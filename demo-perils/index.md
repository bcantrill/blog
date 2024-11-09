---
title: "Demo Perils"
date: "2004-07-26"
categories: 
  - "solaris"
---

One of the downsides of being an operating systems developer is that the demos of the technology that you develop often suck. ("Look, it boots! And hey, we can even run programs and it doesn't crash!") So it's been a pleasant change to develop [DTrace](http://www.sun.com/bigadmin/content/dtrace), a technology that packs a jaw-dropping demo. In demonstrating DTrace for customers around the world, I have had the distinct (and rare) pleasure of impressing the most technically adept (and often jaded) audiences. My typical demonstration is on my [Solaris x86 laptop](http://www.sun.com/bigadmin/features/articles/solarisOnLaptops.html), where I use DTrace to instrument the running system -- exploring with the audience the peculiarities that exist even on an idle laptop. (This usually involves discovering and understanding the unnecessary work being done by acroread, [dhcpagent](http://docs.sun.com/db/doc/817-0690/6mgflnt8b?a=view), [sendmail](http://sendmail.org/), etc.) This _ad hoc_ demo shows DTrace as it's meant to be used: dynamically answering questions that themselves were formed on-the-fly.

And when I demonstrate DTrace, I always do so on the absolute latest Solaris 10 build. Our mantra in Solaris Kernel Development is "FCS Quality All the Time" -- we believe that the product should _always_ be ready to be run in production. And if we're going to tell a customer that it's ready to be run in production, we damn well better run it in production ourselves. This has the added advantage that we tend to run into bugs before our customers do, allowing us to ship a final product that is that much more solid. Over the past year, I have given hundreds of DTrace demonstrations in front of customers running latest bits, and before last week, it had always gone off without a hitch...1

Last week, I had the opportunity to give a DTrace demonstration for a highly technical -- and highly influential -- audience at a Fortune 100 company. When I demonstrate DTrace, I typically do a couple of invocations on the command line before things become sufficiently complicated to merit writing a [DTrace script](http://docs.sun.com/db/doc/817-6223/6mlkidlkk?a=view). And it was when I went to run the first such script (a script that explored the activity of `xclock`) that it happened:

> ```
> # dtrace -s ./xclock.d
> Segmentation Fault (core dumped)
> #
> 
> ```

If you've never had it, there's no feeling quite like having a demo blow up on you: it's as if you peed your pants, failed an exam and were punched in the gut -- all at the same horrifying instant. It's a feeling that every software developer should have exactly once in their lives: that unique rush of shock, and then humiliation and then despair, followed by the adrenal surge of a fight-or-flight reaction. In the time it takes a single process to dump core, you go from an (over)confident technologist to a frightened woodland creature, transfixed by the light of an oncoming freight train. For the woodland creature, at least it all ends mercifully quickly; the creature is spared the suffering of trying to explain away its foolishness. The hapless technologist, on the other hand, is left with several options:

1. Pretend that you didn't write the software: "Boy, will you get a load of those fancy-pants software engineers? Overpriced underworked morons, every last one!"
    
2. Explain that this is demo software and isn't expected to work: "Well, that's why we haven't shipped it yet! I mean, what fool would run this stuff anyway? Other than me, that is."
    
3. Make light of it: "Hey, knock knock! Who's there? Not my software, that's for sure! Wocka wocka wocka!"
    
4. Suck it up: "That's a serious problem. If you can excuse me for a second, let me get a handle on what we've got here that we can demo."
    

I always aim for this last option, but on the rare occasion that this has happened to me (and this is -- honest -- probably the worst that a customer-facing demo has gone for me) I usually end up with some combination of the last three, often with plenty of stuttering, some mild swearing ("Damn! Damn!") and profuse sweating.

In my particular case, the worst part was not knowing the exact pathology of the bug that I had just run into. Was there something basic that was broken or toxic about my machine? Would _all_ scripts that I tried to run dump core? And if this was broken, what else was broken? Would I panic the machine or crash a target app if I continued? (Much more serious problems, both.) In an effort to get a handle on it, I did a quick [pstack](http://docs.sun.com/db/doc/816-5165/6mbb0m9m2?q=pstack&a=view) on the [core file](http://docs.sun.com/db/doc/816-5174/6mbb98udv?a=view):

> ```
> 0804718f ???????? (8046604, 2)
> d137c839 dt_instr_size (82d051a, 8067320, 223, d1380fe2) + 59
> d137c0c2 dt_pid_create_return_probe (81651b8, 8067320, 8046af0, 8047170, 80472d
> d137370d dt_pid_per_sym (80472ac, 8047170, d087b02c) + 15b
> d13739ae dt_pid_sym_filt (80472ac, 8047170, d087b02c, 804715c) + 7c
> d13152ca Psymbol_iter_com (81651b8, ffffffff, 8069060, 1, 407, 1) + 1e0
> d13153ae Psymbol_iter_by_addr (81651b8, 8069060, 1, 407, d1373932, 80472ac) + 1
> d1373b81 dt_pid_per_mod (80472ac, 82cf600, 8069060) + 191
> d1373d56 dt_pid_mod_filt (80472ac, 82cf600, 8069060) + a3
> d1314fe4 Pobject_iter (81651b8, d1373cb3, 80472ac) + 4f
> d13740b4 dt_pid_create_probes (82cafa0, 8067320) + 344
> d1353af8 dt_setcontext (8067320, 82cafa0) + 42
> d13537d4 dt_compile_one_clause (8067320, 82be430, 82cdae0) + 32
> d1353a9c dt_compile_clause (8067320, 82be430) + 26
> d1354d66 dt_compile (8067320, 16a, 3, 0, 80, 1) + 3d9
> d1355263 dtrace_program_strcompile (8067320, 8047ec2, 3, 80, 1, 8066848) + 23
> 080526ef ???????? (8066e48)
> 0805370e main     (3, 8047df8, 8047e08) + 8fc
> 0805177a ???????? (3, 8047eb8, 8047ebf, 8047ec2, 0, 8047edf)
> 
> ```

This was dying in the code that analyzes a target binary as part of creating [`pid` provider](http://docs.sun.com/db/doc/817-6223/6mlkidloa?a=view) probes. There was at least a chance that this problem was localized to something specific about the `xclock` program text -- it was worth trying a similar script on a different process. Fortunately, I was able to stave off total panic long enough to write such a script and -- even better -- this one worked. The problem did indeed seem to be localized to something specific in xclock. And thanks to my [coreadm](http://docs.sun.com/db/doc/816-5166/6mbb1kpv7?a=view) settings, the core file from the seg faulting [dtrace](http://docs.sun.com/db/doc/816-5166/6mbb1kq0f?a=view) had been stashed away for later analysis; the best thing I could do at that point was drive on with the rest of the demo.

And this is what I did. The rest of the demo went well, and the audience was ultimately impressed with the technology. And while I never quite regained my stride (in part because my mind was racing about which change to DTrace could have introduced the problem), I was at least sufficiently effective -- we achieved the goals of the meeting.2 On the plane back home, I root-caused the problem and developed a fix. The next day, I integrated the fix into Solaris -- and I don't think I've ever been so relieved to put latest bits on my laptop!

In the end, having the demo blow up certainly wasn't a pleasant experience -- but I wouldn't change my decision to demo on the latest bits. Not only did we discover a serious bug, we discovered the hole in our test suite that prevented us from finding the bug before it integrated. So who am I to get upset about a little personal humiliation if the upshot is a better product? ;)

* * *

1 This is a slight exaggeration. I had actually run into DTrace bugs in front of customers, but they were always sufficiently small that only a trained eye would realize that something was amiss -- things like slightly incorrect error messages.

2 The primary goal of such a demo is often to get the customer sufficiently excited about [Solaris 10](http://wwws.sun.com/software/solaris/10/) to download [Solaris Express](http://www.sun.com/software/solaris/solaris-express/sol_index.html) (usually for x86) and start playing around with the technology themselves. We are nearly always successful in this -- and I have even had a few customers start downloading Solaris Express before the end of the meeting!
