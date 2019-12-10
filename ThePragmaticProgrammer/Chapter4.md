# Chapter 4. Pragmatic Paranoia

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

