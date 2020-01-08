# Chapter 3. Functional Data Strictures and Immutability

Programs have two basic concepts:

1. Data
2. Data manipulation

Functional programming fits well into this since it promotes a series of data transformations and does so with the original data remaining unchanged; it creates a new copy each time. This reduces the number of potential side-effects caused.

By giving up the ability to mutate values like in C#, and opting for the immutable mindset, you can make synchronous programing much easier and more effortless.

## 3.1 Real-world example: hunting the thread-unsafe object

SignalR is a library for ASP.NET that allows for real-time updates to web pages without the browser having to request the new content. The web sockets open in the browser are able to receive the content from the server as it's available. This is particularly useful in scenarios like a live chat or collaborative editing.

A *thread contention* occurs when a thead is waiting for an object being held by a different thread. This can cause bottlenecks and performance drops.

The example in the book is a web chat application that stores a Dictionary of connected users. This dictionary is thread-safe when multiple threads are only reading it, but when it comes to writing it falls short and can result in thread contention. You can utilize the [lock](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/lock-statement) statement to get around this, but there is a better solution in using immutable objects.

### 3.1.1 .NET immutable collections: a safe solution

You can utilize immutable collections found inside the `System.Collections.Immutable` namespace. Extension methods like `Dictionary<>.ToImmutableDictionary()` cause the function such as `Add()` to instead return a new copy of the `Dictionary`. You can then update the original value with the updated one, while any other threads will still reference the original version, making them thread-safe.

While building the original list, you can utilize the `ImmutableList` class, which simplifies the code for adding new objects.

An issue arises with assigning the updated value to the original, but can be subverted by using a single *compare-and-swap* (CAS) operation.

#### CAS Operations

An operation that alters a state in a single step so that the outcome is autonomous (done or not done) is *atomic*.

*Note to self:* Look this up. He's not doing a great job describing CAS Operations.

Look up information on the `volatile` keyword as well.

Look up the `Interlocked.CompareExchange` function.

#### The `ImmutableInterlocked` Class

Part of the `System.Collections.Immutable` namespace, it will do the same thing the `Atom` class from the previous chapter did. Updating the value of the `ImmutableDictionary` in a thread-safe manner.

This immutability comes at a cost however, and if you want to get truly performant you will need to analyze the best tool for the job. This solution is best for small, thread-safe collections of data.

#### .NET concurrent collections: a faster solution

Utilizing a `ConcurrentDictionary` in the scenario above is the better option if you want more speed.
