---
title: "DTrace on AIX?"
date: "2006-08-17"
categories: 
  - "solaris"
---

So IBM has been on the warpath recently against OpenSolaris, culminating with their [accusation yesterday](http://news.zdnet.co.uk/software/linuxunix/0,39020390,3928134,00.htm) that OpenSolaris is a "facade." This is so obviously untrue that it's not even worth refuting in detail. In fact, being the father of a toddler, I would liken IBM's latest outburst to something of a temper tantrum -- and as with a screaming toddler, the best way to deal with this is to not reward the performance, but rather to offer some constructive alternatives. So, without further ado, here are my constructive suggestions to IBM:

- Open source OS/2. Okay, given the [tumultuous history of OS/2](http://en.wikipedia.org/wiki/OS/2#Breakup), this is almost certainly not possible from a legal perspective -- but it would be a great open source contribution (some reasonably interesting technology went down with that particular ship), and many hobbyists would love you for it. Like I said, it's probably not possible -- but just to throw it out there.
    
- Open source AIX. AIX is one of the true enterprise-class operating systems -- one with a long history of running business-critical applications. As such, it would be both a great contribution to open source and a huge win for AIX customers for AIX to go open source -- if only to be able to look at the source code when chasing a problem that isn't necessarily a bug in the operating system. (And I confess that on a personal level, I'm very curious to browse the source code of an operating system that was [ported from PL/1](http://en.wikipedia.org/wiki/AIX_operating_system#AIX_Version_2).) However, as with OS/2, AIX's history is going to likely make open sourcing it tough from a legal perspective: its Unix license dates from the [Bad Old Days](http://en.wikipedia.org/wiki/System_V#SVR3), and it would probably be time consuming (and expensive) to unencumber the system to allow it to be open sourced.

Okay, those two are admittedly pretty tough for legal reasons. Here are some easier ones:

- Support the [port of OpenSolaris to POWER/PowerPC](http://www.opensolaris.org/os/community/power_pc/). Sun doesn't sell POWER-based gear, so you would have the comfort of knowing that your efforts would in no way assist a Sun hardware sale, and your POWER customers would undoubtedly be excited to have another choice for their deployments.
    
- Support the nascent effort to port OpenSolaris to the S/390. Hey, if Linux makes sense on an S/390, surely OpenSolaris with all of its goodness makes sense too, right? Again, customers love choice -- and even an S/390 customer that has no intention of running OpenSolaris will love having the choice made available to them.

Okay, so those two are easier because the obstacles aren't legal obstacles, but there are undoubtedly internal IBM cultural issues that make them effectively non-starters.

So here's my final suggestion, and it's an absolutely serious one. It's also relatively easy, it clearly and immediately benefits IBM and IBM's customers -- and it even doesn't involve giving up any IP:

- **Port DTrace to AIX.** Your [customers want it](http://www.opensolaris.org/jive/thread.jsp?messageID=50963). Apple has shown that [it can be done](http://www.zdnet.com.au/news/software/soa/Mac_OS_X_Leopard_gets_Sun_s_DTrace/0,2000061733,39265767,00.htm). We'll [help you do it](http://blogs.sun.com/roller/page/ahl?entry=dtrace_on_mac_os_x). And you'll get to participate in the [DTrace community](http://www.opensolaris.org/os/community/dtrace/) (and therefore the [OpenSolaris community](http://www.opensolaris.org)) in a way that doesn't leave you feeling like you've been scalped by Scooter. Hell, you can even follow Apple's lead with Xray and innovate on _top_ of DTrace: from talking to your customers over the years, it's clear that they love [SMIT](http://www.ibm.com/servers/aix/products/aixos/whitepapers/smit.html) -- integrate a SMIT frontend with a DTrace backend! Your customers will love you for it, and the DTrace community will be excited to have yet another system on which that they can use DTrace.

Now, IBM may respond to these alternatives just as a toddler sometimes responds to constructive alternatives ("No! No! NO! Mine! MINE! MIIIIIIIINE!", etc). But if cooler heads prevail at Big Blue, these suggestions -- especially the last one -- will be seen as a way to constructively engage that will have clear benefits for IBM's customers (and therefore for IBM). So to IBM I say what parents have said to screaming toddlers for time immemorial: we're ready when you are.
