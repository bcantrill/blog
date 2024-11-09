---
title: "Be careful what you ask for..."
date: "2004-06-18"
categories: 
  - "solaris"
---

Earlier, [I lamented](http://blogs.sun.com/roller/page/bmc/20040616#open_source_p_s_dtrace) the fact that a press roundtable on three key technology areas in Solaris 10 ([DTrace](http://www.sun.com/bigadmin/content/dtrace), [Zones](http://www.sun.com/bigadmin/content/zones) and ZFS) had yielded only stories about open source -- a topic which we explicitly didn't talk about. Fortunately, there is now a [new story](http://servers.itmanagersjournal.com/servers/04/06/17/2019258.shtml?tid=73&tid=97) by one of attendees of the roundtable that focuses on the three technology areas.  
  
And even better, the larger points about DTrace are certainly correct, e.g.:

> DTrace, which uses more than 30,000 data monitoring points in the kernel alone, lets administrators see their entire system in a new way, revealing systemic problems that were previously invisible and fixing performance issues that used to go unresolved.

And the example that the article is trying to cite has an absolute basis in fact -- it's discussed in depth in Section 9 of [our upcoming USENIX paper](http://www.sun.com/bigadmin/content/dtrace/dtrace_usenix.pdf). But that said, the details of the specific example are incredibly wrong. (So wrong, in fact, that they're just odd; what does "a wild-card desktop applet that had somehow gotten channeled into the central system" even mean?) Perhaps the terms used are so opaque that readers will come away confused, but with the right overall impression -- but given that readers at [LWN.net](http://lwn.net) went so far as to [accuse me of being a pointy-haired boss](http://lwn.net/Articles/89918/) based on the [C++ misquote](http://blogs.sun.com/roller/page/bmc/20040616#open_source_again), I can only imagine [what I'll be accused of being now](http://news.ft.com/comment/columnists/martinlukes)...
