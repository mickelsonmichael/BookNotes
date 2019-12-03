# Chapter 20: Testing your application

**For a broad discussion of Unit Testing, read [*The Art of Unit Testing*](https://www.manning.com/books/the-art-of-unit-testing-second-edition?a_bid=f6ea80f5&a_aid=iserializable) by Roy Osherove, or [*.NET Core in Action*](https://www.manning.com/books/dotnet-core-in-action?query=.net%20core) by Dustin Metzgar.**

## Summary

* Unit tests are console apps that use a framework like xUnit, MSTest, or NUnit and a test runner
* You can run tests from the dotnet CLI by calling `dotnet test` or by using the Visual Studio Test Explorer
* xUnit has emerged as the primary standard for ASP.NET Core projects
* The ASP.NET Core team uses xUnit to test the framework
* You can use `dotnet new xunit` CLI command to create a new xUnit Test Project
* `[Fact]` methods should be public and parameterless
* `[Theory]` methods can contain parameters, so can be used to run a test repeatedly with different inputs
* You can provide data for each `[Theory]` using the `[InlineData]` attribute
* Use assertions in test methods to verify an expected value
* Use the `DefaultHttpContext` class to unit test custom middleware components
* If you need a response body, you must replace the default `Stream.Null` of the `DefaultHttpContext` with a `MemoryStream` instance and read the stream after invoking the middleware.
* MVC controllers can be tested like a normal class, but should contain minimal logic, so may not be worth the effort
* Integration tests allow you to test multiple aspects of the app at once
* `Microsoft.AspNetCore.TestHost` package provides a `TestServer` object that you can use to create a simple web host for testing. It creates an in-memory version of the app to make requests and receive responses from
* Create the `TestServer` by passing in a `WebHostBuilder`.
* The `TestServer` exposes an `HttpClient` property, `TestServer.Client` that you can use to make the requests
* You can configure the `WebHostBuilder` using the Startup file.
* If you need to load config files or Razor views, you'll need to configure the `ContentRootPath` for the app
* To render Razor views, you must add an `MSBuild` target to the test csproj file. This can be avoided in 2.1+ by installing `Microsoft.ASpNetCore.Mvc.Testing` into the project instead.
* If using the same `TestHost` config for all integration tests, you can configure the host in a `T` class, implement `IClassFixture<T>` in your test class, and inject an instance of `T` into the constructor. All tests in the class will use the same instance of `T` which will reduce the number of `TestHost`s created
* You can use the EF Core SQLite provider as an in-memory database to test code
* Can configure the provider by creating a `SqlLiteCOnnection` with `DataSource=:memory:` in the connection string. Create a `DbContextOptionsBuilder<>` object and call `UseSqlLite()` with the connection. Then pass `DbCOntextOptions<>` into the app's `DbCOntext` and call `context.Database.EnsureCreated()`.
* The in-memory database is kept as long as there is an open connection. By opening manually, it can be used with multiple `DbContext`s, othersise EF will close the connection and delete the database when `DbContext` is disposed

## 20.1 An introduction to testing in ASP.NET Core

Since testing now has a bigger role in ASP.NET core, you can use `dotnet test` in the CLI to run all the tests for a project, regardless of testing framework used. It uses the underlying SDK to run the tests, which is the same as the test runner in Visual Studio; both options will give the same result.

Test projects will have three dependencies that can be retrieved from NuGet.

* .NET Test SDK
* a testing framework
* A test-runner adapter for `dotnet test`

Small isolated tests that ensure each component is working correctly are *unit tests*.

Because of the way the framework was designed. It avoids static types, uses interfaces, and has a modular architecture.

Tests that ensure components work together are *integration tests*. They don't necessarily include the entire app, but more components than unit tests.

For UI testing, the author recommends [Selenium](http://www.seleniumhq.org) or [Ghost Inspector](http://ghostinspector.com).

## 20.2 Unit testing with xUnit

 The most commonly used framework with .NET Core is [xUnit](https://xunit.github.io/), which is used by the framework itself. At this point xUnit has become convention for the framework. 
 

### 20.2.1 Creating your first test project
 
 A test project can be created easily with Visual Studio or by running `dotnet new xunit`. 
 
Tests in xUnit are denoted by the `[Facts]` attribute, have no arguments, be public, return `void` or a `Task`, and be within a public, nonstatic class.

There is no `Program` class, because the test SDK automatically injects it at build time. [Read here](http://mng.bz/1NyQ) if you want to add a Program.cs to the test project. 
 
 ### Running tests with dotnet test
 
 You may run tests either using the Visual Studio test explorer or the `dotnet test` command with the CLI.
 
 **Run the `dotnet test` command from the *project* folder which contains the csproj file. If you try to run at the solution level you will get an error.**

`dotnet test` runs `dotnet restore` and `dotnet build` automatically, then runs the tests. It uses the same mechanisms as Visual Studio Test Explorer, so the results will always be the same. You can additionally add the `dotnet xunit` command to run tests with xUnit. [Read here](https://xunit.github.io/docs/getting-started-dotnet-core.html) for more info.

### 20.2.3 Referencing your app from your test project

In order to test a project, your test project must have a reference to the target project. This can be done by adding a dependency in Visual Studio or by adding a `ProjectReference` to your .csproj file.

```xml
<ItemGroup>
    <!-- The path is a relative path -->
    <ProjectReference Include="../path/to/project.csproj" />
</ItemGroup>
```

**The path inside the `Include` attribute is a relative path. A `..` signifies a parent folder.**

> #### Common conventions for project layout
> The default layouts from Visual Studio are slightly different that the layouts chosen by the ASP.NET Core team (and other popular C# projects).
> * .sln file in the root
> * Main projects in "/src" directory
> * Test projects in "/test" directory
> * Each main project has a test equivalent
> * Other folders in the root can include "tools", "samples", or "docs"
>
> You don't have to follow these conventions but it's important to be aware of

### 20.2.4 Adding Fact and Theory unit tests

The author generally follows one of three paths when writing a test
* **Happy Path** - arguments with expected values are provided; meant to test success
* **Error Path** - arguments are invalid; expected to throw an exception
* **Edge Cases** - arguments are on the edge of the expected values

Most unit testing follows a particular three step process
1. Arrange - define all parameters and create the class under test
2. Act - execute the method being tested and capture the result
3. Assert - verify the result of the act stage has the proper value

xUnit provides an `Assert` class that contains several verifications for checking your code worked properly. If they fail, then the test throws a particular exception.

**You can remove the class name from being displayed in xUnit using an [xunit.runner.json file](https://xunit.github.io/docs/configuring-with-json.html).**

Instead of copy-pasting a single `[Fact]` method to test several scenarios, you can create a single `[Theory]` method and adding parameters using the `[InlineData()]` attribute as many times as needed on the method. When ran, each `InlineData` test will appear as a seperate test.

You can also use `[ClassData]` or `[MemberData]` to pass the values for the attributes. The author wrote a blog post about using them [here](http://mng.bz/8ayP) but won't go into further detail in the book.

### 20.2.5 Testing failure conditions

xUnit provides several helper methods on the `Assert` class that allow for the testing of Exceptions, including checking for a particular error message.

**Don't tie test methods too closely with implementation of a method. It makes tests brittle and internal changes to a class can break the tests**

The `Assert.Throws` method accepts a lambda method. In this lambda, you should call the function that you expect to throw an exception. The method will catch the exception and validate that the exception type equals the expected type.

## 20.3 Unit testing custom middleware

See Chapter 19 for a review on creating custom middleware.

Testing middleware is complicated because `HttpContext` is a big class. There's a lot for the middleware to interact with. Leads to tight coupling to the implementation which is undesirable. 

You can pass a `DefaultHttpContext` into tests.It is an implementation of `HttpContext` and is part of the base framework abstractions. [Read the source code here](http://mng.bz/7b43).

In order to test the `Invoke` method, you can also pass a custom delegate that sets a particular boolean when it is called. This will allow you a straightforward way to confirm that the delegate was called. See the code snippet below for the author's example.

```c#
[Fact]
public async Task ForNonMatchingRequest_CallsNextDelegate()
{
    // ARRANGE

    // Creates the context and sets the path
    var context = new DefaultHttpContext();
    context.Request.Path = "/notping";
    
    // Create a boolean to track when the delegate is called
    var wasExecuted = false;
    RequestDelegate next = (innerContext) => 
    {
        wasExecuted = true; // Set the tracking boolean to true when called
        return Task.CompletedTask; // the delegate must return a task
    };
    
    // ACT
    
    // Create the instance of the middleware
    var middleware = new StatusMiddleware(next: next);
    
    // Call the method to be tested
    await middleware.Invoke(context);
    
    // ASSERT
    
    // test whether the delegate was called by checking whether
    // the boolean was set to true
    Assert.True(wasExecuted);
}
```

In the above example, becuase the `HttpContext.Request.Path` did not equal "ping", the pipeline continued down the normal path by calling the delegate method (i.e. `next();`). Because the delegate was called, the value of the boolean will be set to true and the assert will pass. If the execution of the pipeline was stopped and the delegate wasn't called as a result, the boolean would still be false.

In general, when creating a test, it should test one thing and one thing only. The author creates an example test that has multiple asserts which is generally not good practice. If you were to view the test in the test runner and it failed, you would need to do more work to debug which part actually failed; you would lose glance value with the test.

If you needed to verify the `Response.Body` of a request while using `DefaultHttpContext`, you need to properly onfigure the response stream. The `DefaultHttpContext` uses `Stream.Null` as the default value for `Response.Body`, meaning it will not capture any values passed to it and they will be lost. You therefore must replace the body with a `MemoryStream` and use a `StreamReader` to read the contents and verify.

```c#
[Fact]
public async Task ReturnsPongBodyContent()
{
    // Create the memory stream
    var bodyStream = new MemoryStream();
    
    // create the context
    var context = new DefaultHttpContext();
    context.Response.Body = bodyStream;
    context.Request.Path = "/ping";
    
    // set up a simple delegate
    RequestDelegate next (contxt) => Task.CompletedTask;
    
    // Set up the middleware
    var middlware = new StatusMiddleware(next: next);
    
    string response; // empty string to capture the reponse
    bodyStream.Seek(0, SeekOrigin.Begin); // reset the stream to the beginning
    
    using (var reader = new StreamReader(bodyStream))
    {
        reponse = await reader.ReadToEndAsync();
    }
    
    // Verify it is the expected result
    Assert.Equal("pong", response);
}
```

## 20.4 Unit testing MVC Controllers

Unit tests test only parts of logic, while MVC requests contain multiple layers. However, Controllers can be isolated down. You can test for a variety of responses including
* Returning an `IActionResult` for invalid requests
* Calling the business logic and returning an `IActionResult` for valid requests (or an object serialized)
* Apply resource-based authorization as required

Generally controllers shouldn't contain business logic themselves. They work as an intermediate between the request and the services. This makes it easier to write tests.

Because of this, there shouldn't be much left to test within the controller itself.

Controllers are still classes and actions are still methods, so you can easily write tests for controllers and actions. However you are testing the `iActionResult` and not the actual response to the user in these cases.

You may test the actions return a particular view, but this will likely result in a brittle test.

One issue arises when validating `ModelState`. The `MvcMiddleware` sets the `ModelState` when action is called; this does not happen in a unit test. You have to add your own model errors since you cannot rely on model binding to do it for you. 

Because of these factors, the author generally avoids writing unit tests for controllers. He does however write *integration tests* for them. 

## 20.5 Integration testing: testing your whole app with Test Host

In the context of this book, unit tests are for testing components individually and integration tests are for testing multiple components at once and often interact with databases or contexts. This distinction means that unit tests are generally smaller, faster, and much more numerous than the integration tests. 

### 20.5.1 Creating a `TestServer` using the Test Host package

The ideal test for a middleware component would be a standalone copy of your application, with just your component in the pipeline. However, this is error prone and time consuming to configure.

Instead you can create a `TestServer` which will get you close to that ideal and without spinning up a separate app. It creates an in-memory web server that you can send requests to through a particular `HttpClient` configured by the `TestServer`. This will mean requests can be send as if they were to a location on another network, when in reality they are being sent to a memory server.

Just add `Microsoft.ASpNetCore.TestHost` to your test project.

The constructor for the `TestServer` takes in an `IWebHostBuilder` object, so you can create one with all the middleware components you require and only the components you require.

```c#
[Fact]
public async Task TestThings()
{
    // Create a new web host builder
    var hostBuilder = new WebHostBuilder()
        .Configure(app =>
        {
            // Add any middlware under test
            app.UseMiddlware<CustomMiddleware>();
        });
        
    using (var server = new TestServer(hostBuilder))
    {
        // Get the speciall HttpClient from the TestServer
        HttpClient client = server.CreateClient();
        
        // Send the particular request
        var response = await client.GetAsync("/request/url");
        
        var result = await response.Content.ReadAsStringAsync();
        
        Assert.Equal("expected", result);
    }

}
```

If you want to test the actual application itself instead, you can use the `Startup.cs` file directly. See the next section for more info.

### 20.5.2 Using your application's Startup file for integration tests

In an application, when you call `UseStartup<>`, you are diverting some of the logic into a separate standalone class, Startup.cs. That means that you can reference this class from another project easily. You can create a test that uses your application's current configuration by simply adding a reference to the project containing the `Startup.cs` file, then adding  `UseStartup<Startup>()` to your `WebHostBuilder`.

### 20.5.3 Rending Razor views in `TestServer` integration tests

**ASP.NET 2.1+ introduces `Microsoft.AspNetCore.Mvc.Testing` which solves the issues in this section. [See this link](http://mng.bz2QP2) for more information**

Even while using the Startup.cs file in your tests, as shown in the previous section, you may still be missing critical configurations to propertly test your application. Most critically for rendering MVC views, it doesn't include the *content root* directory (base directory of the application).

You have to call `UseContentRoot()` on the `WebHostBuilder` in order to configure this for a standard application. But in a test application, you will have to reference the location using a relative URL for the app starting from the root of the test app. This can lead to particularly ugly paths. 

In order to finish this configuration, you must paste the following code before the final `</Project>` element of the test project `.csproj` file. 

```xml
<Target Name="CopyDespFiles" AfterTargets="Build"
        Condition="'$(TargetFramework)'!=''">
        
        <ItemGroup>
            <DepsFilePaths Include=$([System.IO.Path]::ChangeExtension('%_ResolvedProjectReferencePaths.FullPath)', '.deps.json'))" />
        </ItemGroup>

    <Copy SourceFiles="%(DepsFilePaths.FullPath)"
        DestinationFolder="$(OutputPath)"
        Condition="Exists('%(DepsFilePaths.FullPath)')" />
    </Target>
```

[See this GitHub Issue for more details (1212)](https://github.com/aspnet/Razor/issues/1212).

**Using this method means you will be making calls to all the same configurations as the live application, including databases and third-party services. This may or may not be desirable. You should configure a "Testing" hosting environment with special configurations for tests.**

### 20.5.4 Extracting common setup into an xUnit test fixture

Creating a `TestServer` for every test method is expensive, but xUnit provides *test fixtures* that allow you to share an instance between tests. This will occur *once per class*.

1. Create fixture class, `T`
2. Implement `IClassFixture<T>` on the test class
3. Inject an instance of `T` into the test class constructor

The `IClassFixture<T>` has no methods, but let's xUnit know to create an instance of `T` before running a test method.

**If a test fixture implements `IDisposable`, then xUnit will call `Dispose()` after the tests have ran.**

You can then add a property to your test class to store the instance of `T` and share that instance with all running tests for that class.

## 20.6 Isolating the database with an in-memory EF Core provider

Utilizing a `DbContext` attached to a *real* database makes test slow, potentially unrepeatable, and dependent on the configuration of the database.

Microsoft provides two in-memory providers:

* Microsoft.EntityFrameworkCore.InMemory
    * Doesn't simulate a database, but stores in memory
    * Isn't a relational database so is missing features
    * Can't execute SQL against it
    * Can't enforce constraints
    * **Fast**
* Microsoft.EntityFrameworkCore.Sqllite
    * Uses the relational SQLite
    * Limited in some features, but is a true relational database
    * Normally the data is written to a file, but it can be configured to write to memory, making it much faster

The author describes using SQLite in this chapter. Read about InMemory [here](https://mng.bzhdIq).

Simply add a reference to `Microsoft.EntityFrameworkCore.Sqlite`  or `Microsoft.AspNetCore.All` then use the `UseSqlite()` extension method. Then utilize the `DataSource=:memory:` connection string to tell the provider to use the database in memory.

```c#
    var sqliteConnection = new SqliteConnection("Datasource=:memory:");
    
    // open the connection yourself to prevent it from being destroyed early
    sqliteConenction.Open(); 

    var options = new DbContextOptionsBuilder<MyDbContext>()
        .UseSqlite(sqliteConnection)
        .Options;
```

**The in-memory database is destroyed when the connection is closed. If you want to share a database between multiple `DbContext`'s, you must open them yourself; otherwise they will be automatically disposed.**

1. Create a `SliteConnection` and open it (use `:memory:`)
2. Create a `DbContextOptionsBuilder<>` and call `UseSqlite(openConnection)`
3. Use the `Options` property to get the options object
4. Pass the options to the `DbContext` constructor
5. Call `context.Database.EnsureCreated()` to verify it matches EF Core's model
    * This is simlar to running migrations on a database
6. Create and add any new data and call `SaveChanges()`
7. Create a new instance of the `DbContext` and inject it into the test class

**Use two separate `DbContexts` when adding setup data to prevent issues with EF Core's caching confounding your test results.**