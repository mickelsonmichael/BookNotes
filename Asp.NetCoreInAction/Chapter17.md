# Chapter 17: Monitoring and troubleshooting errors with logging

### 17.3.1 Adding a new logging provider to your application

Third party logging aggregator services
* [Loggr](http://loggr.net)
* [Elmah.io](https://elmah.io)
* [Seq](https://datalust.co/seq)

The author uses the open source library [Serilog](https://serilog.net) to write to multiple log locations at one time.

### 17.3.2 Replacing the default `ILoggerFactory` with Serilog

Serilog is an open source library that can write to multiple locations including the console, a database, the filesystem, or [Elastisearch](https://www.elastic.co/). It does this through logging destinations called [sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks).

Configuring Serilog is easy
1. Add a package reference with VS, the CLI, or the .csproj
2. Create the logger and configure the sinks
3. Call `UserSerilog()` in Program.cs

Instead of a factory going to many providers, serilog is a factory going to a single provider that leads to many sinks.

By calling `UserSerilog()` you are automatically replacing the built in `ILoggingProvider`.

*Enrichers* can be used to automatically add additional information to all the log messages like process ID or environmental variables.

## 17.4 Changing log verbosity with filtering

When applying filters to a log, the **most specific** rule will be chosen. And provider-specific rules override category rules.

It is common practice to include logging settings in the appsettings.json file.

## 17.5 Structured logging: creating searchable, useful logs

Logs stored in a .txt file are *unstructured* logs; they are difficult to traverse and analyze because you are limited to searching for particular strings and values.

Enriched logs allow for additional properties to be stored as key-value pairs. You can then search and filter results based on these properties.

### 17.5.1 Adding a structured logging provider to your app

[Seq](https://getseq.net) lives on a server and collections messages via HTTP much like a REST Api, and is currently (as of writing of the book) undergoing changes to make it cross platform.

ASP.NET Core treats each parameter in a message as a key value pair.

`log.Info("This is the {firstParameter}, and this is the {second}", "first", "second")`

In this example, the structured logger would include a key value pair `<firstParameter, "first">` and `<second, "second">` in the data it collects from the log. You could then use those values to filter your results using SQL-like queries within Seq.

### 17.5.2 using scopes to add additional properties to your logs

Logging scopes group a number of operations by adding the same data to all of them. You can create a scope by creating a using statement, then each log message provided within that scope will have the additional data provided by the scope. Once the using is closed, all further logs will no longer have that additional information.