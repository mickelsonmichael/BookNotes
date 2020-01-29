# Chapter 7: While You Are Coding

**DISCLAIMER** - I have now switched to an audiobook version, so these notes will be significantly more sparse than they previously were. However, due to the conversational nature of the book, I don't necessarily see this as a bad thing. I will now do summaries instead, and write any pertinent notes on the topic.

## 37. Listen to Your Lizard Brain

This chapter essentially tells how you should listen to your instincts. It does this in a really long-form fashion. It then transitions into a discussion about the advantages of prototyping.

Prototyping should be done in a disposable manner. There should be no fear that your code will be used so that you can code as organically as possible without worrying about exceptions or clean code. Once you prove or disprove your concept, you should scap the prototype and start over clean.

**Tip 61:** Listen to Your Inner Lizard

## 38. Programming by Coincidence

This section discusses how you should know why your code works. Don't just accept that "it suddenly works" and move on; you should understand the code you write. You should also meter your assumptions, and always assume the worst. Don't take faith in patterns. Only rely on reliable things.

**Tip 62:** Don't Program by Coincidence

## 39. Algorithm Speed

The Big-O Notation is the focus of this section, and it reviews how to estimate the Big-O and common scenarios that could result in the most common notation forms.

**Tip 63:** Estimate the Order of Your Algorithms

**Tip 64:** Test Your Estimates

## 40. Refactoring

The book references Martin Fowler, which is a common name in programming. I'm not sure what else he's done, but the book references his book about Refactoring, which may be a good potential next read for me.

They also discuss the programming metaphor, and at one point compare it to gardening. Which is fine, except they also claim that construction is more of a science than gardening... which is just wrong.

**Tip 65:** Refactor Early, Refactor Often

## 41. Test to Code

**Tip 66:** Testing Is Not About Finding Bugs

When Unit Testing, the main goal shouldn't be to find bugs, it should be to improve your code. The very nature of Unit Test writing helps you find issues with your code; dependencies or ETC issues among others.

Briefly they discuss TDD and don't exactly give it a glowing review. However, they at no point mention their personal experience with the mindset, which feels like they aren't exactly experts on the topic.

**Tip 67:** A Test Is the First User of Your Code

**Tip 68:** Build End-to-End, Not Top-Down or Bottom Up

**Tip 69:** Design to Test

Essentially this section discusses how the main benefit of testing isn't that it helps you find bugs, but instead that it helps you get into an idealized mindset. You are thinking about how to write your code for your tests and thus thinking about all the good principles a programmer should follow.

There is also a brief mention of log files, which is something we unfortunately don't currently do at LMCU. It would be very nice to get consistent logs outside of just exceptions, but I don't forsee it happening any time soon.

**Tip 70:** Test Your SOftware, or Your Users Will

## 42. Property-Based Testing

**Tip 71:** User Property-Based Tests to Validate Your Assumptions

From what I can tell, the concept of Property-Based testing is already baked-in to all three of the largest Unit Testing frameworks available to .NET. It is the usage of `DataRow` or other attributes that allow you to insert data into your tests as a parameter. This allows you to run a single test a multitude of times without issue.

## 43. Stay Safe Out There

The section is all about safety, including encapsulation, locking down privileges and access, securing default values, encryption, and updating.

One particularly interesting concept they mentioned was granting permissions on a case-by-case basis instead of role-based. This allows for more compartmentalized access to a program, and for each user to have an experienced that is tailored to their requirements and their privileges.

**Tip 72:** Keep It Simple and Minimize Attack Surfaces

**Tip 73:** Apply Security Patches Quickly

There is an additional section on *Password Antipatterns* that discusses how passwords requirements should/could be set up. It speaks at length about requirements like hints and character types. It's definitely something that I would look more to an official source to get my rules from instead of this book, but they make some good points.

## 44. Naming Things

This one was a long-winded explanation (like most sections) of how naming should be done and how it *shouldn't* be done. Name things descriptively and thoroughly.

They briefly talk about how you should follow the semantics of the language when naming variables, and that in C, string are often designated `s` and that you should follow that semantic. However, I disagree with that. While `s` does inform the reader that it contains a string, what string does it contain? The use of `i` and `j` to mean indexes are much more obvious, seeing as the loop is dependent on `i` and `i` should almost always mean the index of the outer loop. You don't have to think about what the value of `i` is in this case: it is the current index. But `s` is much more variable, you could store any number of things in `s`. At that point you'll just read the code to figure out what `s` should be, which defeats the purpose of this chapter. If that were acceptable then you could do that for *any* variable in *any* language. Just read the context, so easy right?

**Tip 74:** Name Well; Rename When Needed
