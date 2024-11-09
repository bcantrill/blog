---
title: "DTrace Tips, Tricks and Gotchas"
date: "2005-02-28"
categories: 
  - "solaris"
---

Last week, [Mike](http://blogs.sun.com/mws), [Adam](http://blogs.sun.com/ahl) and I presented the first "advanced" DTrace talk. That is, since shipping DTrace, our presentations have been to people who have either never heard of DTrace, or have heard of it but haven't seen it before, or have heard of it and seen it before and now want to learn how to use it. The time for an advanced talk was clearly overdue; many of the questions both in the [DTrace forum](http://forum.sun.com/forum.jsp?forum=211) and internally at Sun have indicated that the limits of [the DTrace documentation](http://docs.sun.com/db/doc/817-6223) are being reached1, and that an advanced presentation would be helpful to some users. So given that others may find this presentation useful -- and with the caveats that it doesn't substitute for seeing it in the flesh2 and that it contains some rather arcane detail -- the "Advanced DTrace: Tips, Tricks and Gotchas" presentation can be found [here](http://blogs.sun.com/roller/resources/bmc/dtrace_tips.pdf).

As the title implies, this really _is_ a random collection of tips, tricks and gotchas. Indeed, it's so random the slides could pretty much be in any order -- there is no narrative arc to this presentation whatsoever (there isn't even a conclusion slide because there wasn't much to conclude), and the whole thing thus has a somewhat dream-like (or nightmare-like?) quality. Apologies for that; there didn't seem to be a better way to present such arcana...

* * *

1 Yes, there are limits to the documentation -- which is pretty incredible considering that there's already 400 pages of it.

2 For starters, it's much easier to grok some of these concepts when they can be demo'd on-the-fly. And by not seeing it live, you won't have the pleasure of hearing me speak "a million miles an hour" in [Dan Lacher's words](http://blogs.sun.com/roller/page/dlacher/20050226#dreaming_in_dtrace). (To which all I will say in my defense is that I've never had anyone fall asleep during one of my presentations...)

Technorati tag: [DTrace](http://technorati.com/tag/DTrace)
