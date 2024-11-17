---
title: "UNIX, circa 1984"
date: "2004-11-24"
---

I recently ran across [Xhomer](http://xhomer.isani.org/xhomer/), a simulator for the [DEC Pro 350](http://www.vintage-computer.com/dec_pro_350.shtml). While there are several [historic machine simulators](http://www.sosresearch.org/caale/caalesimulators.html#Historic) out there, Xhomer is polished: it compiled and ran on my Opteron laptop running Solaris 10 with no difficulties. But best of all, Xhomer includes system software: you can download a disk image of VENIX 2.0 -- a "real-time" UNIX variant from VenturCom. Here's a [screenshot of me logging in](images/welcome.png).

While I had heard of VENIX (a System III derivative), I had never used it before today. (And nor, presumably, have the good folks at [pharmanex](http://www.fightcholesterol.com/venix/).) In using it, it's clear that VENIX had some BSD influence: VENIX 2.0 includes both [csh](http://docs.sun.com/app/docs/doc/816-5165/6mbb0m9du?a=view) (blech) and [vi](http://docs.sun.com/app/docs/doc/816-5165/6mbb0ma0u?a=view) (phew). Using a twenty year old UNIX is a strange experience: I'm amazed at how little the most basic things have changed. There was very little that I didn't recognize ("em1" anyone?), and I was familiar with all of the tools necessary to write a program ([vi](http://docs.sun.com/app/docs/doc/816-5165/6mbb0ma0u?a=view)), compile it ([cc](http://docs.sun.com/app/docs/doc/816-5165/6mbb0m9cg?a=view)) and debug it ([adb](http://docs.sun.com/app/docs/doc/816-5165/6mbb0m9b5?a=view)). Using the latter of these was the most amusing; here's a [screenshot of me using adb](images/adb.png).

Compare this output with the output of "$a" on [adb](http://docs.sun.com/app/docs/doc/816-5165/6mbb0m9b5?a=view), and you'll see why this got me excited. (And then try "$a" on [mdb](http://docs.sun.com/app/docs/doc/816-5165/6mbb0m9lr?a=view) for our warped idea of an easter egg.) It should go without saying that I looked (so far in vain) for an Algol 68 compiler on this system. Seeing adb spit out a true Algol stack backtrace would be like sipping from the debugging fountain of youth...

While it's amazing how familiar VENIX feels, I'm also stunned by how anemic its facilites are: it has no TCP/IP stack, no real filesystem, no multiple processor support, no resource management, no dynamic linking, no real virtual memory system, no observability and poor debuggability. It reminds me how far we have come -- even before we embarked on the far more radical technologies found in [Solaris 10](http://wwws.sun.com/software/solaris/10/)...

Now, does anyone know of a [Language H](http://foldoc.doc.ic.ac.uk/foldoc/foldoc.cgi?Language+H) compiler for the PDP-11?

`OBEY`
