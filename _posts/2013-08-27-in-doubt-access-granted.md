---
layout: post
title: 'In doubt: access granted'
date: '2013-08-27T00:00:00.000Z'
category:
- Tips & Tricks
- Best Practices
image: /assets/posts/in-doubt-access-granted-678c61dc164abde3b0ec82ee662d754a3a14b4421f.jpg
redirects:
- /t/99
- /t/99/
- /t/99/1
- /t/99/in-doubt-access-granted
- /t/99/in-doubt-access-granted/1
- /t/in-doubt-access-granted
- /t/in-doubt-access-granted/1
---

When doing Dev Ops or system administration there is an important default rule for keeping your system secured: **forbidden unless explicitly allowed**. Most firewalls work that way: no port is open for connection unless administrator explicit opened it. The reason for that being that you can't trust the source, you have to assume it someone that wants harm. In computer security the rule is "guilty until proven innocent". But that **shouldn't bubble up to become part of your customer communication**. Because having to ask for access for every little thing gets really annoying and destroys the user experience.

Yes, of course, there could be someone doing something harmful. Though the actual odds are that this is a legitimate customer just trying to use your App. But you make them re-enter the password because they switched from one subdomain to another. A password which by enforcing certain length and characters, you made sure they are unable to remember. Similar goes to having the mobile App check online, whether the users is out of their quota when taking a picture. And here comes the real big problem of the distrust against your customer: in an increasingly mobile-internet-ish world, internet connection also becomes much more flaky and unreliable. 

Meaning you aren't always able to check back with the server weather the user is allowed to do a certain action or still has quota. And then the most secure way of dealing with that situation is to deny access or prompt to upgrade. But that - from a product perspective - is by far the worst you can do. There is a customer, who wants to get a job done, most likely one they are totally legitimate to be doing and you deny that request for reasons outside of their control. You should always focus on the task at hand and get that job done, which - under an unreliable connection - is already hard enough. What does it matter to you if the accounts runs 1mb over their 300mb quota? The next time they have internet and they try again, they will be prompted to upgrade - which is soon enough. After all they were out of connection and therefore unable to upgrade before any ways. And by denying them what was rightful theirs, you drove them away. So design your App to always grant access and deny it later.

Btw. Soundcloud is doing a good job here: It accepts your upload no matter what. But if you ran out of quota because of the upload, you'll receive an email informing you that your oldest other sound is not accessible to the public anymore. And then you - as the customer - can deal with the problem. But they don't force you to upgrade while you are on a mobile with an already unreliable connection.

---

Thanks to [Benson Kua](https://www.flickr.com/photos/bensonkua/4886812019) for releasing the picture under CC 2.0.
