---
title: "Catching disk latency in the act"
date: "2009-01-01"
categories: 
  - "fishworks"
---

Today, [Brendan](http://blogs.sun.com/brendan) made a very interesting discovery about the potential sources of disk latency in the datacenter. Here's [a video we made](http://www.youtube.com/watch?v=tDacjrSCeq4) of Brendan explaining (and demonstrating) his discovery:

This may seem silly, but it's not farfetched: Brendan actually made this discovery while exploring drive latency that he had seen in a lab machine due to a missing screw on a drive bracket. (!) Brendan has [more details on the discovery](http://blogs.sun.com/brendan/entry/unusual_disk_latency), demonstrating how he used the [Fishworks analytics](http://dtrace.org/resources/bmc/cec_analytics.pdf) to understand and visualize it.

If this has piqued your curiosity about the nature of disk mechanics, I encourage you to read Jon Elerath's excellent [ACM Queue](http://queue.acm.org/) article, [Hard disk drives: the good, the bad and the ugly!](http://doi2.acm.org/1317394.1317403) As Jon notes, noise is a known cause of what is called a non-repeatable runout (NRRO) -- though it's unclear if Brendan's shouting is exactly the kind of noise-induced NRRO that Jon had in mind...
