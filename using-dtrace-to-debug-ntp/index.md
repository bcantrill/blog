---
title: "Using DTrace to debug NTP"
date: "2005-07-27"
categories: 
  - "solaris"
---

[Brian Utterback](http://blogs.sun.com/roller/page/blu/) has a great blog entry describing [using DTrace to debug a really nasty problem in NTP](http://blogs.sun.com/roller/page/blu?entry=tricky_problem_with_ntp). This problem is a good object lesson for two reasons:

- The pathology -- a signal of mysterious origin killing an app -- is a canonically nasty problem and (before DTrace) it was very difficult (or damned near impossible?) to diagnose.
    
- While Brian was able to use DTrace to get a big jump on the problem, completely understanding it took some great insight on Brian's part. This has been said before, but it merits reemphasis: DTrace is a tool, not a magician. That is, DTrace still needs to be used by someone with a brain. And not, by the way, because DTrace is difficult to use, but rather because systems -- especially misbehahaving or suboptimal ones -- have endemic complexity. We have tried to make it as easy as possible to use DTrace to understand that complexity, but the complexity exists nonetheless.

Summary: DTrace allows you to solve problems that were previously unsolvable (or damned near) -- but it means you'll be using your brain more, not less, so you'd better stop [stabbing it with Q-tips](http://www.snpp.com/guides/brainspeaks.html).
