---
layout: post

title: "Clippy: Linting as a Service"
date: 2016-03-05
image: http://www.bashy.io/assets/projects/clippy.png
categories:
  - tech
tags:
  - rust
  - clippy
  - bashy

---

_Original Posted on [Bashy.io](http://www.bashy.io/news/2016/03/05/clippy-linting-as-a-service/)_


When we [relaunched AreWeWebYet](http://www.bashy.io/news/2016/02/16/we-are-back-baby/) about three weeks ago, we made the bold claim that "you can build stuff" for the web with rust. Of course it didn't take long before people asked: but what for example? And we found there aren't really many well documented, real-world rust-web-projects out there for others to learn from – until today.

## Annoucing: Clippy Service

![Clippy Service](http://www.bashy.io/assets/projects/clippy.png)

[Clippy](https://crates.io/crates/clippy) is a rust compiler plugin, which automatically checks your code for common errors and bad style – commonly known as a linter. Unfortunately setting it up requires latest nightly features and integrating it isn't the easiest task at the moment – until now. Clippy Service offers to check out your github project and run clippy against it without any changes on your side. Once done, it awards you with a neat little badge you can put into your Readme and link to the extended logs:

[![Clippy Linting Result – Always live!](http://clippy.bashy.io/github/ligthyear/clippy-service/master/badge.svg)](http://clippy.bashy.io/github/ligthyear/clippy-service/master/log)


Of course it features a nice website on the front, allowing you to easily configure the badge and get the rendered output for a range of different text renderer (Markdown, AsciiDoc, etc.).

## Annotated Source Code

But more than just the real world service that it, Clippy is also meant as an extensive example and learning project. As such it ships – and advertises with on the homepage directly – a **completely annotated source code and [online viewer](http://clippy.bashy.io/docs/main.html)** – Whaaaaaat!!

Further more, [the documentation index](http://clippy.bashy.io/docs/main.html) also gives links to various interesting features and how they are implemented, as well as gives you one-click access to the Vagrant, Docker and Travis Configuration files to reuse and learn from. And there are some nice features indeed: Redis, Sub-Processing with Output Parsing, Unicode Emojis and in-Memory-Unzipping.

**Check it out: [clippy.bashy.io](http://clippy.bashy.io)**
