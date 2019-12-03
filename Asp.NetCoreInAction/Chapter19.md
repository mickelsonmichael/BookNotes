# Chapter 19: Building Custom Components

* Building custom middleware and branching middleware pipelines
* Using configuration values to set up other configuration providers
* Replacing the built-in DI container with a third-party container
* Creating custom Razor Tag Helpers and view components
* Creating a custom DataAnnotations validation attribute

## Summary
* Use `Run` to create middleware that always returns a response.
* Should always place `Run` at the end of a pipeline or branch, since anything past it will never execute.
* The `Map` extension can be used to create branches in a pipeline.
* When `Map` matches the path, the prefix is removed from the path and inserted into the `HttpContext.PathBase`
* The `Use` extension can create generalized middleware to create a response, modify the request, and pass the request on to subsequent middleware.
* Middleware can be encapsulated in a middleware class to be more reusable and readable. It should take in a `RequestDelegate` object into a constructor and have a `public Task Invoke(HttpContext)` method.
* To call the next middleware in a pipeline, simply call `RequestDelegate()`
* Some configuration providers require their own configuration. You can load these by partially building the `Iconfiguration` object with other sources first, then building it again once the values are used to create the other sources.
* If you need services from DI to configure an `IOptions<T>`, then you should create a separate class that implements `IConfigureOptions<T>` that can use DI in the constructor.
* Can add a third-party DI container into an existing application for additional utilities
* You can configure the built-in container normally, then call a `Populate()` method to import the container into the third-party container
* The name of a tag helper class corresponds to the resulting tag in Razor. `SystemInfoTagHelper` will become `<system-info />`
* The `TagHelperOutput` allows you to modify what is rendered onto the page within the `Process` and `ProcessAsync` methods of a tag helper.
* You may set encoded content using `Content.SetHtmlContent()` and unencoded content using `Content.SetContent()`
* You can stop the processing of a tag by calling `SuppressOutput()` and remove it completely by setting `TagName=null`.
* View components are like partial views with business logic, used for sections of a page that have distinct logic from the main component.
* A view component can be created by inheriting from the `ViewComponent` base class and implementing the `InvokeAsync` method, which can accept parameters from the Razor code.
* View components can use DI, `HttpContext`, and render additional partial views.
* The partial views for a view component should be stored in `Views/Shared/Components/{ComponentName}` folder
* If no view is specified, the `Views/Shared/Components/{ComponentName}/Default.cshtml` file will be loaded
* You can create custom `DataAnnotations` by deriving from `ValidationAttribute` and overriding the `ISValid()` method
* You cannot use Di within custom validation attributes. You must use the Service Locator Pattern

## 19.1 Customizing your middleware pipeline
You may have scenarios where you want to create lightweight functions a user can request    

* Get the current time
* Check the app is running

But you may not want to spin up the entire pipeline to perform these tasks. Or you could also want every request to have a particular header before it reaches the other middleware.

There are "branches" in a pipeline. You may want to divert a request to go down a different branch based on the incoming request.

### 19.1.1 Creating simple endpoints with the `Run` Extension
`Run` extension method returns a lambda response immediately. The requests will not progress further down the pipeline, so it should be the last middleware in the pipe.

You may modify the `HttpContext.Response` within the `Run` method and can write any content you need to it.

`Run` is best used for very simple middleware. Too much complexity will make it difficult to read

### 19.1.2 Branching middleware pipelines with the `Map` extension
Pipelines can actually be branched and have multiple possible paths. Every branch is independent; a response can only pass through on of two branches, not both.

The pipeline selects the branch by looking at the path of the request URL. If it matches the parameter provided to the [`Map`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.mapextensions.map?view=aspnetcore-3.0#Microsoft_AspNetCore_Builder_MapExtensions_Map_Microsoft_AspNetCore_Builder_IApplicationBuilder_Microsoft_AspNetCore_Http_PathString_System_Action_Microsoft_AspNetCore_Builder_IApplicationBuilder__) method, it proceeds down the branch. Otherwise the request stays on the main branch without being modified.

The `Map` function creates a completely new `IApplicationBuilder` that is separate from the original. Middleware added to this builder are not added to the main pipeline.

Really helpful tool but can get confusing very quickly if there are multiple branches, especially if they are nested. [RouteMiddleware](http://mng.bz/QDes) is the routing middleware MVC utilizes. It can be used independently of MVC and is much more powerful than `Map`.

When a route is matched by the `Map` function, the matching segment is removed from the path and stored in `HttpContext.PathBase`
* e.g. "/ping/pong" matches the mapping for "/ping", the request will be modified to "/pong" and the `PathBase == "/ping"`

### 19.1.3 Adding to the pipeline with the `Use` extension

The [`Use`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder.use?view=aspnetcore-3.0#Microsoft_AspNetCore_Builder_IApplicationBuilder_Use_System_Func_Microsoft_AspNetCore_Http_RequestDelegate_Microsoft_AspNetCore_Http_RequestDelegate__) extension method adds a general-purpose piece of middleware to the pipeline, with a lambda method that runs once any request reaches the middleware. The method has two parameters, an `HttpContext` and a pointer to the rest of the pipeline as `Func<Task>`. By calling the pointer, you will proceed to the next middleware
 ``` c#
 app.Use(async (context, next) => {
	 next() // Proceed to the rest of the pipeline
 }); 
 ```

If you do not call this pointer function, the rest of the pipeline *will not execute*.

In general, you should not modify the response after calling the `next()` function. The next step in the pipeline may have already started to send a response back to the user, which would lead to a corrupted response. You also shouldn't call next() if you've called an `async` write to the response for similar reasons; the response could be sent back to the user before the write is complete.

For example, you may want to attach the `HTTP Strict-Transport Header` (HSTS) to every request, which would force the requests to go over HTTPS. The `Use` method can be used to decorate the requests as they come in, effectively making your application HTTPS secure.

```c#
public void Configure(IApplicationBuilder app) 
{
	// add the middleware to the pipeline
	app.Use(async (context, next) =>
	{
		// set a function that should be called before response is sent to browser
		context.Response.OnStarting(() => 
		{
			// add the HSTS header
			// for 60 seconds the browser will send only HTTPS request to your app
			context.Response.Headers["Strict-Transport-Security"] = "max-age=60";
			
			// function passed to OnStarting must return a task
			return Task.CompletedTask; 
		});
		
		// Call the remainder of the pipeline
		await next();
	});

	app.UseMvc(); // the rest of the app will now have the HSTS header
}
```

Multiple `Use` methods in a pipeline can make things cluttered. It's much cleaner to create custom components

### 19.1.4 Building a custom middleware component
Custom components don't derive from a particular base class or interface, but instead have a general template. Each component should have a constructor with a [`RequestDelegate`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.requestdelegate?view=aspnetcore-3.0) object that leads to the rest of the pipeline (e.g. `next();`). Then the component should also include an Invoke method
```c#
public Task Invoke(HttpContext context);
```
This `Invoke` method is equivalent to the lambda method from the `Use` extension. Because it does not inherit directly from any class, the `Invoke` method is fluid and can have any number of parameters that are injected into it via DI, similar to the methods in `Startup.cs`.

Once your component is created, you can then use extension methods on `IApplicationBuilder` to make the app more readable. 
```c#
applicationBuilder.UseMiddleware<HeadersMiddleware>();
```
vs
```c#
applicationBuilder.UseSecurityHeaders()
```
Because the middleware itself is a singleton, any objects in the constructor will be **kept alive for the life of the app**, so only singletons should be added to the constructor of the middleware. If you need scoped or transient dependencies, you can inject them via the `Invoke` method instead.

## 19.2 Handling complex configuration requirements

### 19.2.1 Partially building configuration to configure additional providers
Sometimes in order to set up more advanced configuration, you may need some additional configuration values. For example, to set up a database you will need a connection string, user, password, etc.. This leads to a circular dependency, you need configuration to start the configuration.

You will need to add the provider to build the configuration object but need the configuration object to add the provider.

The solution is several simple steps
1. Load configurations onto the `IConfiguration` object
2. Build the object
3. Add additional configuration
4. Build the object again

```c#
public static IWebHost BuildWebHost(string[] args) => new WebHostBuilder()
	.UseKestrel()
	.UseContentRoot(Directory.GetCurrentDirectory())
	.ConfigureAppConfiguration((context, config) =>
	{
		// 1. Load configurations onto the IConfiguration object
		config.AddXmlFile("baseconfig.xml");

		// 2. Build the object
		var partialConfig = config.Build();

		// get the desired value from the partial configuration
		string filename = partialConfig["SettingsFile"];		

		// 3. Add additional configuration
		config.AddJsonFile(filename)
			.AddEnvironmentVariables();
	})
	.UseStartup<Startup>()
	.Build(); // 4. Build the object again
``` 

### 19.2.2 Using services to configure `IOptions` with `IConfigureOptions`
The `services.Configure<T>()` call from Chapter 11 can be further customized. It provides a second overload that takes in a lambda method that can assign values to the `IOptions<T>` object instead of automatically binding the values from a configuration section. 

```c#
public void COnfigureServices(IServiceCollection services)
{
	services.Configure<CurrencyOptions>(Configuration.GetSection("Currencies"));
	
	services.Configure<CurrencyOptions>(options => 
	{
		// set the Currencies property equal to this manually entered array
		options.Currencies = new string[] { "GBP", "USD" };
	});
	
	services.AddMvc();
}
```

Each call to `Configure<T>()` adds another configuration step to the `CurrencyOptions` object. All are run in order of appearance when DI requests the object (in the example above, when it requests `CurrencyOptions`).

In situations where you need a configured service to set up an `IOptions<T>` object, you must defer it until the last possible moment. ASP.NET Core provided the `IConfigureOptions<T>` interface to help solve this.

```c#
public class COnfigureCurrencyOptions : IConfigureOptions<CurrencyOptions>
{
	private readonly ICurrencyProvider _currencyProvider;
	
	public ConfigureCurrencyOptions(ICurrencyProvider currencyProvider)
	{
		// can inject services that are available after DI is configured
		_currencyProvider = currencyProvider;
	}

	public void Configure(CurrencyOptions options)
	{
		// can use the injected service to load the values
		options.Currencies = _currencyProvider.GetCurrencies();
	}
}
```

You can then register this implementation with the DI container.

## 19.3 Using a third-party dependency injection container
The built in container is **intentionally** limited in features and most likely won't be expanded. You may be interested in installing a third-party 
* [Autofac (https://autofac.org/)](https://autofac.org/)
* [Structure Map (https://structuremap.github.io/)](https://structuremap.github.io/)
* [Ninject (http://www.ninject.org/)](http://www.ninject.org/)
* [Simple Injector (https://simpleinjector.org/index.html)](https://simpleinjector.org/index.html)

The process to configure a new one is pretty simple
1. Install the package
2. Return an `IServiceProvider` from `Startup.ConfigureServices`
	* Normally returns void
3. Configure the container to register services already registered with the pre-built container using the `Populate(services)` method

The `Populate` method registers the already created registrations with the new container and allows the use of the built in extension methods like `AddMvc()` and `AddAuthorization()`.

You may need to call the `AddControllersAsServices()` method in order to register services for controllers and allow DI.

## 19.4 Creating a custom Razor Tag Helper

### 19.4.1 Printing environment information with a custom Tag Helper
In order to create a custom tag helper, you must derive from the [TagHelper](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper?view=aspnetcore-3.0) base class and override the [`Process`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper.process?view=aspnetcore-3.0#Microsoft_AspNetCore_Razor_TagHelpers_TagHelper_Process_Microsoft_AspNetCore_Razor_TagHelpers_TagHelperContext_Microsoft_AspNetCore_Razor_TagHelpers_TagHelperOutput_) and [`ProcessAsync`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper.processasync?view=aspnetcore-3.0#Microsoft_AspNetCore_Razor_TagHelpers_TagHelper_ProcessAsync_Microsoft_AspNetCore_Razor_TagHelpers_TagHelperContext_Microsoft_AspNetCore_Razor_TagHelpers_TagHelperOutput_) methods.

The class name is then converted into kebab-case and used in the Razor pages

| Class Name | Kebab-Case | HTML Tag
|--|--|--
| SystemInfoTagHelper | system-info | `<system-info />`
| BookClubBookTagHelper | book-club-book | `<book-club-book />`
| BookclubBookTagHelper | bookclub-book | `<bookclub-book />`

You can customize the name of the element by applying the [HtmlTargetElementAttribute](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.htmltargetelementattribute?view=aspnetcore-3.0) to the class
```c#
[HtmlTargetElement("EnvInfo")]
SystemInfoTagHelper : TagHelper 
{
	// results in <env-info /> tag
}
```
You should inject an [HtmlEncoder](https://docs.microsoft.com/en-us/dotnet/api/system.text.encodings.web.htmlencoder?view=netcore-3.0) into almost every custom tag, since you should be encoding all text inserted onto the page to prevent from XSS attacks.

You can define attributes on the tag by decorating properties in the class with the [HtmlAttributeNameAttribute](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.htmlattributenameattribute?view=aspnetcore-3.0). For example adding `[HtmlAttributeName("thisSet")]` would result in a tag like `<tag thisSet="value" />`.

In order to replace the contents of the tag, you can utilize the [`Content.SetHtmlContent()`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.html.htmlcontentbuilderextensions.sethtmlcontent?view=aspnetcore-3.0) method. Any content provided to this method should be encoded and will appear within the inside of the tag. You can also use the [SetContent()](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.html.htmlcontentbuilderextensions.setcontent?view=aspnetcore-3.0) method to pass unencoded content, since it will automatically encode it for you.

In order to use the tag helpers within Razor pages, you must regsiter them. It's recommended to register them in the `_ViewImports.cshtml` file by adding a `@addTagHelper` directive and specifying the fully qualified name and assembly. You can also add all tag helpers within an assembly by utilizing the `*` wildcard and specifying the assembly name. 

`@addTagHelper *, AssemblyName`

### 19.4.2 Creating a custom Tag Helper to conditionally hide elements
ASP.NET core also allows you to extend pre-existing tag helpers. The author uses the ability to hide and show a tag based on a boolean value as an example. This makes it easier to read the code without switching your mind between HTML and Razor, and also improves the usability of editors that don't have full Razor syntax support.

For this example, you don't need to create a whole new element like the previous section (19.4.1) but instead can create a custom attribute. 

Like before, create a class that inherits from [TagHelper](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper?view=aspnetcore-3.0), but this time when decorating it with the `[HtmlTargetElement]` attribute, add in the `Attributes` parameter and set the value to the name of the attribute to target.

```c#
[HtmlTargetElement(Attribute = "attr-name")]
```

Next, override the `Process` method and you can modify the element in any way you need. For example:

* Set `TagHelperOutput.TagName` to null to remove the element completely
* Call `TagHelperOutput.SuppressOutput()` to generate nothing at all
	* This method also sets `TagName` to null
	* Clears the `PreElement`, `PreContent`, `Content`, `PostContent`, and `PostElement`
* Remove the tag from the output by using [`TagHelperOutput`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelperoutput?view=aspnetcore-1.1).[`Attributes`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelperoutput.attributes?view=aspnetcore-1.1#Microsoft_AspNetCore_Razor_TagHelpers_TagHelperOutput_Attributes).[`RemoveAll(string name)`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelperattributelist.removeall?view=aspnetcore-1.1#Microsoft_AspNetCore_Razor_TagHelpers_TagHelperAttributeList_RemoveAll_System_String_)

You should also override the [`Order`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.razor.taghelpers.taghelper.order?view=aspnetcore-3.0#Microsoft_AspNetCore_Razor_TagHelpers_TagHelper_Order) property, which determines the order of rendering for tag attributes. By setting it to `int.MinValue` you force it to run first. In the example found in the book, the tag removes the element completely so you would want to run that as soon as possible.

You must still remember to register the helpers in the `_ViewImports.cshtml` file as in the previous section.

[More details and examples](http://mng.bz/Idb0)
[Source code for additional Tag Helpers](https://github.com/DamianEdwards/TagHelperPack)

## 19.5 View components: adding logic to partial views

Partial views aren't always the best fit for a particular scenario. They let you encapsulate *view rendering logic* but not *business logic*.

View components, on the other hand, can contain business logic and can be tested independently like a controller. They are analogous to a mini-MVC controller that is invoked from a view or layout, and are comparable to the ["child actions"](https://haacked.com/archive/2009/11/18/aspnetmvc2-render-action.aspx/) from older version of ASP.NET like `@Html.Action()`.

The best candidates for view components are when the logic for the section will be distinct and independent from the container.

The preferred way to render a view component on a page is **far too similar to Web Forms**.

```htmlmixed=
<vc:component-name />
```

The name of the tag is derived from the class library as is convention. To create the tag above, you would create a class called `ComponentNameViewComponent`. The "ViewComponent" suffix is removed and the remainder is converted to kebab-case. Additionally, you may customize the name further by using a [`ViewComponentAttribute`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.viewcomponentattribute?view=aspnetcore-3.0) reference.

In order to create a view component, you must derive from the `ViewComponent` base class and implement an `InvokeAsync()` function. This base class provides a variety of helper methods similar to the `Controller` and `ControllerBase` classes.

Parameters can be passed into the component using properties on the Tag Helper element like `<vc:component-name parameter="value" />`. The values do not come from model binding like a controller.

The `InvokeASync` method, when implemented, must return a `Task<IViewComponentResult>`, similar to the way controllers must return a `IActionResult` or `Task<IActionResult>`. But the `IViewcomponentResult` does not accept a status code or redirect. The response must be content or a view.

:::info
You may freely use Dependency Injection within the constructor of the class, and you have access to the current `HttpContext`.
:::

When returning a `View()` result, if you do not specify the name of the view, it will default to `Default.cshtml`. Partial views must be stored in `Views/Shared/Components/{ComponentName}/{TemplateName}`.

* `Views/Shared/Components/MyRecipes/Default.cshtml`
* `Views/Shared/Components/MyRecipies/Unauthenticated.cshtml`

There are some additional caveats to consider when creating a view component.

* Must be public, non-nested, and non-abstract
* Can't use filters
* Can't use `@sections` in partial views returned by view component
* When using `<vc:component-name />` you must import it as a custom tag helper
* Can invoke a tag directly using `Component.InvokeASync()` directly in the Razor page.

## 19.6 Building a custom validation attribute
Validation occurs after model binding but before the action or action filters have been executed. The `MvcMiddleware` calls `IsValid()` for each attribute and passes in a value and `ValidationContext`. 

One piece of the `ValidationContext` of note is the `ObjectInstance`, which can be used to reach the top-level model being validated. You can use this access to validate properties that are dependent on the value of other properties, like a start date that must be before an end date.

```c#
var model = validationContext.ObjectInstance as MyModel;
```

::: info
Within `IsValid()`, the value coming in is an `object` type, and must be converted to the expected type.
:::

To return a bad validation result, use `return new ValidationResult("Error message")`, and your error will be added to the current `ModelState`. To return a successful result, use `return ValidationResult.Success`.

By default, ASP.NET Core uses jQuery to perform client-side validation. If you want these values to be validated before being submitted to the server, you will need to use jQuery to do it, or import a new validation library.

You may want to use DI to improve your attribute, but unfortunately only constants can be passed to a constructor of an `Attribute`. Instead you are forced to use the [Service Locator Pattern](https://en.wikipedia.org/wiki/Service_locator_pattern), which should normally be avoided, but is necessary in this instance.
