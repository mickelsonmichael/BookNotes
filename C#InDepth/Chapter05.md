# Chapter 5 - Writing Asynchronous Code

- The Windows Runtime platform is commonly known as *WinRT*
- In general, you don't need to dispose of tasks. See [this blog by Stephen Toub](https://devblogs.microsoft.com/pfxteam/do-i-need-to-dispose-of-tasks/). The blog linked in the book is Archived but this is the best guess at what the link should have been
- `HttpClient` is kind of like an improved `WebClient`. It's preferred for .NET 4.5+ and contains only asynchronous operations
- With an `async` method, it returns as soon as an `await` expression is reached
- A continuation is a callback to be executed when an asynchronous operation (or any `Task`) is completed
- The `Task` class has a `Task.ContinueWith` method for attaching continuations
- The `await` keyword basically asks the compiler to build a continuation on your behalf
- For more information on `SynchronizationContext`, read [this article by Stephen Cleary](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext)
  - Especially important for ASP.NET developers
  - ASP.NET context can easily create deadlocks in code that looks fine
- In reality, the language doesn't actually need the `async` keyword; the keyword could be inferred from an `await` present in the body
  - It is required because it's good practice and leads to much more readable code
- The `async` modifier has not representation in IL Code
- `Task<TResult>` represents an operation that returns a value of `TResult`
- `Task` operations do not produce a result and are essential equivalent to `Task<void>`
- You *can* return `void` from async methods, which was designed for compatibility with event handlers
  - This is the only time you should return `void` from an `async` method, since returning `Task` gives the caller some control
- `async` methods can be generic, static or nonstatic, and specify any of the regular access modifiers
- None of the parameters in an `async` method can use the `out` or `ref` modifiers
- Pointer types can't be used as `async` method parameter types
- When implementing the *awaitable* pattern, the return type of the `GetResult` method determines the type of task
- You cannot use the `await` operator within a `lock`; `lock` statements and asynchronoy don't go well together
- When returning from a method with a return type of `Task<TResult>`, the wrapping is done for you in generated code
  - If your return statement occurs within the scope of a `try` block with a `finally` block, the expresion used to compute the return value is evaluated immediately, but doesn't become the result of the task until everything is cleaned up
  - If the `finally` block throws an exception, the task fails, even if there is a return value available
- When an `await` operation failes, if it captured an exception, the exception is thrown
- You should always write async methods so they return quickly
- You should generaly avoid performing long-running blocking work in an async method
  - Separate it into another method that you can create a `Task` for
- Exception unwrapping is important since if you await an asynchronous operation that's failed, it may have failed a long time ago in a different thread
  - i.e. creating and starting the task, then attempting to get the result later will mean there's been an exception in the stack for several lines of code
  - This prevents the usual way of propogating exceptions up the stack
- When you await a task, if it's either faulted or canceled, an exception will be thrown
  - For convenience, the first exception within the `AggregateException` is thrown
- If an exception occured in the `WhenAll()` method, then only the first exception would be thrown
  - To check the rest of the exceptions, you can use `Task.Exception` for each of the individual tasks

## Lazy Exceptions and Argument Validation

- If you validate parameters in async code as you would in normal code, the caller won't have any indication of the problem until the task is awaited
- You want to write a *nonasync* method that retuns a task and validates the arguments before calling a separate async function that assumes all valid arguments
  - Separate async method
  - Anonymous async method
  - **Local async method (C#7+)**

## Handling Cancellation

- To get a cancellation token, create a `CancellationTokenSource` then use the `Token` property to get the `CancellationToken`
  - This token can then be passed to the async method
- The token source allows you to pass the same token to multiple async methods and cancel them all with one action in any of the methods
- If an async method throws an `OperationCanceledException` the retuned task will have a status of `Canceled`
  - Any other exception would result in a faulted status, even if it is thrown outside of the context of a cancellation token
  - This concept is the contradiction to the C# specification, which states that *any* exception should result in `Faulted`
- A method that is marked `async` but doesn't contain an `await` is run asynchronously
- Task cancellation is propogated normally unless you intercept the `OperationCancelledException`

## Async Anonymous Functions

- You can't use asynchronous anonymous functions to create expression trees
- When creating an async anonymous function, the operation doesn't start until the delegate is invoked, and multiple invocations start multiple separate operations

## Custom task types in C\# 7

- `ValueTask<TResult>` is very similar to a task, but is a value type
- This is useful when you need to create highly performant code, since you don't need to allocate a new `Task` every time
- In most cases, you don't expect the task to be completed before you await it, so `ValueTask` provides little benefit, since the continuation must be created anyway
- `ValueTask<TResult>` shines when, in most conditions, the awaited task is completed by the time the value is awaited
- "The async/await infrastructure caches a task that it can return from any aync method declared to return `Task` that completes synchronously and without exception"
  - Successful tasks are cached

## Async main methods in C\# 7.1

- Unlike most async methods, an async entry point can't have a return type of `void` or use a custom task type
- Async entry points are mostly helpful when writing small tools or demo code

## Usage Tips

- When you're writing library code or in an application that doesn't touch the UI thread, you don't want to come back to the UI thread
  - `ConfigureAwait` method takes a paremeter that determines whether the returned awaitable will capture the context when it's awaited
- `ConfigureAwait(false)` will prevent the context from being captured and the continuation running on the orginal context; a thread-pool thread will be used instead
- Jon has used the `ConfigureAwaitChecker.Analyzer` NuGet package in the past to ensure he doesn't miss a `ConfigureAwait` opportunity
  - [Link to NuGet page](https://www.nuget.org/packages/ConfigureAwaitChecker.Analyzer/)
- You can leverage parallelism by starting tasks early and awaiting their results when you need them
- When running multiple tasks at once, you might want to utilize `Task.WhenAll` so that you can log all the task failures independently
- Be wary of `Task<TResult>.Result` property and `Task.Wait()` methods; they can lead to deadlocks
- Read the blog posts by Stephen Toub for more info on wrapping async methods in synchronous implementations (and visa versa)
  - ["Should I expose synchronous wrappers for asynchronous methods?"](https://devblogs.microsoft.com/pfxteam/should-i-expose-synchronous-wrappers-for-asynchronous-methods/)
  - ["Should I expose asynchronous wrappers for synchronous methods?"](https://devblogs.microsoft.com/pfxteam/should-i-expose-asynchronous-wrappers-for-synchronous-methods/)
  - Summary: No, you should not
- In general, add an option for cancellation from the beginning; better to have it and not need it to need it and not have it. Implementing the cancalltion after the task is much more difficult than adding it right away
- When testing, you can use `Task.FromResult`, `Task.FromException`, and `Task.FromCanceled` to return expected values
- You can also use `TaskCopletionSource<TResult>` for more in-depth scenarios
  - Allows you to create an ongoing task then you can set the value for later to mark as completed

## Blog Post - *Understanding the Whys, Whats, and Whens of ValueTask*

[.NET Blog Post](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/?WT.mc_id=ondotnet-c9-cxa)

- `Task` is essentially a promise; you initiate a `Task` and can wait for the operation to complete.
  - Synchronysly: value is available when you use it; data was already available
  - Asynchronously but complete: value wasn't available, but the `Task` was completed before you used the value
  - Asynchronously but incomplete: value wasn't available and you must wait for the `Task` to complete
- Using the `await` keyword makes it easy to generate a callback once the `Task` is completed. The compiler does all the work and optimization for you
- Because `Task` is a class, it comes with the downsides of allocation; you must allocate memory on the heap and force the GC to free the resources instead of freeing other things