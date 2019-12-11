# Chapter 2. Functional Programming Techniques for Concurrency

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
