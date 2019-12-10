# Chapter 4. Pragmatic Paranoia

**Tip 36:** You Can't Write Perfect Software

As a programmer, you shouldn't trust anyone else's code. Always assume it will fail. And as I pragmatic programmer, you should never trust *yourself* either. Know that you don't write perfect code and that you are going to make mistakes; be ready for them. Build defenses against your mistakes.

## 23. Design by Contract

### DBC

A piece of code should have a *contract* of sorts. It should do no more or no less than what it says it will do. It will also say what it expects to receive to perform that output. 

* Preconditions
  * Routine's requirements
  * It is the caller's responsibility to pass good data
* Postconditions
  * Routine's guarantee
  * Implies that it will conclude
* Class invariants
  * Something that must be true once the proces is *complete*
  * Does not have to be true during the process

> If all the routine's preconditions are met by the caller, the routine shall guarantee that all postconditions and invariants will be true when it completes.

When the contract is breached, a solution needs to be had. Could be an exception or the program terminating. 

**Tip 37:** Design with Contracts

Write "azy" code. Accept little and give little. Only do what the contract states and nothing more.

### Class Invariants and Functional Languages

It's called "class" invariants because it was conceived in the Eiffel language; an object-oriented language. But really it applies to the *state* of things, which can apply to functional languages as well. You pass the state to a function and receive a state back.

### Implementing DBC

When possible, check the inputs of your function to ensure they meet your contract. If your language does not support DBC in code, you can put the contract as comments or in unit tests.

#### Assertions

Assertions are very helpful in at least partially checking these contracts. However, in most OOL there is no propogation of assertions via inheritance. If you override a class, its contract will have to be manually rewritten. There also may come environments where exceptions from assertions are turned off or ignored in the code.

There is also no way to store the "old" values in most languages. You will need to manually save the values in order to check them.

### DBC and Crashing Early

Crash early at the site of the issue. The longer you delay the crash the harder it is to diagnose the cause of the issue and the slower your response time will be. By using preconditions you transfer the burden of responsibility for checking the parameters to the caller instead of the function. That way the function itself can feel safe in performing the actions it needs.

### Semantic Invariants

Use these flexible invariants for contracts that are core to the philosophy of the program. Things that most likely will not change with business rules. In the book, they use an example of debit card transactions and the invariant "Err in the favor of the consumer". A core policy that will most likely not undergo change with a change in management.

## Dynamic Contracts and Agents

You can renegotiate contracts freely. Processes should be able to negotiate contracts themselves ideally. This is some utopia chapter that I really don't think needed to be here...

## 24. Dead Programs Tell No Lies

Each and every switch statement has a *default* for one reason: we want to know when the "impossible" happened.

Often times, instead of blaming the code at fault, we will look for different areas to blame. The data, the server, the user. Instead we should be reading an interpreting the error message.

### Catch and Release is for Fish

Instead of a series of `try...catch` statments that don't provide any additional information, simply allow the exception to occur. This cleans up code significantly and allows you to better get to the source of the issue through the error message.

**Tip 38**: Crash Early

### Crash, Don't Trash

Instead of catching an error and allowing it to continue with a bad configuratio or improper data, *let it crash*. It is easier to determine the problem if you crash straight away at the source of the issue, and it will prevent future issues related to the error from occuring; you stop the damage early.

## 25. Assertive Programming

**Tip 39:** Use Assertions to Prevent the Impossible

Assertions *assert* that something should be true. Use them to check for the impossible, not the expected. Instead of asserting that a function returns an expected result, assert that it does not return an "impossible" result. Asserts plan for the worst case scenario.

You can also catch the errors thrown by assertions and handle them particularly.

### Assertions and Side Effects

Make sure that if you do write assertions, that they do not introduce *more* errors into your code. For instance, accidentally incrementing a value while checking it in the assert.

### Leave Assertions Turned On

Once shipping to production, dont turn off assertions to save resources or speed up code. If you really need to, you can disable the more taxing assertions, but don't remove them all.

An assertion can pay for itself if it catches a dangerous and unexpected issue.

## 26. How to Balance Resources

**Tip 40:** Finish What You Start

The function or object that creates and object should be responsible for deallocating it. This prevents issues with future refactors and coupling of code.

When available, utilize the automatic creation and closing of objects like C#'s `using` statments.

**Tip 41:** Act Localy

### Nest Allocations

Two recommendations

1. Deallocate resources in the *reverse* order they were allocated to prevent orphaned dependancies
2. When allocating the same set of resources in two places, do them in the same order to prevent deadlocks

> If process A claims `resource1` and is about to claim `resource2`, while process B has claimed `resource2` and is trying to get `resource1`, the two processes will wait forever.

### Objects and Exceptions

When coding in object-oriented languages, you can wrap resources in classes that will automatically allocate the resource on object creation and deallocate it when the object has been garbage collected.

### Balancing and Exceptions

In languages that support exceptions, you may run into issues with resource allocation. Generally you have two options

1. Use variable scope
  * In languages with scoping rules, once the variable has gone out of scope, it is removed
  * Rust, C++
2. Use `finally` clause in a `try..catch` block (or C#'s `using` statement)
  * If the language supports it, will automatically occur even if there is an exception

#### An Exception Antipattern

If you are relying on the `finally` pattern to deallocate a resource, be sure that deallocation does not occur before the allocation even happens. If you wrap the allocation in the `try` statement and allocation fails, the `finally` statement will then try to deallocate something that does not exist, resulting in another exception.

### When You Can't Balance Resources

In a process with nested allocations, you have three options

1. The top-level structure frees its resources and the resources of its children
2. The top-level sturcture is deallocated and the sub-structures are orphaned
3. The top-level structure refuses to deallocate until all sub-structures are removed

### Checking the Balance

Create wrappers that check for the status of resources, or utilize analyzers to check the allocations of your program while it's running.

## 27. Don't Outrun Your Headlights

**Tip 42:** Take Small Steps -- Always

Don't outrun your headlights in the sense of getting ahead of yourself while coding. Make small incremental changes and wait for feedback either from customers, unit tests, or other sources. You never know how things will change, so don't get too far ahead before you know what those changes could be. Don't plant *too* far ahead eitehr. Keep your code DRY and ETC.

### Black Swans

**Tip 43:** Avoid Fortune Telling

You can't predict the future; things out of your control may change significantly someday. The concept of a "black swan" means that the significant events in history are not the expected events; they're the unusual and unexpected.

## Research: Assertions in C\#

C# does have the ability to utilize assertions in ways like `Debug.Assert` and `Trace.Assert`.

See [here](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.debug.assert?view=netframework-4.8) for the official documentation on using `Debug.Assert`.

## Research: Contracts in C\#

[Code Contracts in .NET](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/code-contracts)

Code contracts allow you to specify preconditions, postconditions, and object invariants in code.

* Preconditions - requirements that must be met when entering a method or property
* Postconditions - expectations at the time the moethod or property code exits
* Object invariants - expected state for a class that is in a good state

Code contracts have classes for marking code, a static analyzer for compile-time analysis, and a runtime analyzer.

All .NET Framework languages can utilize code conditions without any additional installations. You can also download [Code Contracts for .NET Extension for VS](https://marketplace.visualstudio.com/items?itemName=RiSEResearchinSoftwareEngineering.CodeContractsforNET) to specify the level of anaylsis to perform. Analyzers confirm that contracts are well formed.

Most contract methods are conditionally compiled. They are only compiled when you specify the `CONTRACTS_FULL` symbol using the [`#define` directive](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-define). 

### Preconditions

Utilize the `Contract.Requires` method and specify the state when a method is invoked. Generally specify valid parameter rules. *Be sure that your preconditions have no side effects*; they should read, not write. You can throw a particular type of exception by specifying the generics for the `Requires` call like `Contract.Requires<ArgumentNullException>`.

#### Legacy Requires Statements

The contract tool can recognize bits of parameter checking code if they meet one of the following scenarios:

* Statement appears first in a method (before any others)
* The statement is followed by a `Contract` method call

```c#
if (x == null) throw new SomethingException();
Contract.EndContractBlock(); // All previous "if" checks are preconditions
```

### Postconditions

Determine the state of a method upon termination and is checked just before exit. Unlike preconditions, post conditions can check private or internal properties and variables.

#### Standard Postconditions

`Ensures` method expresses a condition that must be true upon normal termination of the method.

```c#
Contract.Ensures(this.Price > 0);
```

#### Exceptional Postconditions

Conditions that should be `true` when an exception of type `T` is thrown by a method.

```c#
Contract.EnsuresOnThrow<T>(this.Price > 0);
```

You should utilize exceptional postconditions with specific exceptions that may be called when a member is called, not with other "impossible-to-control" exceptions like stack overflow or `Exception`.

#### Special Postconditions

The following methods may be used only within postconditions:

* You can check the values in postconditions with `Contract.Result<T>()` where `T` is the return type of the method, but you must provide the type if the compiler is unable to infer it. `void` methods cannot utilize `Contract.Result<T>()` in their postconditions.
* You can retrieve the original values of a condition using `Contract.OldValue<T>(e)` where `T` is the type of `e`; `T` can be omitted if the type can be inferred.
  * An old expression cannot contain another old expression
  * Must refer to a vlue that existed in the method's precondition
  * Must be an expression that can be evaluated as long as the method's precondition is true
  * To reference a field on an object, the preconditions must guarantee that the object is always non-null
  * You cannot refer to the metho'd return value (`Contract.OldValue(Contract.Result<int>() + x) //Invalid`)
  * Cannot refer to `out` parameters
  * Cannot depend on the bound variable of a quantified if the range depends on the return value of the method
  * Cannot refer to the parameter of the `ForAll` or `Exists` call unless it is used as an indexer or argument to a method call
  * Cannot occur in anonymous method if the value depends on any parameters of the anonymous delegate (unless it is an argument for the `ForAll` or `Exists` methods)

``` c#
  Contract.ForAll(0, xs.Length, i => Contract.OldValue(xs[i]) > 3); // OK
  Contract.ForAll(0, xs.Length, i => Contract.OldValue(i) > 3); // ERROR
```

* You can check the status of `out` parameters using the `Contract.ValueAtReturn` method, which may appear only in postconditions

``` c#
public void OutParam(out int x)
{
    Contract.Ensures(Contract.ValueAtReturn(out x) == 3);
    x = 3;
}
```

### Invariants

> Object invariants are conditions that should be true for each instance of a class whenever that object is visible to a client. They express the conditions under which the object is considered to be correct.

In order to define the invariants, you must create a method and decorate it with the `ContractInvariantMethodAttribute`, which contains a sequence of calls to the `Contract.Invariant` method.

``` c#
[ContractInvariantMethod]
protected void ObjectInvariant ()
{
    Contract.Invariant(this.y >= 0);
    Contract.Invariant(this.x > this.y);
    ...
}
```

Invariants are defined by the `CONCTRACTS_FULL` preprocessor symbol. They are checked at the end of each public method. If public methods are nested, then only the outermost invariants are checked. 

### User Guidelines

#### Contract Ordering

The following table shows the order of elements you should use when you write method contracts.

|1. | If-then-throw statements | Backward-compatible public preconditions |
|2. | Requires | All public preconditions. |
|3. | Ensures | All public (normal) postconditions. |
|4. | EnsuresOnThrow | All public exceptional postconditions. |
|5. | Ensures | All private/internal (normal) postconditions. |
|6. | EnsuresOnThrow | All private/internal exceptional postconditions. |
|7. | EndContractBlock | If using if-then-throw style preconditions without any other contracts, place a call to EndContractBlock to indicate that all previous if checks are preconditions. |

#### Purity

Methods within a contract must not modify any preexisting state - they must be *pure*. You can find a list of the code elements that are considered "pure" in the [original documentation](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/code-contracts#purity). You can also read about the [PureAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.contracts.pureattribute).

#### Visibility

Members mentioned in a contract must be at least as visible as the method they are in. A contract in a public method cannot require something from a private field. This would put an impossible burdon on the caller. You can except a property from these rules by using the [ContractPublicPropertyNameAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.contracts.contractpublicpropertynameattribute)

## Meeting Notes

Many were confused about preconditions and postconditions and who is "responsible" for it. We've determined that in the languages they specify, the pre- and postconditions are not part of the usual method; they are considered separate and independant. You should be checking these *before* the method is actually called; the closest relative in C# seems to be attributes?

Lots of discussion about whether or not to let the application just crash. As web developers, you would never want the user to see the actual error message itself; you almost always catch exceptions and return an error.

Always fall back on allowing your code to be easily replacable. Lyn and Dan especially liked this. As times move ahead, it's best to make things swappable.
