---
title: "Hello Joyent!"
date: "2010-07-30"
---

As I mentioned in [my farewell to Sun](http://dtrace.org/blogs/bmc/2010/07/25/good-bye-sun/), I am excited by the future; as you may have seen, that future is [joining Joyent](http://joyeur.com/2010/07/28/bryan-is-now-a-joyeur/) as their VP of Engineering.

So, why Joyent? I have known Joyeurs like [Jason](http://twitter.com/jasonh), [Dave](http://twitter.com/davidpaulyoung), [Mark](http://twitter.com/mmayo) and [Ben](http://www.cuddletech.com/blog/) since back when the "cloud" was still just something that you drew up on a whiteboard as a placeholder for the in-between crap that someone else was going to build and operate. But what Joyent was doing _was_ very much what we now call cloud computing -- it was just that in describing Joyent in those pre-cloud days, I found it difficult to convey exactly why what they were doing was exciting (even though to me it clearly was). I found that my conversations with others about Joyent always ended up in the ditch of "virtual hosting", a label that grossly diminished the level of innovation that I saw in Joyent. Fortunately for my ability to explain the company, "cloud" became infused with much deeper meaning -- one that matched Joyent's vision for itself.

So Joyent was cloud before there was cloud, but so what? When I started to consider what was next for me, one of the problems that I kept coming back to was DTrace for the cloud. What does dynamic instrumentation look like in the cloud? How do you make data aggregation and retention scale across many nodes? How do you support the ad hoc capabilities that make DTrace so powerful? And how do you visualize the data that in a way that allows for those ad hoc queries to be visually phrased? To me, these are very interesting questions -- but looking around the industry, it didn't seem that too many of the cloud providers were really interested in tackling these problems. However, in a conversation at [my younger son](http://dtrace.org/blogs/bmc/alexander_morgan_gaffikin_cantrill)'s third birthday party with Joyeur (and friend) [Rod Boothby](http://twitter.com/rod11), it became clear that Joyent very much shared my enthusiasm for this problem -- and more importantly, that they had made the right architectural decisions to allow for solving it.

My conversation with Rod kicked off more conversations, and I quickly learned that this was not the Joyent that I had known -- that the company was going through a very important inflection point whereby they sought a leadership position in innovating in the cloud. To match this lofty rhetoric, the company has a very important proof point: the hiring of [Ryan Dahl](http://tinyclouds.org/), inventor and author of [node.js](http://nodejs.org/).

Before getting into the details of node.js, one should know that I am a JavaScript lover. (If you didn't already know this about me, you might be somewhat surprised by this -- and indeed, there was a time when such a confession would have to be whispered, if it could be said at all -- but times have changed, and I'm loud and proud!) My affection for the language came about over a number of years, and crescendoed at Fishworks when [I realized that I needed to rewrite our CLI in JavaScript](http://dtrace.org/blogs/bmc/2008/11/16/on-modalities-and-misadventures/). And while I'm not sure if I'm the only person or even the first to write JavaScript that was designed to be executed over a 9600 baud TTY, it sure as hell felt like I was a pioneer in some perverse way...

Given my history, I clearly have a natural predisposition towards server-side JavaScript -- but node.js is much more than that: its event driven model coupled with the implicitly single-threadedness of JavaScript constrains the programmer into a model that allows for highly scalable control logic, but only with sequential code. (For more on this, see [Ryan's recent Google tech talk](http://www.youtube.com/watch?v=F6k8lTrAE2g) -- though I have no idea what was meant when Ryan was introduced as "controversial".) This idea -- that one can (and should!) build a concurrent system out of sequential components -- is one that Jeff and I discussed this in [our ACM Queue article on real-world concurrency](http://queue.acm.org/detail.cfm?id=1454462):

> To make this concrete, in a typical MVC (model-view-controller) application, the view (typically implemented in environments such as JavaScript, PHP, or Flash) and the controller (typically implemented in environments such as J2EE or Ruby on Rails) can consist purely of sequential logic and still achieve high levels of concurrency, provided that the model (typically implemented in terms of a database) allows for parallelism. Given that most don’t write their own databases (and virtually no one writes their own operating systems), it is possible to build (and indeed, many have built) highly concurrent, highly scalable MVC systems without explicitly creating a single thread or acquiring a single lock; it is concurrency by architecture instead of by implementation.

But Ryan says all that much more concisely at 21:40 in the talk: "there's this great thing in Unix called 'processes.'" Amen! So node.js to me represents a confluence of many important ideas -- and it's clean, well-implemented, and just plain fun to work with.

While I am excited about node.js, it's more than just a great idea that's well executed -- it also represents Joyent's vision for itself as an innovator up and down the stack. One can view node.js as being to Joyent was Java was to Sun: transforming the company from one confined to a certain layer into a true systems company that innovates up and down the stack. Heady enough, but if anything this analogy understates the case: Joyent's development of node.js is not merely an outgrowth of an innovative culture, but also a reflection of a singular focus to deliver on the economies of scale that are the great promise of cloud computing.

Add it all up -- the history in the cloud space, the disposition to solving tough cloud problems that I want to solve like instrumentation and observability, and the exciting development of node.js -- and you have a company in Joyent that I believe could be the next great systems company and I'm deeply honored (and incredibly excited) to be a part of it!