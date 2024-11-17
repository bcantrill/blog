---
title: "Yet more blog sifting..."
date: "2005-06-20"
categories: 
  - "solaris"
---

Despite there being now four blog entries to sift through the [Opening Day entries](http://opensolaris.org/os/blogs/?startDate=2005-06-14) ([one from me](http://blogs.sun.com/roller/page/bmc/20050615#sifting_through_the_blogs), [one from Liane](http://blogs.sun.com/roller/page/lianep/20050615#opensolaris_blogs_networking_drivers_security), [one from Claire](http://blogs.sun.com/roller/page/cmh/20050620#sifting_opensolaris_blogs_from_tokyo) and [another from me](http://blogs.sun.com/roller/page/bmc/20050617#more_blog_sifting)), there are still some great entries that have gone uncategorized. Many of these entries fall into the category of "debugging war stories" -- engineers describing a particularly tough or interesting bug that they nailed. The proliferation of these kinds of stories among the Opening Day entries is revealing: in some organizations, you're a hero among engineers if you come up with a whizzy new app (even if the implementation is a cesspool), while in others you might gain respect among engineers by getting a big performance win (even if it makes things a bit flakey), but in Solaris development, nothing gains the respect of peers like debugging some diabolical bug that has plagued the operating system for years. So if you're new to [OpenSolaris](http://opensolaris.org) and you're looking to make a name for yourself, a good place to start is in the never-ending search for elusive bugs. To get a flavor for the thrill of the hunt, check out:

- [George Shepherd](http://blogs.sun.com/roller/page/georges) on [using DTrace to debug a nasty STREAMS bug](http://blogs.sun.com/roller/page/georges/20050614#dtrace_as_a_streams_debugger).
    
- [Jim Carlson](http://blogs.sun.com/roller/page/carlson) on [debugging a hang in `ldtermclose`](http://blogs.sun.com/roller/page/carlson/20050614#close_hang_or_4028137). You can infer how much we obsess about debugging from the first line of Jim's entry: "Every once in a while, a bug sticks with you long enough that you remember the ID number without having to think about it."
    
- [Adam Leventhal](http://blogs.sun.com/roller/page/ahl) on [debugging a cross call hang](http://blogs.sun.com/roller/page/ahl/20050614#debugging_cross_calls_on_opensolaris).
    
- [Chris Gerhard](http://blogs.sun.com/roller/page/chrisg) on a [two line fix in init](http://blogs.sun.com/roller/page/chrisg/20050614#two_line_fix_in_init).
    
- [Sarah Jelinek](http://blogs.sun.com/roller/page/sarahsblog) on [debugging a UFS file truncation bug](http://blogs.sun.com/roller/page/sarahsblog/20050614#debugging_a_ufs_file_truncation). For me personally, it's always very gratifying to see a bug like Sarah's -- where [DTrace](http://opensolaris.org/os/community/dtrace/) was guided by expert hands to help out on a tough problem.
    
- [Sherry Moore](http://blogs.sun.com/roller/page/sherrym) on [debugging a bug at the murky interface between compiler and operating system](http://blogs.sun.com/roller/page/sherrym/20050614#whose_bug_is_it_anyway).
    
- [Saurabh Mishra](http://blogs.sun.com/roller/page/saurabh_mishra) on [debugging a memory ordering problem](http://blogs.sun.com/roller/page/saurabh_mishra/20050614#compiler_reordering_problem).
    
- [Alok Aggarwal](http://blogs.sun.com/roller/page/aalok) on [debugging an NFSv4 problem on SPARC](http://blogs.sun.com/roller/page/aalok/20050614#debugging_on_sparc).
    
- [Narayana Kadoor](http://blogs.sun.com/roller/page/nkadoor) on [debugging a logic error in dynamic intimate shared memory (DISM)](http://blogs.sun.com/roller/page/nkadoor/20050614#small_logic_error_big_trouble).
    
- [Surya Prakki](http://blogs.sun.com/roller/page/sprakki) on [debugging kernel memory corruption](http://blogs.sun.com/roller/page/sprakki/20050614#kernel_memory_corruption_detected_and), surely the most pernicious of software pathologies.
    
- [Raja Gopal Andra](http://blogs.sun.com/roller/page/raja) on [debugging an application bug](http://blogs.sun.com/roller/page/raja/20050614#application_popper_mis_behaving_when).
    
- [Martin Englund](http://blogs.sun.com/roller/page/martin) on [tracking down a bug in the audit daemon](http://blogs.sun.com/roller/page/martin/20050614#a_quick_peek_into_a).
    
- [Peter Harvey](http://blogs.sun.com/roller/page/peteh) on [increasing UNIX group membership](http://blogs.sun.com/roller/page/peteh/20050614#increasing_unix_group_membership_easy). In this entry, Peter doesn't dwell on the bug -- which is clear in this case -- but rather the complexities of fixing it. It's a good example of how something that appears simple can be stubbornly complicated.
    
- [Prabahar Jeyaram](http://blogs.sun.com/roller/page/prabahar) on [debugging a nasty panic in the UFS lockfs protocol](http://blogs.sun.com/roller/page/prabahar/20050614#doctype_html_public_w3c_dtd).
    

And here are a few more on the simple (but thorough) satisfaction from fixing old bugs:

- [Paul Roberts](http://blogs.sun.com/roller/page/paulr) on [an ancient bug in `xargs`](http://blogs.sun.com/roller/page/paulr/20050614#a_simple_userland_bugfix).
    
- [Stacey Marshall](http://blogs.sun.com/roller/page/ace) on [a four year old bug in the name service switch](http://blogs.sun.com/roller/page/ace/20050614#investigating_nss). Stacey's entry touches on the larger satisfaction of going from a bug in a strange subsystem to understanding the code, developing the right fix, verifying the fix and then writing the test case to be sure. The dedication to this kind of craftsmanship -- taking the time to do it the Right Way, even for small stuff -- is what has drawn many of us to Solaris; it is, as Stacey says, "what I love about this job."
    
- [Darren Moffat](http://blogs.sun.com/roller/page/darren) on [a nine year old bug in `usermod`](http://blogs.sun.com/roller/page/darren/20050614#fixed_9_year_old_bug).
    

That about does it for today. There are still many more entries to categorize, but fortunately, I think I can see the light at the end of the tunnel -- or is that just the approach of death from the exhaustion of sifting through [all of this content](http://opensolaris.org/os/blogs/?startDate=2005-06-14)?

* * *

Technorati tags: [OpenSolaris](http://technorati.com/tag/OpenSolaris) [Solaris](http://technorati.com/tag/Solaris) [DTrace](http://technorati.com/tag/DTrace)
