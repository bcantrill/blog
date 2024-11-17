---
title: "Still more blog sifting..."
date: "2005-06-25"
categories: 
  - "solaris"
---

Even though the launch of [OpenSolaris](http://opensolaris.org) was well over a week ago, and even though the [Opening Day entries](http://opensolaris.org/os/blogs/?startDate=2005-06-14) have now been sifted through in _five_ different blog entries ([here](http://blogs.sun.com/roller/page/bmc/20050615#sifting_through_the_blogs), [here](http://blogs.sun.com/roller/page/lianep/20050615#opensolaris_blogs_networking_drivers_security), [here](http://blogs.sun.com/roller/page/cmh/20050620#sifting_opensolaris_blogs_from_tokyo) [here](http://blogs.sun.com/roller/page/bmc/20050617#more_blog_sifting) and [here](http://blogs.sun.com/roller/page/bmc?entry=yet_more_blog_sifting)), there are _still_ some great uncategorized entries. Without further ado...

- **Zones**. [Zones](http://opensolaris.org/os/community/zones/) are among the most talked-about new features in Solaris, and the engineers on the team have developed some highly-readable Opening Day entries describing the implementation. Start with [David Comay's](http://blogs.sun.com/roller/page/comay) entry taking you on a [tour of zone state transitions](http://blogs.sun.com/roller/page/comay/20050614#these_boots_are_made_for), and then check out [Dan Price's](http://blogs.sun.com/roller/page/dp) entry to extend your tour to the [innards of the zones console](http://blogs.sun.com/roller/page/dp/20050614#inside_the_zones_console_a). Dan's tour is notable because he takes you by an [ASCII art comment of his](http://cvs.opensolaris.org/source/xref/usr/src/cmd/zoneadmd/zcons.c) that is one of the more elaborate in Solaris. (One of these days, I'll take you on a tour of my favorite ASCII art comments in the source base -- but today we have too much to see, so everyone back in the car!) Wrap up your tour of zones with [John Beck's](http://blogs.sun.com/roller/page/jbeck) entry describing [adding command-line editing and history to zonecfg](http://blogs.sun.com/roller/page/jbeck/20050614#adding_command_line_editing_history).This entry has applicability beyond zones -- it's a useful how-to for adding command-line editing and history to any C-based or C++-based application.
    
- **Booting**. The most exciting project to integrate into Solaris since Solaris 10 shipped is surely the reachitecture of the booting system to use GRUB -- and [Shudong Zhou's](http://blogs.sun.com/roller/page/szhou) entry describing [testing the code at Fry's](http://blogs.sun.com/roller/page/szhou/20050614#booting_solaris_x86_at_fry) is a must-read. For more details, check out [Jan Setje-Eilers'](http://blogs.sun.com/roller/page/setje) entry describing [the new boot architecture](http://blogs.sun.com/roller/page/setje/20050614#a_new_boot_architecture). Look for more from these two -- using GRUB allows for many new possibilities, and Shudong and Jan have much to blog about.
    
- **Solaris Volume Manager**. Despite the fact that it could save them gobs in licensing fees to certain [third parties](http://www.veritas.com), many seem to not realize that we bundle an [industrial-strength volume manager](http://www.informit.com/articles/article.asp?p=169475&rl=1) with Solaris. Hell-bent on describing what they've been working on for so long, the Solaris Volume Manager team was out in force on Opening Day. If you have any storage responsibilities in your shop and you're not running SVM, you should carefully read these entries; SVM may well save you a bundle -- making you such a hero that you'll be able to tell the suits exactly what they can do with their [TPS report](http://www.wormulon.net/files/TPSreport.pdf).
    
    - [Andre Molyneux](http://blogs.sun.com/roller/page/andresblog) on [RAID 0+1 vs. RAID 1+0](http://blogs.sun.com/roller/page/andresblog/20050614#raid_0_1_vs_raid) and again on [the role of timing in testing RAID 5 in SVM](http://blogs.sun.com/roller/page/andresblog/20050615#timing_is_everything).
        
    - [Jerry Jelinek](http://blogs.sun.com/roller/page/jerrysblog) on [improving SVM response to disk failure](http://blogs.sun.com/roller/page/jerrysblog/20050614#svm_and_the_b_failfast).
        
    - [Sanjay Nadkarni](http://blogs.sun.com/roller/page/nadkarni) on [resync regions and optimized resyncs](http://blogs.sun.com/roller/page/nadkarni/20050614#resync_regions_and_optimized_resyncs) -- a feature which VxVM users will recognize as the SVM equivalent of dirty region logging (DRL).
        
    - [Susan Kamm-Worrell](http://blogs.sun.com/roller/page/skamm) on [multi-owner disksets](http://blogs.sun.com/roller/page/skamm/20050614#multi_owner_diskset_in_svm), an SVM feature that allows multiple nodes to simultaneously access volumes.
        
    - [Steve Peng](http://blogs.sun.com/roller/page/stevep) on [disk Relocation in SVM](http://blogs.sun.com/roller/page/stevep/20050614#title_open_solaris_blog_title).
        
    - [Tony Nguyen](http://blogs.sun.com/roller/page/tonyn) on the [SVM default interlace and resync buffer values](http://blogs.sun.com/roller/page/tonyn/20050614#svm_default_interlace_and_resync).
        
    
- **Miscellany**. There are a handful of great entries that don't fit neatly into any one category -- or are so specific as to be their own category. Be sure not to miss these:
    
    - [Jeff Bonwick](http://blogs.sun.com/roller/page/bonwick) on [revealing the origins of the slab allocator](http://blogs.sun.com/roller/page/bonwick/20050614#now_it_can_be_told).
        
    - [Phil Harman](http://blogs.sun.com/roller/page/pgdh) on [getting getenv to scale](http://blogs.sun.com/roller/page/pgdh/20050614#caring_for_the_environment_making). Because so much Solaris scalability work happened many years ago, Phil's is the only entry describing the specifics of getting a subsystem to scale with CPUs; Phil's entry should be considered a must-read for anyone working on scalability.
        
    - [Peter Memishian](http://blogs.sun.com/roller/page/meem) on [using doors as a synchronization primitive](http://blogs.sun.com/roller/page/meem?entry=head_title_dsvclockd_1m_using). Peter's work is a good example of why -- contrary to the [beliefs of some pinheads](http://marc.theaimsgroup.com/?l=git&m=111349140005505&w=2) -- developing a user-level service provider can be quite a bit more challenging than developing in the kernel.
        
    - [Tim Marsland](http://blogs.sun.com/roller/page/tpm) continuing his _magnum opus_ on the implementation of Solaris 10 on x64 with [Part 4: Userland](http://blogs.sun.com/roller/page/tpm/20050614#solaris_10_on_x64_processors3).
        
    - [Ienup Sung](http://blogs.sun.com/roller/page/is) on [efficiently handling illegal UTF-8 byte sequences](http://blogs.sun.com/roller/page/is/20050614#secure_utf_8).
        
    - [Chandan](http://blogs.sun.com/roller/page/chandan) describing the [implementation of the OpenSolaris Source Browser](http://blogs.sun.com/roller/page/chandan/20050614#opensolaris_source_browser)
        
    - [Cyndi Eastham](http://blogs.sun.com/roller/page/ceastha) on [developing libavl](http://blogs.sun.com/roller/page/ceastha/20050613#in_search_of).
        
    - [Chris Beal](http://blogs.sun.com/roller/page/cwb) describing [the implementation of signal delivery](http://blogs.sun.com/roller/page/cwb/20050614#doctype_html_public_w3c_dtd).
        
    - [Dave Miner](http://blogs.sun.com/roller/page/dminer) leading a [tour of the DHCP server](http://blogs.sun.com/roller/page/dminer/20050613#dhcp_server_tour).
        
    - [Dave Powell](http://blogs.sun.com/roller/page/dep) on the [implementation of pseudo-filesystems](http://blogs.sun.com/roller/page/dep/20050614#simplified_psuedo_filesystem_implementation) -- and why `/system` is the new home for all such filesystems.
        
    - [Jonathan Adams](http://blogs.sun.com/roller/page/jwadams) on [macros and powers of two](http://blogs.sun.com/roller/page/jwadams/20050615#macros_and_powers_of_two) -- and the simple pleasures of [bit-twiddling](http://catb.org/~esr/jargon/html/B/bit-twiddling.html).
        
    - [Tom Erickson](http://blogs.sun.com/roller/page/tomee) on [the implementation of libdtrace](http://blogs.sun.com/roller/page/tomee/20050614#tom_erickson_s_weblog). libdtrace is still a private interface (we still have some sanding and polishing to do), but Tom's entry is a must-read for anyone considering writing their own DTrace consumer.
        
    - [Keith M Wesolowski](http://blogs.sun.com/roller/page/wesolows) on [getting SPARC inlines to work with GCC](http://blogs.sun.com/roller/page/wesolows/20050614#the_first_opensolaris_project_gcc1).
        

Phew! I think that about does it. When we first tried to encourage Solaris engineers to blog on Opening Day, I thought we were going to have a hard time convincing engineers to blog -- I knew that providing in-depth, technical content takes a lot of time, and I knew that everyone had other priorities. So when we were planning the launch and talking about the possibility of dealing with a massive amount of Opening Day content, my response was "hurt me with that problem." Well, as it turns out, most engineers didn't need much convincing -- many provided rich, deep content -- and I was indeed hurt with that problem! While it was time consuming to sift through them, hopefully you've enjoyed reading these entries as much as I have. And let it be said once more: welcome to [OpenSolaris](http://opensolaris.org)!

* * *

Technorati tags: [OpenSolaris](http://technorati.com/tag/OpenSolaris) [Solaris](http://technorati.com/tag/Solaris) [DTrace](http://technorati.com/tag/DTrace)
