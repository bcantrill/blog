---
title: "DTrace on QNX!"
date: "2007-11-08"
categories: 
  - "solaris"
---

There are moments in anyone's life that seem mundane at the time, but become pivotal in retrospect: that chance meeting of your spouse, or the job you applied for on a lark, or the inspirational course that you just stumbled upon.

For me, one of those moments was in late December, 1993. I was a sophomore in college, waiting in Chicago O'Hare for a connecting flight home to Denver for the winter break, reading an issue of [the late Byte magazine](http://en.wikipedia.org/wiki/Byte_(magazine)) dedicated to operating systems. At the time, I had just completed what would prove to be the most intense course of my college career -- [Brown's (in)famous CS169](http://dtrace.org/blogs/bmc/the_inculcation_of_systems_thinking) -- and the magazine contained an article on a very interesting microkernel-based operating system called [QNX](http://en.wikipedia.org/wiki/QNX). The article, written by the [late, great Dan Hildebrand](http://www.openqnx.com/index.php?name=News&file=article&sid=298), was exciting: we had learned about microkernels in CS169 (indeed, we had implemented one) and here was one running in the wild, solving real, hard problems. I was inspired. When I returned to school, I cold e-mailed Dan and asked about the possibilites of summer employment. Much to my surprise, he responded! When I returned to school in January, Dan and QNX co-founder [Dan Dodge](http://en.wikipedia.org/wiki/Dan_Dodge) gave me an intense phone interview -- and then offered me work for the summer. I worked at QNX (based in Kanata, outside of Ottawa) that summer, and then came back the next. While I didn't end up working for QNX after graduation, I have always thought highly of the company, the people -- and especially the technology.

So you can imagine that for me, it's a unique pleasure -- and an an odd sort of homecoming -- to announce that [DTrace is being ported to QNX](http://sendreceivereply.wordpress.com/2007/11/08/i-trace-you-trace-what-about-dtrace/). In my [DTrace talk at Google](xxx) I made passing reference (at about 1:11:53) to one "other system" that DTrace was being ported to, but that I was not at liberty to mention which. Some speculated that this "surely" must be Windows -- so hopefully the fact that it was QNX will serve to remind that it's a big heterogeneous world out there. So, to [Colin Burgess and Thomas Fletcher](http://sendreceivereply.wordpress.com/about/) at QNX: congratulations on your fine work. And to QNX'ers: welcome to the [DTrace community](http://www.opensolaris.org/os/community/dtrace/)!

Now, with the drinks poured and the toasts made, I must confess that this is also a moment colored by some personal sadness, for I would love nothing more right now than to call up `danh`, chat about the exciting prospects for DTrace on QNX, and reminisce about the many conversations over our shared cube wall so long ago. (And if nothing else, I would kid him about his summer-of-1994 enthusiasm for Merced!) Dan, you are sorely missed...