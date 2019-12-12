# Chapter 2. Functional Programming Techniques for Concurrency

## Summary

* Function composition applies the result of one function to the input of another, creating a new function
* Closure is an in-line delegate/anonymous method attached to it's parent method, where the variables defined in the parent's body can be referenced from within the anonymous method
* Closure provides a convenient way to give a function access to local state, even outside of scope
* Memoization is a technique that caches results instead of recomputing them.
* Precomputation performs an initial computation that generates a series of results, usually in the form of a lookup table.
  * This can be shared to reduce the number of unnecessary computations.
  * Generally replaces memoization and is used in combination with partial applied functions
* Lazy initialization is another variation of caching
  * Defers the computation of a factory function until needed
  * Creates the object only once
  * Improves performance by reducing memory consumption and reducing unnecessary computation

## 2.1. Using function composition to solve complex problems

*Function composition* - combining of functions in a manner where the output from one function becomes the input for the next function, leading to the creation of a new function

* Can continue endlessly
* Creates complex functions to solve complex problems

The functional paradigm serves to make more readable code with no side effects, preserving the logic of parallelism. You can work from a big problem and simplify it one step at a time until you can simply solve it, then reconnect all the smaller solutions into one large solution.

### 2.1.1 Function composition in C\#

C# doesn't include function composition natively, but it can be implemented using a simple method. You can create an extension method that can be used to combine any two functions with on or more generic input arguments.

``` c#
static Func<A, C> Compose<A, B, C>(this Func<A, B> f, Func<B, C> g) => (n) => g(f(n));

Func<CoffeBeans, Espresso> makeEspresso = grindCoffe.Compose(brewCoffe);
Espresso espresso = makeEspresso(coffeBeans);
```

If you felt so inclined, you could look at the source code for this book and find several more extension functions that could help you achieve this paradigm. 

### 2.1.2 Function composition in F\#

F# natively supports function composition and is built into the language with the `>>` infix operator. The operator will combine existing functions to build new ones.

``` f#
let add4 x = x + 4
let multiplyBy3 x = x * 3

let list = [0..10] // defines a range of numbers from 1 to 10

// List.map is equivalent to the .Select() statement in LINQ

let newList = List.map(fun x -> multiplyBy3(add(x)) list)
// vs
let newList = list |> List.map(add4 >> multiplyBy3)
```

The function composition style native to F# allows the process to be read from left to right, which is the more natural way to read the line and adds to the comprehensibility.

## 2.2. Closures to simplify functional thinking

*Closure* - first-class function with free variables that are bound in the lexical environment

They are the more convenient way to give functions access to local state and pass data into background operations. They allow a function to access one ore more non-local variables when invoked outside its immediate lexical scope.

*[Free variable](https://stackoverflow.com/a/21856306/3338349)* - a variable used in some function that its value depends on the context where the function is invoked, called or used. The function will need to be determined at runtime by stepping backwards up the call chain to find the sender.

Closures have been in .NET since 2.0, so are a common feature and comes naturally. Lambda expressions are a common use case for closures. 

``` c#
double salesTax = 0.06;

// the salesTax variable is referenced within the scope of the lambda function
Func<double> getTotal = (price) => price * salesTax;

var total = getTotal(10.00);
```

### 2.2.1 Captured variables in closures with lambda expressions

Closures also allow variables to be maintained even when they would have normally gone out of scope; the variable is captured and not subject to garbage collection.

You can consider the captured variables as snapshots of themselves. Snapshots taken at the time of evaluation and not at the time of capture. If you were to alter the variable outside of a closure before the closure had executed, the updated value would be read by the closure instead of the value at time of capture.

### 2.2.2 Closures in a multithreading environment

The issue of the value of a variable being updated before a closure is ran can be avoided by using the `Parallel.For` method from the Task Parallel Library. You could also capture a new unique temporary variable for each task, but this is not ideal.

When F# runs a `for` loop, instead of updating the value of the index with each loop, instead a new immutable index is created for each loop. That means the issues that C# suffers from in terms of closures not having access to the loop is solved.

## 2.3 Memoization-caching technique for program speedup

*Memoization* - see also *tabling*. Caching the results of a function to avoid repeated calculations. It keeps the result in memory so it can be returned immediately in future calls.

Memoization doesn't always result in performance increases. For instance, in the case of a Dictionary, the time it takes to calculate the hash for a string is proportional to the length. So it is your best interests to utilize a profiler and determine whether your scenario is ideal for using memoization.

## 2.4 Memoization in action for a fast web crawler

The book gives an example of a web crawler that takes a URL, finds the title and records it, then finds all links on the page and performs the same action again for the new URLs, recursively returning a list of titles.

In this scenario, there will inevitably be duplicate URLs and a lot of time would be wasted doing the same calculation repeatedly. Instead, memoization should be leveraged so that duplicate computations are avoided.

You could also utilize the LINQ extension [AsParallel](https://docs.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.asparallel?view=netframework-4.8) to automatically bind the query to PLINQ.

Issues arise from this however when threads come into play and you are retrieving from a non-thread-safe cache. Instead utilize something like the `ConcurrentDictionary`, which not only allows for the process to be run in parallel but even reduces the amount of code.

## 2.5 Lazy memoization for better performance

Even with the caching, you are still missing some critical performance boosts. If the functions are running in parallel, it may be possible that two or more may try to call for the same URL at the same time; this will result in two calls to the same URL instead of the ideal one. To get around this you can utilize the `Lazy<>` construct to get your results.

``` c#
Func<T, R> MemoizeLazyThreadSafe<T, R>(Func<T, R> func) where T : IComparable
{
    var cache = new ConcurrentDictionary<T, Lazy<R>>();

    return arg => cache.GetOrAdd(arg, a => new Lazy<R>(() => func(a))).Value;
}
```

### 2.5.1 Gotchas for function memoization

Storing the cache in a dictionary isn't a good long-term solution; items are not removed and only added. Leading to a potential memory leak issue. 

This can be averted by utilizing the `ConditionalWeakDictionary`, a dictionary where the key is held as a *weak reference* and the values are kept only as long as the key is. The key is subject to garbage collection, thus the data is also subject to garbage collection. 

Another solution is to store a timestamp with your cached objects, and occasionally check for the oldest items and remove them.

You should only perform memoization when the cost of storing the items is less than the cost of calculating them. You should benchmark your process with and without memoization before you commit to the changes.

## 2.6 Effective concurrent speculation to amortize the cost of expensive computations

*Speculative Processing* - Precomputation. Computations are performed before the results are needed, when all the required parameters are available.

*Amortization* - paying off an amount owed over time by making planned, incremental payments of principal and interest.

*Jaro-Winkler distance* - measures the similarity between two strings. The higher the distance the mor similar the strings. Best suited for short strings like names. 0 is no similarity 1 is an exact match

### 2.6.1 Precomputation with natural functional support

F# will allow you to leverage currying and partial application of functions to calculate the value of some underlying data source ahead of time.

### 2.6.2 Let the best computation win

You can also utilize a method that returns the quickest of two potential calculations first.

The book uses the example of a weather app. You could enter a search to retrieve the weather in a particular city, then request two different services make the computation. Whichever service returns first will be the winner and their result will be returned; the other computation will be cancelled.

## 2.7 Being lazy is a good thing

*Lazy evaluation* is a technique to defer the evaluation of an expression until the last possible moment. When it is accessed. This results in faster programs by preventing excessive computations. 

### 2.7.1 Strict languages for understanding concurrent behaviors

*Eager evaluation*, or *strict evaluation*, is the opposite of lazy evaluation. C# and F# are both strict languages.

**Book Recommendation**: ["Why Functional Programming Matters" by John Hughes](http://mng.bz/qp3B)

Microsoft introduced the `Lazy<T>` construct with Framework 4.0. You define what type of object the `Lazy` is meant to represent then provide it with a value factory; a method of retrieving the value when it is needed.

### 2.7.2 Lazy caching technique and threading-safe Singleton pattern

Because the operations are done on demand and only once, the `Lazy<T>` construct is an ideal mechanism for implementing a Singleton pattern.

[Implementing a Singleton in C#, MSDN](http://mng.bz/pLf4)

*Double-Checked Locking* - design pattern that reduces the overhead of acquiring a lock by first testing the locking criterion without acquiring the lock.

You can pass an optional second parameter into the `Lazy<T>` constructor to force it to e thread-safe. This is enabled by default. No matter how many threads call for the object, they will all receive the same instance, which is cached after the first call.

Keep note, while retrieving the object may be thread safe, it doesn't guarantee that the properties of that object are thread safe as well.

#### LazyInitializer

`LazyInitializer` is an alternative static class like `Lazy<T>`, but with optimized initialization performance and more convenient access.

``` c#
private Image bigImage;

public Image BigImage => LazyInitializer.EnsureInitialized(ref bigImage, () => new Image());
```

### 2.7.3 Lazy support in F\#

F# uses the same `Lazy` construct as C# but slightly differently. The result `T` is automatically calculated from the result, and instead of utilizing `.Value`, you use `.Force()` to force the calculation of the object when you need it.

### 2.7.4 Lazy and Task, a powerful combination

You can use `Task` and `Lazy` together to improve your program even further, by returning a `Lazy<Task<T>>`. Then your requested value can be retrieved asynchronously at the time it is needed.

However, with an asynchronous lambda expression, it can be executed on any thread that calls `Value` and the expression will run within the context.

You are better of wrapping the expression in a `Task`, which will force the asynchronous execution on a thread-pool thread.

## Meeting Notes (12/12/2019)

Finding that in chapter 2 he shows C# examples, then mentions the issues and moves on to F# examples. So we've been glancing past the F# examples since we aren't well versed with the language and staring isn't helping.
