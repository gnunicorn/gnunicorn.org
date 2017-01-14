---
layout: post
title: Tech Creationist Canvas Editor
date: '2014-08-19T16:03:00.000Z'
category:
- Tools
- Tech Creationist Canvas
- business
image: /assets/posts/tech-creationist-canvas-editor-16fd32cfefd48d139280a2294735bb6f95d8db3092.png
redirects:
- /t/119
- /t/119/
- /t/119/1
- /t/119/tech-creationist-canvas-editor
- /t/119/tech-creationist-canvas-editor/1
- /t/tech-creationist-canvas-editor
- /t/tech-creationist-canvas-editor/1
---

If you [open the editor](http://canvas.create-build-execute.com) without any further instructions it will pre-fill with the Tech Creationist Canvas (TCC) and open the source file in the editor on the bottom – yes, Tech Creationist Canvas is [self-hosting ;)](http://en.wikipedia.org/wiki/Self-hosting).

## One YAML to rule them all
The source format for the canvas is a [YAML](http://en.wikipedia.org/wiki/YAML) file. The Editor is a live-editor: every change you do will directly rerender the view. I recommend using the default TCC yaml or one of the other examples provided and get started by adapting it to fit your project.

On the top of the canvas you find a few meta information to fill in like:

{% highlight yaml %}

# the name of the project
name: "Tech Creationist Canvas"

# Describe the project in a one-line elevator pitch
tagline: Evalutating tech ideas for increased, sustainable impact

# tech creationist canvas version
renderer: tcc

# versioning of the idea
version: 2.0

# when did you change this?
date: July 29th, 2014 – 11:33

{% endhighlight %}

Out of them only `renderer` has a meaning for the Canvas itself. It allows you to switch between this Tech Creationist Canvas (`tcc`),  the Lean Startup Canvas (`lsc`) and the Business Model Canvas (`bmc`) renderers. Each one comes with their own set of keys to set in a different format. They aren't compatible with each other and can't be automatically converted. If you want to switch to a different renderer, I recommend loading an example with such a renderer and take it from there. **This document will only describe the TCC renderer in detail**.

`version` and `date` are just meta information to keep track for yourself.


## May the source be with you

As the Canvas can be described (and rendered from) this easy to handle source yaml plain text file, I recommend including it in the source code repository alongside  the Licence and Readme file. It is easily readable and structured in a straight forward manner. This also allows for easy updates and revision tracking through [git](http://git-scm.com/) alongside your normal source.

As you might have noticed the editor loads its own examples from Github. The same way it does it, you can also specify your repo in the URL to make the editor load your canvas and render it (for example from your Readme). The format is as follows:

`http://canvas.create-build-execute.com/#github=REPO:FILE`

Where `REPO` is the name of the repository on github (including the Organisation or Username) and `FILE` being the path (on the main branch) to the canvas file to load. If `FILE` is omitted, the editor tries to load the `tcc.yml` in the root per default.

Examples:

  
  `http://canvas.create-build-execute.com/#github=ligthyear/tech-creationist-canvas`

  `http://canvas.create-build-execute.com/#github=ligthyear/tech-creationist-canvas:examples/discourse.yml`
  

**Gist Support**

If you'd rather want the editor to load the Canvas source from a [Gist](https://gist.github.com/), you can use the `#gist=GISTID` feature in the URL

**Full-Data Support**

If you click the "permalink" feature, the editor will create a link including the entire yaml from the editor *as is* as a base64-string prefixed with `#data`. This is very helpful to quickly share your state with someone else or send around an experiment via email without having to store the source file somewhere. Just copy-paste that link and send it over and it will always and for ever render the same version of the canvas. Of course this could also be auto generated from the source file directly, if you prefer that.


## Embedding

This data feature is also used to allow for easy embedding. Just click on the `embed` link to receive an html-iframe-blob to copy-paste where ever you like. It will link and load to the current version from the editor. This is just a handy function you can also simply embed whatever URL you are looking at the moment. The editor is smart enough to detect that it is loaded by an iframe and won't show the editor functionality. That's it.

## Collaboration

The editor is [open source and on Github](https://github.com/ligthyear/tech-creationist-canvas). It is licensed under BSD. Please feel free to open Issue or Send Pull-Requests. And of course share your feedback, tips and tricks or how you are using it either here in the comments or [directly via email](mailto:ben[at]create-build-execute[dot]com). Any feedback highly appreciated!


---

**Read Next**

 - [Introducing the Tech Creationist Canvas](/2014/08/19/introducing-the-tech-creationist-canvas/)
 - [The Principles of The Tech Creationist Canvas](/2014/08/19/tech-creationist-canvas-the-principles/)
 - [The Tech Creationist Canvas explained](/2014/08/19/tech-creationist-canvas-explained/)

---
