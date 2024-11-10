---
title: "DTrace vs. DProbes/LTT"
date: "2004-07-17"
categories: 
  - "solaris"
---

Recently, Karim Yaghmour posted [the following](http://groups.google.com/groups?selm=2g5aM-5mq-27%40gated-at.bofh.it&output=gplain) to the [linux-kernel mailing list](http://lkml.org):

> ```
> As I noted when discussing this with Andrew, we've been trying to get
> LTT into the kernel for the past five (5) years. During that time we've
> repeatedly encountered the same type of arguments for not including it,
> and have provided proof as to why those arguments are not substantiated.
> Lately I've at least got Andrew to admit that there were no maintenance
> issues with the LTT trace statements (given that they've literally
> remained unchanged ever since LTT was introduced.) In an effort to
> address the issues regarding the usefulness of such a tool, I direct
> those interested to this article on DTrace, a trace utility for Solaris:
> http://www.theregister.co.uk/2004/07/08/dtrace_user_take/
> <rant>
> With LTT and DProbes, we've basically got almost everything this tool
> claims to provide, save that we would be even further down the road if
> we did not need to spend so much time updating patches ...
> </rant>
> Karim
> --
> Author, Speaker, Developer, Consultant
> 
> ```

Now, Karim's really only interested in [DTrace](http://www.sun.com/bigadmin/content/dtrace) it that it helps him make his larger point that his project has been unfairly (or unwisely) denied entry into the Linux kernel. His is a legitimate point, and something that is often lost in the assertions that Linux is developed faster than other operating systems: for all of its putative development speed, Linux has a surprising number of otherwise valuable projects that have been repeatedly denied entry for reasons that seem to be petty and non-technical. DProbes/LTT is certainly one example of such a project, and [LKCD](http://librenix.com/?inode=525) is probably another.

But what of Karim's assertion that LTT and DProbes "basically \[have\] everything \[DTrace\] claims to provide"? This claim is false, and indicates that while Karim may have scanned [The Register article](http://www.theregister.co.uk/2004/07/08/dtrace_user_take/), he didn't bother to browse even [our USENIX paper](http://www.sun.com/bigadmin/content/dtrace/dtrace_usenix.pdf) -- let alone [our documentation](http://www.sun.com/bigadmin/content/d10_latest.pdf). From these, one will note that while LTT lacks many DTrace niceties, it also lacks several vital features. Two among these are [aggregations](http://docs.sun.com/db/doc/817-6223/6mlkidli7?a=view) and [thread-local variables](http://docs.sun.com/db/doc/817-6223/6mlkidlft?a=view) -- two features that are not syntactic sugar or bolted-on afterthoughts, but rather are core to the DTrace architecture. These features turn out to be **essential** in using DTrace to quickly resolve problems. For an example of how these features are used, see Section 9 of [our USENIX paper](http://www.sun.com/bigadmin/content/dtrace/dtrace_usenix.pdf) -- and note that every script that we wrote to debug that problem used aggregations, and that several critical steps were only possible with thread-local variables.

And fortunately, you don't even have to take my word for it: [RedHat](http://redhat.com) developer [Daniel Berrangé](http://berrange.com) has posted a [comparison of DTrace and DProbes/LTT](http://berrange.com/bitsbobs/mlp/2004/07/dtrace) that reaches roughly the same conclusions...
