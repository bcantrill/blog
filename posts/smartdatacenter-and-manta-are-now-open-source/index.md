---
title: "SmartDataCenter and Manta are now open source"
date: "2014-11-03"
---

Today we are announcing that we are open sourcing the two systems at the heart of our business: [SmartDataCenter](https://www.joyent.com/products/private-cloud) and the [Manta object storage platform](https://www.joyent.com/products/manta). SmartDataCenter is the container-based orchestration software that runs [the Joyent public cloud](https://www.joyent.com/products/compute-service); we have used it for the better half of a decade to run on-the-metal OS containers -- securely and at scale. Manta is our multi-tenant ZFS-based [object storage platform that provides first-class compute](http://queue.acm.org/detail.cfm?id=2645649) by allowing OS containers to be spun up directly upon objects -- effecting arbitrary computation at scale without data movement. The unifying technological foundation beneath both SmartDataCenter and Manta is [OS-based virtualization](http://www.slideshare.net/bcantrill/os-virtualization-40700689), a technology that Joyent pioneered in the cloud [way back in 2006](http://www.slideshare.net/bcantrill/joyent-circa-2006). We have long known the transformative power of OS containers, so it has been both exciting and validating for us to see the rise of Docker and the broadening of appreciation for OS-based virtualization. SmartDataCenter and Manta show that containers aren't merely a fad or developer plaything but rather a fundamental technological advance that represents the foundation for the next generation of computing -- and we believe that open sourcing them advances the adoption of container-based architectures more broadly.

Without any further ado -- and to assure that we don't fall into the most prominent of my own [corporate open source anti-patterns](https://www.youtube.com/watch?v=Pm8P4oCIY3g#t=21m59s) -- here is [the source for SmartDataCenter](https://github.com/joyent/sdc/) and [the source for Manta](https://github.com/joyent/manta/). These are sophisticated systems with many moving parts, and you'll see that these two repositories are in fact meta-repositories that explain the design of each of the systems and then point to the (many) components that comprise them (all now open source, natch). We believe that some of these subcomponents will likely find use entirely outside of SDC and Manta. For example, [Manatee](https://github.com/joyent/manatee) is a ZooKeeper-based system that manages Postgres replication and automates failover; [Moray](https://github.com/joyent/moray) is a key-value service that lives on top of Postgres. Taken together, Manatee and Moray implement a highly-available key-value service that we use as the foundation for many other components in SDC and Manta -- and one that we think others will find useful as well.

In terms of source code mechanics, you'll see that many of the components are implemented in either node.js or by extending C-based systems. This is not by fiat but rather by the choices of individual engineers; over the past four years, as we learned about [the nuances of node.js error handling](http://www.joyent.com/developers/node/design/errors) and as we invested heavily in [tooling for running node.js in production](http://www.joyent.com/developers/node/debug), node.js became the right tool for many of our jobs -- and we used it for many of the services that constitute SDC and Manta.

And because any conversation about open source has to address licensing at some point or another, let's get that out of the way: we opted for the [Mozilla Public License 2.0](https://www.mozilla.org/MPL/2.0/). While relatively new, there is a lot to like about this license: its file-based copyleft allows it to be proprietary-friendly while also forcing certain kinds of derived work to be contributed back; its explicit patent license discourages litigation, offering some measure of troll protection; its explicit warranting of original work obviates the need for a contributor license agreement ([we're not so into CLAs](http://dtrace.org/blogs/bmc/2014/06/11/broadening-nodejs/)); and (best of all, in my opinion), it has been explicitly designed to co-exist with other open source licenses in larger derived works. Mozilla did terrific work on MPL 2.0, and we hope to see it adopted by other companies that share our thinking around open source!

In terms of the business ramifications, at Joyent we have long been believers in open source as a business model; as the leaders of the [node.js](http://nodejs.org/) and [SmartOS](http://smartos.org/) projects, we have seen the power of open source to start new conversations, open up new markets and (importantly) yield new customers. Ten years ago, I wrote that [open source is "a loss leader -- minus the loss, of course"](http://dtrace.org/blogs/bmc/2004/08/28/the-economics-of-software/); after a decade of experience with open source business models, I would add that open source also serves as sales outreach without cold calls, as a channel without loss of margin, and as a marketing campaign without advertisements. But while we have directly experienced the business advantages of open source, we at Joyent have also lived something of a dual life: node.js and SmartOS have been open source, but the distributed systems that we have built using these core technologies have remained largely behind our walls. So that these systems are now open source does not change the fundamentals of our business model: if you would like to consume SmartDataCenter or Manta as a service, you can [spin up an instance on the public cloud](https://www.joyent.com/products/compute-service/pricing) or [use our Manta storage service](https://www.joyent.com/products/manta). Similarly, if you want a support contract and/or professional services to run either SmartDataCenter or Manta on-premises, [we'll sell them to you](https://www.joyent.com/products/private-cloud). Based on our past experiences with open source, we do know that there will be one important change: these technologies will find their way into the hands of those that we have no other way of reaching -- and that some fraction of these will become customers. Also based on past experience, we know that some (presumably much smaller) fraction of these new technologists will -- by merits of their interest in and contributions to these projects -- one day join us as engineers at Joyent. Bluntly, [open source is our farm system](https://www.youtube.com/watch?v=1KeYzjILqDo#t=55m55s), and broadening our hiring channel during a blazingly hot market for software talent is playing no small role in our decision here. In short, this is not an act of altruism: it is a business decision -- if a multifaceted one that we believe has benefits beyond the balance sheet.

Welcome to open source SDC and Manta -- and long-live the container revolution!