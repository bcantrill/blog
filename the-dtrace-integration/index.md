---
title: "The DTrace integration"
date: "2004-07-25"
categories: 
  - "solaris"
---

I've been following with interest [this thread](http://groups.google.com/groups?hl=en&lr=&ie=UTF-8&threadm=2kSPF-4SA-7%40gated-at.bofh.it&rnum=1&prev=/groups%3Fhl%3Den%26lr%3D%26ie%3DUTF-8%26selm%3D2kSPF-4SA-7%2540gated-at.bofh.it) on the [linux-kernel mailing list](http://lkml.org/). The LTT folks have apparently given up on [the claim](http://groups.google.com/groups?selm=2g5aM-5mq-27%40gated-at.bofh.it&output=gplain) that they've got "basically almost everything \[DTrace\] claims to provide." They now acknowledge the difference in functionality between LTT and DTrace, but they're using it to continue an attack on the Linux development model. (Or more accurately, to attack how that model was applied to them.) The most interesting paragraph is this one [in a post](http://groups.google.com/groups?hl=en&lr=&ie=UTF-8&selm=2lXtV-QF-15%40gated-at.bofh.it) by Karim Yaghmour:

> ```
> As for DTrace, then its existence and current feature set is only a
> testament to the fact that the Linux development model can sometimes have
> perverted effects, as in the case of LTT. The only reason some people can
> rave and wax about DTrace's capabilities and others, as yourself, drag
> LTT through the mud because of not having these features, is because the
> DTrace developers did not have to put up with having to be excluded from
> their kernel for 5 years. As I had said earlier, we would be eons ahead
> if LTT had been integrated into the kernel in one of the multiple attempts
> that was made to get it in in the past few years. Lest you think that
> DTrace actually got all its features in a single merge iteration ...
> 
> ```

Some points of clarification: we actually _did_ get most of our features in our initial integration into Solaris (on September 3, 2003). In Solaris, projects that integrate are expected to be in a completed state; if there is follow-on work planned, fine -- but what integrates into the gate must be something that is usable and complete from a customer's perspective. So contrary to Karim's assertion, most of DTrace came in that first (giant) putback. As a consequence of this, DTrace spent a _long_ time living the same way LTT has lived: outside the gate, trying to keep in sync while development was underway. Admittedly, DTrace did this for two years -- not five. And this is Solaris, not Linux; it's easier to keep in sync if only because there is only one definition of the latest Solaris. (The newest DTrace build was always based off of the latest Solaris 10 build.) Still: we didn't let the fact that we had not yet integrated prevent us from developing DTrace, nor did we let it prevent us from building a user community around DTrace. By the time DTrace (finally!) integrated into Solaris 10, we had hundreds of internal users, and a long list of actual problems that were found only with the help of DTrace. Not that DTrace would have been unable to integrate without these things, but having them certainly accelerated the process of integration.

More generally, though, I'm getting a little tired of this argument that LTT would be exactly where DTrace is had they only been allowed into the Linux kernel five years ago. I believe that there is some fundamental innovation in DTrace that LTT simply did not anticipate. For insight into what LTT _did_ anticipate, look at the [LTT To Do List](http://www.opersys.com/LTT/dox/ltt-to-do-list-MD.html) from 2002. In that document, you will find many future directions, but not so much as a _whisper_ of the need for DTrace basics like [aggregations](http://docs.sun.com/db/doc/817-6223/6mlkidli7?a=view), [speculative tracing](http://docs.sun.com/db/doc/817-6223/6mlkidljq?a=view), [thread-local variables](http://docs.sun.com/db/doc/817-6223/6mlkidlft?a=view) or [associative arrays](http://docs.sun.com/db/doc/817-6223/6mlkidlfr?a=view) -- let alone DTrace arcana like [stability](http://docs.sun.com/db/doc/817-6223/6mlkidlqn?a=view) or [translators](http://docs.sun.com/db/doc/817-6223/6mlkidlr3?a=view). Would LTT be further along now had it been allowed to integrate into Linux five years ago? Surely. Would it be anywhere near where DTrace is today? In the immortal words of [the Magic 8-Ball](http://8ball.ofb.net/), "very doubtful."
