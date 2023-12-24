+++
title = "My path to becoming a Rustacean"
description = "How I'm learning Rust while coming from a Python background, and how you can too"
date = 2023-12-21
draft = true

[taxonomies]
tags = ["rust"]
categories = ["Languages"]

[extra]
toc = true
comment = true
+++

## The future is Rusty 🦀

> *"It wasn't always so clear, but the Rust programming language is fundamentally about empowerment: no matter what kind of code you are writing now, Rust empowers you to reach farther, to program with confidence in a wider variety of domains than you did before."*
>
> -- The Rust Book, [foreword](https://doc.rust-lang.org/book/foreword.html)

The quote above is easy to gloss over when first reading the Rust Book, but it's a powerful statement that I've come to appreciate more and more as I've spent time learning Rust this year. And, it feels good to know there are a lot of others like me. 😅

I'm a long-time Python developer who always struggled with C and C++, from a syntax and a usability perspective . For the most part, I've avoided thinking about lower-level concepts like memory management and gotten away with it, because there have always been excellent Python libraries that do a lot of the heavy lifting from a performance perspective.

However, in recent years, as I've become a better developer who can tackle more complex problems, the need for a different style of programming with static typing and compiler checks has become that much more apparent in my work.

In this post, I'll highlight my learning journey with Rust, and how I've come to appreciate the language and its community. I'll also share some tips and resources on how you can get started and most importantly, stay motivated in your journey.

## Why I need more than Python

Python has withstood the test of time, and I've been using it for over a decade now in numerical/scientific computing, data science and machine learning. It's *really* easy to get started with, and there are a ton of great libraries out there that make it easy for relatively inexperienced programmers to build scalable and complex applications. However, it's also in Python's strengths that there are footguns that can lead to problems down the line.

{% tip(header="TL;DR") %}
The issues listed below are **not** related to Python (supposedly) being slow. In fact, it's relatively easy these days to write performant Python code, via it's amazing library ecosystem, that's fast enough for most use cases. The key areas of interest for me are related to *safety* (primarily type safety), and *reliability* of code that runs in production.
{% end %}

### Python discovers bugs at run time

Python is a dynamically typed, interpreted language, which means that the type of a variable is inferred at runtime. This is great for getting started quickly, but it also means that your initial runs in production code will almost always error out, commonly due to type mismatches. This leads to a frustrating developer experience, and worse, some of these bugs can be incredibly hard to track down in a large codebase.

In a recent project, I spent 3 days tracking down a bug (with unhelpful error messages) that was related to an invalid datetime object. This is a rather trivial bug that would have been caught at compile time in a statically typed language.

{% note() %}
Rust is a statically typed, compiled language, which is *great* for type safety. The Rust compiler is famously strict, and can catch a host of bugs prior to run time. In my time learning Rust, I've found that the compiler is almost always helpful in its error messages as well (though there are definitely still situations where I can't understand head or tail of error messages -- this will hopefully improve with time).
{% end %}

### Python type annotations can be a mess

The original intention behind Python type annotations, or more simply, [*typing*](https://docs.python.org/3/library/typing.html), was to help developers catch type-related bugs early on via static type checking (like mypy), and to help with code readability. However, as the docs state, the Python runtime doesn't enforce types -- code that has invalid type annotations will still run perfectly fine -- it's purely a voluntary exercise by the developer, which doesn't exactly increase confidence in the code.

In practice, type annotations [can be a mess](https://x.com/mitsuhiko/status/1632675582176526337) from a readability perspective, especially when you're dealing with complex data structures that are very common in production-scale projects.

{% note() %}
Rust's has an immensely powerful and expressive type system, including strong support for generics, and it's easy to define complex, custom data structures that make code easier to read and understand. The compiler is *very* helpful in catching type-related bugs early on, and the error messages are usually very helpful in pointing you in the right direction, avoiding a lot of frustration in catching and fixing trivial bugs prior running any code in production.
{% end %}

### Refactoring can be a nightmare in Python

When refactoring even a moderately large codebase in Python, it's all too easy to miss edge cases and have faith that you wrote all the tests that were required. Although the availability of static type checkers in Python have reduced the need to write tests for type-related bugs (which is a good thing), it's very common to face all sorts of nasty bugs in production, largely due to a developer's invalid assumptions about the codebase.

{% note() %}
Rust's compiler is very strict, which can be frustrating initially, but it *really* helps with catching bugs prior to run time. This means that you can focus on writing tests mainly for logical bugs, allowing the rich type system and compiler checks to assist you. Refactoring a large code base in Rust (I'd imagine), is a far more pleasant task than in Python.
{% end %}

---

## A learning path for new Rustaceans

When learning a new programming language, conventional wisdom states that you go "bottom-up", starting with the fundamental concepts and building up from there -- this is typically done via books and tutorials. Alternatively, a lot of folks choose to begin "top-down", starting with a high-level overview of a problem of interest and then diving into the details that are relevant to that problem, by directly working with code and examples.

Personally, I've found that the best way to learn Rust is to **do both at the same time**, i.e., switch between the bottom-up and top-down approaches when you're starting off. This is because Rust has a lot of  concepts that are hard for beginners, like traits, borrow-checking and lifetimes, but these concepts are ubiquitous throughout the language, in a way that you can't sequentially increase the complexity of your learning without getting stuck.

{{figure(src="rust-learning-modes.png", alt="Learning Rust by mixing top-down and bottom-up approaches")}}

Start with [The Rust Book](https://doc.rust-lang.org/book/) and pre-built exercises (like [Rustlings](https://github.com/rust-lang/rustlings)) that aim to demonstrate key concepts in Rust, but, also pair this with a top-down approach where you study tools and frameworks that are relevant to your problem of interest. Write your own code to solve really simple problems that you know how to do in another language (in my case, Python). This helps build a mental model of how Rust works, why it's good (and not so good) at certain things, while also giving you a sense of accomplishment as you build something that's relevant to you.

The following subsections will go into more detail on the steps I followed that were very helpful to a) get motivated and b) remain motivated to continue learning even when I got stuck.

### 1. Consume all the content

It's very helpful to gain a birds-eye view of a language, its ecosystem and its community before diving in. I spent a *lot* of time reading blog posts, watching YouTube videos, listening to podcasts and simply following other Rustaceans on Twitter.

Venn diagram of podcasts, books, blog posts, YouTube channels and Discord servers. Cover all the modes of learning to reinforce the concepts.

In my journey, I learned that Rust is a language with a lot of history and *lore* (Ferris and Corro).

### 2. Understand what makes Rust great for *you*

If you ask yourself the question: *Why am I interested in Rust?* and the answer is "*because everyone else is talking about it*", you're probably not going to get very far. The **first** step is to convince yourself of what aspects of Rust resonate with you. Depending on your technical background and career interests, your answer could be vastly different from mine.

This stage is the most unique to each individual, and it's important to spend some time thinking about it. For me, it was the exact reasons I listed [above](#why-i-need-more-than-python) and my frustrations with Python's type system (or lack thereof), that led me to think about Rust. And, having gone through step 1 above, I'd learned a great deal about Rust's expressive type system, strong emphasis on safety and what having good compiler checks can do for your confidence in your code.

### 3. Embrace the meme game

Learning is about having fun -- when the going gets tough, the best way to keep at it is by laughing at yourself. Rust's meme game is strong!

https://x.com/shuttle_dev/status/1699375589746995644?s=20

https://programmerhumor.io/programming-memes/rust-devs-footprints/

### 4. Apply it to your existing problems

Don't just follow what the books say -- that's the fastest path to boredom. Try to apply it to your existing problems that you already know how to solve in the language you're coming from.

### 5. Document your journey publicly

This really helps with accountability, and also helps others who are in the same boat as you. Building and experimenting in private leads to drop-offs and dead ends, and it's all too easy to fall of the wagon without any accountability. You can always ask for help on Discord.

### 6. Spread the word

Once you have a handle on the core concepts, you'll find yourself naturally thinking about your code in a way that encapsulates the Rust way of doing things -- performance, safety and low resource utilization. Before you know it, you'll find yourself evangelizing Rust to your friends and colleagues, not to mention you'll be posting about your work publicly and collaborating with like-minded people who've gotten there before you did.

## How hard is Rust, really?

If you're coming from an interpreted language (like I did from Python), the best way to approach this is to think of Rust as a language that's *different* from what you're used to, rather than a language that's hard in and of itself. In fact, I'd argue that writing Rust *makes me a better Python programmer*, as I find myself thinking about my code mch more carefully than I would if I were haphazardly prototyping. In Rust, you pay a cost upfront to write safer and correct code, unlike in interpreted, dynamically typed languages.

Although the initial "honeymoon" period can come to an end when you hit a wall with concepts like ownership, borrowing, traits and lifetimes, they are all part of what makes Rust so powerful. The beauty of Rust is in the fact that the parts that are difficult, *need to be*, because they address the core aspects of computing right down to the machine level, whereas the parts that are easy (like managing dependencies and distributing your code) are truly a breeze because they've been so well designed from the ground up.

A key stage in your Rust journey is when you stop feeling frustrated with the compiler, but instead start viewing it as your ally. **The compiler exists to help you write better code**. You spend your valuable time working with logical issues rather than the trivial stuff, allowing you to solve problems of greater complexity with the confidence that your code is safe, correct, and will run reliably in production.

Granted, there will be periods early on where you don't feel as productive as you did in your erstwhile favourite language, but here, [The Rust Book](https://doc.rust-lang.org/book/) is forever your friend, and you'll find yourself re-reading the book multiple times as you pick up on things you missed the first time around.


## Conclusions

It's very easy to feel impostor syndrome when you're learning Rust -- after all, a lot of really smart people have gotten there before you. Tim McNamara has the best solution: start small, and grow gradually[^1]. When you're starting off, *it's okay to write basic code that might be inefficient*, as long as it works and achieves the intended goal. Chances are, your relatively naive Rust code will be faster than your Python code. 😀

As you keep writing Rust code, you eventually reach a point where you understand how to improve efficiency by reducing allocations, using iterators, etc. I'm not there myself yet, but it's just a matter of time. This leads to an important question: When can you call yourself a Rustacean? Jon Gjengset's take[^2] is quite valid one: "You are a Rustacean from just before the first time you ask yourself whether you might be one".

Having spent some *serious* time studying Rust this past year, I've come to frame it for myself as follows:  

> *If you find yourself thinking about Rust, writing Rust code and spreading the word about why the language matters to you, you are a Rustacean.* 😎

---

## References

[^1]: [Tim McNamara on Learning Rust](https://www.manning.com/livevideo/rust-in-action?query=rust%20in%20action)

[^2]: [Jon Gjengset Gets into the Nitty-Gritty of Rust](https://nostarch.com/blog/software-engineer-jon-gjengset-gets-nitty-gritty-rust)

---

## Learning materials

This section summarizes some great learning resources I've come across and used in my Rust learning journey. Feel free to comment to add some more to this list so that others can benefit from them as well!

### Books

I recommend reading the following books in the order listed below:

- [The Rust Book](https://doc.rust-lang.org/book/) by Carol Nichols and Steve Klabnik
- [Rust by example](https://doc.rust-lang.org/rust-by-example/) (Extension to the Rust Book)
- [Rust in Action](https://www.manning.com/books/rust-in-action) by Tim McNamara
- [Zero to Production in Rust](https://www.zero2prod.com/) by Luca Palmieri
- [Rust for Rustaceans](https://rust-for-rustaceans.com/) by Jon Gjengset

### YouTube channels

The following YouTube channels are great for tackling specific concepts in Rust, one by one, and to gain a birds-eye view of the langauage.

- Tim McNamara [@timClicks](https://www.youtube.com/c/timclicks)
- Tristram Oaten [@NoBoilerPlate](https://www.youtube.com/@NoBoilerplate)
- Trevor Sullivan: [@TrevorSullivan](https://www.youtube.com/watch?v=lTjGt17bQ3k&list=PLDbRgZ0OOEpUkWDGqp91ODn0dk7LPBAUL)
- Jon Gjengset: [@jonhoo](https://www.youtube.com/c/JonGjengset)