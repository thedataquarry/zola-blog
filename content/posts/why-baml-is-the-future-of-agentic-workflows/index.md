+++
title = "Why I'm excited about BAML and the future of agentic workflows"
description = "How BAML's innovations on structured generation with LLMs allows you to build more reliable agents, chatbots with RAG, and more."
date = 2025-01-28
draft = true

[taxonomies]
tags = ["baml", "agentic", "workflow", "orchestration", "rust"]
categories = ["Frameworks"]

[extra]
toc = true
comment = true
+++

## What is BAML and why should we care?

It's a new year, and judging by the frenzy of new AI frameworks over the last several months, I think it's fair to state that 2025 is going to largely be about agents and AI workflow orchestration. In 2024, a lot of well-known companies pivoted their strategies to enable better agentic workflows. Having dabbled in a variety of existing frameworks over the last several months, I always felt something was lacking. Either some functionality was missing, code felt too verbose or was too highly abstracted, or the API was too rigid. It always felt like wearing a straitjacket when trying to build even the most simple orchestrations and workflows.

For me, all this changed when I recently discovered [BAML](https://docs.boundaryml.com/home):

> BAML is a domain-specific language (DSL) to generate structured outputs from LLMs — with the best developer experience.
> With BAML you can build reliable Agents, Chatbots with RAG, extract data from PDFs, and more.

The part that says "best developer experience" is particularly important from my perspective, and is why I believe BAML is going to rapidly gain in popularity over the coming months. Although there have been new agentic and AI workflow orchestration frameworks coming out seemingly every month lately -- I think that BAML stands out from the rest of the pack. While these may seem like tall and potentially premature claims, I hope that this post at least highlights the key features of BAML that make it so very exciting to begin building with.

## What do we mean by "agents"?

Before getting into BAML and its role in the agentic and AI workflow orchestration ecosystem, it's worth describing what we mean when we use the terms "agent" or "agentic workflows". I like the definition given by Hugging Face [in their blog](https://huggingface.co/blog/smolagents#%F0%9F%A4%94-what-are-agents):

> AI Agents are programs where LLM outputs control the workflow.

Whether or not you agree with the definition above, there's no doubt that the term "agent" is commonly used synonymously with parts of the system that use LLMs to dictate the next action. The level of autonomy and the complexity of steps may vary in your case, but the core idea is that agentic workflows of today use LLMs at various stages for decision making and control flow.

The biggest problem that developers face when building LLM-based or agentic workflows is a lack of determinism in the outcome. A large part of this is because these workflows can break in various ways, and many issues tend to surface only once the system is in production, leading the developer back to the drawing board once issues are discovered.

## Limitations of structured generation with LLMs

JSON is an amazing format for passing data around via REST APIs, but when it comes to generating JSON, LLMs need special treatment to control their token generation in a way that ensures the JSON is valid. Also, not all LLMs support structured output generation. Because LLMs output text a token at a time and most of them do not deterministically output valid JSON or other stuctured formats, most frameworks approach this by explicitly reprompting the LLM to redo the task if it fails. Every now and then, the LLM might miss double quotes in the key/value of the JSON entry, resulting in the entire JSON failing to parse.

{{ figure(src="baml-1.png" alt="LLMs like outputting text. Sometimes, that text is not valid JSON") }}

## How BAML works under the hood

BAML is designed to address the problem of consistent and reliable structured generation with LLMs.

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

The example above shows how to use BAML to extract structured data from a resume. Models are defined using the `class` keyword. The `Experience` class contains the person's role, company, start and end dates of employment. `Resume` consists of a person's name, a list of their experiences (which are each instances of the `Experience` class) and a list of skills. Note how the schema in the prompt that BAML sends to the LLM isn't stringified JSON, but rather a more concise version without double quotes with added comments (provided by the `description` annotation in the BAML model). This helps the LLM better understand the task. Template strings that use the Jinja templating language provide a way to insert variables and expressions into the prompt.

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
      - Data and AI Engineer at RBC (2022 May - 2024 Dec)

      Skills:
      - Python
      - Rust
    "#
  }
}
```

Prompt transparency, immediate visual feedback and testability are all key features that make BAML a joy to work with.

> BAML brings a level of rigour normally seen in software engineering workflows to LLM orchestration and agentic workflows.

### Language agnosticity

BAML supports codegen for generating library code in several languages. As of writing this post, Python, TypeScript, and Ruby are supported but support for Go, Rust and many other languages are coming soon.
The benefit of storing the LLM-related logic in a custom DSL is that it allows developers in various parts of the organization (who may be using different languages) to all know to look in just one place -- `baml_src`,
implement the library code in their language of choice, and then use the generated code in `baml_client` to power their application.

Application code can call the necessary library code that BAML generates (whose prompts havealready been engineered and tested beforehand). This means that over time, a much larger pool of developers can experience the same productivity and feature richness in the LLM ecosystem that Python developers have come to expect.

### Lossless compression in token space

Using BAML and only BAML to interface with the LLM achieves a form of **lossless compression** of the token space.
When passing instructions to the prompt, you can allow BAML to generate
a more concise string representation of the instructions, rather than passing stringified JSON, which has a lot more useless
tokens like double quotes and curly braces. LLMs benefit from concise, high quality instructions, so BAML formulates the
prompt with the fewest tokens possible, which compresses the prompt without losing any information. The example below shows how a JSON schema that explains to
an LLM what fields are required and what fields aren't, can be translated into an equivalent BAML-generated schema that uses far fewer tokens.

{{ figure(src="baml-4.png" alt="JSON schema (left) vs. BAML schema (right)") }}

Upon receiving the LLM's output, the BAML parser then validates the output and handles any errors, while coercing the output to the appropriate types so that it can be safely handled by any downstream part of the workflow.

> BAML lets the LLM focus on what it does best: generating strings token by token. The LLM's attention window is largely occupied with understanding the task, not the schema.

## Results of the BAML approach

BAML offers all the above features in a remarkably developer friendly interface. There are plugins for VS Code and Cursor (with Neovim and PyCharm coming in the future), a [prompt playground](https://www.promptfiddle.com/) fiddle, a [chat interface](https://dashboard.boundaryml.com/chat) to write BAML models using an LLM, and more!

All of these innovations result in workflows that are faster, cheaper and more robust/reliable, making developers more productive and confident about their ability to deliver working apps in production.

How does BAML achieve this? It does so by letting LLMs do what they do best: generating strings, but while applying post facto fixes to the LLM's output via a rule-based parsing engine written in Rust. Instead of costly methods like re-prompting the LLM to fix minor issues in the outputs (which takes seconds), the BAML engine corrects the output in milliseconds, thus saving money on API calls. In addition, the upstream steps of creating higher quality, losslessly compressed prompts, less verbose prompts results in a very high quality of output with even smaller and open source LLMs. It's a win-win on multiple fronts.

## BAML's philosophy

BAML's fundamental mission since its inception has been to deliver the best possible developer experience when creating AI-powered solutions.

However, with AI applications, the _time-to-value_ proposition is hugely important for their larger adoption across enterprise organizations.

BAML addresses this as follows: LLMs are incredibly powerful and versatile at a variety of tasks, so it makes sense to use them early on at various points in the workflow, to quickly deliver testable, working PoCs that offer immediate value to stakeholders. Once the AI workflow is tested end to end and usage metrics can be gathered, developers can go back and write specialized tools/engines to replace LLMs with specialized tools where necessary. This enables larger and broader adoption of AI in the industry.

The key idea is to ship fast (thanks to BAML's rapid built/test cycle), and then come back later and optimize portions of the workflow for cost and performance. All this is enabled by a tool that's language agnostic, offers a great developer experience and is fully open source.

## But wait, didn't we say agents?

