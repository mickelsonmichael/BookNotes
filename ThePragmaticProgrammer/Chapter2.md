# Chapter 2: A Pragmatic Approach

## 8. The Essence of Good Design

:::info
**Tip 14 |** Good Design is Easier to Change Than Bad Design
:::

The ETC Principle means *Easier to Change*. Decoupling is good because it is easier to change. The single responsibility principle is important because things are easier to change. Naming is important because it makes code more readable and thus easier to change.

Follow ETC as a general value but not a hard and fast rule. When given a choice of how to code something, think about which option will make it easier to change in the future and take note so you can reflect on this later when you do need to change it.

## 9. DRY - The Evils of Duplication

Maintenance begins the second you start writing an application. Business rules change, requirements change, algorithms don't work; there is inherit instability in the code that you cannot always control.

:::info
**Tip 15 |** DRY - Don't Repeat Yourself
:::

> Every piece of knowledge must have a single, unambiguous, authoriatative representation within a system.

If you express something in two places and a change needs to be made, you must make that change in *both* places. It's not about whether you'll remember, it's about when you'll forget.

::: warning
First Edition Note: The first edition doesn't put enough emphasis on the fact that DRY expands beyond code; it is the duplication of *knowledge* and *intent*.
:::

Not all code duplication is knowledge duplication. While code may look identical, the naming scheme behind around it may provide a different set of information. A function `ValidateAge(int n)` may be structurally identical to `ValidateQuantity(int n)`, but are fundamentally different from one another. If rules behind verification of age were to change, you can modify the `ValidateAge` function without impacting the `ValidateQuantity` method.

Comments can lie. By writing the logic in two places, you work against the DRY principle and you must now update information in two places. Inevitably the comments will be overlooked and become out of sync with the code. Comments should enhance the code, not repeat it.

Data structures can also be non-DRY as well. If a property on a class can be a calculation of two other property on that class, then it is a repetition of knowledge to store it again; instead that property should be a calculation of the other two. However, it may be better for performance to cache these values in a property. If that's the case ensure that the external information outside of the class does not reflect that; use getters and setters to mask the calculations.

### Representational Duplication

APIs are a large source of duplication. When you use external sources you almost always introduce a DRY violation. Utilize specifications like the [OpenAPI](https://github.com/OAI/OpenAPI-Specification) to import the specifications into your local tools and improve the reliability of your integration. The authors recommend that instead of writing static structures for APIs (classes), instead use a series of key-value pairs.

::: spoiler
I'm not sure why they would recommend this. If you're expecting data from the API and it isn't there it's going to error out either way. Why is this error preferable to the other?
:::

### Interdeveloper Duplication

One  of the hardest forms of duplication to prevent is duplication by multiple developers. Multiple devs may create the same functionality multiple times without realizing it, and this problem may remain unnoticed for the life of the application, until a change is required.

The only way to mitigate this issue is by having good team communication with a local source of information. The authors recommend having "project librarian" who can facilitate the exchange of knowledge, or having a centralized repository where utilities can be placed.

:::info
**Tip 16 |** Make It Easy to Reuse
:::

If a library or utility you write isn't easy to use, people won't use it.

## 10. Orthogonality

> Two lines are *orthogonal* if they meet at right angles, such as axes on a graph. In vector terms, the two lines are *independent*.

As you move up on a graph, only the `y` value changes and not the `x`, meaning that they are independent. In computing two things are orthogonal if a change in one does not affect the other (e.g. database code changes should not affect the UI).

By using orthogonality, you are able to eliminate the effects between unrelated things. A change in one area of your program will not affect another.

::: info
**Tip 17 |** Eliminate Effects Between Unrelated Things
:::

* It is easier to write smaller bits of code than one large block
* It is easier to test orthogonal code
* It is easier to reuse orthogonal code
* There is less overlap in the functionality of two components, making them more efficient
* Issues in one orthogonal module do not affect others
* Small changes in one area do not affect others (less fragile)
* You will be less tied to a particular vendor or 3rd party implementation

There is an easy test for orthogonal design:
*If I dramatically change the requirements behind a particular function, how many modules are affected?*

> Do not rely on the properties of things you can't control

Postal codes, SSNs, email addresses, domains, and government IDs are all external identifiers that you have no control over and could change at any time. If they did change, how easily would your system adapt?

If you are bringing in a third party tool or library, consider how it will affect the orthogonality of your system.

1. Keep your code decoupled
    * Don't reveal anything unnecessary to other modules
    * If you need to change an object's state, let the object do it for you
2. Avoid global data
    * Global data binds things too close together
    * Explicitly pass data into a module instead (parameters)
    * Be careful with singletons
3. Avoid similar functions

Orthogonality also applies to documentation. You should be able to change the appearance of something without modifying the content at all. Utilize style sheets, macros, or even Markdown languages.

Orthogonality supports the concept of ETC. Orthogonal components are much easier to change.

## 11. Reversibility

::: info 
**Tip 18 |** There Are No Final Decisions
:::

Any choices you make should be easily reversed at a later date. If you need to change databases that should be doable. If you need to change interfaces that should be doable. There should be enough orthogonality that you can swap components without much issue.

Fads come and go. Make your architecture able to adapt to the latest fads without much issue. Break code down into smaller, swappable components.

::: info
**Tip 19 |** Forgo Following Fads
:::

> No one knows what the future may hold, especially not us! So enable your code to rock-n-roll: to "rock on" when it can, to roll with the punches when it must.

Yes they really wrote that.

## 12. Tracer Bullets

"Tracer bullet development" allows for immediate feedback under actual conditions with a moving goal. 

::: info
**Tip 20 |** Use Tracer Bullets to Find the Target
:::

You can build out the architecture of a system before building all the functionality to ensure that the core concepts work together first. Build a skeleton app and then future development will instead be extending that application with more functionality. 

Unlike prototypes, tracer code is written to be used. It should contain all the fail safes and testing that normal code contains, it just isn't yet fully functional.

This mode of development supports the mindset that "development is never finished."

1. Users get to see something working early
    * You can warn them that they are seeing something immature
    * They won't be disappointed by lack of functionality
    * They will be able to give recommendations and guidance earlier on
2. Developers build a structure to work in
    * It's easier to extend something than to create it from scratch
3. You have an integration platform
    * The system is connected end to end
    * You can implement the unit tests for your code before hooking it into the actual system
4. You have something to demonstrate
    * You will always have something to show the product sponsors
5. You have a better feel for progress
    * It is easier to measure performance
    * It is easier to demonstrate progress to your user

Prototyping is creating test modules with no intention of utilizing the actual code. Creating something that is meant to be disposed of and replaced with a real implementation. It is often not written in a target code or framework but instead in a code that is faster to write and get off the ground with minimal effort.

## 13. Prototypes and Post-it Notes

::: info
**Tip 21 |** Prototype to Learn
:::

Prototype anything that carries risk or hasn't been tried before, or things that are critical to the final system. Anything unproven, experimental, new, doubtful, dubious.

* Architecture
* New functionality in an existing system
* Structure or contents of external data
* Third-party tools or components
* Performance issues
* User interface design

When using prototypes, **ignore** the following
1. Correctness
2. Completeness
3. Robustness
4. Style


Instead focus on the core concepts and on making sure the product actually works.

When writing a prototype, be sure that everyone knows you are writing *disposable* code. They may give people a false sense of progress; be completely clear that the code you are creating is not permanent and will be replaced.

## 14. Domain Languages

::: info
**Tip 22 |** Program Close to the Problem Domain
:::

Create a language that resembles the end goal you want to create. Use language in the domain that you are interested in. 

Codes like RSpec or *internal* domain languages. They are written in the language of interest and do not need to be compiled by the language to function. They can take advantage of the features of their languages. This allows the use of automation as well, like creating loops for running processes quickly. However, you are bound by the syntax of that language.

Cucumber and Ansible use *external* languages; written in a different language than their destination languages. They are not bound by the limitations of the language(s) they work with. As long as you write a parser for the language you can use your external language.

## 15. Estimating

::: info
**Tip 23 |** Estimate to Avoid Surprises
:::

All answers are estimations, just some are more accurate than others. Context matters greatly when making an estimation.

How you deliver the estimate also affects how people interpret it. You can use the table below to determine how it may be best to phrase a particular estimate before giving it.

| Duration | Quote the estimate in... |
| -------- | -------- |
| 1-15 days     | Days     |
| 3-6 Weeks | Weeks|
| 8-20 Weeks | Months |
| 20+ weeks | Think hard before giving an estimate |

Before giving an estimate make sure you understand what is being asked. Create a model of a system and break that model into components. Try the model out with some values. Calculate the answers from those values. 

Keep track of how well you estimate. If you are consistently giving over or under-estimations, then you may need to increase or decrease your usual guesses.

Give a number of estimations based on a worst, best, and median case scenario; the *Program Evaluation Review Technique (PERT)*.

> Every PERT task has an *optimistic*, a *most likely*, and a *pessimistic estimate*. 

However, the only really accurate way to estimate is to get started with the project. Gain experience with the project first and use incremental development.

Repeat the following steps with slices of functionality
1. Check requirements
2. Analyze risk (and prioritize riskiest items earlier)
3. Design, implement, integrate
4. Validate with the users

Don't restrict the number of iterations you expect to go through. You never know how many it may take.

>That's also how the old joke says to eat an elephant: one bite at a time

::: info
**Tip 23 |** Iterate the Schedule with the Code
:::
