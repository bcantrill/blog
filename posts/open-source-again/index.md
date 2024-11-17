---
title: "Open source, again!"
date: "2004-06-16"
categories: 
  - "solaris"
---

So [another article](http://servers.itmanagersjournal.com/servers/04/06/16/2037217.shtml?tid=73&tid=96) showed up covering the same press meeting that [I discussed earlier](http://blogs.sun.com/roller/page/bmc/20040616#open_source_p_s_dtrace). Again, despite the fact that it was less than one tenth of one percent of the content of the rountable, the headline is open source. On the one hand, I feel slightly vindicated in that this one at least quoted me a little more accurately. And then there's this:

> "But I'm also sure we'll be revisiting a few comments in the code here and there -- I just thought of a particularly disparaging one I might have left in having to do with C++ unions," Cantrill said with a laugh.

This was actually in response to a question -- someone asked (lightheartedly, I had thought) if there were any inappropriate comments in Solaris that would have to be cleaned up. I responded -- indeed with a laugh -- that there were some profane comments I could think of that had to do with unions in C. (Note: C, not C++ -- not that C++ unions don't deserve the same coarse words, only that C++ has bigger problems than just unions.)  
  
Needless to say, I was quite surprised to see this off-hand comment show up -- and the reason for the disparaging comment probably deserves a little context. The comment in question is in code that I wrote for a loadable module in [mdb](http://docs.sun.com/db/doc/806-6545?q=modular+debugger+guide&s=t), the Solaris modular debugger. This particular code is part of postmortem object type identification, a mechanism for identifying arbitrary memory objects from a system crash dump. I actually wrote [a paper](http://arxiv.org/abs/cs/0309037) on this, and presented it at [AADEBUG 2003](http://www.elis.ugent.be/aadebug2003/) in September. Anyway, if you read the paper, you'll see why I was feeling malice towards unions when I wrote the comment: the presence of unions makes type identification needlessly difficult.  
  
So that's the explanation for the disparaging comment about unions, for whatever that's worth. And I'm still holding out some hope that we'll see an article on the actual technical content that we presented, and not the open sourcing of Solaris that we refused to talk about...
