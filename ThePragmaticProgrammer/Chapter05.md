# Bend, Or Break

- [Bend, Or Break](#bend-or-break)
  - [28. Decoupling](#28-decoupling)
    - [Law of Demeter](#law-of-demeter)
      - [Chains and Pipelines](#chains-and-pipelines)
    - [The Evils of Globalization](#the-evils-of-globalization)
      - [Global Data Includes Singletons](#global-data-includes-singletons)
      - [Global Data Includes External Resources](#global-data-includes-external-resources)
    - [Inheritance Adds Coupling](#inheritance-adds-coupling)
    - [Again, It's All About Change](#again-its-all-about-change)
  - [29. Juggling the Real World](#29-juggling-the-real-world)
    - [Events](#events)
    - [Finite State Machines](#finite-state-machines)
      - [The Anatomy of a Pragmatic FSM (Finite State Machine)](#the-anatomy-of-a-pragmatic-fsm-finite-state-machine)
      - [Adding Actions](#adding-actions)
      - [State Machines Are a Start](#state-machines-are-a-start)
    - [The Observer Pattern](#the-observer-pattern)
    - [Publish/Subscribe](#publishsubscribe)
    - [Reactive Programming, Streams, and Events](#reactive-programming-streams-and-events)
      - [Streams of Events Are Asynchronous Collections](#streams-of-events-are-asynchronous-collections)
    - [Events are Ubiquitous](#events-are-ubiquitous)
  - [20. Transforming Programming](#20-transforming-programming)
    - [Finding Transformations](#finding-transformations)
      - [Transformations All the Way Down](#transformations-all-the-way-down)
      - [What's with the |&gt; Operator](#whats-with-the-gt-operator)
      - [Keep on Transforming...](#keep-on-transforming)
      - [Putting It All Together](#putting-it-all-together)
    - [Why Is This So Great?](#why-is-this-so-great)
    - [What About Error Handling?](#what-about-error-handling)
      - [First, CHoose a Representation](#first-choose-a-representation)
      - [Then Handle It Inside Each Transformation](#then-handle-it-inside-each-transformation)
      - [Or Handle It in the Pipeline](#or-handle-it-in-the-pipeline)
    - [Transformation Transform Programming](#transformation-transform-programming)
  - [31. Inheritance Tax](#31-inheritance-tax)
    - [Some Background](#some-background)
    - [Problems Using Inheritance to Share Code](#problems-using-inheritance-to-share-code)
      - [Problems Using Inheritance to Build Types](#problems-using-inheritance-to-build-types)
    - [The Alternatives Are Better](#the-alternatives-are-better)
      - [Interfaces and Protocols](#interfaces-and-protocols)
      - [Delegation](#delegation)
      - [Mixins, Traits, Categories, Protocol Extensions, ...](#mixins-traits-categories-protocol-extensions-)
    - [Inheritance is Rarely the Answer](#inheritance-is-rarely-the-answer)
  - [32. COnfiguration](#32-configuration)
    - [Static Configuration](#static-configuration)
    - [Configuration-As-A-Service](#configuration-as-a-service)
    - [Don't Write Dodo-Code](#dont-write-dodo-code)
  - [Meeting Notes](#meeting-notes)

## 28. Decoupling

**Tip 44:** Decoupled Code Is Easier to Change

Like they've said in earlier chapters, code should be *easier to change*. Decoupled code is in fact easier to change because less code has to change in order to make any modification. Tightly coupled code will require multiple changes in multiple places.

In a bit of example code, they have a customer object, which has a list of orders, which have lists of totals. In a function they calculate the order total by selecting all the totals from all the orders and adding them up. They suggest this is an example of a "train wreck," since the function now needs knowledge of customer, orders, and totals.

``` c#
// converted from their language into c#
public void ApplyDiscount(Customer customer, int orderId, double discount) 
{
    Totals totals = customer.orders.FirstOrDefault(x => x.OrderId == orderId).GetTotals();

    totals.GrandTotal = totals.GrandTotal - discount;
    totals.Discount = discount;
}
```

**Tip 45:** Tell, Don't Ask

The first issue is that Totals isn't responsible for the totals. It is simply a container for the fields other functions like the example can query and update. So the first fix is to delegate the discounting to the total object:

``` c#
public void ApplyDiscount(Customer customer, int orderId, double discount) 
{
    Totals totals = customer
        .orders
        .FirstOrDefault(x => x.OrderId == orderId)
        .GetTotals()
        .ApplyDiscount(discount);
}
```

Next, they say the same issue is found with the customer and orders. The functions shouldn't have to search through the orders themselves:

``` c#
public void ApplyDiscount(Customer customer, int orderId, double discount) 
{
    Totals totals = customer
        .FindOrder(orderId)
        .GetTotals()
        .ApplyDiscount(discount);
}
```

They'd go one final step and simplify the retrieval of the totals for the order:

``` c#
public void ApplyDiscount(Customer customer, int orderId, double discount) 
{
    Totals totals = customer
        .FindOrder(orderId)
        .ApplyDiscount(discount);
}
```

They stop there even though you could add one more layer and give the `Customer` object an `ApplyDiscount(double)` function as well. But they argue that since `Orders` have their own meaning when they are not associated with a customer - you could start with orders and reason about to get a customer - they are candidates for exposure to other functions.

### Law of Demeter

Often talked about in relation to coupling, it is a set of guidelines to help developers on the Demeter Project keep their code cleaner and decoupled. 

A method in a class `C` should only call:

- Other instance methods in `C`
- Its parameters
- Methods in objects that it creates
  - On the stack and the heap
- Global variables

They have fallen out of favor with this particular ruleset, particularly the final rule about global variables. Instead they boil it down to a simpler mantra

**Tip 46:** Don't Chain Method Calls

Try not to have more than one "." when accessing something. The exception occurs with objects that are really unlikely to change. This does not include code we write or 3rd party applications; only libraries that come with the language can be considered safe.

It would seem that that most LINQ queries are considered acceptable, since the core logic is unlikely to change (being a core language feature).

#### Chains and Pipelines

They clarify that they are not referring to pipelines in this chapter, because pipelines to not rely on hidden implementation details. They do concede that pipelines add a level of coupling, but that it is negligible and less of a barrier to change than train wrecks.

### The Evils of Globalization

Global data causes fairly tight coupling. A change to the global variable requires a change to all functions that utilize it, which is potentially all the code in the system.

**Tip 47:** Avoid Global Data

#### Global Data Includes Singletons

If you are going to use global data, put it in a singleton method and cover everything in accessor methods, that way the "global variables" have some sort of intelligence to them and are more malleable to change.

#### Global Data Includes External Resources

Any mutable external resource is global data. Make sure you wrap any resources (database, datastore, file system, service API) in methods that you control.

**Tip 48:** If It's Important Enough to Be Global, Wrap It in an API

### Inheritance Adds Coupling

They discuss this more in Topic 31. But classes that inherit state and behavior from another class is indeed a form of coupling.

### Again, It's All About Change

Decoupled code is easier to change. Coupled code could have ramifications when changed that are not apparent for months later.

Keep code "shy" so it only deals with things it directly knows about.

## 29. Juggling the Real World

> This section is all about writing... responsive applications.

### Events

Event driven programming has four main strategies to work with it

1. Finite State Machines
2. The Observer Pattern
3. Publish/Subscribe
4. Reactive Programming and Streams

### Finite State Machines

#### The Anatomy of a Pragmatic FSM (Finite State Machine)

A state machine has a set of states, including the *current state*. For each state, there are events associated with it. For each event, a new state is set to the current.

#### Adding Actions

A pure finite state machine has only one output, the final state. More actions can be added that are triggered on certain transitions.

They give an example of a string reader. The final output is a string, but with each character passed, they add to that string using an action. For instance, the first quote (") is sent, so the `result` is initialized as an empty string. Then an "h" is passed, so `result += 'h'`. Next an "i" is passed; `result += 'i'`. Finally the second quote (") is passed, and the `result` is returned.

#### State Machines Are a Start

They feel that state machines are underused. However, they don't solve all the problems. That's really all this section says...

### The Observer Pattern

- *Observable* - the source of events
- *Observers* - the list of clients interested in those events

The observable stores a list of clients and function references. When an event is processed, the observable loops through its list of clients and calls the function it was provided with the event as the parameter.

This is a relatively simple method to implement, but it has two major drawbacks:

- It introduces coupling
- The callbacks introduce a bottleneck

These issues are solved by the next method.

### Publish/Subscribe

- *Publishers*
- *Subscribers*

Publishers and subscribers are connected via channels, which are implemented somewhere else hidden from your code. Channels have a name, and a subscriber may register with one or more channels. Publishers write events to channels.

Unlike the observer pattern, the communication is handled outside of your code and can potentially be handled asynchronously. You can often find cloud services to host your channels, so it is not in your best interest to host them yourself; let the specialists specialize.

The main downside is that it is not always clear which subscribers are listening to which channel and which publishers are publishing to which channels.

### Reactive Programming, Streams, and Events

Spreadsheets are an example of reactive programming. A change in one cell may cause others to update if those cells reference the updated cell. *React* and Vue.js are also considered reactive programming frameworks.

<http://reactivex.io>

Streams can be though of as lists of events that get longer as new events arrive. This allows sorting, filtering, joining, and asynchronous methodology.

The authors then geek out about streams being "zipped" together.

#### Streams of Events Are Asynchronous Collections

In the scenario where User information is retrieved from an API, they utilize a static list of User IDs. But they note that this doesn't have to be a static list, and that instead they could create a new observable each time a user logs in to the site.

### Events are Ubiquitous

That's pretty much it. They're just very common, and you might not notice it at first.

## 20. Transforming Programming

Programs transform inputs into outputs, but design talks apparently only ever revolve around classes, modules, data structures, algorithms, languages, frameworks etc. They feel that this is not good, that focusing on the I/O of an application instead can lead to clearer code.

**Tip 49:** Programming Is About code, But Programs Are About Data

### Finding Transformations

The "top-down" approach involves taking a requirement and determining its inputs and outputs. 

#### Transformations All the Way Down

Pretty much they're building up to functional languages here using an example with dictionaries and the Elixir functional language.

#### What's with the |> Operator

It's a pipe. Every time you use a pipe should be an opportunity to see data moving between two functions

Apparently there's talk of the pipe coming to JavaScript to really seal in the fact that it's a functional language at this point.

#### Keep on Transforming...

This section just moves to the second step of their example application, still using Elixir.

#### Putting It All Together

Just summarizes the string of functions from the examples previously.

### Why Is This So Great?

**Tip 50:** Don't Hoard State; Pass It Around

By utilizing the functional approach instead of the OO approach, we can get a better degree of decoupling. The data isn't spread everywhere anymore, instead it is in a flow. Here is where they really lay into FP being superior to OO.

I'm finding myself somewhat happy that Lyn and I are reading through Concurrency in .NET at the same time, since it's tackling a lot of the same concepts.

### What About Error Handling?

Instead of passing the raw data, you can wrap the data in a data structure (or type) that can determine whether the data is still valid. **In F# this is the `Option`.**. In general, you can either check for errors inside transformations, or outside them.

#### First, CHoose a Representation

You must create a wrapper that will be used to transfer the data. In the example of Elixir, there is a convention for functions to return a tuple containing OK and the value, or an error and the reason.

#### Then Handle It Inside Each Transformation

Each transformation in their example does a bit of overload matching. In one case, the tuple passed starts with an `:ok` parameter, and the function proceeds as normal. In another case, the tuple begins with `:error` and that message is simply passed through the pipeline in this manner. This puts the burden of error checking on the transformations themselves.

#### Or Handle It in the Pipeline

An issue occurs with the previous error handling implementation in that all the functions are called, and the burden of error checking is put on them. You can move this burden into the pipe by changing the transformation from function *calls* into function *values* that can be called later.

A function that takes a value wrapped in something and applies a function to that value and returns the newly wrapped values is a *bind* function.

### Transformation Transform Programming

Once you develop the habit of thinking of your code as a series of transformation, things become cleaner, designs become flatter, and functions become shorter.

## 31. Inheritance Tax

They start this chapter off by saying that you shouldn't use inheritance in OO. Let's see why

### Some Background

There are two rough types of inheritance

1. A way of combining types (C++, Java)
2. Organization of behaviors (Ruby, JavaScript)

They then group OO programmers into two categories

1. Don't like typing - use inheritance to add common functionality to a base class
   - Car and Motorcycle both implement the same `.Model()` method
2. Don't like types - use inheritance to express relationships between classes
   - A car is a vehicle

Both kinds apparently have problems.

### Problems Using Inheritance to Share Code

*Inheritance is coupling*. A child class is coupled to it's parent class and all the parents of that parent class. A minor change to a parent class could have a cascade of effects and break a multitude of other classes.

#### Problems Using Inheritance to Build Types

Writing classes as a series of relationships can cause even more issues, especially considering multiple inheritance. A car can be a vehicle, an asset, an insured item, etc. Not all languages provide this type of inheritance, so it's probably best to avoid getting into the habit.

**Tip 51:** Don't Pay Inheritance Tax

### The Alternatives Are Better

They suggest three techniques to avoid using inheritance

- Interfaces and Protocols
- Delegation
- Mixins and traits

#### Interfaces and Protocols

Here they mention interfaces as organizational units that create no code, only determine what an inheriting class *must* implement.

Any object that implements a particular interface can be reasoned with as an instance of that interface. A car can be converted into an `IVehicle` and you could stash a list of cars and trucks inside a `List<Vehicle>` and perform operations on the vehicles using the common interface implementations.

**Tip 52:** Prefer Interfaces to Express Polymorphism

#### Delegation

One problem with interfaces are unused methods. You may not need all of the methods that a particular interface requires; this leads to unused code.

**Tip 53:** Delegate to Services: Has-A Trumps Is-A

Whenever possible, move logic out of a class and into a service that is tailor-made to perform that operation. Instead of every class having an implementation of `Save()`, have them all call the same `Repo.Save()` method instead.

#### Mixins, Traits, Categories, Protocol Extensions, ...

All of the names in the title are the same basic idea. We want to extend classes and objects with new functionality without using inheritance.

Create a set of functions, give them a name, and extend a class or object with them and you have a mixin.

**Tip 53:** Use Mixins to Share Functionality

Am I crazy, or is this more coupled code that they were just complaining about? Having classes inherit from a utility class would have the same effect, would it not? This may be something I want to bring up in discussion, because to me it seems they ended on Mixins because they feel it is the solution to all our woes, but somehow I get the sense we're back at square one.

### Inheritance is Rarely the Answer

Use one of the methods listed in the last sections instead.

## 32. COnfiguration

When a value may change after an application goes live, keep that value external. Parameterize your function, that way it can easily adapt how it needs without rewriting.

**Tip 55:** Parameterize Your App Using External Configuration

Anything you know will have to change that you can express outside your code should be moved into configuration.

### Static Configuration

Generally, configuration values are stored either in flat files like JSON or YAML or in a database. However, the book writers would prefer this information be stored in a Service API instead.

### Configuration-As-A-Service

Storing configurations in a service has multiple benefits:

- Sharing of the configuration between multiple applications
  - Can limit what each application is able to see
- Changes can be made globally
- You can utilize a UI to maintain the configuration
- the data becomes dynamic

They take special note of that last point. The ability to update the configuration of a service without having to restart that service is highly desirable, and the service API approach will better allow that to happen.

### Don't Write Dodo-Code

That's dodo as in the bird, not the poop. Code should be adaptable, because if it can't adapt it won't make it.

## Meeting Notes

With chaining method calls, it's OK to chain if the methods are unlikely to change; this is why chaining LINQ statements are considered OK. The logic inside may change but the `.Where` call will remain consistent.
