# Chapter 2 - C# 2

## 2.1 Generics

[Blog Post by Eric Lippert on covariance and contravariance](http://mng.bz/gYPv). This links to just part two of eight, and it may not be necessary to read them all.

When using type constraints for generics (`where T : constraint`), you can use multiple different constraints.

- Reference type constraint
  - `where T : class`
  - Not only classes, but **any** reference type
- Value type constraint
  - `where T : struct`
  - must be a **non-nullable** value type (struct or an enum)
- Constructor constraint
  - must have a public, parameterless constructor
  - Allows the use of `new T()` inside the method
- Conversion constraint
  - Class, interface, or another type parameter
    - `where T : Person`
    - `where T : IPerson`
    - `where T1 : T2`

When there are multiple generic parameters, each can have its own `where` statement, like below

```c#
TResult DoStuff<TArg, TResult>(TArg input)
    where TArg : IPerson
    where TResult : class, new();
```

The `default` operator can be used on nullable value types and will return the "null value for the type." When using it on a `string` it defaults to null as well.

If you use the `typeof` operator on something like a collection of generic types (e.g. `List<T>`), then the result will be something like:

```c#
System.Collections.Generic.List`1[System.String]
```

The 1 in this instance corresponds to the arity of the generic. There is only 1 generic parameter, so the arity is 1. If you had `IDictionary<T1,T2>` then you would get an arity of 2 and

```c#
System.Collections.Generic.Dictionary`2[System.String, System.Int32]
```

## 2.2 Nullable Value Types

Nullable value types were added to make it easier to represent the absence of information while also reducing the amount of errors programmers run in to.

There is a parameterized version of the `GetValueOrDefault(T defaultVal)` that will return the passed argument instead of the default value of the type. For example, doing `GetValueOrDefault(100)` will return `100` instead of `0` when the value is null.

Using `Nullable<int>` is the same as `Nullable<Int32>` is the same as `int?` is the same as `Int32?`. Whichever you use, the IL code generated will be the same.

The line `if (x != null)` and `if (x.HasValue)` are equivalent, since the former is simply a null literal that checks the `HasValue` property for `false`.

Most operations between two `Nullable<T>` operate how you would think; the one exception being that the `<=` or `>=` operators return false when comparing two null values, despite the fact that `==` would return true. It's unlikely that this will cause issues, but it's possible.

Note that some languages like VB treat lifted operators differently.

> Using the as operator with nullable types is surprisingly slow. In most code, this is unlikely to matter (it’s not going to be slow compared with any I/O, for example), but it’s slower than is and then a cast in all the framework and compiler combinations I’ve tried.

In a null-coalesce function, if the first operand is `Nullable<T>` and the second operand is `T` then the result of the whole expression is `T`.

```c#
int? first = 5;
int second = 6;

int result = first ?? second; // ok
```

## 2.3 Simplified Delegate Creation

The purpose of a delegate is to "encapsulate a piece of code so that it can be passed around and executed as necessary in a type-safe fashion in terms of the return type and parameters."

A method group is a group of methods with the same name but different parameters.

## 2.4 Iterators

The use of the `yield` keyword is ideal because it allows for lazy-execution. You can break out of a loop/iteration and stop the execution of the rest of the function since each `yield` is calculated one at a time with each loop. The program remembers the last `yield` it touched and continues form the point where it left off each time.

When breaking out of an iteration function with a try...finally block, once you break out of the execution (because some condition is met to stop iteration), then the finally block is automatically called, even if breaking before iterating through all the options. This is because a `foreach` loop is essentially wrapped in a `using` statement, which automatically runs the `finally`.

## Meeting \#1 (January 23, 2020)

Lyn and I both didn't finish this chapter, so we've decided to break it down into two parts. We probably will also do this for the main Book Club as well, since this chapter is quite a bit to absorb.

She figures we could go to 2.3, but I think we should make it to 2.4 since 2.3 is on Delegates, which is more of an FYI in my opinion, since most of it is superseded by lambdas.

We were talking briefly about partial classes and how we could use them win the www project to break up the Accord.cs file. That way it's multiple files but it would be compiled into one large class.

We think a lot of the older stuff is just fun to know but not very useful, like `StringCollection` and delegates.

## Meeting 2/4/2019

Sounds like everyone is on board for doing a short presentation for each of the sections (Dan wasn't here but I think he's overruled). See below for the breakdown. It's uncertain whether or not we're doing all of chapter 2 or not, however. Right now it's a tentative plan to read the entire thing, but we'll do a check-in on Monday or perhaps later this week to see how people are feeling about it, and go from there.

- Brian - Generics (2.1)
- Kal - Nullable value (2.2)
- Dan - Delegate creation (2.3)
- Kent - Iterators (2.4)
- Mike - Minor Features (2.5)
