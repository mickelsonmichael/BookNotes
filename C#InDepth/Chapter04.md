# Chapter 4 - Improving Interoperability

- Thanks to the `dynamic` type, C# is both statically *and* dynamically typed, which is rare for a programming language.

## 4.1 Dynamic Typing

- When using dynamic operations, the lookups are done **at execution** time, so the compiler won't be able to catch any typing issues.

```c#
    dynamic text = "my text";
    string my = text.Substring(2); // the lookup for this method happens at runtime, but will succeed
    string broken = text.SUBSTR(2); // the lookup here fails, but not until runtime
```

- There is no CLR type for `dynamic`, instead it is an `object` decorated with the `[Dynamic]` attribute (for method names, this is not necessary for local variables).

### Rules for the `dynamic` type

1. Implicit conversion from any (non-pointer) type to `dynamic`
    - `dynamic str = "string";`
2. Implicit conversion from an expression of type `dynamic` to any (non-pointer) type.
    - `string result = str.Substring(2);`
3. Expressions that involve a value of type `dynamic` are *usually* bound at execution time
    - `var result2 = str + "here I go"`
4. *Most* expressions that involve `dynamic` have a compile-time type of `dynamic` as well
    - `var result = str.Substring(2); // compile time type = dynamic`

- Binding at execution time can utilize user-defined implicit conversions as well
- *Almost* all operations with dynamic values have a result that is also dynamic
  - Exceptions include constructors, the `is` keyword, and the `as` keyword

## ExpandoObject: A Dynamic Bag of Data and Methods

- An object found in the `System.Dynamic` namespace
- Operates in two modes; dynamic and static
  - In a static context, it is an `IDictionary<string, object>` and behaves like a normal Dictionary.
- It also implements `IDynamicMetaObjectProvider`
  - Allows the definition of custom dynamic behavior
  - Is a complex option and not the usual go-to method
- Allows you to add and read properties via the dictionary, but in a dynamic way (e.g. `var val = myObject.randomProperty);`)

## Creating a Dynamic Implementation

- Besides implementing the complicated `IDynamicMetaObjectProvider`, you can also simply inherit the `DynamicObject` class
- The `DynamicObject` class is much simpler and makes implementation much faster
- All the methods return a `bool` with the result of the operation; if a `false` is returned, then a `RuntimeBinderException` will be thrown

## Limitations of Dynamic Typing

- The execution-time binder doesn't resolve extension methods
- Anonymous methods can't be assigned to a variable of type `dynamic`
  - `dynamic funciton = x => x * 2;` is **invalid**
  - You can use a cast or an intermediate statically typed variable instead
    - `dynamic function = (Func<dynamic, dynamic>) (x = x * 2);` is **valid** because there is an explicit cast
- Lambda expressions can't appear within dynamically bound operations
  - `dynamic result = dynamicObj.Select(x => x * 2);` is **invalid**
  - Again, if you cast the method explicitly you can work around this like `dynamic result = dynamicObj.Select((Func<dynamic, dynamic>)(x => x * 2));`
- Lambda expressions that are converted to expression trees *must not contain any dynamic operations*
  - Even with wcasting, `var query = dynamicObj.AsQueryable().Select(...);` will always fail to compile since `AsQueryable` builds an expression tree

## Optional Parameters and Named Arguments

- Parameters with `ref` or `out` modifiers are *not* permitted to have default values
- An argument without a name is called a *positional argument* and is the most common type
- The default value for an argument can be a default expression like `default(int)`
  - In C# 7.1 and later, you can just write `default` using the `default literal`
- The default value for an argument can be a `new` expression for value types (e.g. `new Guid()`)
- Named arguments can be specified in any order
- Arguments are evaluated in the order that they appear in the source code, left to right
  - This is important if the argument evaluation has side-effects (i.e. incrementing a value like `Method(i++, i)` will result in a different value of `i` for both parameters)
- When overload resolution occures, a method that has no optional parameters without corresponding arguments is matched before a method with at least one optional parameter without a corresponding argument
  - `Method(int i, int j)` will be matched instead of `Method(int i, int j = 0)`
  - However, more optional parameters does not make a method a worse or better match. `Method(int i, int j = 0)` will not be a better or worse match than `Method(int i = 0, int j = 0)`

## Generic Variance

- **Covariance** occures when values are returned only as output
  - `out`
  - `public interface IEnumerable<out T>`
- **Contravariance** occurs when values are accepted only as input
  - `in`
  - `public delegate void Action<in T>`
- **Invaraince** occurs when values are used as input and output
  - `public interface IList<T>`
- The modifiers `in` and `out` are used to specify the variance of a type parameter
- Each type parameter can have only one of the modifiers, but a type can declare multiple
  - `public TResult Func<in T, out TResult>(T arg)`
- Variance isn't inherited by the classes or structs implementing the interface
  - *Classes and structs are always invariant*
- Variance plays a big role in how conversions can happen, via *variance conversion* (a type of *reference conversion*)
  - *Reference Conversion* - a conversion that doesn't change the value involved (always a reference) and only changes the compile-time type
  - *Identity Conversion* - a conversion from one type to the same type as far as the CLR is concerned
    - e.g. `string => string` or `object => dynamic`

### Variance Conversion

The basic rules given a generic declaration `T<X>` and attempting to convert from `T<A>` to `T<B>`:

1. If `X` is covariant (`T<out X>`) there must be an identity or implicity reference conversion from `A` to `B`
2. If `X` is contravariant (`T<in X>`) there must be an identity or implicit conversion from `B` to `A`
3. If `X` is invariant, there must be an identity conversion from `A` to `B`

In a real example using `Func<in T, out TResult>`, the rules result in the following:

- Conversion `Func<object, int> => Func<string, int>` is **valid** because
  - `T` is contravariant, and there's an implicit reference conversion `string => object`
  - `TResult` is covariant and there's an identity conversion `int => int`
- Conversion `Func<dynamic, string> => Func<object, IConvertable>` is **valid** because:
  - `T` is contravariant and there's an idtenity conversion `dynamic => object`
  - `TResult` is covariant and there's an implicit reference conversion `string => IConvertible`
- Conversion `Func<string, int> => Func<object, int>` is **invalid** because:
  - `T` is contravariant, but there's no implicit reference conversion `object => string`

Jon suggests reading the third edition to dive into this concept a little more in-depth if you're still having troubles. But most of the time you won't even need to know these things are happening.

The real benefit of generic variance is being able to implicitly cast objects without using the `Cast<T>()` method or any other forms of explicity casting.
