+++
title = "What I've learned about open source software, visualized in graphs"
description = "Analyzing the network effects of open source communities and what it means at an individual level"
date = 2023-12-06
draft = true

[taxonomies]
tags = ["general"]
categories = ["General"]

[extra]
lang = "en"
toc = true
comment = true
+++

## An individual can have an impact on large-scale networks

Anyone who's ever worked in industry has been told about the importance of networking. The idea is that the more people you know, the more opportunities you'll have. However, for the majority of people, this is a chicken-and-egg problem: how do you get to work on interesting ideas/projects if you don't know the people working on them? And how do you get to know people working on interesting projects if you don't know what projects exist in the first place?

It's well known that open source software (OSS) communities are a kind of large-scale social network. In this post, I want to cover how you, an individual, can leverage the vast OSS network to your advantage, while also actively contributing back to the network, and toward the open sharing of knowledge.

Because I'm a visual thinker, I'll be using **graphs** to illustrate my points.

{% warning() %}

The term "graph" used in this post is from graph theory. A graph is a form of structured data representation, in which a set of nodes (or vertices) that represent a concept or entity in the real world are connected by edges (or links) that represent the relationship between them.

{% end %}

## Part 1: My views on OSS communities

The term "open source software" can mean a lot of things to a lot of people. It's a lot more than just a license to use software for free.

To me, the outstanding feature of open source is that it's a form of [publicly visible](https://github.com/apache/arrow-datafusion/issues/6782) collaboration between people who share a common interest in a particular area, whether it's a programming language or a specific domain of knowledge. This work can take on many forms, from filing a simple issue on a public repository, to improving documentation, to collaborating on a full-blown research paper with people distributed all over the globe.

Contrary to what your mind may tell you, you **do not** need to be an "expert" to be able to contribute. In a healthy open source ecosystem, *everybody* is on their own continuous learning journey -- you're just joining the train at a point in time. The next few subsections will highlight this, and more.

### I. Open source is an intellectual social network

Just like most of us keep some form of social media presence to remain in touch with our friends and family, open source platforms and forums are a means to find people who share your intellectual interests. The reward for inserting yourself into these networks, is the ability to communicate with people you may never meet in person, but who you can still learn from and grow in your skills, with zero expectations of reciprocity.

Imagine communicating with a single individual on one of these forums (e.g., GitHub), whose own networks then expand your reach out to a whole host of potential collaborators with similar interests. It takes *action* on our part to insert yourself into this global network of smart people.

{{ figure(src="oss-social-network.png", alt="The intellectual social network")}}

It's arguable that the network effects of OSS communities is, in large part, responsible for the explosion of innovation in the tech world over the past decade.

### II. Open source distribution channels are governed by social dynamics

> Open source is best viewed as a social network, with a distribution channel tagged onto it, and it isn't really anything to do with technology[^1].

[^1]: Jeremiah Lewin, founder of [prefect.io](https://docs.prefect.io/latest/), on the [Open source startup podcast](https://oss-startup-podcast.launchnotes.io/announcements/episode-10-prefect-open-source-data-flow-automation)

When deciding to join any social network, you're presented with the cyclic conundrum of "why should I join this platform if there's no interesting content on it yet?", and "why would I post any content on this platform if there's nobody on it yet?". With open source software, the same problem presents itself -- why would someone use a tool, framework or language if there isn't any content or community around it, and why would someone contribute to it if there isn't yet a substantial number of users?

The reality is that the best tools (purely on technical grounds) are not just "discovered" by an existing user base -- open source forums and communities (like [HackerNews](https://news.ycombinator.com/)) are how the best tools and frameworks get discovered, via *social dynamics*. Any company building on top of an OSS framework would be wise to invest heavily in building a community around their product, because that's how they'll get the most exposure to motivated individuals who advocate for their use (good luck to any proprietary tool vendors who are hoping for this).

{{ figure(src="oss-distribution-channel.png", alt="Open source as a distribution channel: all it takes is that one motivated user!")}}

### III. Open source thrives on positive feedback loops

Open source software can be broadly categorized into three buckets: **tools**, **frameworks** and **languages**. The beauty lies in how these categories are synergistic with one another via positive feedback loops that benefit the overall ecosystem. This is illustrated below at the individual and systemic levels.

#### At an individual level

Say you, an enterprising individual, wrote a framework that solves a problem you had. You then publish it on GitHub, and it gets picked up by a few people who find it useful. However, the framework you built itself depends on other frameworks and tools, which connect you to a much larger ecosystem.

The more people who use your framework, the more people will use the tools it depends on, and the overall effect at each level propagates upstream (because people tend to give back to the systems they benefit from). This is the essence of a positive feedback loop.

{{ figure(src="oss-feedback-loops-2.png", alt="OSS communities and feedback loops (1)")}}

#### At a systemic level

This idea can also be viewed from a systemic lens.

* *Languages* are the bedrock of the OSS ecosystem.
* *Frameworks* are built on top of languages, and are the primary means by which large-scale user-bases interact with the underlying language.
* *Tools* are built on top of languages and frameworks, allowing people to interact with combinations of underlying frameworks.

In the graph below, each node represents a *collection* of tools, frameworks and languages, and the networks of people that interact with them. The positive effects from the usage of a tool propagate backwards to the frameworks, and the same holds true for frameworks to languages. In the end, a thriving OSS ecosystem is a self-reinforcing network of positive feedback loops.

{{ figure(src="oss-feedback-loops-1.png", alt="OSS communities and feedback loops (2)")}}

### IV. Open source yields compounding returns

We've all seen/used apps like Uber, in which multiple parallel innovations from decades of prior research in networking, GPS and mobile computing, all came together at a point in time to create a new product that was greater than the sum of its parts. The same holds true for open source software and ecosystems.

Say you, a developer, spend time learning and contributing to multiple frameworks, written in your language of choice. Over time, you end up working with many people, utilizing and improving the same tools and frameworks, all with the broader vision toward solving real world problems. Because of the positive feedback loops described above, the compounding returns from your efforts are far greater than the sum of those from the individual components. This is what allows a project's complexity to grow with time, while delivering more and more value.

{{ figure(src="oss-compounding.png", alt="Compounding returns from open source")}}

The [Hugging Face Hub](https://huggingface.co/docs/hub/index) in the machine learning world is a great example of such compounding in action, where the entire ecosystem has moved forward while making knowledge and tools more broadly available to a large user base.

### V. Open source communities ensure knowledge redistribution

Tying back to the previous point, the compounding returns enabled by open source communities are only possible because of knowledge redistribution. This is illustrated below.

A tool, framework or language doesn't *start off* as popular. Once passionate communities advocating for their use appear, they inspire novice users to learn more from existing, more established users. Unlike the ivory towers of academia where you need prior accomplishments to be taken seriously, the beauty of open source software lies in its utilitarian nature -- developers of these systems are driven by pragmatism and the desire to solve real world problems, and, by and large, are more than happy to share their knowledge with anyone who's willing to learn.

The end result of such an ecosystem is that knowledge isn't hoarded by a select few, but is instead redistributed to the broader community, allowing greater scrutiny of the underlying components and the overall advancement of the field.

{{ figure(src="oss-knowledge-redistribution.png", alt="Knowledge redistribution is common in open source communities")}}

---

## Part 2: What I've gained through my OSS contributions

I'm writing this section to reinforce the idea that you **do not** need prior expertise in a tool, framework or language to begin contributing to open source. Just like many others, I've experienced my fair share of impostor syndrome, and my best experiences in the open source world came purely through my *inserting myself into the network* and letting events take their natural course. I'll highlight three specific examples below.

### I. A fruitful collaboration that might have never happened

What started with me filing a simple [GitHub issue](https://github.com/Pantalaymon/neuralcoref-for-french/issues/1) on an existing project [`neuralcoref-for-french`](https://github.com/Pantalaymon/neuralcoref-for-french) led toward a year-long, highly fruitful collaboration with a fellow developer who shared the same interests as me. We ended up working with a talented team that had funding for this kind of work and [published our findings](https://caiac.pubpub.org/pub/a7a2cy9j/release/1) on French NLP. This simple initial action led me to have to a ton of experiences I'd never have had if I'd stayed silent.

> Key takeway: **Don't hesitate to reach out to people.** The worst that can happen is that they don't respond. But the more likely outcome is that you'll end up having interesting conversations and learn a ton along the way, while broadening your network and the impact you have on others.

### II. Improvements to my own projects that might have never happened

How my using a database SDK, `meilisearch-python-sdk`, on an open source project led to its creator [finding me](https://github.com/prrao87/db-hub-fastapi/pull/12), and how the resulting PRs that we worked on together, improved both my own projects and the framework itself. I ended up going down deep rabbit holes related to [debugging concurrency issues](https://github.com/prrao87/db-hub-fastapi/pull/42) and [improving performance](https://github.com/prrao87/db-hub-fastapi/pull/15) using async and multi-threading in Python, and database I/O, all because I chose to put my code out there and run my experiments publicly.

> Key takeaway: **Don't be afraid to put your work out there, in a public repo.** The worst that can happen is that nobody notices it. However, if someone knowledgeable cares enough to spend a bit of time on it, you'll end up learning enough from the experience that you can improve your own work (and skills) and be able to take on even more complex projects in the future.

{% tip() %}
Side note: Always write good documentation, so that anyone who stumbles across your work in the future can easily reproduce your findings. When it comes to documentation, you get back in the future what you pay now! 🤓
{% end %}

### III. Deeper technical understanding that might never have happened

How my [blog post](../why-pydantic-v2-matters/) and [benchmark study](https://github.com/prrao87/pydantic-benchmarks) on a popular open source framework `pydantic`, went (mini-)viral with tens of thousands of engagements on LinkedIn in the first 24 hours of its posting, leading to its creator [reaching out to me](https://github.com/prrao87/pydantic-benchmarks/pull/1) with a PR that doubled the performance of my original code 🤯.

Studying these performance improvements gave me a far deeper understanding of the framework than I otherwise would have had by tinkering with it on my own. Furthermore, engaging in fruitful [discussions](https://github.com/pydantic/pydantic/discussions/6517) with some of the smartest people in the community was a large part of my learnings from this journey.

> Key takeaway: **Thorough understanding of a system comes by interacting and engaging with it.** Learning by simply reading others' work is fine, but the best way to learn is by *doing*, in a public manner wherever possible. Sometimes, by random chance, you may end up interacting with the people who built the system, or at least, with people who know it well enough to teach you a thing or two.

## Conclusions

I hope this post did a fair job of highlighting why I'm so passionate about supporting great open source tools, frameworks and languages. From a career development standpoint, working with proprietary software simply cannot yield the rich social and intellectual experiences that open source communities can provide.

To wrap up, I'll list some key points to remember regarding the larger open source ecosystem.

* The majority of people out there don't know you exist. OSS communities are a great way to become visible to people all over the world, but this takes consistent effort and time.

* The advancement of knowledge in the open source ecosystem relies on publicly visible contributions and collaboration, so do your best to put your work out there (whenever permissible). Use artificial data and anonymized results to showcase your projects, if required.

* Do not expect rewards for contributions. The best way to approach open source is to think of it as a way to give back to the community that you benefit from. The rewards will come in the form of new opportunities and connections that you'd never have had otherwise.

* Most importantly, OSS contributions can come in *many* forms, not just code. Documentation, issue filing, and even just sharing your experiences with others and spreading the word about tools you've enjoyed working with, as I have done in this post, are all a key part of building a thriving community that advances the state of the art.

As the title of this post suggests, it's not just the network that impacts the individual, the reverse is also equally true. Good luck on your journey!

{% important() %}

Learning from founders that base their companies on open source is hugely inspiring, so if you're looking for a great tool or framework to experiment with as your passion project, I'd highly recommend checking out the [Open Source Startup Podcast](https://oss-startup-podcast.launchnotes.io/).

{% end %}