---
layout: post
title: It&rsquo;s time for a change
date: '2012-03-09T00:00:00.000Z'
category:
- Thoughts
redirects:
- /t/42
- /t/42/
- /t/42/1
- /t/42/its-time-for-a-change
- /t/42/its-time-for-a-change/1
- /t/its-time-for-a-change
- /t/its-time-for-a-change/1
---




_Disclaimer: I published this Post originally over at [our Company Blog](http://blog.dieBeschleuniger.de) of [Club der Beschleuniger eG](http://www.dieBeschleuniger.de). You can find the original of this cross-post [here](blog.diebeschleuniger.de/2012/03/its-time-for-change-inside.html)._


Lately there is has been a vivid and fruitful discussion about the <a href="http://www.ted.com/talks/eli_pariser_beware_online_filter_bubbles.html">filter bubble</a> created by recommendation services all around is. The hyposis is that filtering data based on what we want to hear will stop exposing us to stuff we don't want to know but should. And the more this kind of filtering happens the less people care about stuff outside their circle which could have potentially a bad influence for society. While thinking about that problem I realised that recommendations - as of today - don't work in the long run anyways because they are too static.


The way recommendations algorithms work today is basically on top of an so called intrest-graph: The system looks at the stuff you liked using those a link of other people, who liked the same. Then it looks for stuff the other "who are very like you" liked and recommends this as something you probably like, too. And every time you tell the system another thing you liked or disliked it gets more accurate the next time. This is how it works from last.fm to netflix, from facebook to twitter. Smart system, don't you think?


But how come I - and with me so many others - stopped using last.fm? With more than one and a half years of historical data it should be able to predict the stuff I like pretty good, shouldn't it? So what happend? The algorithm didn't change. Neither did the data. But I did. Almost over night, I switched from listening hard rock/punk to trance (electronic). I realised I am way more productive that way. So when I wanted to listen to the new stuff, last.fm kept interrupting me with the wrong stuff - the stuff I didn't like anymore. And it was almost impossible to make it change. Because trying to fight two years of recorded history is like fighting wind mills. **The more data you have in the recommendation system the harder a transition to becomes**. As a result I quit last.fm.


This may sound rare and uncommon, but think about it for a second; **Everyone who has ever moved from one flat to another knows how liberating it feels to start from scratch**. And you'd also agree &nbsp;that there is at least one box you won't open ever. In there is mostly baggage that you don't need anymore; It could be old (hurtful) picture of your ex or a box of books and CDs you never were able to throw away. Similar to last.fm, **where the baggage was all this music I used to listen to**. And as it was impossible to make last.fm adopt to my changes, I had to move on as well.


Social Networks in comparison are still very young. But eventually they have to face the same problem. Someone I totally admired last year and all his posts can be very annoying to me today. And I am not only thinking about the break up - as the hardest of all relationship changes - I am talking about everyday life; **we and our relationships to others change constantly**. A bunch of people already face the problems like having their ex-girl friends liking the post where you are complaining about your wife. In relationships you have a lot of baggage - way more than your music history. But it is **not in the interest of Facebook** or Twitter to **offer a you clean slate**. And this is also true for the relationship to companies and other fan pages.


But you know me, I don't bitch about a problem without a at least looking for a solution: When we were working on the relaunch of <a href="http://www.happyyuppie.com/">happyYuppie.com</a>, we ran into a similar problem. The way stock forecasts work is called time-series-analysis. You look at the values in the past and try to find a mathematical model describing it in a way that allows you to predict the future values. While rewriting that algorithm, there was a huge collaps in the Solar-Panel-Market in Germany; all over the places their stocks fell. And though this was a common case of market cleansing, our models predicted that they'd get back to old strength soon. **Based on the old baggage** (the long-term history) and not taking into account the latest strong trends, **our prediction models were simply wrong**. **As recommendations algorithms are when there is a huge change or general transition.**


The way we solved this was by creating a series of models for different time periods: one for monthly data of the last three years, one of weekly data of the last year and another of daily data for the last three months. While doing this over a series of different data and time horizonts we've learnt one common thing: **the shorter the time span the more "trend aware" the forecast is**. What follows is mixing those three models to create one surprisingly precise model. The weight of each models depends highly on the&nbsp;volatility&nbsp;and some other parameters of the stock, which I can't disclose here. But essentially it means the model for solar panel companies are much more aware of the latest trends of losses happening.


Resulting in a system that creates forecasts for in a slowly fading weight the further you go back in history. You could apply these principles to recommendation systems easily; taking into account when it happened and put a **higher emphasis on the younger event **when there is an detectable change in taste, **the recommendation system would become much more change-aware and long-lasting**. On top it would also **allow the user to influence his&nbsp;personalized&nbsp;system&nbsp;easily** and **break out of his filter bubble by&nbsp;actively&nbsp;including something** that hasn't been there before. This would give him **immediate insight on that topic with the best content available** and slowly something the user is less interested in now fades out.
