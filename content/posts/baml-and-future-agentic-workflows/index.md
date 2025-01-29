+++
title = "Why I'm excited about BAML and the future of agentic workflows"
description = "How BAML's innovations on structured generation with LLMs allows you to build more reliable agents, chatbots with RAG, and more."
date = 2025-01-29
draft = false

[taxonomies]
tags = ["baml", "agentic", "workflow", "orchestration", "rust"]
categories = ["Tools"]

[extra]
toc = true
comment = true
+++

## What is BAML and why should we care?

It's a new year, and judging by the frenzy of new AI frameworks over the last several months, I think it's fair to state that 2025 is going to largely be about agents and AI workflow orchestration. In 2024, some well-known companies pivoted[^1],[^2] toward enabling better agentic workflows and observability and monitoring. Having dabbled in a variety of existing frameworks over the last several months, I always felt something was lacking. Either some functionality was missing, code felt too verbose or was too highly abstracted, or the API was too rigid. It always felt like wearing a straitjacket when trying to build even the most simple orchestrations and workflows.

For me, all this changed when I recently discovered [BAML](https://docs.boundaryml.com/home):

> BAML is a domain-specific language (DSL) to generate structured outputs from LLMs — with the best developer experience.
> With BAML you can build reliable Agents, Chatbots with RAG, extract data from PDFs, and more.

The part that says "best developer experience" is particularly important from my perspective, and is why I believe BAML is going to rapidly gain in popularity in 2025. Although there have been new agentic and AI workflow orchestration frameworks coming out seemingly every month lately -- I think that BAML stands out from the rest of the pack. While these may seem like tall and potentially premature claims, I hope that this post (at the very least) inspires some curiosity in you to begin building with it.

## What do we mean by "agents"?

Before getting into BAML and its role in the agentic and AI workflow orchestration ecosystem, it's worth describing what we mean when we use the terms "agent" or "agentic workflows". I like the definition given by Hugging Face [in their blog](https://huggingface.co/blog/smolagents#%F0%9F%A4%94-what-are-agents):

> AI Agents are programs where LLM outputs control the workflow.

Whether or not you agree with the definition above, there's no doubt that the term "agent" is commonly used synonymously with parts of the system that use LLMs to dictate the next action. The level of autonomy and the complexity of steps may vary in your case, but the core idea is that agentic workflows of today use LLMs at various stages for decision making and control flow.

The biggest problem that developers face when building LLM-based or agentic workflows is a lack of determinism in the outcome. Things can fail quite spectacularly in production, in large part because these workflows can break in many ways, and a lot of the time, issues tend to surface only once the system is already in production. BAML aims to address this by using a powerful, concise type system. More on this below.

## Limitations of structured generation with LLMs

JSON is an amazing format for passing data around via REST APIs. LLMs that are required to produce JSON need special treatment[^3] to control their token generation in a way that ensures the JSON is valid. Also, not all LLMs support structured output generation. Because LLMs output text a token at a time and most of them do not deterministically output valid JSON or other stuctured formats, most frameworks approach this by explicitly reprompting the LLM to redo the task if it fails. Every now and then, the LLM might miss double quotes in the key/value of the JSON entry, resulting in the entire JSON failing to parse.

{{ figure(src="baml-1.png" alt="LLMs are great at outputing strings, not JSON (unless you control generation)") }}

## How BAML works under the hood

The BAML toolchain is designed to address the problem of consistent and reliable structured generation with LLMs. It improves reliability in your AI workflows, while also being faster _and_ cheaper.

Upstream, BAML massively improves the prompt engineering experience by providing a type-safe domain-specific language (DSL), and generates compact prompts that are easy to write, read and test.

Downstream, the BAML parser obtains the LLM's output and applies _post facto_ fixes and validation to the output. Instead of relying on costly methods like re-prompting the LLM to fix minor issues in the outputs (which takes seconds), the BAML engine corrects the output in milliseconds, thus saving money on API calls while also allowing you to use smaller, cheaper models that can achieve largely the same outcome as bigger, more expensive models[^4].

{{ figure(src="baml-5.png" alt="BAML's upstream and downstream prompting and parsing workflow") }}

The BAML parser uses a technique called [Schema-Aligned Parsing](https://www.boundaryml.com/blog/schema-aligned-parsing#sap) (SAP) to fix the LLM's output via a rule-based engine, applying error correction techniques to get the output to conform to the known schema. Because the parser is written in Rust 🦀, it's able to apply the fixes in <10 ms, which is orders of magnitude faster (and cheaper) than re-prompting the LLM to fix its mistakes. It's an _engineering solution_ to a problem that shouldn't require the sophistication of a full-fledged LLM. Even if LLMs improve remarkably over the coming years, which they likely will, token generation per a strict format would still be more expensive than a rule-based parser, which is **free**.

## Strengths of BAML

From my perspective as someone who cares deeply about developer experience, BAML achieves a stellar design. This is due to a combination of the features described below.

### Right level of abstraction

All logic and prompts are transparent and customizable at **just one directory deep** and not hidden away from the developer in obscure files somewhere inside the code base. Prompts that are modified by an internal function but are not transparent to the developer are just bugs waiting to happen!

Here's how it would look once you initialize a new BAML project:

```
./my_project/
├── baml_src/
    ├── clients.baml
    ├── generators.baml
    └── models.baml
├── baml_client/
    ├── ...
    ├── ...
    └── ...
```

The file `clients.baml` contains the settings and the fallback logic for the LLMs used in the project. `generators.baml` contains the information on the codegen component in your client language of choice (e.g. Python, TypeScript, etc.). And finally, `models.baml` and any other files like it contain the data models, tools, prompts and tests used in the project.
A typical project might have many components that power different parts of the workflow, so any number of additional `.baml` files can be added to the `baml_src/` directory to separate things out cleanly.

{{ figure(src="baml-2.png" alt="The structure of a BAML project") }}

The `baml_client/` directory contains the generated code in the application language of choice for the project. This is the library code that you can use in your application (that leverages BAML under the hood to power the workflow).
Keeping the relevant files cleanly separated with full transparency and a high degree of customizability is why I think BAML hits upon the perfect level of abstraction.

### Tight iterative feedback loop

Prompts in BAML are immediately testable, as is common in software engineering workflows, with the LLM's inputs and outputs being very easy to review by a human over a range of test cases. This makes prompt engineering a breeze!

{{ figure(src="baml-3.png" alt="The model and functions in BAML (left) and the actual prompt sent to the LLM (right)") }}

The example above shows how to use BAML's expressive type system to extract structured data from a resume. Models are defined using the `class` keyword. The `Experience` class contains the person's role, company, start dates and end dates of employment. `Resume` consists of a person's name, a list of their experiences (which are each instances of the `Experience` class) and a list of skills. Note how the schema in the prompt that BAML sends to the LLM isn't stringified JSON, but rather a more concise version without double quotes. The comment lines beginning with `//` were provided by the `description` annotation when specifying the model in BAML. This helps the LLM better understand the task. Template strings that use the Jinja templating language provide a way to insert variables and expressions into the prompt.

The moment the developer writes a BAML function and defines the prompt, the resulting prompt generated by BAML _is immediately visible and testable in the editor_.

Here's an example test function in BAML for my resume -- it's super easy to read, write and run the test.
```rs
test my_resume {
  functions [ExtractResume]
  args {
    resume #"
      Prashanth Rao

      Experience:
      - AI Engineer at Kùzu Inc. (2024 Jan - Present)
      - AI and Data Engineer at RBC (2022 May - 2023 Dec)

      Skills:
      - Python
      - Rust
    "#
  }
}
```

And here is the result obtained via a call to `gpt-4o-mini`, which is perfectly valid JSON, with dates in the desired format as requested
in the prompt. All of this was done before a line of application code was written (right in the editor)!

```json
{
  "name": "Prashanth Rao",
  "experience": [
    {
      "role": "AI Engineer",
      "company": "Kùzu Inc.",
      "start_date": "2024-01",
      "end_date": "Unknown"
    },
    {
      "role": "AI and Data Engineer",
      "company": "RBC",
      "start_date": "2022-05",
      "end_date": "2023-12"
    }
  ],
  "skills": [
    "Python",
    "Rust"
  ]
}
```

Prompt transparency, immediate visual feedback and testability are all key features that make BAML a joy to work with.

{% tip() %}
BAML brings a level of rigour normally seen in software engineering workflows to LLM orchestration and agentic workflows.
{% end %}

### Language agnosticity

BAML supports codegen for generating library code in several languages. As of writing this post, Python, TypeScript, and Ruby are supported but support for Go, Rust, PHP and many other languages is coming soon.
The benefit of using a DSL to set the LLM-related logic is that it can be statically defined upfront while allowing developers in various parts of the organization (who may be using different languages) to know to look in just one place, `baml_src`,
implement the library code in their language of choice, and then use the generated code in `baml_client` to power their application.

{% tip() %}
Due to its design, BAML potentially allows a much larger pool of developers using different languages to experience the same productivity and feature richness in the LLM ecosystem that Python developers have come to expect.
{% end %}

### Lossless compression in token space

With BAML as the interface between the developer and the LLM, we achieve a form of **lossless compression** in token space.
When passing instructions to the prompt, you can allow BAML to generate
a more concise string representation of the instructions, rather than passing stringified JSON, which has a lot more extraneous
tokens like double quotes and curly braces. LLMs benefit from concise, high quality instructions in the prompt. BAML formulates the
prompt with the fewest tokens possible without losing any schema information.

{{ figure(src="baml-4.png" alt="JSON schema (left): 370 tokens vs. BAML-generated schema (right): 168 tokens") }}

As can be seen, the resume schema example above shows how a JSON schema that explains to
an LLM what fields are required and what fields aren't, is quite verbose and hard to understand (both for humans and LLMs).
This can be translated into an equivalent BAML-generated schema that uses far fewer tokens.

Upon receiving the LLM's output, the BAML parser then validates the output and handles any errors, while coercing the output to the appropriate types so that it can be safely handled by any downstream part of the workflow.

{% tip() %}
BAML lets the LLM focus on what it does best: generating strings token by token.
The LLM's attention window is largely occupied with understanding the task, not the schema.
{% end %}

## BAML's philosophy

Since its inception, BAML's goal has been to deliver the best possible developer experience when working with LLMs.
However, for larger adoption of AI solutions across enterprise organizations, the _time-to-value_ proposition is also hugely important.

BAML addresses this as follows:
- LLMs are incredibly powerful and versatile at a variety of tasks
- Use them early on at various points in the workflow
- Quickly deliver testable, working PoCs that offer immediate value to stakeholders
- Use software engineering best practices to ship to production, iteratively adding complexity as necessary
- Test the domain and business logic end-to-end and gather usage/success metrics
- Replace LLMs with specialized tools where necessary (avoiding premature optimization and over-engineering)

Developers can **ship fast** and iterate quickly (thanks to BAML's rapid built/test cycle). All this without restricting software engineers to specific languages like Python or TypeScript, while being 100% open source and transparent.

## But wait, didn't we say agents?

This isn't the end of the story. BAML is just getting started 🔥. It's all well and good to take a cursory glance at BAML and say, "*it doesn't support graph-based workflows yet*", or "*it doesn't have features A, B, C like frameworks X, Y, Z*". What's clear once you begin using BAML is that it has a **broad surface area**. What started off as a goal towards improving structured generation with LLMs has now evolved into a toolchain that will eventually support a variety of complex control flows with logic-based retries, and more.

With the history of the BAML team's relentless focus toward developer experience, it's not a stretch to expect some great UX features for building agentic workflows in the coming weeks and months.

## Conclusions

To summarize, BAML is particularly appealing to developers like me because it simultaneously innovates along multiple dimensions that improve productivity and make coding more fun:

- Right level of abstraction, making code and logic more transparent and accessible
- Tight iterative feedback loop, making prompt engineering a breeze
- Language agnosticity, allowing developers to use their language of choice
- Lossless compression in token space, meaning cheaper, faster workflows

For my part, I'm excited to join the journey and to use BAML in my own projects related to knowledge graph extraction, graph quality improvement, Graph RAG, agentic RAG, and more. I'll be exploring a lot more of these ideas using tools like Kùzu and LanceDB in conjunction with BAML in the coming months, so stay tuned! 🚀

## Learning resources

I highly recommend the following blog posts by the Boundary team building BAML to understand their vision, philosophy and implementation. Follow along their journey, as I'm doing!

- [Your prompts are using 4x more tokens than you need](https://www.boundaryml.com/blog/type-definition-prompting-baml)
- [Every way to structured outputs from LLMs](https://www.boundaryml.com/blog/structured-output-from-llms)
- [What is Schema-Aligned Parsing (SAP)?](https://www.boundaryml.com/blog/schema-aligned-parsing#sap)
- [Building a new programming language in 2024](https://www.boundaryml.com/blog/building-a-new-programming-language)
- [BAML documentation](https://docs.boundaryml.com/home/welcome)

The Data Exchange Podcast by Ben Lorica with Vaibhav Gupta, the founder of Boundary, is another great resource to get started learning about BAML. Let's get building!

{{ youtube(id="VBNNm4NyaPw", class="youtube") }}

---

[^1]: [LangChain's second birthday](https://blog.langchain.dev/langchain-second-birthday/), LangChain blog

[^2]: [Python data validator Pydantic launches model-agnostic AI agent development platform](https://venturebeat.com/programming-development/python-data-validator-pydantic-launches-model-agnostic-ai-agent-development-platform/), Venture Beat

[^3]: [Outlines](https://dottxt-ai.github.io/outlines/latest/welcome/), a Python library that allows you to use LLMs with structured generation

[^4]: [Beating OpenAI's structured outputs on cost, accuracy and speed](https://www.boundaryml.com/blog/sota-function-calling)
