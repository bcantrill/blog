---
title: "DTrace is not a security risk!"
date: "2005-03-02"
---

Recently, there was [a presentation](http://www.ccc.de/congress/2004/fahrplan/event/57.de.html) at the annual meeting of [Chaos Computer Club](http://en.wikipedia.org/wiki/Chaos_Computer_Club) in Berlin. As the presentation describes DTrace at some length, several have asked the question: is DTrace a security risk? The answer is an emphatic "no" -- quite the contrary in fact -- but it merits some explanation.

DTrace can only be used by users on the system that have the appropriate privileges (as discussed in the Security chapter of the [DTrace documentation](http://docs.sun.com/app/docs/doc/817-6223)). By default, the only user with sufficient privileges to use DTrace is `root` -- the super-user. The techniques described in the paper and in the presentation are only for use on a system **that one has already compromised**. Of course, once a system is compromised, all bets are off; a nefarious user can:

- Load their own daemons to act as [trojan horses](http://en.wikipedia.org/wiki/Trojan_horse_%28computing%29), potentially sniffing passwords and compromising subsequent machines
    
- Examine `/etc/shadow` and crack it to obtain cleartext for every password on the system
    
- Use the pre-existing Solaris observability tools (`truss`(1), `gcore`(1), `mdb`(1), etc.) to observe and modify arbitrary processes
    
- Crash and/or destroy the system beyond repair
    
- Load their own kernel modules to spoof arbitrary parts of the system
    

Yes, you can use DTrace on a compromised system to glean additional information, but everything you can do with DTrace was in principle possible before DTrace -- DTrace just happens to make it a little easier. Indeed, the presentation doesn't even discuss the ways in which a nefarious user on a compromised system can use DTrace -- rather it describes how DTrace can be used to _understand_ the system well enough to design a nefarious spoofing kernel module in the first place. And revealingly, the presentation spends quite a bit of time describing how to design a nefarious kernel module such that it _evades_ instrumentation by DTrace.[^1] The fact that time and effort were spend on DTrace evasion is telling: as a tool designed to expose the inner workings of a production system, DTrace is much more feared by the Black Hats than it is useful to them; far from being a security _risk_, DTrace is very much a security _asset_.

[^1]: I hasten to add that the author's techniques for evading DTrace won't actually work completely. They will successfully evade one form of instrumentation, but they leave the nefarious module completely exposed to several other forms of instrumentation and detection by DTrace. A more devilish rootkit would completely replace DTrace with some sort of [Bizarro](http://theages.superman.ws/Encyclopaedia/bizarro.php) DTrace that knew how to _completely_ deny the existence of its cohorts...

