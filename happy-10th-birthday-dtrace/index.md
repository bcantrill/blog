---
title: "Happy 10th Birthday, DTrace!"
date: "2013-09-03"
---

Ten years ago this morning -- at 10:27a local time, to be exact -- DTrace was integrated. On the occasion of DTrace's fifth birthday a half-decade ago, [I reflected on much of the drama of the final DTrace splashdown](http://dtrace.org/blogs/bmc/2008/09/03/happy-5th-birthday-dtrace/), but much has happend in the DTrace community in the last five years; some highlights:

- [DTrace was ported to Linux](http://www.computerworld.com/s/article/9237268/Oracle_ports_DTrace_to_Oracle_Linux) -- [twice](https://github.com/dtrace4linux/linux)
    
- [Brendan](http://dtrace.org/blogs/brendan)'s [DTrace book](http://dtracebook.com/index.php/Main_Page) became the canonical guide to using DTrace in practice, bringing DTrace to a new generation of practitioners
    
- [Chris Andrews](http://chrisa.github.io/) lit the way for bringing USDT to dyanamic languages with his terrific [libusdt](https://github.com/chrisa/libusdt), using it to [bring DTrace to node.js](https://github.com/chrisa/node-dtrace-provider), [bring DTrace to Lua](https://github.com/chrisa/lua-usdt), [bring DTrace to Perl](https://github.com/chrisa/perl-dtrace/tree/master/Devel-DTrace-Provider) and [bring DTrace to Ruby](https://github.com/chrisa/ruby-dtrace).
    
- [Dave Pacheco](http://dtrace.org/blogs/dap/) became a master of the black art of DTrace ustack helpers, and developed [a DTrace ustack helper for node.js](http://dtrace.org/blogs/dap/2012/01/05/where-does-your-node-program-spend-its-time/), allowing for [Brendan Gregg](http://dtrace.org/blogs/brendan)'s [amazing flamegraphs](http://dtrace.org/blogs/brendan/2011/12/16/flame-graphs/) for node.js programs on SmartOS. Dave's work was [both extended to 64-bit and explained beautifully](http://blog.indutny.com/3.dtrace-ustack-helper) by the incomparable [Fedor Indutny](https://github.com/indutny).
    
- In part shamed by Joyent customers, I [expanded DTrace in the non-global zone](http://dtrace.org/blogs/bmc/2012/06/07/dtrace-in-the-zone/) -- which was embarrassingly a problem that I had previously ducked by claiming it was "technically impossible." (If you don't know by now, this is often correctly translated as "I really, really don't feel like it.")
    
- [Eric Schrock](http://dtrace.org/blogs/eschrock/) added the [print action](http://dtrace.org/blogs/ahl/2012/07/28/my-new-dtrace-favorite/) -- which saved my butt as recently as this past weekend.
    
- We got the DTrace community together at the first DTrace unconference -- [dtrace.conf(08)](http://dtrace.org/blogs/bmc/2008/03/16/dtrace-conf08/) -- and then followed that up at an Olympiad cadence with [dtrace.conf(12)](http://dtrace.org/blogs/ahl/2012/07/28/my-new-dtrace-favorite/). The International DTrace Committee will soon be selecting the host city for dtrace.conf(16), so get those bribes ready!
    

And a bunch of other stuff that's either more esoteric (e.g., [llquantize](http://dtrace.org/blogs/bmc/2011/02/08/llquantize/)) or that I now use so frequently that I've forgotten that there was a time that we didn't have it ([\-x temporal](https://github.com/illumos/illumos-gate/commit/e5803b76927480e8f9b67b22201c484ccf4c2bcf) FTW!). But if I could go back to my five-year-ago self, the most surprising development to my past-self may be how smoothly DTrace and its community survived the collapse of Sun and the [craven re-closing of Solaris](http://dtrace.org/blogs/bmc/2010/08/19/the-liberation-of-opensolaris/). It's a testament to the power of open source and open communities that DTrace has -- along with its operating system of record, [illumos](http://en.wikipedia.org/wiki/Illumos) -- emerged from these events empowered and invigorated. So I would probably show my past-self my [LISA talk on the rise of illumos](http://smartos.org/2011/12/15/fork-yeah-the-rise-and-development-of-illumos-2/), and he and I would both get a good chuckle out of it all -- and then no doubt erupt in fisticuffs over the technical impossibility of [fds\[\] in the non-global zone](http://www.slideshare.net/bcantrill/dtrace-in-the-nonglobal-zone)...

And lest anyone think that we're done, the future is bright, as the El Dorado of [user-land types in DTrace](http://dtrace.org/blogs/bmc/2012/06/07/dtrace-in-the-zone/) may be gleaming on the horizon courtesy of [Adam](http://dtrace.org/blogs/ahl) and [Robert Mustacchi](http://dtrace.org/blogs/rm/). I think I speak on behalf of all of us in the broader DTrace community when I say that I'm very much looking forward to continuing to use, improve and expand DTrace for the next decade and beyond!
