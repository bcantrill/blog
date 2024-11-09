---
title: "Software: Immaculate, fetid and grimy"
date: "2015-09-03"
---

Once, long ago, there was an engineer who broke the operating system particularly badly. Now, if you've implemented important software for any serious length of time, you've seriously screwed up at least once -- but this was notable for a few reasons. First, the change that the engineer committed was egregiously broken: the machine that served as our building's central NFS server wasn't even up for 24 hours running the change before the operating system crashed -- an outcome so bad that the commit was unceremoniously reverted (which we called a "backout"). Second, this wasn't the first time that the engineer had been backed out; being backed out was serious, and that this had happened before was disconcerting. But most notable of all: instead of taking personal responsibility for it, the engineer had the audacity to blame the subsystem that had been the subject of the change. Now on the one hand, this wasn't entirely wrong: the change had been complicated and the subsystem that was being modified was a bit of a mess -- and it was arguably a preexisting issue that had merely been exposed by the change. But on the other hand, it _was_ the change that exposed it: the subsystem might have been brittle with respect to such changes, but it had at least worked correctly prior to it. My conclusion was that the problem wasn't the change per se, but rather the engineer's decided lack of caution when modifying such a fragile subsystem. While the recklessness that had become a troubling pattern for this particular engineer, it seemed that there was a more abstract issue: how does one safely make changes to a large, complicated, mature software system?

Hoping to channel my frustration into something positive, I wrote up [an essay on the challenges of developing Solaris](https://web.archive.org/web/20100106051229/http://hub.opensolaris.org/bin/view/Community+Group+on/dev_solaris), and sent it out to everyone doing work on the operating system. The taxonomy it proposed turned out to be useful and embedded itself in our engineering culture -- but the essay itself remained private (it pre-dated [blogs.sun.com](http://www.technologyreview.com/article/403929/sun-microsystems-blog-heaven/) by several years). When we [opened the operating system](https://www.youtube.com/watch?v=-zRN7XLCRhc#t=21m54s) some years later, the essay was featured on [opensolaris.org](https://web.archive.org/web/20100125065252/http://hub.opensolaris.org/bin/view/Main/). But as that's obviously been ripped down, and because the taxonomy seems to hold as much as ever, I think it's worth reiterating; what follows is a polished (and lightly updated) version of the original essay.

In my experience, large software systems -- be they proprietary or open source -- have a complete range of software quality within their many subsystems.

## Immaculate

Some subsystems you find are beautiful works of engineering -- they are squeaky clean, well-designed and well-crafted. These subsystems are a joy to work in but (and here's the catch) by virtue of being well-designed and well-implemented, they generally don't need a whole lot of work. So you'll get to use them, appreciate them, and be inspired by them -- but you probably won't spend much time modifying them. (And because these subsystems are such a pleasure to work in, you may find that the engineer who originally did the work is still active in some capacity -- or that there is otherwise a long line of engineers eager to do any necessary work in such a rewarding environment.)

## Fetid

Other subsystems are cobbled-together piles of junk -- reeking garbage barges that have been around longer than anyone remembers, floating from one release to the next. These subsystems have little-to-no comments (or what comments they have are clearly wrong), are poorly designed, needlessly complex, badly implemented and virtually undebuggable. There are often parts that work by accident, and unused or little-used parts that simply never worked at all. They manage to survive for one or more of the following reasons:

- They work just well enough to not justify the cost of either rewriting them or switching them out
    
- The problem they solve isn't important enough to justify the cost of rewriting them or switching them out
    
- The problem they solve is so nasty that the cost of a rewrite or a switch is enormous -- or at least that it dwarfs the cost of ongoing maintenance
    

If you find yourself having to do work in one of these subsystems, you must exercise _extreme_ caution: you will need to write as many test cases as you can think of to beat the snot out of your modification, and you will need to perform extensive self-review. You can try asking around for assistance, but you'll quickly discover that no one is around who understands the subsystem. Your code reviewers probably won't be able to help much either -- maybe you'll find one or two people that have had the same misfortune that you find yourself experiencing, but it's more likely that you will have to explain most aspects of the subsystem to your reviewers. You may discover as you work in the subsystem that maintaining it is simply untenable -- and it may be time to consider rewriting the subsystem from scratch. (After all, most of the subsystems that are in the first category replaced subsystems that were in the second.) One should not come to this decision too quickly -- rewriting a subsystem from scratch is enormously difficult and time-consuming. Still, don't rule it out _a priori_.

Even if you decide not to rewrite such a subsystem, you should improve it while you're there in manners that don't introduce excessive risk. For example, if something took you a while to figure out, don't hesitate to add a block comment to explain your discoveries. And if it was a pain in the ass to debug, you should add the debugging support that you found lacking. This will make it slightly easier on the next engineer -- and it will make it easier on you when you need to debug your own modifications.

## Grimy

Most subsystems, however, don't actually fall neatly into either of these categories -- they are somewhere in the middle. That is, they have parts that are well thought-out, or design elements that are sound, but they are also littered with implicit intradependencies within the subsystem or implicit interdependencies with other subsystems. They may have debugging support, but perhaps it is incomplete or out of date. Perhaps the subsystem effectively met its original design goals, but it has been extended to solve a new problem in a way that has left it brittle or overly complex. Many of these subsystems have been fixed to the point that they work reliably -- but they are delicate and they must be modified with care.

The majority of work that you will do on existing code will be to subsystems in this last category. You must be _very cautious_ when making changes to these subsystems. Sometimes these subsystems have local experts, but many changes will go beyond their expertise. (After all, part of the problem with these subsystems is that they often weren't designed to accommodate the kind of change you might want to make.) You _must_ extensively test your change to the subsystem. Run your change in every environment you can get your hands on, and don't be be content that the software seems to basically work -- you must beat the hell out of it. Obviously, you should run any tests that might apply to the subsystem, but you must go further. Sometimes there is a stress test available that you may run, but this _**is not a substitute for writing your own tests**_. You should review your own changes extensively. If it's multithreaded, are you obeying all of the locking rules? (What _are_ the locking rules, anyway?) Are you building implicit new dependencies into the subsystem? Are you using interfaces in a new way that may present some new risk? Are the interfaces that the subsystem exports being changed in a way that violates an implicit assumption that one of the consumers was making? These are not questions with easy answers, and you'll find that it will often be grueling work just to gain confidence that you are not breaking or being broken by anything else.

If you think you're done, review your changes again. Then, print your changes out, take them to a place where you can concentrate, and review them yet again. And when you review your own code, review it not as someone who believes that the code is right, but as someone who is certain that the code is wrong: review the code as if written by an archrival who has dared you to find anything wrong with it. As you perform your self-review, look for novel angles from which to test your code. Then test and test and test.

It can all be summed up by asking yourself one question: have you reviewed and tested your change every way that you know how? **You should not even _contemplate_ pushing until your answer to this is an unequivocal YES.**. Remember: you are (or should be!) _always_ empowered as an engineer to take more time to test your work. This is true of every engineering team that I have ever or would ever work on, and it's what makes companies worth working for: engineers that are empowered to do the Right Thing.

## Production quality all the time

You should assume that once you push, the rest of the world will be running your code _in production_. If the software that you're developing matters, downtime induced by it will be painful and expensive. But if the software matters so much, who would be so far out of their mind as to run your changes so shortly after they integrate? Because software isn't (or shouldn't be) fruit that needs to ripen as it makes its way to market -- it should be correct when it's integrated. And if we don't demand production quality all the time, we are concerned that we will be gripped by the [Quality Death Spiral](https://web.archive.org/web/20091028095830/http://hub.opensolaris.org/bin/view/Community+Group+on/qual_death_spiral). The Quality Death Spiral is much more expensive than a handful of outages, so it's worth the risk -- but you must do your part by delivering **production quality all the time**.

Does this mean that you should contemplate ritual suicide if you introduce a serious bug? Of course not -- _everyone_ who has made enough modifications to delicate, critical subsystems has introduced a change that has induced expensive downtime somewhere. We _know_ that this will be so because writing system software is just so damned tricky and hard. Indeed, it is _because_ of this truism that you **_must demand of yourself_** that you not integrate a change until you are out of ideas of how to test it. Because you will one day introduce a bug of such subtlety that it will seem that _no one_ could have caught it.

And what do you do when that awful, black day arrives? Here's a quick coping manual from those of us who have been there:

- Don't pretend it didn't happen -- you screwed up, but your mother still loves you
    
- Don't minimize the problem, shrug it off or otherwise make light of it -- this is serious business, and your colleagues take it seriously
    
- If someone spent time debugging your bug, thank them
    
- If someone was inconvenienced by your bug, apologize to them
    
- Take responsibility for your bug -- don't bother to blame other subsystems, the inherent complexity of the software, your code reviewers, your testers, the community, etc.
    
- If it was caught before it was running in production, be thankful that a production user wasn't affected by it
    

But most importantly, you must ask yourself: **what could I have done differently?** If you honestly don't know, ask a fellow engineer to help you. We've all been there, and we want to make sure that you are able to learn from it. Once you have an answer, take solace in it; no matter how bad you feel for having introduced a problem, you can know that the experience has improved you as an engineer -- and that's the most anyone can ask for.
