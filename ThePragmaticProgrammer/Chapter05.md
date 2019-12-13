# Bend, Or Break

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

