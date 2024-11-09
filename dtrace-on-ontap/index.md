---
title: "DTrace on ONTAP?"
date: "2007-09-06"
categories: 
  - "solaris"
---

As presumably many have seen, [NetApp is suing Sun over ZFS](http://www.theregister.co.uk/2007/09/05/netapp_sues_sun_over_zfs/). I was particularly disturbed to read [Dave Hitz's account](http://blogs.netapp.com/dave/2007/09/netapp-sues-sun.html), whereby we supposedly contacted NetApp 18 months ago requesting that they pay us "lots of money" (his words) for infringing our patents. Now, as a Sun patent holder, reading this was quite a shock: I'm not enamored with the US patent system, but I have been willing to contribute to Sun's patent portfolio because we have always used them defensively. Had we somehow lost our collective way?

Now, I should say that there was something that smelled fishy about Dave's account from the start: I have always said that a major advantage of working for or doing business with Sun is that **we're too disorganized to be evil**. Being disorganized causes lots of problems, but actively doing evil isn't among them; approaching NetApp to extract gobs of money shows, if nothing else, a level of organization that we as a company are rarely able to summon. So if Dave's account were true, we had silently effected two changes at once: we had somehow become organized _and_ evil!

Given this, I wasn't too surprised to learn that [Dave's account isn't true](http://blogs.sun.com/jonathan/entry/on_patent_trolling). As Jonathan explains, NetApp first approached STK through a third party intermediary (boy, are they ever organized!) seeking to buy the STK patents. When we bought STK, we also bought the ongoing negotiations, which obviously, um, broke down.

Beyond clarifying how this went down, I think two other facts merit note: one is that NetApp has filed suit in East Texas, [the infamous den of patent trolls](http://www.technologyreview.com/read_article.aspx?id=16280&ch=infotech). If you'll excuse my frankness for a moment, WTF? I mean, I'm not a lawyer, but we're both headquartered in California -- let's duke this out in California like grown-ups.

The second fact is that this is not the first time that NetApp has engaged in this kind of behavior: in 2003, shortly after buying the patents from bankrupt Auspex, NetApp went after BlueArc for infringing three of the patents that they had just bought. Importantly, [NetApp lost on all counts](http://www.marketwire.com/mw/release.do?id=702043&k=) -- even after an appeal. To me, the misrepresentation of how the suit came to be, coupled with the choice of venue and history show that NetApp -- despite their claptrap to the contrary -- would prefer to meet their competition in a court of law than in the marketplace.

In the spirit of offering constructive alternatives, how about [porting DTrace to ONTAP](http://www.vmunix.com/mark/blog/archives/2007/07/30/dtrace-for-netapp-please/)? As [I offered to IBM](http://dtrace.org/blogs/bmc/dtrace_on_aix), we'll help you do it -- and thanks to the [CDDL](http://en.wikipedia.org/wiki/Common_Development_and_Distribution_License), you could do it without fear of infringing anyone's patents. After all, everyone deserves DTrace -- even patent trolls. ;)
