# Chapter 1. Functional Concurrency Foundations

This chapter covers

* Why you need concurrency
* Differences between concurrency, parallelism, and multithreading
* Avoiding common pitfalls
* Sharing variables between threads
* Using the functional paradigm

For a single CPU core, the limiting factor is not the speed of processing, but the heat.

A shorter circuit means a faster CPU; smaller is faster. We've nearly reached the peak of how small we can makes CPUs because we've focused on making them smaller for so long. Increasing the frequency of the signal exponentially increases the heat generated, until we reach the point where it causes damage such as changes to the crystal structure.

To continue the increase in speed, we've begun using multiple processors working together. This requires the programmer to utilize the processors to their fullest. 

## 1.1. What you will learn from this book

* How to combine asynchronous operations with the Task Parallel Library
* Avoid common problems and troubleshoot multithreaded and asynchronous programs
* Concurrent programming models to adopt functional paradigms
  * Functional
  * Asynchronous
  * Event-Driven
  * Message passing with agents and actors
* Building concurrent systems using the functional paradigm
* Write asynchronous computations in a declarative style
* Accelerate sequential programs by using data-parallel programming
* Implement reactive and event-based programs declaratively with Rx-style event streams
* Use functional concurrent collections for building lock-free, multi-threaded programs
* Write server-side applications
* Solve proglems using patterns like For/Join, parallel aggregation, and Divide and Conquer
* Process data sets with parallel streams and parallel Map/Reduce implementations

## 1.2. Let's start with terminology

Concurrency is multithreading, but multithreading isn't necessarily concurrent. 

### 1.2.1. Sequential programming performs one task at a time

*Sequential programming* is the act of accomplishing things in steps. One step after the other, in sequence.

Easier to reason about and less prone to confusion or mixups. However, causes ineffective "blocks" where the application is waiting for a process to finish when it could be performing other steps.

### 1.2.2. Concurrent programming runs multiple tasks at the same time

A multitasking operation can give the illusion of being efficient. If only one operation is being performed at a time, then there is no real gain, even if the order is more complex. 

*Concurrency* is the ability to run several programs or parts of a program at the same time.

Concurrency can make it seem like threads are running in parallel and that they can run simultaneously, but single-core environments pause the execution of one thread to continue or start another. Parallelism is when the number of resources is increased to provide actual multitasking.

### 1.2.3. Parallel programming executes multiple tasks simultaneously

*Parallelism* is executing multiple tasks at once, concurrently. Same time, different cores. 

All parallel programs are concurrent, but not all concurrency is parallel. Parallelism is achieved only through multiple cores. 

Multicore processing is when two processors work on two separate tasks, independently. In the barista example, one barista brews the coffee, the other prepares the milk.

Operations are considered *concurrent* if they can be executed in parallel.

Operations are considered *parallel* if the executions overlap in time.

Parallel programming is a subset of concurrent programming. 
>Concurrency refers to the design of the system, parallelism relates to the execution.

#### Additional Research into the Difference between Parallelism and Concurrency

>Concurrency and parallelism are related terms but not the same, and often misconceived as the similar terms. The crucial difference between concurrency and parallelism is that concurrency is about dealing with a lot of things at same time (gives the illusion of simultaneity) or handling concurrent events essentially hiding latency. On the contrary, parallelism is about doing a lot of things at the same time for increasing the speed.
>Parallelly executing processes must be concurrent unless they are operated at the same instant but concurrently executing processes could never be parallel because these are not processed at the same instant.

| Basis for Comparison | Concurrency | Parallelism |
| -------- | -------- | -------- |
| Basic     | Managing and running multiple computations at the same time     | Running multiple computations simultaneously     |
| Achieved through | Interleaving Operation | Using multiple CPU's |
| Benefits | Increased amount of work accomplished at a time | IMproved throughput, computational speed-up |
|Make use of | Context switching | Multiple CPUs for multiple processes |
| Processing units | Probably single | Multiple |
| Example | Running multiple applications at the same time | Running a web crawler on a cluster |

> simultaneous: existing, occurring, or operating at the same time; concurrent:

**Best example I can find: Concurrency is like eating and talking during a meal. You are doing two tasks at the same time, *eating* and *talking*, but you can only do one or the other. You can eat, then talk, then eat. This is concurrency. Parallelism would be when one individual is eating and the other is talking. Two operations at the same time.
Parallelism is like singing and dancing. You can sing and dance at the same time at once.
[:movie_camera: YouTube Video](https://www.youtube.com/watch?v=ltTQaMSk6ME)**

### 1.2.4. Multitasking performs multiple tasks concurrently over time

*Multitasking*: performing multiple tasks over a period of time by executing them concurrently. Doing two (or more) things at once.

*Time slice*: scheduling logic that coordinates execution between multiple threads

*Thread quantum*: the amount of time the schedule allows a thread to run before scheduling a different thread.

Context switching is handled by the operating system. But in a single-core computer, the overhead from switching contexts may be more taxing than the time saved from switching.

Two kinds of multitasking operating systems

* *Cooperative multitasking systems*
  * Scheduler lets each task run until finished *or* explicitly yields control back to scheduler
* *Preemptive multitasking systems*
  * Windows and most operating systems
  * Scheduler prioritizes the tasks
  * System switches execution based on priority when the task time allocation is completed

### 1.2.5. Multithreading for performance improvement

*Multithreading*: a form of concurrency that uses multiple threads of execution. Multithreading implies concurrency, but concurrency does not necessarily imply multithreading.

## 1.3. Why the need for concurrency

### 1.3.1. Present and future of concurrent programming

## 1.4. The pitfalls of concurrent programming

### 1.4.1. Concurrency hazards

### 1.4.2. The sharing of state evolution

### 1.4.3. A simple real-world example: parallel quicksort

### 1.4.4. Benchmarking in F\#

## 1.5. Why choose functional programming for concurrency

### 1.5.1. Benefits of functional programming

## 1.6. Embracing the functional paradigm

## 1.7. Why use F# and C# for functional concurrent programming
