---
layout: post
title: Passwords are so 2007
date: '2013-02-05T00:00:00.000Z'
category:
- Tips & Tricks
- Best Practices
- tech
redirects:
- /t/74
- /t/74/
- /t/74/1
- /t/74/passwords-are-so-2007
- /t/74/passwords-are-so-2007/1
- /t/passwords-are-so-2007
- /t/passwords-are-so-2007/1
---



The other day I was looking into how first start processes of nowadays Mobile-Apps look and could be optimized for a side-project I am supervising. And though there has been plenty talking about "first time experience optimisation" and such over the last couples of years, many still seem to be stuck in the year 2007 - the time before the iPhone/Smartphones. In the sense of that they still do not harvest the possibilities those bring but stick with old and obsolete principles. Passwords for example.

I was looking at one of those famous exercising apps for example. Aside from the fact that it requires another download of 225Mb (holy cow), that tampers with the first user experience, as no one would want to wait with their exercise, especially when already out of the door to get started. They also **required me to register an account and come up with a hopefully super un-guess-able password**, before I could continue. That is where I stop and got me thinking: **Why?**

Why does this App that focusses on providing me with help at exercising require me to register that way? Sure, on a website, the server would need such information to figure out what data actually to present, but that is because of the fact that **browsers are anonymous. Apps are not.** An App knows not also the platform but even the **Device-ID, Google- or Apple-ID** and specificly the **IMEI** without having to ask me ever. The App and as such the web-service behind **can easily identify this connection** and serve the data it requires. No need for neither my email-address, nor yet another password. Especially if you take into consideration, that this is not a high-security-app and probably never ever asks about these credentials again but safes them for me (because everything else would be a real bummer).

**But what about accessing the data from different devices?**, you might ask. Well, honestly, that point I'd first have validated as an actual problem with real data: **how many people do really switch their device for the usage of your app** I seriously doubt that is an actual issue for anyone ever. And you could simple always allow to sign-in again with your email-address and access all data that way. You would want that anyway for data migration purposes. If you asked for the email at the first start, you can see that there is already an account existing and offer to **send an email with a one-time-token-password to authenticate that app**, send it and save the IMEI as one that accesses that account. Done.

And don't even get me get started on Apps/Webservices which ask for setting up a password after I logged in via Facebook/Twitter. Guys, you have all my information after this, my email-address in particular. Even if I ever decide to not login via Facebook or Twitter again or you scratch that service, a simple usage of the "please email me a new password"-function will grant me authenticated access. Stop asking not only useless but actually stupid questions.

So, the next time, you are thinking about **optimising the first-time-user-experience don't hesitate rethinking already established procedures, methods and protocols as they might simply be obsolete on this device and prevent you from harvesting the potentials of the technology you are having at hand**.
