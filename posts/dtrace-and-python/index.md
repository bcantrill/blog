---
title: "DTrace and Python"
date: "2005-05-10"
categories: 
  - "solaris"
---

[Sean McGrath](http://blogs.sun.com/smg) has a [great blog entry](http://blogs.sun.com/roller/page/smg/20050510#beer_python_and_stuff) on adding DTrace probes to Python. As Sean points out, this isn't a perfect solution -- but damn is it cool! For whatever it's worth, we're currently working on infrastructure that will address many of the problems that Sean discussed; once completed, this infrastructure will allow for user-level providers to export the same semantic richness as the stable kernel-level providers like `io`, `sched` and `proc`. But as Sean's work shows, you can still do very interesting things even with what exists today...  

* * *

Technorati tag: [DTrace](http://technorati.com/tag/DTrace)
