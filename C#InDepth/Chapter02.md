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
