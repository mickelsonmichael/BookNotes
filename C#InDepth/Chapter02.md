# Chapter 2 - C# 2

## 2.1 Generics

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
