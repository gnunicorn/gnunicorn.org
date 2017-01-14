---
layout: post
title: Fully Flexible, highly extendable CMS with I18N support - introducing HTML5
  Static Pages
date: '2012-05-13T00:00:00.000Z'
category:
- Tips & Tricks
- tech
redirects:
- /t/47
- /t/47/
- /t/47/1
- /t/47/fully-flexible-highly-extendable-cms-with-i18n-support-introducing-html5-static-pages
- /t/47/fully-flexible-highly-extendable-cms-with-i18n-support-introducing-html5-static-pages/1
- /t/fully-flexible-highly-extendable-cms-with-i18n-support-introducing-html5-static-pages
- /t/fully-flexible-highly-extendable-cms-with-i18n-support-introducing-html5-static-pages/1
---



You need a highly functional, super flexible and easily extendable Content-Management-System? With localisation support for multi-language, multi-currency and even localised design? With support for up to N+1-Landing-Pages fully customizable in both content and design? With a highly extendable design and UI-interaction framework? Fully customizable, full localised, multi-Level-Menu-support? With a full revision history of every change and reveriability? With responsive designs and support for mobile? Hosted as cheap as possible, potentially for free? And all of it delivered with the speed of light and lowest latency possible? 

## Introducing HTML5 Static Pages

Seriously. You want full-flexibility in Design and Content? Then, why even bother the pain of a half-ass content management solutions like [plone](http://plone.org/) or [django-cms](https://www.django-cms.org/) with their crappy WYSIWIG editor that spits out bad HTML and you end up writing your own code any ways? And then those frameworks actually force a frame on you (hence the name) in which you have to stay and make everything you want to do impossible. And their performance sucks.

I really keep it with what [37Signals](http://37signals.com/svn/posts/3103-no-framework-needed) said earlier this year: for promotion pages with almost no dynamic content, there is no framework needed. Not even PHP. Sure I understand you might like to have some parts of the template rendered using includes, the menu dynamically change without you having to put it into every page yourself and an easy way to write content. But that is exactly what frameworks like [jekyll](http://jekyllbootstrap.com/) are made for: you write Markdown-Files and it renders them into nice and fully functional HTML5-Pages. And the whole layout lies to your finger tips. So you want a quick responsive Design? Stitch it together with [Twitter Bootstrap](http://twitter.github.com/bootstrap/) or any other [Compass](http://compass-style.org/) or any other CSS-Build-system and include it with a line.

And once compiled, they are hosted as simple HTML-Files and can be hosted by the simplest of all webservers (like a simple ``python -m SimpleHTTPServer`` with python2.7+) with a speed no caching service is ever gonna beat. Or use any of those Cloud Solutions which host html for super cheap, like [Amazon AWS Simple Storage Service (S3)](http://aws.amazon.com/s3/) or [Github Pages](http://pages.github.com) - the later is actually free and run on jekyll for you...

And for Version control: Static-Pages can be easily paired up with plenty of version control systems like [Mercurial](http://mercurial.selenic.com/) or [Git](http://git-scm.com/) (and the later then can directly be published with GH-Pages btw) - tools your developer and designer, who'll ask to do the actual changes to your page anyways at some point once your editor failed on you again, should be able to use already anyways.

Now you think, that is "unprofessional" because it isn't a fully blown Database-Driven Blablabla-Framework? Well, you know by generalising you always make the trade-off in performance and flexibility and perfection; for each individual case a more specific solution would outperforme any optmised general approach. And with performance I do not only mean in delivering content to the customer but also in time you spend to get content into the system. So, using HTML5-Pages actually often is the "professional" thing to do. It just needs to be done right in any case.
