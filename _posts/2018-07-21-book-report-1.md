---
layout: default
title:  "Book Report: Data-Oriented Design by Richard Fabian"
comments: true
---

# Book Report: Data-Oriented Design by Richard Fabian

This book is a very good resource for game programmers to understand the gist of data oriented design and its benefits. While part of the book is just giving some well-known techniques an academic term, the other parts contain many implementation examples and suggestions to think in a data oriented manner. Especially for we, as game engine programmer, we should realize the fact that a game engine is just a giant stream data processing engine, which programming for the data makes more sense.

My sharing will be divided into 3 parts: A quick comparison between object and data oriented design, 3 keys to think about before creating a data oriented solution, and my feeling towards data oriented design in game.

## Comparison

Here I gives a simple comparison between object oriented design and data oriented design by summarizing all the chapters in the book. Only simple rationale is given, please read the book for a more thorough proof of concept.

__Object oriented design is__
* A modelling, a abstraction
* Not wrong on its own
* Is wrong when it is used in high performance/low latency programming
* A way to fit real world concept into problem solving by programming
* Not the best way to design a solution that runs on current generation hardware
* Reusable in terms of code, class and interface
* Easy to understand by looking at the relationship between objects
* Evicting data in cache more easily especially when there are a lot of polymorphic calls
* Promoting inheritance and dynamic polymorphism which increases cyclomatic complexity when used heavily

__Data oriented design is__
* A way of thinking, an anti-abstraction
* All about the data, its transformation and the environment it runs on
* Not only about cache friendly data structure
* Taken from database concept
* Reusable in terms of data transformation
* Easy to add new features because there are no coupling between data and what we need is just to another transform on a selection of data
* Easy to debug and test because the input and output is the only factors and the data transform has no side effect
* Hard to understand relationship by just looking at interfaces
* Not friendly for writing generic code but optimization friendly

## 3 Keys

I summarize three key characteristics of data oriented design, hopefully with these it would be easier for anyone to start thinking around data and create data oriented solutions.
1. Context
2. Separation
3. Explicitness

__Context__

Give context. We want to know more about the problem we are going to solve, the data layout of our data, the amount of our data, the platform we are going to run on, the hardware specification of that platform, the compiler of our choice. Basically gather as much information as we can. A quick example, sometimes we don’t need a full sort if we know how our data is inserted into a data structure and approximately where our target data will be located at. 

Learn about CPU and compiler. For example, We should know how branch prediction works on cpu level and why it can affect our performance greatly when we are having a large amount of wrong prediction in a tight loop. Another example, we should understand why chain pointer lookup is bad. It is not just non-cache friendly, it also requires the previous pointer to be finished looking up before accessing the next one. In this case there is no room for out of order execution optimization in the CPU or instructions rearrangement done by the compiler.

Knowing the potential size of our data also helps a lot. When our data is small enough or we structure it to a small enough size that fits better in cache, we can use some more cache-friendly structures and algorithms. Such as a cache-oblivious B-tree in place of a normal tree. Think about searching, when our data or element count is small enough, a binary or linear search can be faster than another other sophisticated sorting algorithm.

__Separation__

The author described a technique called “Existential Programming”. A very cool term but basically it just means that we prepare our data and algorithm so that computation is only carried out according to necessity. A common example of the opposite is a if-clause checking if a pointer is nullptr then carry out the respective action, it causes wrong branch prediction and possibly evict data from instruction and memory cache. Except from separating between necessary and unnecessary, we should also try to separate between hot and code path, and read and write access.
```cpp
for all gameobjects
	if mesh is not null
		draw
```
```cpp
for all visible meshs
	draw
```

A gameplay programming example, by using a table based finite state machine (FSM) instead of typical class and function based FSM, we can separate all states and their transition operations. Each state only write to and read from the table, instead of doing callback or direct function call. In this way, it is not only easier to debug because repeating a certain situation is only a matter of value in the table, it is also scalable to parallel processing and open up an opportunity to allow multiple events and transitions simultaneously.

One of the problems of OOP is that it always result in very fine grain of processing unit, and it’s very hard to switch between different granularity. Take a forest as an example, usually when we create object oriented interface it looks like this - a forest consists of trees, a tree consists of leaves. In this case, no matter we want to compute in forest's unit or leave's unit, we have to iterate through all the hierarchies and it worsens the situation if we have to jump between memory or process numerous dynamic polymorphic calls. Using data oriented design, the data is just plainly accessible without any hierarchy and meaning, so the granularity of processing can be changed easily from frame to frame.

__Explicitness__

Be as explicit as we can in our coding and thinking process to help ourselves and the compiler. This is not just a matter of coding style to write cleaner code, it also allows easier optimization done by the compiler. For example, reducing order of dependence in memory access. Sometimes it can be more performant by calculating a value again instead of waiting for the result when multiple assignment for the same value is needed.

Expressing unchanged memory is also one way to express our variable usage explicitly for the compiler. Using constness is one way, using reference and pointer in a smart way is more difficult. Take the below as an example, if the compiler cannot be sure that a value in a for-loop won’t be changed through the whole loop, it will have to load the value again every loop to play safe. In this case, q has to reloaded every loop.
```cpp
void foo(const int& q){
	for (int i = 0; i < q; ++i){
		// do something
	}
}
```

Also be explicit in our plan of attack to data oriented optimization. When planning for optimization, we should follow well defined procedures for example as suggested in the book: Define the problem -> Measure -> Analyze -> Implement -> Confirm. The book contains more details on how each step should be done. Of course it is not the only way to do optimization, but it is surely a simple and basic reminder for everyone trying to optimize their code, to use a structured method and stick close to it.

## My Feeling

__The Past__

Just as any other one who entered the industry within this decade, I started learning programming with Object Oriented Design. Even until now it seems to be the easiest way to introduce a programming language for beginners because that mimics how human thinks naturally.

By reading this book, we can understand that Data Oriented Design is not as intuitive if we are not familiar with the thought process and have certain experience on working with intensive data. But does it mean that it is not worth the time and effort to educate the students or others about it? Obviously not. I believe many veteran in the industry have already started using Data Oriented Design because they have noticed the problem when they are doing iterations to deal with ever changing users’ requirement, just that the term is not there yet.


__The Present__

I have been trying data oriented design out in the recent particle runtime development. Turns out it proved to be extremely easy to add new features and iterate on it. But there are two points that I should have been more cautious about. First, it takes time to come up with the implementation. Not only it is not as intuitive, it is also because I accidentally put too much focus on performance at the beginning of implementation. Cache friendliness overtook my mind and I became very worried about does the data always lies sequential in cache. Second, jobification poses a question to the granularity of processing. By using the archetype-based ECS system to store particle emitters and their modules, the most direct unit of processing is archetype. However, when spreading the workload across threads, it might not be the perfect unit because some archetypes might contain very little emitters or less modules that requires simulating. At the end I partitioned the archetype array to simulate a fixed number of emitters per job and it results in logical speed up. It might be the correct way to go but I wonder is there a better way if I take jobification into account from the very beginning.

__The Future__

Sometimes programmers have an unconscious pursuit of genericity. When I was a beginner it appeared to me writing generic code that can handle many different situations is a showcase of engineering intelligence. Writing generic code might be important in companies that focus on building commercial software or libraries, but for game industry, that really should not be the main focus of our work. Especially in a AAA studio which works on multiple high-end titles at the same time and targeting multiple platforms, with each of them having their own set of data, the importance of optimization outruns reusability. And often optimization breaks reusability.

Is this the future? Yes, but definitely not the only future. With advancement in hardware, it can be guaranteed that the future architecture will be vastly different from the one we have now. Taking an extreme example, quantum computing, I am confident to say that the same rule will not simply apply. But I think the value of Data Oriented Design is to remind us to look at the thing that is actually important and conquers the most part of a software. In this case, it is the data not the programmer.

## Reference
1. The book website and there's a online version: http://www.dataorienteddesign.com/site.php

