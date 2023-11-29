+++
title = "Why I chose Zola to build this site"
description = "My thoughts on blogging, static sites and how to deploy a Zola site via GitHub Pages"
date = 2023-11-29
draft = true

[taxonomies]
tags = ["general"]
categories = ["General"]

[extra]
lang = "en"
toc = true
comment = true
+++

## My trysts with blogging

I've been writing blogs on and off for a few years now. Like most people, I began my journey via a platform that had a low barrier to entry, namely, [Medium](https://medium.com/@tech_optimist), because I could just get to writing straight away, and I didn't have to worry about hosting, security, or any of the other things that come with running a website. I just wanted to write good technical content, and Medium was a good place to do that a few years ago.

Fast-forward to today, and my thoughts have evolved with the changing landscape, not to mention that my tech skills are *way* better than when I started off. 😅

### Why do I blog?

This is a good time to bring up some [@karpathy](https://x.com/karpathy/status/1325154823856033793?s=20) philosophy:

{{ figure(src="karpathy-philosophy.png" w=600 h=100) }}

Point #1 deserves a deeper exploration and a blog post of its own, but with regard to blogging, I think point #2 is very relevant, and in my view, leads directly to point #3.

It's hard to explain a concept to someone else unless you understand it at a deep enough level yourself. But, you can't really understand something well enough unless you've had to explain it to someone else. How does one break this virtuous cycle? I think the answer is in writing, or blogging, about the things you're learning.

How does blogging lead me to point #3 in the above Tweet? I write primarily for myself, and then when I look back at how far I've come, I end up comparing myself only to younger me, and never to others. Deep, indeed!

### Why not Medium?

When I was writing on Medium, I found myself constantly worrying about how to stand out in the ocean of existing content already on there. This is a natural consequence of writing on a platform that's designed to get you more views, and more clicks, and more engagement. I found that this was taking away from the joy of writing in itself.

We now live in a world of hype and self-promotion -- at least, for those of us in the A.I. and software engineering space. Platforms like Medium have been inundated with content that's just trying to grab the reader's attention, garner clicks, and include every buzz word in the modern tech lexicon. The content out there *just looks the damn same* as everything else.

Add to this the fact that half the writers on Medium are putting their articles behind paywalls, and you have a recipe for disaster. I'm not saying that writers shouldn't be paid for their work, but I think the paywall model that Medium is pushing is fundamentally flawed.

I'm not going to pay for a subscription to a platform that has a ton of content that I don't care about, just to read one article that I do care about. I'd rather pay the writer directly.

### Why not Substack?

Substack is fundamentally a newsletter site, not a blogging platform. It's designed to encourage the reader to subscribe to a writer's newsletter, and receive updates via email. Not only this, but there are also very limited features for things I care about to write a technical blog, such as code syntax highlighting, LaTeX support, and so on.

Oh, and just like Medium, every Substack blog *looks* the same, visually, which again makes it hard to discern the work of one individual writer from another, unless you subscribe to a ton of newsletters.

### Why I self-host

Below, I list some of my fundamental ideas about blogging and open knowledge-sharing.

* Knowledge that I gained from the public domain should, ideally, be shared back within the public domain -- I believe this is what advances human knowledge and understanding.
* Reading and writing content should, to a certain degree, be a personal experience and not a cookie-cutter one.
* Content should appear in a format that's easy to read, and not be cluttered with ads, pop-ups, and other distractions.
* To stand out in a sea of otherwise similar-looking content, you need the ability to customize aspects of your blog, such as the layout, the fonts, the colours, and so on.

I've been thinking about these ideas for a while now, and the only thing preventing me from going all in was the perceived difficulty of setting up a static site.

## Static vs. Dynamic sites

Anyone who's read anything on Medium will have noticed the dynamic nature of how it shows content. A dynamic website loads pages on the fly, typically relying on an underlying CMS (Content Management System) to store and retrieve content from a database. This is in contrast to a static website, which is just a collection of HTML, CSS, and JavaScript files that are served as-is to the user.

Static sites are faster to load and, in my view, provide a much smoother reading experience than dynamic websites. For blogs, where the primary medium of interest is text, using a static site seems like a no-brainer. However, it does require some understanding of the modern web, including HTML, CSS and JavaScript.

## Static Site Generators (SSGs)

Of course, nobody in their right mind would want to put in days of work to structure their website. This is where Static Site Generators come in.

> *If you’ve ever taken a good look at HTML, it should be clear that it isn’t fun to write it all by hand. Therefore, we invented the Static Site Generator (SSG): a program that takes an HTML template and some text in a more human-friendly form (usually Markdown), and mashes them together, yielding a servable HTML file[^1]*.

[^1]: Marcus R.A. Newman, [in the Prefetch blog](https://prefetch.eu/blog/2022/website-adventures-generators/)

There are a ton of SSGs out there, including Jekyll, Hugo and Pelican. The main benefit of using an SSG is that it allows you to write content in a human-friendly format, such as Markdown, and then generates the HTML files prior to deployment. All it then takes is a bit of tweaking the CSS and adding some minimal JavaScript to get the static site up and running.

## What is Zola?

Zola is:

> *A fast static site generator in a single binary with everything built-in.
> This tool and the template engine it is using were born from an intense dislike of the (insane) Golang template engine and therefore of Hugo that I was using before for 6+ sites.[^2]*

[^2]: Creator of Zola on their [GitHub repo](https://github.com/getzola/zola)

Hugo, the world's most popular SSG written in Golang, was my first choice until recently. However, once I discovered Zola, an equivalent SSG written in Rust, I was immediately hooked.

## The anatomy of a Zola site

A Zola site is just a collection of files and folders, with a specific structure.

```bash
.
├── config.toml
├── content
│   ├── about
│   │   └── _index.md
│   ├── posts
│   │   ├── post-1.md
│   │   ├── post-2.md
│   │   └── _index.md
│   └── projects
│   │   └── _index.md
│   └── _index.md
├── sass
│   └── main.scss
├── static
│   ├── font
│   │   └── custom_font.woff2
│   ├── img
│   │   └── icon.png
│   └── js
│       └── main.js
└── templates
    ├── base.html
    ├── index.html
    ├── page.html
    ├── post.html
    └── section.html
```

* `config.toml` is the main configuration file for your site. It contains information such as the site's title, description, the theme to use, and so on.
* `_index.md` at each directory is a special file that tells Zola to generate a page structure based on the declared front matter.
* `content` is the folder where you'll write your content. Each file in this folder is a page on your site.
  * Zola allows the concept of `section` and `page` to keep things clean and organized
* `sass` is the folder where you'll keep your CSS stylesheets. Zola uses the [Sass](https://sass-lang.com/) CSS preprocessor, which allows you to write CSS in a more human-friendly way.
* `static` is the folder where you'll keep all your static assets, such as images, fonts, and JavaScript files.
* `templates` is the folder where you'll keep your HTML templates, which can be customized via [shortcodes](https://www.getzola.org/documentation/content/shortcodes/)
  * Zola uses the [Tera](https://tera.netlify.app/) template engine, which allows you to generate HTML pages via a template language very much like Python's Jinja2.

## Importance of template engine

The main issue for me with Hugo is its underlying Go template engine, which made it immensely difficult to customize the look and feel of a theme I liked. I found myself spending hours, or even days, trying to figure out how to get my site to look the way I wanted it to, rather than actually writing content.

Although Hugo is hugely popular and there's tons of examples online, it was still proving too hard and unintuitive for to learn how templates work. Zola addressed all these concerns, and more. Within a few days of looking up the documentation on Zola's template engine, [Tera](https://keats.github.io/tera/), I was productive enough to get my site's customizations up and running.

## Steps to setting up a Zola site

If you've never set up and deployed a static site before, don't worry! It's not as hard as it sounds.

### Install Zola

Zola is a single binary, which means you can just download the binary for your platform. I'm on Mac, so I just used Homebrew to install Zola:

```bash
brew install zola
```

Navigate to an empty directory and type in `zola init` to initiate a base Zola site. This will create a `config.toml` file and a `content` folder.

### Define your requirements

For me, these were the main requirements I had in order to write productively. Spend a bit of time thinking about what matters to you, and define a list accordingly!

* LaTeX math equations
* Code syntax highlighting
* Customizable themes and layouts
* Table of contents
* Callout boxes (`info`, `warning`, etc.)
* Comments
* Privacy-focused site analytics

### Pick a base theme

Zola has an excellent list of starter [themes](https://www.getzola.org/themes/) that you can choose from. For this site, I chose the [Serene](https://github.com/isunjn/serene) theme and then customized it to my liking. Always pick a theme that gets you most of the way towards the requirements you defined above -- the styles, layouts and templates can then be customized to meet all the requirements later on.

### Understand the theme's structure

Spend a bit of time on the theme's documentation and examples to understand which files control the layout, styles and fonts. In my case, I found that the `templates` folder contained the HTML templates that had Tera code in them, and the `sass` folder contained the Sass CSS stylesheets. Once you tinker around with them for a bit, it becomes clear how to make modifications to the theme.

### Customize the theme

Decide on what features matter to you the most, and begin tinkering with the stylesheets, templates and configuration files to get the theme to look the way you want it to. This is a time-consuming step up front, but with Zola, I found this to be much more intuitive and fun than I did with Hugo. As I mentioned earlier, within a few days of tinkering with the Tera template files, I was able to be productive enough to get the first version site up and running on my local machine.

### Write content and test locally

Navigate to the `content` directory and begin writing a post in Markdown. You can use the `zola serve` command to start a local server that will serve your site on `localhost:1111`. You can then navigate to `localhost:1111` in your browser to see how your site looks. This is a great way to test your site locally before deploying it.

### Deploy your site with a "Hello World!" post

To get the site up and running on the web, you'll need to deploy it somewhere. I chose [GitHub Pages](https://pages.github.com/) because it's free and easy to set up right within the source repo. I also chose to use a custom domain name, which I purchased from [Namecheap](https://www.namecheap.com/), but this step is totally optional -- GitHub Pages allows you to host the page on `https://myusername.github.io` as well, for free.

## How to deploy?

### GitHub Pages

GitHub Pages is a free service that allows you to host static sites right from your GitHub repo. It's super easy to set up, and you can get started in a few minutes. The only caveat is that if you want to host a site without specifying a custom domain name, it has to have the same name as your GitHub username. For example, this site is based in the `thedataquarry` GitHub organization, so the site is available at the following URL: `https://thedataquarry.github.io`.

To deploy a GitHub pages site, it helps to have some basic familiarity with [GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions). GitHub Actions are a way to automate tasks within your GitHub repo, such as running tests, building and deploying your site, and so on.

#### Set up a GitHub Action

Create a directory called `.github/workflows` in the root of your repo, and create a file called `build.yml` in it. This file will contain the instructions for GitHub to build and deploy your site. This site uses the standard Zola GitHub Action, which is available [here](https://github.com/shalzz/zola-deploy-action).

```yaml
name: Deploy Zola site to Pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Build and deploy
        uses: shalzz/zola-deploy-action@v0.17.1
        env:
          PAGES_BRANCH: gh-pages
          TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Once a Markdown file has been written and pushed (along with any other changes to other files) to the `main` branch, this action triggers and builds the site on the `gh-pages` branch.

{% warning(header="Note") %}
To deploy the site, make sure that the following settings are set in your repo's `Settings` > `Pages` section:

* Build and Deployment:
  * Source: "Deploy from a branch"
  * Select the `gh-pages` branch
{% end %}

### Netlify or Vercel

An alternative to GitHub Pages, if you aren't comfortable with using GitHub actions, is to use a service like [Netlify](https://www.netlify.com/) or [Vercel](https://vercel.com/). Both of these services allow you to deploy static sites on a free tier, and directly connect to the source repo in which the code is based. I haven't used either of these services, but the Zola [docs](https://www.getzola.org/documentation/deployment/overview/) contain good instructions for them.

## Conclusions

I hope this post inspires you to write, and how to get started with setting up your own static site over which you have full control. I think that putting your thoughts and learnings down into a blog is a great way to push your boundaries, contribute to open source, and to make a mark on the world, no matter how small. Good luck on your journey! 🚀

## Code

The code for this website is available [here](https://github.com/thedataquarry/thedataquarry.github.io). If you like this template, free to use it and begin tinkering with your settings, and consider providing attribution to the [original author](https://github.com/isunjn/serene), just like I did. 😄