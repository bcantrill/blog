---
title: "DTrace on LKML"
date: "2004-08-20"
categories: 
  - "solaris"
---

So DTrace was [recently mentioned](http://groups.google.com/groups?hl=en&lr=&ie=UTF-8&selm=2v3Ad-5tc-29%40gated-at.bofh.it) on the [linux-kernel mailing list](http://lkml.org/). The question in the subject was is "DTrace-like analysis possible with future Linux kernels?" The [responses](http://groups.google.com/groups?hl=en&lr=&ie=UTF-8&threadm=2v3Ad-5tc-29%40gated-at.bofh.it&rnum=1&prev=/groups%3Fhl%3Den%26lr%3D%26ie%3DUTF-8%26selm%3D2v3Ad-5tc-29%2540gated-at.bofh.it) have been interesting. Karim Yaghmour rattled in with his usual blather about the existence of DTrace proving that LTT should have been accepted into the Linux kernel long ago. I find this argument incredibly tedious, and have [addressed it](http://blogs.sun.com/roller/page/bmc/20040718#dtrace_vs_dprobes_ltt) at some length. Then there was [this inane post](http://groups.google.com/groups?hl=en&lr=&ie=UTF-8&selm=2v4w9-6aQ-5%40gated-at.bofh.it):

> ```
> > http://www.theregister.co.uk/2004/07/08/dtrace_user_take/:
> > "Sun sees DTrace as a big advantage for Solaris over other versions of Unix
> > and Linux."
> That article is way too hypey.
> It sounds like one of those strange american commercials you see
> sometimes at night, where two overenthusiastic persons are telling you
> how much that strange fruit juice machine has changed their lives,
> with making them loose 200 pounds in 6 days and improving their
> performance at beach volleyball a lot due to subneutronic antigravity
> manipulation. You usually can't watch those commercials for longer
> than 5 minutes.
> The same applies to that article, I couldn't even read it completely,
> it was just too much.
> And is it just me or did that article really take that long to
> mentioning what dtrace actually IS?
> Come on, it's profiling. As presented by that article, it is even more
> micro optimization than one would think. What with tweaking the disk
> I/O improvements and all... If my harddisk accesses were a microsecond
> more immediate or my filesystem giving a quantum more transfer rate,
> it would be nice, but I certainly wouldn't get enthusiastic and I bet
> nobody would even notice.
> Maybe, without that article, I would recognize it as a fine thing (and
> by "fine" I don't mean "the best thing since sliced bread"), but that
> piece of text was just too ridiculous to take anything serious.
> I sure hope that article is meant sarcastically. By the way, did I
> miss something or is profiling suddenly a new thing again?
> Regards,
> Julien
> 
> ```

Yes, you missed something Julien: **you forgot to type "dtrace" into google**. (If there were a super-nerd equivalent of the [Daily Show](http://www.comedycentral.com/tv_shows/thedailyshowwithjonstewart/), we might expect [Lewis Black](http://www.lewisblack.net/) to say that -- probably punctuated with his usual "you moron!") If you had done this, you would have been taken to the [DTrace BigAdmin site](http://www.sun.com/bigadmin/content/dtrace) which contains links to the [DTrace USENIX paper](http://www.sun.com/bigadmin/content/dtrace/dtrace_usenix.pdf), the [DTrace documentation](http://www.sun.com/bigadmin/content/dtrace/d10_latest.pdf), and a boatload of other material that supports the claims in [The Register story](http://www.theregister.co.uk/2004/07/08/dtrace_user_take/). In fact, if you had just **scrolled to the bottom** of that story you would have read the "Bootnotes" section of the story -- which provides plenty of low-level supporting detail. (Indeed, I'm not sure that I've _ever_ seen The Register publish such user-supplied detail to support a story.)

Sometimes the bigotry surrounding Linux suprises even me: in the time he took to record his misconceptions, Julien could have (easily!) figured out that he was completely wrong. But I guess that even this is too much work for someone who is looking to confirm preconceived notions rather than understand new technology...

Fortunately, [one of the responses](http://groups.google.com/groups?hl=en&lr=&ie=UTF-8&selm=2v5se-6PN-9%40gated-at.bofh.it) did call Julien on this, if only slightly:

> ```
> * Julien Oster:
> > Miles Lane  writes:
> >
> >> http://www.theregister.co.uk/2004/07/08/dtrace_user_take/:
> >> "Sun sees DTrace as a big advantage for Solaris over other versions of Unix
> >> and Linux."
> >
> > That article is way too hypey.
> Maybe, but DTrace seems to solve one really pressing problem: tracking
> disk I/O to the processes causing it.  Unexplained high I/O
> utilization is a *very* common problem, and there aren't any tools to
> diagnose it.
> Most other system resources can be tracked quite easily: disk space,
> CPU time, committed address space, even network I/O (with tcpdump and
> netstat -p).  But there's no such thing for disk I/O.
> 
> ```

Of course, the responder misses the larger point about DTrace -- that one can instrument one's system _arbitrarily_ and _safely_ with DTrace -- but at least he correctly identifies one capacity in which DTrace clearly leads the pack. And I suppose that this is the best that a rival technology can expect to do, so close to the epicenter of [Linux development](http://familyfun.go.com/arts-and-crafts/season/feature/famf107project/famf107project24.html)...
