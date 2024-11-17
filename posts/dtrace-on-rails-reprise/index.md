---
title: "DTrace on Rails, reprise"
date: "2006-10-12"
categories: 
  - "solaris"
---

A little while ago, I blogged about [DTrace on Rails](http://dtrace.org/blogs/bmc/dtrace_on_rails). In particular, I promised that I would get diffs based on [is-enabled probes](http://www.opensolaris.org/jive/thread.jspa?messageID=31314) out "shortly." In giving a guest lecture for a [class at Berkeley](http://radlab.cs.berkeley.edu/wiki/RADSClassFall06) yesterday, I was reminded that I still hadn't made this available. With my apologies for the many-months delay, the diff (against Ruby 1.8.2) is [here](http://dtrace.org/resources/bmc/ruby-1.8.2-dtrace-isenabled.diff).

And as long as I have your eyeballs, let me join [Adam](http://blogs.sun.com/ahl/entry/dtrace_is_a_web_developer) in directing you to [Brendan Gregg](http://blogs.sun.com/brendan)'s amazing [Helper Monkey](http://blogs.sun.com/brendan/entry/dtrace_meets_javascript). Brendan's work is an indisputable quantum leap for what has become one of the most important, one of the most misunderstood, and certainly one of the most undebuggable platforms on the planet: JavaScript. If you do anything AJAX-ian that's evenly vaguely performance or footprint sensitive, you're wasting your time if you're not using Helper Monkey. (When Brendan first demo'd Helper Monkey to Adam and me, Adam's line was that Brendan had just propelled JavaScript debugging forward by "100,000 years" -- which is not to say that Helper Monkey is like debugging in the 1021st century, but rather that debugging JavaScript without Helper Monkey is like [debugging in the late Pleistocene](http://rocr.xepher.net/weblog/images/neanderthalers.jpg).)
