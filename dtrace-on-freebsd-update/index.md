---
title: "DTrace on FreeBSD, update"
date: "2006-05-25"
categories: 
  - "solaris"
---

A while ago, I blogged about the possibility of a [FreeBSD port of DTrace](http://blogs.sun.com/roller/page/bmc?entry=dtrace_on_freebsd). For the past few months, [John Birrell](http://people.freebsd.org/~jb/) has been hard at work on the port, and has [announced recently](http://marc.theaimsgroup.com/?l=freebsd-current&m=114854018213275&w=2) that he has much of the core DTrace functionality working. Over here at [DTrace Central](http://www.flickr.com/photos/66572791@N00/18206393/), we've been excitedly watching John's progress for a little while, providing help and guidance where we can -- albeit not always solicited ;) -- and have been very impressed with how far he's come. And while John has quite a bit [further to go](http://people.freebsd.org/~jb/dtrace/todo.html) before one could call it a complete port, what he has now is indisputably useful. If you run FreeBSD in production, you're going to want John's port as it stands today -- and if you develop for the FreeBSD kernel (drivers or otherwise), you're going to _need_ it. (Once you've done kernel development with DTrace, there's no going back.)

So this is obviously a win for FreeBSD users, who can now benefit from the leap in software observability that DTrace provides. It's also clearly a win for DTrace users, because now you have another platform on which you can observe your production software -- and a larger community with whom to share your experiences and thoughts. And finally, it's a huge win for OpenSolaris users: the presence of a FreeBSD port of DTrace validates that OpenSolaris _is_ an open, innovative platform (despite what [some buttheads say](http://www.theregister.co.uk/2006/05/24/bea_open_source_sun_jboss/)) -- one that will benefit from and contribute to the undeniable economics of open source.

So congrats to John! And to the FreeBSD folks: welcome to [the DTrace community](http://www.opensolaris.org/os/community/dtrace/)!
