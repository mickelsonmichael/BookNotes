# Appendix A: Getting to grips with .NET Core and .NET Standard

This chapter is aimed at clarifying why .NET Standard was created and why .NET Core was made the way it was, as well as some guidance on how to select a target framework when creating a new application.

## Summary

* .NET has many implementations
    * .NET Framework, Mono, Unity
    * Separate platforms with different Base Class Libraries (BCLs)
    * Separate app models
    * .NET Core is another separate platform
* Each platform has a BCL with .NET types and classes like strings and streams
* .NET Standard defines a set of APIs across all platforms that support it
    * You can write a library that targets a specific version of .NET standard
    * Any platform that supports the version you specify will be compatible
* Each version of .NET Standard is a superset of the previous
    * .NET Standard 1.2 includes all APIs from .NET Standard 1.1
    * .NET Standard 1.1 includes all APIs from .NET Standard 1.0
* Each version of a platform supports a version of .NET Standard
* .NET Framework 4.6.1 *technically* supports .NET Standard 1.4
    * There is a compatibility shim that allows referencing of .NET Standard 2.0 libraries
    * Can reference a .NET Framework 4.6.1 library from .NET Standard 2.0
* If you reference a Framework 4.6.1 library from Standard 2.0, and the library uses an unsupported API, you get a runtime exception
* You must target a .NET implementation when developing an **app**, not .NET Standard
* Target .NET Core 2.0 when writing apps
* Target .NET Framework 4.7.1 if you can't use .NET Core
* For libraries, target the lowest version of .NET Standard you can
* If your library may be used by a .NET Framework 4.6.1 app and you're targeting .NET Standard 2.0, multi-target both.
* When using one framework, use `<TargetFramework>`
* When using multiple frameworks, use `<TargetFrameworks>`
* Use `Microsoft.DotNet.Analyzers.Compatibility` package to analyze and check for incompatibilities

## A.1 Exploring the motivations for .NET Core

Xamarin uses the cross-platform Mono implementation of the .NET Framework; an alternative *platform* to the .NET Framework that you can use to build mobile applications.

There are three layers to each stack
1. Complier & CLR
2. Base Class Libraries (BCL)
3. App-specific libraries like Windows Forms and iOS

The *Base Class Libraries (BCL)* are the fundamental .NET types like `int` and `string`, as well as additional APIs for reading files, the API for performing a `Task`, and more. These types are similar between platforms, but not all features of one are available on the other.

Some other platforms include:
* .NET Framework
* Xamarin
* Windows 8/8.1
* Universal Windows Platoform (UWP)
* Windows Phone 8.1
* Silverlight Phone
* Unity
* .NET Compact Framework (CF)
* .NET Micro

Each of these APIs varies *slightly* which made it difficult to switch between them. Developers had to know the minute differences, and sharing code between platforms was difficult.

Also, .NET Framework is an older style of monolithic coding, while the development world is trending towards smaller, cross-platform microservices.

The .NET Core platform includes
* Cross-platform BCL
    * Sometimes called CoreFX
    * https://github.com/dotnet/corefx
* Cross-platform runtime
    * CoreCLR
    * https://github.com/dotnet/coreclr
* .NET CLI tooling
    * `dotnet` tool in the command line
    * https://github.com/dotnet/cli
* ASP.NET Core Libraries
    * Build ASP.NET Core Applications
    * Later moved to run on the .NET Framework as well

.NET Core is just another platform, but the real advantage came with the release of .NET Standard. Which ensures each platform implements a standard set of APIs.

## A.2 Sharing code between projects

Before the creation of .NET Core, there was a desire to share code between platforms. This was done using Portable CLass Libraries (PCLs). 

### A.2.1 Finding a common intersection with Portable Class Libraries

When creating a library, you can specify which platforms you want to support. The project would then only have access to **the set of common APIs** between the platforms. Each platform added reduces the number of available APIs, since the API must be included in *all* of the selected platforms.

![platform venn diagram](https://dpzbhybb2pdcj.cloudfront.net/lock/Figures/afig02_alt.jpg)

You would then end up with a single project and single package that worked on all the platforms.

[See here for Stephen Cleary's list of profiles you could choose](https://portablelibraryprofiles.stephencleary.com/)

A final issue is that if you wanted to add an additional target platform, you would need to recompile your PCL. That means you need to wait for the profile author to recompile their profile to support the new platform.

### A.2.2 .NET Standard: a common interface for .NET

.NET Standard is the successor to PCL Libraries. It defines a set of APIs that a platform must implement. It doesn't require any downloads on your part, but instead requires that the platform authors must update their code to match the implementations.

Each new version of .NET Standard includes *all* features of the previous versions. So v1.2 includes all APIs for v1.1 *and* v1.0. When developing an app on v1.2, you can reference any library that implements any version of .NET Standard lower than 1.2 as well. 

Example: UWP version 10 supports .NET Standard 1.4. Therefore it also supports .NET Standard versions 1.0 - 1.3. 

![.net backwards compatibility](https://dpzbhybb2pdcj.cloudfront.net/lock/Figures/afig04_alt.jpg)

[See here for a list of platforms and their .NET Standard version support](https://docs.microsoft.com/en-us/dotnet/standard/net-standard).

The author prefers [this example by David Fowler from the ASP.NET team](http://mng.bz/F9q3) to really understand what .NET Standard is in terms of C#. Each interface defines an implementation of .NET Standard, and each new version inherits from the previous version. i.e. `NETStandard1_1 : NETStandard1_0`. You can then target a particular standard, and any platform that implements that Standard will be able to use all the APIs available to it.

**Even if a platform implements a given version of .NET Standard, the method may still throw a `PlatformNotSupportedException`. See section A.3 for more info.**

*Mike's note: what kind of bullcrap is this? Can't wait to hear the justification for this nonsense.*

Somehow, apps targeting .NET Framework 4.6.1 can reference .NET Standard 2.0 libraries despite it only implementing .NET Standard 1.4. This is a special case and only applies to .NET Version 4.6.1 - 4.7.0. 

.NET Framework v4.7.1 targets .NET Standard 2.0.

The .NET Core team built .NET Core 2.0 to match all the APIs available in .NET Framework 4.6.1, the most popular version of .NET Framework. They also added these APIs to .NET Standard 2.0, since they wanted Standard 2.0 to match the API of Framework 4.6.1. However, Framework 4.6.1 *doesn't* contain implementations for some APIs in Standard 1.5 or 1.6. Therefore Framework cannot support Standard 2.0 directly.

Microsoft decided to **allow .NET Framework 4.6.1 to reference .NET Standard 2.0 libraries** in order to promote developers to write for .NET Standard 2.0. 

#### .NET Blog Post

[Link to the blog post here](http://mng.bz/tU6u) which further explains .NET Standard and this situation.

![implementations](https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2016/09/dotnet-today.png)

The .NET platform was forked many times. This was beneficial because it allowed for tailoring .NET to work on a specific platform that a single-platform wouldn't be able to do. It causes issues for developers, however, because there is no unified class library.

The diagram above show the "three main flavors of .NET" that you can target. You need to know all three in order to write code that works on all three. With this system, someone would need to develop new versions of .NET to target new operating systems or device capabilities. .NET Standard helps mitigate this

![.net core chart](https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2016/09/dotnet-tomorrow.png)

Developers only need to know one base class library. Libraries targeting .NET Standard will be able to run on all .NET platforms.

* Applications
    * You don't need to use .NET Standard directly
    * Still benefit indrectly
    * Learning the base class library for one means you can switch to other platforms easily
* Portable Class Libraries
    * PCLs must select the platform you want to target and then shows which API set you can use
    * They are not as predictable as .NET Standard. APIs are the intersection between the selected platforms and will vary.
* Consistency in APIs
    * Of .NET Framework, Core, and Xamarin/Mono, Core has the fewest APIs available
    * Certain platforms had drastically different availability of foundational APIs
    * Differences in the shape of core pieces, especially in reflection
* Versioning and Tooling
    * The goal of .NET Core was to lay a foundation for a portable .NET platform that can unify APIs
    * It was meant to be the next version of PCLs
    * Didn't result in a good tooling experience
    * Resulted in smaller NuGet packages, that could be updated independently
    * However, abstract specifications don't mix with them well because it would require a very specific combinations to run on a platform
    * To avoid it, they defined .NET Standard as a single NuGet package
    * Only represents the set of required APIs, no need to break it up further
    * Only important dimension is version; the higher the version, the more APIs
    * The lower the version, the more platforms that have implemented it


To summarize, we need .NET Standard for two reasons:
1. Driving force for consistency
2. Foundation for great cross-platform tooling

.NET Standard is also compatible with PCLs. You can see the mapping from .NET Standard version [here](https://docs.microsoft.com/en-us/dotnet/articles/standard/library).

> In .NET Standard 2.0, we’ll make it possible for libraries that target .NET Standard to also reference existing .NET Framework binaries through a compatibility shim

They're starting to lose me here...

* .NET Framework 4.6.1 is the most frequently used version, so .NET Standard 2.0 should implement all its features
* .NET Core would need to be majorly updated so that it supports the upgrades to .NET Standard 2.0
    * Only requires updates to the SDK and NuGet packages
* Xamarin already supports most of .NET Standard, but they hope to updated it to include all the APIs for Standard 2.0 as well

> In order to allow .NET Framework 4.6.1 to support .NET Standard 2.0, we had to remove all the APIs from .NET Standard that were introduced in .NET Standard 1.5 and 1.6.

After looking at NuGet, only six non-Microsoft packages used APIs on .NET Standard 1.5 or later, and Microsoft has reached out to them to mitigate the effects and replace the calls with implementations from .NET Standard.

At this point I stopped reading the blog post because it was a bit over my head and it's a Tuesday after a 3 day weekend...

![.NET Framework compatibility](https://dpzbhybb2pdcj.cloudfront.net/lock/Figures/afig06_alt.jpg)

.NEt Framework can also be reference by .NET Standard libraries through the use of compatibility shims.

### A.2.3 Fudging .NET Standard 2.0 support with the compatibility shim

When .NET Standard was first released, no libraries (NuGet packages) would implement it. Since .NET Standard libraries can only reference other .NET Standard libraries, the packages those packages reference must also be updated. 

The compatibility shim was an attempt to speed up this implementation. It allows .NET Standard 2.0 libraries to reference .NET Framework 4.6.1 libraries. This is a special case. 

**If the library uses a .NET Framework specific API, it will throw a runtime exception. .NET tooling will raise a warning every time you build.**

## A.3 Choosing a target framework for your libraries and apps

The framework you use will depend on whether you're going to make an application or a class library.

### A.3.1 If you're building an ASP.NET Core app

**You can't target .NET Standard, since it isn't a platform, it's a specification. Instead you need to run on an implementation like .NET Framework or .NET Core.** You will then be able to refences libraries that target .NET STandard.

The author recommends targeting .NET Core when possible, or .NET Framework 4.7.1. Only use .NET Framework 4.6.1 as a last resort. You can update which framework you are targeting by editing the `<TargetFramework>` tag in the .csproj file of your project.

You can also multi-target frameworks, but the author has never found this particularly useful. When doing this, use the `TargetFrameworks` tag instead.

### A.3.2 If you're building a shared library or NuGet package

The author recommends using .NET Standard in most cases. It lets you reach the largest audience. You should also target the lowest possible version, since it will reach the largest audience. This will require trial-and-error to figure out which version has the minimum APIs you require.

Rules from the author:
* If you need to target .NET Standard 1.4 or below
    * No issues
* .NEt Standard 1.5 or 1.6
    * May contain APIs found in .NET Core 1.x but not in .NET Framework 4.6.1
    * Your library will likely fail on .NEt Framework 4.6.1
    * May be best to target .NET Core directly
* .NET Standard 2.0
    * Most common case probably
    * Multi-target your libraries to .NET Framework 4.6.1 and .NET Standard 2.0 to be safe

**If you're targeting .NET Standard 2.0 and there's a chance it will be used by a library targeting .NET Framework 4.6.1, then multi-target to .NEt Standard 2.0 and .NET Framework 4.6.1**
`<TargetFrameworks>netstandard2.0;net461</TargetFrameworks>`

### A.3.3 Referencing .NET Framework libraries from .NET Standard projects

If you reference a .NEt Framework library from a .NEt Standard Library or .NEt Core 2.0 app, you will receive multiple warnings
* `Package 'package name' was restored using '.NetFramework,Version=v4.6.1' instead of the project target framework '.NetCoreApp,Version=v2.0'. This package may not be fully compatible with your project.`
    * This warning will show up on the library reference in Solution Explorer
    * This warning will show up every time you build


You can ignore these errors by adding `NoWarn=NU1701` attribute to the package reference in the .csproj file, or right-clicking the Solution Explorer, selecting Properties, and enter `NU1701` for `NoWarn`. 

![enabling no-warn for a reference using the gui](https://dpzbhybb2pdcj.cloudfront.net/lock/Figures/afig09_alt.jpg)

You can also use the Roslyn API analyzer NuGet package. It will check your code for problems. `Microsoft.DotNet.Analyzers.Compatibility`. This package will analyze your code and warn you when you make an incompatible call or are using deprecated APIs.

![Roslyn analyzer warnings](https://dpzbhybb2pdcj.cloudfront.net/lock/Figures/afig10_alt.jpg)
