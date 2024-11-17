---
title: "DTrace on Mac OS X!"
date: "2006-08-07"
categories: 
  - "solaris"
---

From WWDC here in San Francisco: Apple has just announced [support for DTrace in Leopard](http://www.apple.com/macosx/leopard/xcode.html), the upcoming release of Mac OS X! Often (or even usually?) announcements at conferences are more vapor than ware. In this case, though, Apple is being quite modest: they have done a tremendous amount of engineering work to bring DTrace to their platform (including, it turns out, implementing DTrace's FBT provider for PowerPC!), and they are using DTrace as part of the foundation for their new Xray performance tool. This is very exciting news, as it brings DTrace to a whole slew of new users. (And speaking personally, it will be a relief to finally have DTrace on the only computer under my roof that doesn't run Solaris!) Having laid hands on DTrace on Mac OS X myself just a few hours ago, I can tell you that while it's not yet a complete port, it's certainly enough to be uniquely useful -- and it was quite a thrill to see Objective C frames in a `ustack` action! So kudos to the Apple engineering team working on the port: Steve Peters, James McIlree, Terry Lambert, Tom Duffy and Sean Callanan.

It's been fun for us to work with the Apple team, and gratifying to see their results. And it isn't just rewarding for us; the entire OpenSolaris community should feel proud about this development because it gives the lie to [IBM's nauseating assertion](http://blogs.sun.com/roller/page/jimgris?entry=ibm_s_ross_mauri) that we in the OpenSolaris community aren't in the "spirit" of open source.

So to Apple users: welcome to DTrace! (And to DTrace users: welcome to Mac OS X!) Be sure to join us in the [DTrace community](http://www.opensolaris.org/os/community/dtrace/) -- and if you're at the WWDC, we on [Team DTrace](http://www.flickr.com/photos/66572791@N00/18206393/) will be on hand on Friday at the DTrace session, so swing by to see a demo of DTrace running on MacOS and to meet both the team at Apple that worked on the port and we at Sun who developed DTrace. And with a little luck, you might even hear [Adam](http://blogs.sun.com/ahl) commemorate the occasion by singing his beautiful and enchanting [ISA](http://en.wikipedia.org/wiki/Instruction_set) aria...

**Update:** For disbelievers, [Adam posted photos](http://blogs.sun.com/roller/page/ahl?entry=dtrace_on_mac_os_x) -- and [Mike went into greater detail](http://blogs.sun.com/roller/page/mws#dtrace_on_macos_x_at) on the state of the Leopard DTrace port, and what it might mean for the direction of DTrace.
