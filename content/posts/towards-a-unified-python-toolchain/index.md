+++
title = "uv: Towards a unified vision for Python tooling"
description = "An introduction to using uv to get the most out of Python project and dependency management, without worrying about virtual environments"
date = 2024-12-26
draft = false

[taxonomies]
tags = ["python", "rust", "tooling"]
categories = ["Tools"]

[extra]
toc = true
comment = true
+++

## A long-term Python user's perspective on getting productive with uv

As many readers of this blog will know, I've been coding in Python for what feels like forever, having written my first line of Python in 2009. Over the years, I've seen and used countless tools to manage dependencies, projects, virtual environments, packages and more, which are collectively known as the Python tooling ecosystem. It's clear that this ecosystem has always been fragmented, seemingly beyond hope. There are _so_ many tools that do similar things, many of which overlap in functionality, and it's not clear to even experienced users which one to use when collaborating with others on a project. Having worked with countless users who are just entering the Python ecosystem, I've seen first-hand how confusing this fragmented tool chain is for newcomers. Recall that Python has only grown in popularity over the years, and is now the most popular language [on GitHub](https://twitter.com/ashtom/status/1851345368984011220), so user-friendliness in the ecosystem is crucial for the language's long term success.

The arrival of [Astral](https://astral.sh/), and their tools: [ruff](https://docs.astral.sh/ruff/) and more recently, uv, has proven to be a breath of fresh air to the entire Python community. Early in 2024, uv's release blog post[^1] created quite a buzz across the Python community, but as I reflect on my own usage of uv over the course of this past year, I felt the need to write this post as a tribute to uv's success, and to describe how it's markedly improved my productivity when working with Python.

A lot of you reading this post are probably like me -- i.e., you are a _user_ of Python libraries and frameworks written by other people. In this blog post, I'll summarize a bit of history of prior tools that did each did their individual jobs well, but uv is now changing the game by unifying the benefits of these tools into one.

{% tip(header="TL;DR") %}
As an AI Engineer, I typically use uv in two distinct "mindsets" during my development workflow.

- **Experimentation**: In a project's early stages, I use uv in combination with `ipykernel` to iterate on ideas and experiments (make and break quickly) directly in my editor (no Jupyter Notebooks).
- **Distribution**: Once the overall flow of the project is clearer, I use uv to execute code while seamlessly translating my initial working code to something that's production-ready
or for sharing with others.

If you want to see examples and a timing comparison, skip ahead to [this section](#usage) to see the commands I regularly use in my workflows.
{% end %}

Armin Ronacher, creator of Flask and the `rye` package manager for Python, explains the reasons for his excitement towards Astral's toolchain very well in his [blog post](https://lucumr.pocoo.org/2024/8/21/harvest-season/). With his vast experience in the Python ecosystem, he approaches uv from a library developer's perspective. This particular comment from his blog post stands out:

> _Domination is a goal because it means that most investment will go into one stack. For me uv is poised to be that tool. It's not quite there today yet for all cases, but it will be in no time, and now is the moment to step up as a community and start to start to rally around it_  
> -- Armin Ronacher[^2]

## Unraveling the messy Python tooling ecosystem

### The early days: `easy_install`

When I first began using Python in early 2009, the Python Package Index (PyPI) and `pip` weren't mature (`pip` had only just come out a few months before), and the only way to install packages was by either manually downloading the source and running `setup.py`, or use the `easy_install` utility. `easy_install` had some benefits, but came with a host of problems[^3]. It didn't have a way to _uninstall_ packages, and it didn't have a way to reproducibly and reliably install the right versions of libraries via a `requirements.txt` file. `pip` was created to solve these problems, and it did so admirably well.

`pip` was a huge step forward for the Python community, and it quickly became the de-facto package manager for Python. It was so good that it was eventually incorporated into the Python standard library, shipping with every default distribution of Python native to unix operating systems.

### The conda affliction

The package manager `conda`, was developed to solve dependency resolution and library compilation challenges in the scientific Python ecosystem, which made sense when key libraries like `numpy` and `scipy` simply didn't compile on certain operating systems via `pip`. However, in hindsight, as `pip` became more and more mature, it's clear that `conda` further fragmented the Python user community, because it introduced a new package and environment manager totally distinct from `pip`. It also used an entirely different repository for new, up and coming libraries (`conda-forge`), so every library maintainer who released a new package on PyPI to be installable via `pip` had to also release the same package on `conda-forge` to be installable via `conda`. If you've worked on large Python codebases, especially those that involve scientific computing, you likely faced the issue of `conda` environments and `pip` environments not playing well together, forcing your to either migrate your entire codebase into one ecosystem or the other.

I've always found it tragic that an entire generation of scientific Python developers came into the Python ecosystem circa 2010-2020 and only knew `conda` as their go-to Python dependency management tool. In all my time using Python over the last 15 years, I've found that "regular" software engineers or backend developers never wanted/used `conda`, preferring other alternatives, and so I've seen my fair share of teams that are a mix of scientific developers and software engineers who have to painstakingly stitch together their workflows because they simply cannot agree on a single toolchain.

{{ figure(src="reddit-python-packaging.webp" alt="<i>Why is Python Packaging such a mess?</i> <a href='https://www.reddit.com/r/Python/comments/cry1fn/rant_after_going_through_literal_hell_with/'>Reddit rant</a>" ) }}

### Why `poetry` rose to prominence

`poetry` is a tool that a lot of developers (especially library maintainers) swear by, and for good reason -- it's a great tool for managing environments and dependencies via locked dependency files. While `pip` relied on the `requirements.txt` file to manage dependencies, `poetry` introduced a much more structured `pyproject.toml` (and its associated `poetry.lock` file), which results in deterministic builds across different platforms and far fewer cases of failure in resolving dependencies.

However, there's no denying that `poetry` is _noticeably slow_. In most machine learning or scientific computing projects where you're dealing with a ton of dependencies, `poetry`'s dependency resolver can take a long time to resolve the dependency graph and get a project set up. The justification has always been that it at least "works", and reduces the burden of portability on library developers who are distributing packages to a large number of users across a variety of platforms. I've seen a lot of mature projects in the Python ecosystem adopt `poetry` for these reasons, and it's more or less become the de-facto standard package manager for Python projects involving larger teams where collaboration is of essence.

### A host of other tools

I won't go into the details of a whole host of other tools that address specific painpoints in Python tooling, but each one of them such as `pyenv`, `pipx`, `pipenv`, `pip-tools`, `pixi` and many others, tried to bring their own perspective to the table, to solve everything from dependency management, from managing Python versions to creating virtual environments. The `pipx` documentation has an [entire page](https://pipx.pypa.io/stable/comparisons/) to just list the differences between theirs and a ton of other tools. Mind-numbing stuff, to say the least.

## uv: The solution to *all* these problems

Since its launch in February 2024, [uv](https://github.com/astral-sh/uv) has exploded in popularity, and many large Python projects have completely migrated to it in their CI/CD as well as local build/execution pipelines. Below, I've summarized a few key reasons why I think uv has become this much of an unstoppable force in such a short time since its launch:

- **Performance**: uv is _fast_. It's ~**10x-100x** faster than `pip` and `poetry` in realistic environments, partly because its dependency resolver is written in Rust 🦀, but also because it implements a lot of smart caching and optimizations that are laser-focused on providing the best performance possible.
See [this video](https://www.youtube.com/watch?v=gSKTfG1GXYQ) by Astral's CEO Charlie Marsh for a deep dive into some of these optimizations.
- **Usability**: uv is _easy to use_. It's a command line tool that's easy to install, and it's very easy to set up a new project and get started with it, because it's also so well-documented.
- **Learned lessons from Rust 🦀 packaging**: Just like the Rust programming language has the Cargo ecosystem for a single unified toolchain, uv brings a lot of the best practices that Rust developers have come to appreciate, into Python.
- **Learned lessons from Python 🐍 packaging**: Charlie Marsh, Astral's CEO, has been open about how he and the core team that developed uv spent a lot of time studying the history of the Python tooling ecosystem (including what worked, and what didn't), and worked closely with the creator of `rye`, Armin Ronacher -- so this is a tool that's been built with a lot of prior thought and consideration put into it.
- **Engagement with the Python community**: A very important aspect of Charlie's and Astral's approach is _humility_, by not pushing their solution on the community and continually engaging with long-term users of other tools in the Python ecosystem. And because the project is completely open source, we get [direct insights](https://github.com/astral-sh/rye/discussions/1342) into how they're learning from the community.

{{ twitter(url="https://twitter.com/charliermarsh/status/1825958674239803515" class="twitter") }}

---

## Usage

Alright, with that bit of background out of the way, I hope it's clear why so many Python developers other than myself are just as excited about uv as I am.

### What makes a good Python project setup?

Let's first start by asking ourselves what the requirements are for a good Python project setup.

- **Python version management**: How do you manage the Python version for your project? How do you ensure that the user executing your code is using the same Python version as you intended?
- **Dependency management**: How do you manage dependencies for your project? How do you ensure that the dependencies are locked and that you can reproduce the environment on another machine?
- **Virtual environments**: How do you manage virtual environments for your project? How do you ensure that the virtual environment is isolated from the system Python installation?
- **Interactive code execution**: Some times, you may want to run exploratory code to test out ideas without needing a full-fledged Python project. How do you link your interactive code execution tool (e.g., Jupyter notebook) to your project's dependencies?
- **Project packaging**: How do you package your project for distribution? How do you ensure that the package is built correctly and that it can be installed on another machine?
- **Ease of use**: How do you ensure that the tools and workflow you applied to your project is easy enough for you, yourself, and then to train others who are new to the project?

As you go read this post and through the uv [docs](https://docs.astral.sh/uv/), you'll see that it addresses all of these requirements (and more), while being a one-stop-shop where
you may have earlier resorted to multiple other tools.

### Create project

The following steps assumes that you've [installed uv](https://docs.astral.sh/uv/getting-started/installation/) globally on your machine.
To begin, let's set up a new uv project.

```bash
uv init uv-demo
cd uv-demo
```

This will create a new directory called `uv-demo` and initialize a new uv project with some basic files in it. The directory structure should look like this:

```bash
uv-demo/
├── .python-version
├── .gitignore
├── pyproject.toml
├── hello.py
└── README.md
```

The `hello.py` file is a simple Python script that prints "Hello, World!" to the console. You can either delete this file or leave it in there, but it's a good starting point
to begin writing some code. A `.python-version` file is created to specify the Python version that was used for this project.
The `pyproject.toml` file is the main configuration file for the project, and it's where you can specify the dependencies for your project -- initially, it's empty.

### Set up environment

Once of the best things about uv is that you don't need to worry about virtual environments. uv will automatically create a virtual environment for your project, while
also ensuring that code that it executes in the current project directory is executed from within this virtual environment -- no more worrying about
whether you're in the right virtual environment or not!

For a demo of how fast uv is, let's install the following dependencies:

{% codeblock(name="requirements.txt") %}
```bash
kuzu==0.7.1
lancedb==0.17.0
llama-index==0.12.8
llama-index-llms-openai==0.3.12
llama-index-embeddings-openai==0.3.1
llama-index-graph-stores-kuzu==0.6.0
llama-index-vector-stores-lancedb==0.3.0
numpy==2.2.1
polars==1.18.0
pyarrow==18.1.0
python-dotenv==1.0.1
```
{% end %}

These requirements contain a non-trivial set of dependencies if, say, you want to build a Graph RAG system using [Kùzu](https://kuzudb.com/) as the graph store,
[LanceDB](https://lancedb.com/) as the vector store, and LlamaIndex as
the orchestration framework. These tools/frameworks themselves depend on libraries like `pyarrow`, `numpy`, `polars`, etc., and there needs to be a way to ensure that
all of these dependencies are installed a) reproducibly and portably across different machines, and b) without any conflicts in the dependency graph
that brings together all of these libraries in a working project.

### Performance comparison

All tests below are run on my 2023 M3 Macbook Pro.
We will install these dependencies using three different tools: `pip`, `poetry` and `uv`, and compare the
performance of each when starting "cold", i.e., with a clean cache. Let's first purge the cache for all the tools on our local machine:

```bash
uv clean cache
pip cache purge
poetry cache clear --all .
```

#### `pip`

Let's first run the following three commands to install the dependencies via `pip`:

```bash
# Create a new virtual environment
python -m venv .venv
# Activate the virtual environment
source .venv/bin/activate
# Time the execution of the pip install command
time pip install -r requirements.txt
```

Installing the dependencies via `pip` took 18.3 seconds. Note that when using `pip`, you needed to
manually keep track of the local `.venv` directory, and activate it so that the dependencies are
installed in the correct environment. It's all too easy to forget to activate the virtual environment,
which leads to the dependencies being installed in the system Python environment instead (not at all
a good practice!).

#### `poetry`

Let's remove the virtual environment via `rm -rf .venv` and run the same command again,
but now let's install the dependencies via `poetry`:

```bash
# Create a new virtual environment and activate it
poetry init
# Accept the default values and then enter the poetry shell
poetry shell
# Install the dependencies
time poetry install
```

Installing via `poetry` took 6.3 seconds. Again, note how the steps are multifold -- you needed to
initialize a new poetry project, enter the poetry shell, and then install the dependencies. To reuse
the environment created, you have to enter the poetry shell every time.

To clean things up for the next step, let's remove the virtual environment via `poetry env remove uv-demo-_xPlEE72-py3.12`,
and delete the `poetry.lock` and `pyproject.toml` files.

#### `uv`

Let's now install the same dependencies using uv:

```bash
# Create an empty pyproject.toml file and initialize a uv project
uv init
# Install the dependencies
time uv add -r requirements.txt
```

Installing the dependencies via uv took just 2.3 seconds! Upon completion of this command,
we now have a `.venv` directory in our project directory, which contains the virtual environment for this project.
We do not need to worry about its contents, or about activating it, as we will see below. uv manages this
directory and its usage entirely for us.

The ease of use is already evident here -- we simply run `uv add -r requirements.txt` and uv handles the rest.
There's also no need to keep track of activating the virtual environment. Indeed, you may as well not even
know that the `.venv` directory it created exists.

To summarize, here's a timing table for this simple dependency resolution task, done
via a cold cache using each of the three tools:

Tool | Timing
--- | ---
`pip` | 18.3 sec
`poetry` | 6.3 sec
`uv` | 2.3 sec

uv was **8x faster** than `pip` and **3x faster** than `poetry` in this relatively simple dependency resolution task.
In a realistic scenario, you're likely dealing with a lot more dependencies that are more complex, and the performance
gap between uv and the other tools is even more pronounced.

### Interactive code execution

In the beginnings of a project, I typically experiment with ideas and iterate quickly, during which I often rapidly update or modify my dependencies. uv is
_great_ for this, because it allows me to seamlessly link my interactive code execution tool (e.g., the Cursor IDE) to my project's dependencies.
Modern IDEs like VS Code and Cursor have great support for interactive Python code execution because they embed `ipykernel` the core utility
that powers Jupyter Notebooks, but they make the execution workflow with virtual environments far more seamless than using Jupyter notebooks.
It's amazing how easily a uv-managed virtual environment can be linked to an interactive code session in VS Code or Cursor.

uv has the concept of [dependency groups](https://docs.astral.sh/uv/concepts/projects/dependencies/#dependency-groups), which allows you to 
clearly separate the development dependencies from the project's core dependencies. Because `ipykernel` is something only I may use on
my local machine -- my collaborators may prefer to use their own tools to rerun my code -- I normally install `ipykernel` in the `dev` dependency group.
When executing code interactively in Cursor, I simply type in `Shift` + `Enter` to run the code, and Cursor will automatically
link the `ipython` kernel to the local virtual environment directory, `.venv`, that uv created for us when we ran `uv add -r requirements.txt`.

```bash
# We ran this command earlier to install dependencies from requirements.txt
uv add -r requirements.txt
# Add ipykernel to the dev dependency group
# to be able to interactively execute code in Cursor
uv add --dev ipykernel
```

See below for a demo of how simple it is to use a local uv-managed virtual environment to interactively execute code in Cursor or VS Code.

{{ figure(src="uv-interactive-demo.gif" ) }}

The above approach is in stark contrast to Jupyter Lab or Jupyter notebooks, which typically require you to install a
custom `ipython` kernel that's stored and managed separately from the project's virtual environment that may
have been created via `pip` or `poetry`. This leads to a disconnect between the environment on your machine and the
environment that someone else may use on their machine, because it required them to perform a bunch of extra steps
to ensure that the kernel is installed and linked to the correct virtual environment. uv solves this problem
by allowing you to use the **same** tool and virtual environment for both interactive code execution _and_ command line execution.

### Command line execution

The `.venv` directory in which uv manages the virtual environment is created when you run `uv add` or `uv sync` (which looks for a local `pyproject.toml` file and creates
a `.venv` directory that holds the virtual environment, if it doesn't already exist). If you're using uv in a CI/CD pipeline, you
can run `uv sync` to ensure that the virtual environment is created at the location where the git repository is cloned.

```bash
❯ uv sync
Resolved 113 packages in 0.75ms
Audited 108 packages in 0.09ms
```

Let's run the same code as shown in the interactive execution section above, but this time, we'll run the Python script via the command line.
Here's the code we'll run:

{% codeblock(name="hello.py") %}
```python
import polars as pl

df = pl.DataFrame(
    {
        "a": [1, 2, 3],
        "b": [4, 5, 6],
    }
)

print("Hello from uv-demo!")
```
{% end %}

We can run the `hello.py` script using `uv run` as follows:

```bash
❯ uv run hello.py
Hello from uv-demo!
```

Although it may not be immediately obvious, this very simple command does the following under the hood:

1. Installs Python if it's not already installed
2. Creates and activates the virtual environment
3. Installs the dependencies
4. Runs the code

This sequence of steps is very powerful, as it simplifies the thought process of the developer while also
ensuring that the code is executed in a consistent environment across all platforms. To ensure this, uv
also creates a `uv.lock` file that locks the exact versions of the dependencies that were used to run the code,
which is similar a `poetry.lock` file, but is created much faster.

As a developer, all I need to do is focus on writing good code that translates my thought process into something that can be executed,
and uv will handle the rest!

### Useful commands

In summary, here are some of the most useful commands that I more frequently use when working with uv:

- `uv init`: Initialize a new uv project
- `uv add <dependency>`: Add a specific dependency to the project
- `uv add -r <requirements.txt>`: Add dependencies from a `requirements.txt` file (which is a common way to distribute dependencies in a Python project)
- `uv sync`: Sync the project dependencies with the existing `pyproject.toml` file (if it doesn't exist, simply run `uv init` to create it)
- `uv run myfile.py`: Run a Python file in the project (similar to `python myfile.py`)

### Tool usage

Another very useful feature that I've begun using more in my workflows is the `uv tool run` command, whose shorthand is `uvx`. This command
allows you to run the `ruff` formatter on your code via a similar workflow to `uv run`. The following command requires that `ruff` is also
installed globally on your machine, similar to uv.

```bash
uvx ruff check .
uvx ruff format *.py --line-length 100
```

This eliminates the need for older tools like `black`, `isort` and `flake8` that were necessary to format code in Python projects
(these tools were also far slower than `ruff`). Using uv in combination with `ruff` makes the entire process of writing clean,
well formatted code just so seamless and convenient!

## Takeaways

uv can obviously do a lot more than what I've shown here, and I encourage you to read the [docs](https://docs.astral.sh/uv/) to deep dive into its functionality.
Other benefits include managing multiple Python versions, executing standalone scripts without the need to create a virtual environment, and more.
This is only the beginning!

In my Python project workflows (both for experimentation and distribution of clean, reproducible code), uv has completely replaced the following tools:

- `pip`
- `pyenv`
- `poetry`
- `venv`
- `pipenv`

Depending on what background you're coming from, you may have other tools that you've used, that are also likely easily replaced by uv.

I've become very passionate about communicating these benefits of unification in the Python tooling ecosystem, which in my opinion, brings Python on par
with other languages like Rust, Ruby and Go, which have long been lauded for their excellent tooling ecosystems. I also believe that sometimes,
these thoughts are best communicated via live demos, so in light of that, I highly recommend watching the
the following video for a live coding tutorial that I did on this topic with [YourTechBud](https://www.youtube.com/@YourTechBudCodes).
Feel free to add your thoughts in the video's comments section!

{{ youtube(id="-J5SnWR4UXw", class="youtube") }}

If you've made it this far, thanks for reading! Please help spread the word about uv and begin trying it out for yourself.
Your future self will thank you for it, and there's no going back!

## Example code

The example code and `requirements.txt` file used in this post can be found [here](https://github.com/prrao87/uv-demo).
Feel free to clone the repo and compare uv's performance to other tools in your own projects!

---

[^1]: _uv: Unifying Python Packaging_, Astral's [blog](https://astral.sh/blog/uv-unified-python-packaging) post.

[^2]: _Rye and uv: August is Harvest Season for Python Packaging_, Armin Ronacher's [blog](https://lucumr.pocoo.org/2024/8/21/harvest-season/)

[^3]: _Why use `pip` over `easy_install`_?, [Stack Overflow](https://stackoverflow.com/questions/3220404/why-use-pip-over-easy-install) question from 2010.
