---
title: "Some prehistory"
date: "2004-06-14"
---

I am an engineer in Solaris Kernel Development here at Sun. I have spent many (well, eight) years at Sun, and I've done work in quite a few different kernel subsystems. Most recently, I -- along with Mike Shapiro and Adam Leventhal[^1] -- designed, implemented and shipped [DTrace](http://www.sun.com/bigadmin/content/dtrace), a new facility for dynamic instrumentation of production systems. As DTrace has been a nearly decade-long dream for me personally, much of this blog will focus on DTrace -- its history, its [architecture](http://www.sun.com/bigadmin/content/dtrace/dtrace_usenix.pdf), its use, its [budding community](http://forum.sun.com/forum.jsp?forum=211), its ongoing development and its futures.  
  
But first, some prehistory to let you know where I'm coming from. (And apologies in advance for the length.)  
  
Above all else, I believe that software should:

- Work correctly
- Work efficiently

One would think that it would be very straightforward to achieve these ends, and yet (historically, at least) it's been damned near impossible -- even for very bright people with high levels of expertise. (It may help to know that my definition of "works correctly" is not "works once", "seems to work" or "works for me" but rather "rock-solid, never-breaks, lean-my-business-on-it and trust-my-life-to-it".) So why is there so much software out there that's slow and broken? The fundamental difficulty is that we cannot _see_ software. And why can we not see it? Because software has the peculiar property that to see it, you must make it slower. That is, the act of indicating that a particular instruction is being executed will slow down the execution of that instruction. Worse, software has the even stranger property that **if you ever want to even _be able_ to see it, you must make it slower.** That is, let's say you want to optionally be able to trace the entry to a function `foo()`. You might have something like this:

```
foo(int arg)
{
	if (tracing_enabled)
		trace(FOO_ENTRY, arg);
	...

```

This boils down to instructions that look something like this (using a RISC-ish proto instruction set):

```
	set   tracing_enabled, %o0
	ld    [%o0], %o1
	cmp   %o0, 0
	bne   go_around
	set   FOO_ENTRY, %o0
	mov   %i0, %o0
	call  trace
	...
go_around:
	!
	! The rest of the function foo()...
	!
	...

```

That is, it boils down to a load, a compare and a branch. This slows down execution (loads hurt -- especially if they're not in the cache), causes a branch-taken in the common path, increases instruction cache footprint, etc. Not a big deal if `foo()` is the only function in which you do this to, but start putting this in every function, and you'll find that you have a system that is too slow to ship -- it has suffered the infamous "death of a thousand cuts."  
  
(Yes, if you're lucky or if the compiler supports a hint to indicate that `trace()` isn't called in the common case, the sense of the branch may change such that the branch will be not-taken in the common case -- which is better, but this still hurts too much to do it everywhere.)  
  
So we can't leave this kind of code in our optimized, production code, so what do we do? Many do something like this:

```
#ifdef DEBUG
#define TRACE(tok, arg) if (tracing_enabled) trace((tok), (arg))
#else
#define TRACE(tok, arg)
#endif
```

This is better -- at least the production version isn't hindered and we still have debug support in a DEBUG version. But now we have a new problem. We now have essentially two versions of our software: the slow one that we can see, and the fast one that we can't. So what do we do when we see a performance problem in production? Well, we might try to run the DEBUG version (or worse, one with custom instrumentation) in production. But that requires downtime, and additional risk -- and usually doesn't fly. So what do we do? We try to reproduce this in development on the DEBUG version that we can see. (This is not "we" as in Sun, by the way, this is "we" as in humanity -- you and me and everyone.) And reproducing perfomance problems is bad, bad news: when you're reproducing performance problems, you're reproducing _symptoms_. (Naturally, because the symptom are all you've got; if you knew the root-cause you would be watching [Knight Rider](http://www.imdb.com/title/tt0083437/) reruns instead of horsing around at work.) And why is reproducing symptoms such bad news? Because **disjoint problems can manifest the same symptoms**.  
  
To borrow a medical analogy: let's say that you discover that you're running a fever in production. So you take your development or test environment, and you try to make it look closer and closer to your production environment until you have a fever. Maybe you add more artificial load, more hardware, more users, whatever. Finally, you see the fever in your development environment. You get all of the developers in the room, and they start throwing instrumented binaries on the development machine. Maybe you think you've got an OS issue, so you have Solaris engineers throwing on new kernels -- or maybe you have your ISVs giving you instrumented binaries of their products. Finally, after a huge amount of time and escalation and more time and frustration, you discover the problem: the fever is due to [influenza](http://www.cdc.gov/flu/about/disease.htm). Okay, this isn't the end of the world: if the production environment stays off its feet, drinks fluids and gets some rest for the next few weeks, it should be fine.  
  
But here's the problem: it **was** influenza in the development environment -- that much was correct. But it's not influenza in the production environment. In the production environment, the problem was [cerebral malaria](http://www.cdc.gov/malaria/disease.htm). No amount of rest is going to help -- our diagnosis is completely wrong.[^2] It may strike you as a glib analogy, but it's an accurate one for the experiences of many. Just think: how many times have you found "a" problem without finding "the" problem?  
  
Okay, so we're clearly down a blind alley of sorts. We actually need to start over with a clean sheet of paper -- we need to change the model. We need to be able to ship an optimized kernel and optimized apps, and when we want to be able to see the software, we need to be able to _dynamically instrument_ it to answer our question. And when we're done answering our question, we want the system to go back to be completely optimized. And we want to do all this in _production environments_. This means that is must be **absolutely safe** -- there must be no way to crash the system through user error.  
  
And this, in essense, is what we've done with [DTrace](http://www.sun.com/bigadmin/content/dtrace). DTrace is a new facility in [Solaris 10](http://wwws.sun.com/software/solaris/10/) for the dynamic instrumentation of production systems. DTrace is available today via [Solaris Express](http://wwws.sun.com/software/solaris/solaris-express/get.html). It has been available since November, and many people have already used it to diagnose real problems. You can read some of their thoughts in the [DTrace feature story](http://www.sun.com/2004-0518/feature/index.html) that ran on [sun.com](http://www.sun.com) late last month.  
  

[^1]: I would have hyperlinked to Mike and Adam's blogs, but they don't (yet) exist. I would expect Adam to have a blog shortly, but given that Mike doesn't yet have a cell phone, it might be a longer wait. Then again, Mike bought a [TiVo](http://www.tivo.com) the first weekend they were on sale at the Palo Alto Fry's back in 1998 -- so you never know when he's going to adopt a technology.  
  
[^2]: Lest you think medical science has figured this one out: I encourage you to contract cerebral malaria and present at your local emergency department -- and observe just how many weeks you spend bouncing around the health care system before some clever doc finally cracks the case...
