# Chapter 18: Improving your application

[The Open Web Application Security Project (OWASP)](http://www.owasp.org)

[Troy Hunt courses and workshops on security](http://www.troyhunt.com)

Asynchronous Javascript and XML (AJAX)

## 18.1 Adding HTTPS to an application

By default, HTTP requests are unencrypted. HTTPS is the same as HTTP but uses an SSL certificate to encrypt the requests and responses.

:::info
SSL protocol has been technically superseded by TLS, so for the context of this book when the author says SSL he really means TLS/SSL.
:::

Thanks to websites like [Let's Encrypt](https://letsencrypt.org), SSL certificates are much easier to get, and thus the web has become much more secure. If hosting in the cloud, many providers have one-click SSL certifications to make it even easier. Or you could use the CloudFlare CDN as well.

Because of the reverse-proxy architecture, you can essentially ignore encryption within your application; it will already be decrypted by the proxy before it even reaches your application. This is known as SSL offloading/termination.

Regardless, you may need to deal with SSL directly, and testing locally will require an SSL certificate on your machine. You must also configure Kestrel to recognize and use the SSL certificate.

### 18.1.1 Setting up IIS and IIS Express to use encryption

IIS Express will work as a reverse proxy and handle the SSL decryption so you don't have to. You can check the project's Debug properties and enable settings there in seconds.

:::info
The default port for HTTPS is 443
:::

### 18.1.2 Creating a self-signed certificate for local development

HTTPS uses *public key cryptography*. There are two keys:
* *Public key* that anyone can see
* *Private key* that only *your server* can see

Anything encrypted with the public key can only be decrypted by the private key.

#### Creating a Self-Signed Certificate using PowerShell on Windows

The easiest method is to use PowerShell because it can be scripted. However you must use the administrative powershell. You then provide a password to use the certification in ASP.NET Core 2.0; blank passwords will result in Kestrel throwing an exception. You can then trust the certificate to avoid warnings from the browser.

This process may not be necessary since Visual Studio already sets up IIS Express to self-sign.

### 18.1.3 Using HTTPS directly in Kestrel

You can configure Kestrel to listen to a specific port and url for HTTPS requests. Unfortunately you cannot use environmental variables for this.

You can call `UseKestrel()` inside Program.cs. This is normally called automatically by `CreateDefaultBuilder()`, but can be called a second time to provide more options.

Kestrel was supposed to have a configuration based API for defining the values but got delayed. However, this feature could potentially have been released with 3.0 or be picked back up by now.

The author recommends using files as the easiest method for storing and reading certifications.

### 18.1.4 Enforcing HTTPS for your whole app

HTTP/2 has better performance over HTTP/1.x, but browsers require HTTPS to enable it. This means that SSL can actually improve the performance of your application. [See here](http://mng.bz/K6M2) for an introduction to HTTP/2.

You can attach a particular header to force HTTPS called the **Http Strict Transport Security (HSTS)** header. [See here](http://mng.bz/K8it) for additional security-related headers you can attach.

Alternatively, you could redirect any HTTP requests to the HTTPS equivalent using middleware or even MVC. The [RewriteMiddleware](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/url-rewriting?view=aspnetcore-3.0) comes with ASP.NET Core and can be added to the pipeline to redirect any request that pass through it.

You could also add the `[RequireHttps]` attribute globally, or to any action or controller. This redirects insecure requests to HTTPS. [See here](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.requirehttpsattribute?view=aspnet-mvc-5.2) for documentation.

#### SSL offloading, header forwarding, and detecting secure requests

Kestrel doesn't know if a request was HTTPS if the reverse proxy intercepts the message and decodes it first. You must put headers on to requests to notify Kestrel that the request was HTTPS. You can set the `X-Forwarded-Proto` header to HTTPS. [See the MDN Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto) for more information on the header.

This forwarding is done automatically with `UseIisIntegration()`, but the call must be placed before `UseMvc()` and `UseRewriter()`.

## 18.2 Defending against cross-site scripting (XSS) attacks

[See here for some additional OWASP discussion](http://www.owasp.org/index.php/Cross-site_Scripting_(XSS)). 

A cross-site scripting (XSS) attack happens when you render user input in an unsafe way. It allows attacks to put `<script>` tags onto your page without your consent. Razor prevents this by design by encoding everything you place into onto the page that isn't wrapped in `Html.Raw()`. You should only use `Html.Raw()` in secure contexts where you can verify the contents are not malicious.

You can pass unencoded values from Razor to Javascript safely by using a Javascript encoder like `System.Text.Encodings/Web.JavaScriptEncoder`. The author however does not recommend this, and instead suggests placing the value into an HTML element that can the be read from Javascript at a later time.

## 18.3 Protecting from cross-site request forgery (CSRF) attacks

CSRF attacks are an issue for sites that use cookies for authentication. A malicious site can make a request on behalf of the user without their knowledge or consent.

You can mitigate these dangers by using the anti-forgery tokens built into MVC and the [Synchronizer Token pattern](http://mng.bz/Qt49).

The Synchronizer Token pattern give the user a specific, unique token. One token is stored into a cookie and the other is added to the form you want to protect. The tokens are created at runtime based on the current user, so there is no way for an attacker to create on for their forged form. 

ASP.NET Core automatically adds an anti-forgery token to the forms, but does not automatically validate it. You instead need to call add a `[ValidateAntiForgeryToken]` attribute to the controller, action, or the global context.

If you want to apply `[ValidateAntiForgeryToken]` globally, it's better to use the `[AutoValidateAntiForgeryToken]`, since it ignores GET requests and other safe verbs.

Unfortunately, issues arise when doing mainly API calls and passing JSON data since the anti-forgery token cannot be used. However if you aren't using cookie authentication you don't have to worry about CSRF at all.

#### Generating unique tokens with the Data Protection APIs

Anti-forgery tokens rely on the frameworks use of strong, symmetric encryption. On ore more keys are used to initialize encryption and reproduce the result. If you have the key you can encrypt and decrypt data.

In ASP.NET Core, encryption is handled by the Data Protection APIs, which can create secure tokens and encrypt cookies. They are used to conrol management of key files for encryption as well.

A key file is a small XML file that contains a random key value for encryption. It must be stored securely, and depending on hosting the Data Protection system will store them in a different location.

This creates issues for server farms; they must have a centralized authentication location in order to work properly.

[See here](http://mng.bz/d4Oi) for more information on configuring data protect and [see here](http://mng.bz/5pW6) for more information on configuring a key storage provider.

## 18.4 Calling your web APIs from other domains using CORS

**Origins** are considered the same if they match the scheme, domain, and ports. CORS blocks requests that are not on the same origin unless it is configured otherwise.

### 18.4.1 Understanding CORS and how it works

Cors is a web standard allowing API to restrict who can make requests to it. These restrictions are defined in policies, which can be targeted to different levels from application-wide to a single action. It utilizes HTTP headers on the response to specify if cross-origin is allowed.

[See here](https://www.w3.org/TR/cors) for the CORS Specification from W3.


### 18.4.2 Adding CORS to you whol eapp with middleware

You should not configure CORS unless you need it, since it opens up avenues of attack into your application.

To set up CORS, you must add the service to the app, configure at least one policy, and add the CORS middleware *or* decorate actions with `[EnableCors]`.

The policies control how applications will respond to cross-origin requests. When listing origins, ensure that they do not have a trailing "/" or the app will fail.

It is important to add the CORS middleware before the MVC middleware.

### 18.4.3 Adding CORS to specific MVC actions with `EnableCorsAttribute`

It's best to only enable CORS for the actions that need it, to reduce potential vulnerability, using the `[EnableCors]` attribute. 

You may then add policies in `ConfigureServes` using an `AddPolicy()` call.

An attribute placed on an action takes precedence over an attribute placed on a controller.

You may use the `[DisableCors]` attribute to disable access entirely, but only for an action that has CORS enabled via the `[EnabledCors]` attribute; it will not ignore the CorsMiddleware.

You cannot selectively disable the CorsMiddlware, you must instead add the `[EnableCors]` attribute globally.

`[EnableCors]` is a marker attribute an contains no real functionality itself.

### 18.4.4 Configuring CORS policies

Whenever possible, avoid cross-origin requests completely, and avoid HTTPS and HTTP issues by forcing the application to use HTTPS.

## 18.5 Exploring other attack vectors

Check out the [OWASP Top Ten Vulnerabilities List](http://mng.bz/yXd3). It is updated yearly with the latest vulnerabilities and provides way to avoid them.

### 18.5.1 Detecting and avoiding open redirect attacks

Open redirect attacks are the #10 vulnerability on the OWASP list as of the writing of this book. It occurs when a user clicks a link to the safe application but is redirected to a malicious link, even though the safe app contains no reference to the malicious application.

This can be done on pages when the next page or a redirect is passed as a parameter to an action method. Most commonly done when logging in to an application.

A hacker will convince a user to click the link to the safe page, but attaches a malicious redirect. In the login example, the user will fill out their information, click login, then be redirected back to the malicious site.

The simple solution is to always validate the return Url is a local Url. ASP.NET Core provides some helpers for doing this including `Url.IsLocalUrl()` and `LocalRedirect()`.

### 18.5.2 Avoiding SQL injection attacks with EF Core and parameterization

Injections attacks are one of the most dangerous threats. Hackers inject code that is executed within your database by piggybacking off SQL strings. 

EF Core (and most ORMs) prevent this by default using built-in protections. However EF Core provides the `FromSQl()` function that you will need to be wary of.

You can avoid this altogether by simply using parameterized queries.

[Little Bobby Tables](https://www.xkcd.com/327/)

### 18.5.3 Preventing Insecure direct object references

Users can gain access to things they shouldn't by noticing patterns in URLs (hackable URLs). You should utilize resource-based authorization and authentication  to prevent this.

While hiding UI elements is nice for the user, it isn't ideal for security.

You can also avoid this by not using integers for your entity keys.

### 18.5.4 Protecting your users' password and data

As a general rule, don't store data you don't need.

Checkout the [Digital Identity Guidelines from NIST](https://pages.nist.gov/800-63-3/sp800-63-3.html).

Here are some basic rules of thumb:
* Never store user passwords anywhere directly; only encrypted
* Don't store more data than you need
* Never store credit card details
* Allow for two-factor authentication
* Prevent weak or compromised passwords
* Mark authentication cookies as "http" and "secure"
* Don't expose whether a user is already registered with you app or not

This is not a full list, but instead the bare minimum you should do.

See [Troy Hunt: Understanding Account Enumeration](https://www.troyhunt.com/understanding-account-enumeration-the-video-tutorial-edition/)
