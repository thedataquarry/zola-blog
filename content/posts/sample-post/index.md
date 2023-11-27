+++
title = "Sample post"
date = 2023-02-10

[taxonomies]
categories = ["General"]
tags = ["markdown"]

[extra]
lang = "en"
toc = true
comment = true
math = true
+++

This is a post showing all supported elements.

<!-- more -->

## Heading

As you can see in this post.

## Paragraph

Lorem ipsum dolor sit, amet consectetur adipisicing elit. Praesentium, nisi saepe dolor unde iusto dolore nam, vero optio consequuntur repudiandae et! Atque libero expedita laudantium cupiditate, sit explicabo sequi ipsa! 

Atque libero expedita laudantium cupiditate, sit explicabo sequi ipsa! 

## Emphasis

normal text, **bold text**, *ltalic text*, ***bold and ltalic text***

## Blockquotes

> Lorem ipsum dolor sit, amet consectetur adipisicing elit. Praesentium, nisi saepe dolor unde iusto dolore nam, vero optio consequuntur repudiandae et! Atque libero expedita laudantium cupiditate, sit explicabo sequi ipsa! 

> Lorem ipsum dolor sit, amet consectetur adipisicing elit. 
>> Praesentium, nisi saepe dolor unde iusto dolore nam, vero optio consequuntur repudiandae et! Atque libero expedita laudantium cupiditate, sit explicabo sequi ipsa! 

## List

unordered:

- First item

- Second item

- Third item

ordered:

1. First item

    1. Indented item

    2. Indented item

2. Second item

3. Third item


## Link

There is a [link](https://example.com) and [another link to example.com](https://example.com)

## Table

| Syntax      | Description | Test Text     |
| ---         |    ----     |           --- |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |

with alignment:

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |

## Footnote

Lorem ipsum dolor sit, amet[^1] words consectetur[^2] adipisicing elit.

## Strikethrough

~~The world is flat.~~ We now know that the world is round.

## Horizontal Rule

---

## Image

with caption:

{{ figure(src="./dataquarry.png", alt="Data quarry banner") }}


## Code

inline `code` and more `inline code`

block level code:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

```python
def plus_one(x):
    return x + 1 if x else None

five = 5
six = plus_one(five)
none = plus_one(None)
```

with file type:

{% codeblock(name="rust") %}
```rs
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```
{% end %}


## Callout

{% note() %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit.
{% end %}

{% important() %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit.
{% end %}

{% warning() %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit.
{% end %}

{% alert() %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit.
{% end %}

{% question() %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit.
{% end %}

{% tip() %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit.
{% end %}

with header:

{% note(header="Note") %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit. Praesentium, nisi saepe dolor unde iusto dolore nam, vero optio consequuntur repudiandae et! Atque libero expedita laudantium cupiditate, sit explicabo sequi ipsa! 
{% end %}

{% alert(header="BREAKING!") %}
Lorem ipsum dolor sit, amet consectetur adipisicing elit.
{% end %}


## Math (KaTex)

$$
f(x) = \int_{-\infty}^\infin \hat f(\xi) e^{2 \pi i \xi x}\ d\xi
$$


When $ a \ne 0 $, there are two solutions to $ (ax^2 + bx + c = 0) $ and they are 
$$ x = {-b \pm \sqrt{b^2-4ac} \over 2a} $$

The Cauchy-Schwarz Inequality

$$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$


## Comment (Giscus)

As you can see in this post.

---

[^1]: First footnote.

[^2]: Here's the second footnote.