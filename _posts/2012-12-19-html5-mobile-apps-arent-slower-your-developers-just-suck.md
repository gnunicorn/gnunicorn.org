---
layout: post
title: HTML5 mobile apps aren&rsquo;t slower - your developers just suck
date: '2012-12-19T00:00:00.000Z'
category:
- Tips & Tricks
- Best Practices
- tech
redirects:
- /t/68
- /t/68/
- /t/68/1
- /t/68/html5-mobile-apps-arent-slower-your-developers-just-suck
- /t/68/html5-mobile-apps-arent-slower-your-developers-just-suck/1
- /t/html5-mobile-apps-arent-slower-your-developers-just-suck
- /t/html5-mobile-apps-arent-slower-your-developers-just-suck/1
---



I already argued last year that [switching from HTML5 Mobiles Apps to native ones might actually not be an argument FOR going HTML5](/2011/12/04/why-6wunderkinder-rewritting-their-apps-as-native-is-an-argument-pro-cross-platform-frameworks) and not against it - from a business perspective. Now recently the discussion came up again, when Facebook rewrote their mobile apps from HTML5 into native and [Mark Zuckerberg stating that HTML5 wasn't there yet](http://techcrunch.com/2012/09/11/mark-zuckerberg-our-biggest-mistake-with-mobile-was-betting-too-much-on-html5/). I was suspscisious about that remark from early on because their boost in performance, even with the embedded-safari lack seemed very unlikely. Now Sencha Touch - an HTML5-Mobile-Framework - [released their rewrite of the _new Facebook_-App as HTML5](http://www.sencha.com/blog/the-making-of-fastbook-an-html5-love-story), which isn't only faster than the HTML5-App facebook used to have but is even more responsive than their current native Apps. Oups.

But how can that be? Well, they only say it in a roundabout way, but it is because their developers simply sucked. The biggest bunch of performance improvements that came with rewriting it as a Native up came from the fact that they simply do smart things now. Instead of querying for full-html-parts of the news-stream every time, they now request data and render it on the device. Leading to way smaller amount of data transfered and allowing the rendering to do smart things like only render the picture once the User is looking at it. We called this lazy-loading and it is a common practice to safe the amount of data requested from the servers as well as improve loading times.

So what the sencha guys are saying that actually, HTML5 in no comparision is slower than native on neither Android nor iPhone. Unless it is build by someone not actually that good. But while iPhone and Android native versions, because of restrictions in the SDK and the framework don't only foster but actuall enforce you to do it right (or it won't work at all), in HTML5 versions you can get away with lower quality. Meaning someone, who is doing a native apps usually, just simply makes it right because he has to, while for an HTML5-Developer it requires actual skills to make it right.

But bringing it back to my point that it is simply easier and quicker to get something out using HTML5. And what Sencha showed here, actually means: once you got tracktion with your HTML5-App, it is the classic question of doing it the right way which gives you performance boost and not resorting to native. So in the end, if you can easier and quicker develop an App in HTML5 for multiple platforms in one code-base and once it is successful, improve it to the point, where it is even faster and more responsive than a native rewrite, raises the question; [As long as you have a good developer](2011/06/10/so-what-language-are-you-programming-in/) - why go native at all?
