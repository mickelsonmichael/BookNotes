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

**NOTE: MISSING NOTES HERE**

### 3.3.6 Building a persistent data structure: an immutable binary tree

This section reviews building a binary tree, or B-Tree, in F# with recursion. Tree structures prevent cycles and are used for performance. The .NET Framework never shipped with a tree collection, however. A binary tree is a tree where each node has between zero and two branches and the height of the tree between any leaves is at most one.

**Depth of a Node:** the number of edges from the node to the root node

**Leaf:** a node with no children

**Siblings:** children of the same parent

**Root:** the only node in a tree with no parent; can also be a leaf if there is only one object in the tree

#### B-Trees in Functional F\#

Binary trees are easily represented in F# because of the presence of ADTs and discriminated unions (DUs). F# also seems to provide a `Node` type for this as well. [See the MDSN documentation for more information](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions#using-discriminated-unions-for-tree-data-structures).

## 3.4 Recursive functions: a natural way to iterate

In functional programming, each iteration passes a new copy of the value due to the immutable nature of FP. This concept is the basis of the *Divide and Conquer* pattern, which involves breaking a problem down into sub-problems that can be solved individually and then brought together for the final solution. This pattern also leads to *dynamic task parallelism* where tasks are added as the iterations advance.

Recursion can lead to slow computations and `StackOverflowException`, but you can utilize the strategies of *tail recursion* and *CPS* to minimize this risk.

### 3.4.1 The tail of a correct recursive function: tail-call optimization

**Tail Call:** also known as *tail-call optimization (TCO)* is the call performed as the last act of a procedure. This improves efficiency because the callee uses the same stack space as the caller.

**Tail Recursive:** when a tail-call calls the same subroutine again

**Tail-Call Recursion:** converts a recursive function into an optimized version; the last call of the function is a call to itself

Unfortunately, the **C# compiler doesn't currently do the optimization necessary for the tail-call**, and there will be no real gain in writing code this way.

### 3.4.2 Continuation passing style to optimize recursive function

Since C# cannot use tail-call optimizations, we can instead implement CPS, where the result of a function is passed into a continuation, which avoids stack allocation. This method is used in the `async-await` TPL in C#. See the example below for a non-recrusive CPS.

```c#
public static void GetMaxCPS(int x, int y, Action<int> action)
{
    var val = x > y ? x : y;
    action(val);
}

public static void Main(string[] args)
{
    GetMaxCPS(5, 7, n => Console.WriteLine(n));
}
```

Note that the function `GetMaxCPS` doesn't do anything with the result itself, but merely passes it on to the delegate `action`. It passes the value to the "continuation procedure."

#### Recursive Functions with CPS
