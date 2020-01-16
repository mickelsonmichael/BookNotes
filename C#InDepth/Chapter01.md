# C\# in Depth - Chapter 1

## 1.1 An evolving language

C# takes a lot of things from other languages; good features that could improve the language. There has been a large development in the type system, starting form static types and growing eventually to dynamic types and generics.

Another huge development was the release of LINQ, which can take advantage of anonymous functions, implicit typing, and anonymous typing to make very clear, concise functions.

Most of the functions introduced with each C# version have been focused on making code more concise and more clear. Things that can be inferred from the code around them don't need to be written which lead to features like implicit typing and ignoring parameters. Other features are meant to clean code up like the object and collection initializers or the automatically implemented properties.

Since C# 5, developers have been able to take advantage of _async/await_, which has made asynchronous programming much easier to read and much easier to write. The author will go into more detail about the mechanisms behind this in later chapters.

Of course, not all updates are focused only on cleaner code. C# 7 especially had a large focus on performance improvements that developers can take advantage of. However, it's important to remember that just because a feature exists, doesn't mean you have to use it. If the feature gives little benefit and only serves to make your code more confusing, then it's best to skip it.

With the release of C# 7, the C# team has started using minor versioning again, which they haven't done since C# 1. You must configure whether or not your application will use these latest versions, but the ability to do so allows the language to grow more incrementally without disturbing developers who don't necessarily need the functionality.

## 1.2 An evolving platform

.NET has become open source and, with the release of .NET Core, more focused on being cross-platform. With the rise of microservices on Linux boxes, it became more important for .NET to be less proprietary, or else risk falling behind the curve in usage.

## 1.3 An evolving community

The C# environment is moving into the realm of open source. The Framework is open source not, Entity Framework Core is open source, ASP.NET Core is open source.

## 1.4 An evolving book

The book covers a lot of version of C#, and he will go over each of those version in turn. This is a more compressed version than the third edition, which had 400 pages on C# 2-4.

A lot of the example in the book will be in Noda Time, which is a library he wrote to improve on the the Date API. He uses these examples just because he knows the inner workings and we can the actual code on GitHub.
