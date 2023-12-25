+++
title = "My path to becoming a Rustacean"
description = "How I'm learning Rust while coming from a Python background, and how you can too"
date = 2023-12-24
draft = false

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

The quote above is easy to gloss over when first reading the Rust Book, but it's a powerful statement that I've come to appreciate more and more as I've spent time learning Rust this year.

I'm a long-time Python developer who always struggled with C and C++, from a syntax and a usability perspective . For the most part, I've avoided thinking about lower-level concepts like memory management and static typing and gotten away with it, because there have always been excellent Python libraries that do a lot of the heavy lifting from a performance perspective.

However, in recent years, as I've become a more experienced developer tackling ever more complex problems, the need for a different style of programming with compile-time checking has become that much more apparent in my work.

In this post, I'll highlight my learning journey with Rust, and how I've come to appreciate the language and its community. I'll also share some tips and resources on how you can get started and most importantly, stay motivated in your journey.

{% note() %}
A list of Rust learning resources that's greatly helped me in my journey is available at [the end of this post](#learning-materials).
{% end %}

## Why I need more than Python

Python has withstood the test of time, and I've enjoyed using it (for the most part) for numerical/scientific computing, data science and machine learning tasks over the last decade. It's *really* easy to get started with, and there are a ton of great libraries out there that make it easy for relatively inexperienced programmers to build complex applications. However, it's in Python's very strengths that there lie footguns that could lead to problems down the line.

{% tip(header="TL;DR") %}
The issues listed below are **not** related to Python (supposedly) being slow. In fact, it's relatively easy these days to write performant Python code, via it's amazing library ecosystem, that's fast enough for most use cases. The key areas of interest for me are related to *safety* (primarily type safety), and *reliability* of code that runs in production.
{% end %}

### Python discovers bugs at run time

Python is a dynamically typed, interpreted language, which means that the type of a variable is inferred at runtime. This is great for getting something up and running quickly, but it also means that your initial runs in production will almost always error out, due to bugs that can only be discovered at run time depending on the existing environment. This leads to a frustrating developer experience, and worse, some of these bugs can be incredibly hard to track down in a large codebase.

In a recent Python project, I spent 3 days tracking down a bug (with unhelpful error messages) that I painstakingly discovered was related to an invalid datetime object. This is a rather trivial bug that would have been caught at compile time in a statically typed language.

{% note() %}
Rust is a statically typed, compiled language that greatly emphasizes type safety. The Rust compiler is famously strict, and can catch whole classes of bugs prior to run time. In my early days using Rust, I've found that the compiler is (mostly) helpful via its error messages, though there are definitely still situations where I can't make head or tail of the error messages -- this will hopefully improve with time.
{% end %}

### Python type annotations can be a mess

The original intention behind Python type annotations, or more simply, [*typing*](https://docs.python.org/3/library/typing.html), was to help developers catch type-related bugs early on via static type checking (like mypy), and to help with code readability. However, as the docs state, the Python runtime doesn't *enforce* types -- code that has invalid type annotations will still run perfectly fine -- typing is a *voluntary* exercise by the developer, which doesn't exactly help increase one's confidence in the code itself.

In practice, type annotations [can be a mess](https://x.com/mitsuhiko/status/1632675582176526337) from a readability perspective, while still not eliminating type-related bugs entirely, especially when you're dealing with the complex data structures commonly seen in production-scale projects.

{% note() %}
Rust has an immensely powerful and expressive type system, now with an [entire team](https://blog.rust-lang.org/2023/01/20/types-announcement.html) dedicated to supporting it. It's relatively easy in Rust to define complex, custom data structures that make code easier to read and understand. The compiler is *very* helpful in catching type-related bugs early on, avoiding a lot of developer frustration prior to running any code in production.
{% end %}

### Refactoring can be a nightmare in Python

When refactoring even a moderately large codebase in Python, it's all too easy to miss edge cases and lose faith in your existing tests due to relatively trivial bugs that surface in production. Although the availability of static type checkers in Python have reduced the need to write certain kinds of tests (which is a good thing), type-checking in Python isn't enforced by the interpreter -- so it's still very common to face all sorts of nasty bugs in production following a major refactor.

{% note() %}
Rust's strict compiler can be frustrating for newcomers to the language, but it *really* helps with catching entire classes of bugs prior to run time. This means that you can focus on testing mainly for logical bugs, allowing the rich type system and compiler checks to assist you in other areas. Refactoring a large code base in Rust (I'd imagine), is a far more pleasant task than in Python.
{% end %}

---

## A learning path for new Rustaceans

I've been learning about Rust for the past year, and, like many others, I've found it to be a very different language from what I'm used to. When learning a new programming language, conventional wisdom states that you go "bottom-up", starting with the fundamental concepts and building up from there -- this is typically done via books and tutorials. Alternatively, a lot of folks choose to begin "top-down", starting with a high-level overview of a problem of interest and then diving into the details that are relevant to that problem, by directly working with code and examples.

Personally, I've found that the best way to learn Rust is to approach it from **both directions at the same time**, i.e., alternate between the bottom-up and top-down approaches when you're starting off. This is because Rust has a lot of concepts that are hard for beginners, like traits, references, borrowing and lifetimes, but these concepts are ubiquitous throughout the language, in a way that you can't sequentially increase the complexity of your learning without getting stuck.

{{figure(src="rust-learning-modes.png" alt="Learning Rust by mixing top-down and bottom-up approaches")}}

Start by reading [The Rust Book](https://doc.rust-lang.org/book/) cover to cover and then go through some pre-built exercises (like [Rustlings](https://github.com/rust-lang/rustlings)). **It's okay if it all doesn't make sense in the beginning** -- the primary goal should be to get a sense of the terminologies used in Rust, and to get a feel for the language's syntax and structure.

The following subsections will go into more detail on the steps I followed that were very helpful to a) get motivated and b) remain motivated to continue learning even when the going got tough.

### 1. Consume all the content

I had a *lot* of fun this year reading blog posts, watching YouTube videos, listening to podcasts and simply following other Rustaceans on Twitter, giving me a very clear drive and focus towards wanting to go deeper on my own terms. Just like with any other language, I believe the best way to learn Rust is to consume as much content on it as you can to gain a birds-eye view of useful features, and to then try and apply it to your own problems.

{{figure(src="rust-learning-materials.png", alt="Consuming Rust resources of a broad variety is the most effective way to learn")}}

Keep alternating between the various forms of learning material, allowing your brain to process the information in different forms. Over time, you get hooked, and the pieces start to come together in your mind. There are so many great learning resources out there, which I list at the end of this article.

### 2. Describe in words what makes Rust great for *you*

Ask yourself the question: *Why am I interested in learning Rust?* If the answer is "*because everyone else is talking about it*", or anything else related to FOMO, you're probably not going to get very far. The **key** is to convince yourself of what aspects of Rust resonate with **you**. Depending on your technical background and career interests, your answer could be vastly different from mine.

In my case, it was the exact reasons I listed [above](#why-i-need-more-than-python) and my frustrations with Python's type system (or lack thereof), that led me to discover Rust. Having consumed my fair share of podcasts, YouTube videos, blogs and books over the course of a year, I'd learned a great deal about Rust's expressive type system, strong emphasis on safety and what having good compiler checks can do in terms of catching bugs early on.

### 3. Embrace the memes

Learning is also about having fun -- when the going gets tough, the best way to keep at it is by laughing at yourself and the situation you're in. [@shuttle_dev](https://twitter.com/shuttle_dev) has among the strongest meme games out there, not to mention that they regularly post great learning material as well. 😄

{{ twitter(url="https://twitter.com/shuttle_dev/status/1699375589746995644?s=20" class="twitter") }}

Twitter and Reddit are also great places to find self-deprecating content and memes on Rust, and the Rust community in general is very welcoming to newcomers. 🚀

### 4. Apply Rust to your existing problems

Don't just follow along with what the books or tutorials show -- that's the fastest path to boredom. Try applying Rust to existing problems that you *already know how to solve* in your primary language of choice.

In my case, I'm actively working on small, self-contained projects that I have no difficulty doing in Python, but am now writing from scratch in Rust. Because I already have a mental map of how I'd solve the existing problem logically, I can focus my time and energy on learning Rust terminology and syntax, rather than getting bogged down by the problem itself.

### 5. Document your learning journey publicly

Don't build and experiment in private -- in my experience, this just leads to dead ends and falling off the wagon entirely. Although it may seem intimidating at first, documenting your code and writing about your work publicly *really* helps with accountability. It can also help others who are in the same boat (or those who are just starting off).

Just like I've done in this article, I'm making it a point to write in detail about my learning journey whenever I can. I'm also sharing my code on GitHub and will soon be writing a book tutorial series that I'm calling [*Rust in Pieces*](https://github.com/thedataquarry/rustinpieces). The project aims to document like-for-like Python and Rust code that perform the same tasks, to highlight the differences in code structure and to help gain familiarity with the "Rust way" of doing things.

Consider a simple Python function that converts unicode text to ASCII:

{% codeblock(name="Python") %}
```python
import unicodedata

def convert_unicode_to_ascii(text: str) -> str:
    # Normalize unicode text NFKD form, then to ASCII
    text = unicodedata.normalize("NFKD", text)
    text_ascii = text.encode("ASCII", "ignore").decode("utf-8")
    return text_ascii
```
{% end %}

In many cases, Rust code appears more functional than the equivalent Python code, giving me that pleasant feeling that I'm learning a new way of thinking about coding.

{% codeblock(name="Rust") %}
```rust
use unicode_normalization::UnicodeNormalization;

fn convert_unicode_to_ascii(s: &str) -> String {
    // Normalize unicode text NFKD form, then to ASCII
    s.nfkd().filter(|c| c.is_ascii()).collect::<String>()
}
```
{% end %}

As you move forward inch by inch, you'll find you are slowly but surely writing more idiomatic Rust code (with the compiler's help, of course), while also learning about the language's standard library and the amazing [crates.io](https://crates.io/) ecosystem.

### 6. Spread the word

Once you have a handle on the core concepts, you'll find yourself naturally thinking about your code in a way that encapsulates the Rust way of doing things -- performance, safety and low resource utilization. Before you know it, you'll find yourself evangelizing Rust to your friends and colleagues, not to mention you'll be posting about your work publicly and potentially collaborating on bigger projects with like-minded people.

{{figure(src="rust-shuttle-tweet.png" alt="Yes, I might soon become one of <a href='(https://x.com/shuttle_dev/status/1731780419870241177)'>these guys</a> 😄")}}

## Rust isn't hard, it's just *different*

No matter what language you're coming from (or, even if this is your first exposure to a programming language), the best way to approach Rust is to think of it as a language that's *different* from what you're used to, rather than a language that's hard in and of itself. Rust is a big and feature-rich language whose core concepts can take time and effort to grasp, but there are plenty of resources out there to learn it over a sustained period.

I'd argue that writing Rust *makes me a better Python programmer*, as I find myself thinking about my code's structure much more carefully than I would if I were haphazardly prototyping in a dynamically typed language. In Rust, you pay a cost upfront to write safer and correct code, which in turn makes you think about your code's structure and design from the get-go.

Although the initial "honeymoon" period will come to an end and you might hit a wall with concepts like ownership, borrowing, traits and lifetimes, these are all part of what makes Rust so powerful. The beauty of Rust is in the fact that the parts that are hard, _**need to be**_, because they deal with core aspects of computing right down to the machine level, whereas the parts that are easy (like managing dependencies and distributing your code) are truly a breeze because they've been so well designed from the ground up.

A key stage in your Rust journey is when you stop feeling frustrated with the compiler, but instead start viewing it as your ally. **The compiler exists to help you write better code**. You spend your valuable time working with logical issues rather than the mundane, allowing you to solve problems of greater complexity with the confidence that your code is safe and will run reliably in production.

Is Rust the perfect language? No, but no language really is. There will most definitely be periods early on in the journey where you don't feel as productive as you did in your erstwhile favourite language, but here, [The Rust Book](https://doc.rust-lang.org/book/) is forever your friend, and you'll find yourself re-reading the book multiple times as you pick up on things you missed the first time around.

## Conclusions

For me, this is just the start of a multi-year journey learning Rust, and I'm excited to see what this ongoing wave of enthusiasm among new and established Rustaceans blossoms into over the coming years. After all, Rust is a language that's been designed to be around [for the next 40 years](https://www.youtube.com/watch?v=A3AdN7U24iU). Having come to Rust from Python for the *safety* and *reliability* that it offers, the speed/efficiency gains it offers are just the icing on the cake.

It's very easy for one to feel impostor syndrome while learning Rust. Tim McNamara frames this really well: [*start small, and grow gradually*](https://www.youtube.com/watch?v=sDtQaO5_SOw&t=30s). When you're starting off, it's **OK**** to write basic code that might be inefficient, as long as it works and achieves the intended goal. Chances are, your relatively naive Rust code will still be way faster than your Python code. 😀

When do you call yourself a Rustacean? [Jon Gjengset's take](https://nostarch.com/blog/software-engineer-jon-gjengset-gets-nitty-gritty-rust) is: "You are a Rustacean from just before the first time you ask yourself whether you might be one".

Having spent some *serious* time studying Rust this past year, I've come to frame it for myself as follows:  

> *If you find yourself thinking about Rust, writing Rust code and spreading the word about it, you are a Rustacean.* 😎

---

## Learning materials

This section summarizes some great learning resources I've come across and benefitted from in my learning path. Feel free to comment and add some more to this list so that others can benefit from them as well!

### Books

I recommend reading the following books in the order listed below:

- [The Rust Book](https://doc.rust-lang.org/book/) by Carol Nichols and Steve Klabnik
- [Rust in Action](https://www.manning.com/books/rust-in-action) by Tim McNamara
- [Zero to Production in Rust](https://www.zero2prod.com/) by Luca Palmieri
- [Rust for Rustaceans](https://rust-for-rustaceans.com/) by Jon Gjengset

### YouTube channels

The following YouTube channels are great for tackling specific concepts in Rust, one by one, and to gain a birds-eye view of the langauage.

- Tim McNamara [@timClicks](https://www.youtube.com/c/timclicks)
- Tristram Oaten [@NoBoilerPlate](https://www.youtube.com/@NoBoilerplate)
- Trevor Sullivan: [@TrevorSullivan](https://www.youtube.com/watch?v=lTjGt17bQ3k&list=PLDbRgZ0OOEpUkWDGqp91ODn0dk7LPBAUL)
- Jon Gjengset: [@jonhoo](https://www.youtube.com/c/JonGjengset)

### Podcasts

The following podcasts are great for getting a sense of the Rust community and the people behind the language. I've listed individual podcast episodes from other podcasts that are also worth listening to.

- [The Rustacean Station](https://rustacean-station.org/)
- [Are we podcast yet?](https://arewepodcastyet.com/)
- [Rust: A language for the next 40 years](https://hanselminutes.com/713/rust-a-language-for-the-next-40-years-with-carol-nichols): Carol Nichols, Hanselminutes
- [The future of safe and efficient programming](https://soundcloud.com/awsdevelopers/episode-083-decoding-the-future-of-safe-and-efficient-programming-with-rust-and-tim-mcnamara): Tim McNamara, AWS Developer Podcast
- [History of Rust](https://rustacean-station.org/episode/042-ben-striegel/): Ben Striegel, Rustacean Station
- [Rust for Rustaceans](https://rustacean-station.org/episode/038-jon-gjengset/): Jon Gjengset, Rustacean Station

### Exercises/Tutorials

- [Rustlings](https://github.com/rust-lang/rustlings): Small exercises to get you used to reading and writing Rust code
- [Rust on Exercism](https://exercism.org/tracks/rust): 96 exercises to help you write better Rust code
- [Rust by example](https://doc.rust-lang.org/rust-by-example/): Extension to the Rust Book, with examples of common Rust concepts
- [Shuttle Launchpad](https://www.shuttle.rs/launchpad): A series of exercises by the folks at [Shuttle](https://www.shuttle.rs/blog/tags/all) and Stefan Baumgartner, organizer of the biggest Rust meetup in Europe, to help you learn Rust by building a real-world applications

### Blogs/Newsletters

The following blogs cover an extensive range of topics related to Rust, and are great for keeping up with the high-level features and developments in the language.

- [This Week in Rust](https://this-week-in-rust.org/): Links to all the latest Rust news, blog posts, RFCs, etc.
- [Armin Ronacher's thoughts and writings](https://lucumr.pocoo.org/tags/rust/)
- [Steve Klabnik blog](https://steveklabnik.com/writing)
- [Niko Matsakis' blog](https://smallcultfollowing.com/babysteps/)
- [Shuttle blog](https://www.shuttle.rs/blog/tags/all): Great blog by the makers of a Rust infra-as-code framework

Have fun learning, and let's build more things in Rust! 🦀
