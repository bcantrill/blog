---
title: "Using DTrace to understand GNOME"
date: "2005-07-11"
categories: 
  - "solaris"
---

I read with some interest about the [GNOME startup bounty](http://www.gnome.org/bounties/Optimization.html#306042). As [Stephen O'Grady pointed out](http://www.redmonk.com/sogrady/archives/000809.html), this problem is indeed perfect for [DTrace](http://opensolaris.org/os/community/dtrace/). To get a feel for the problem, I wrote a very simple D script:

```
#!/usr/sbin/dtrace -s
#pragma D option quiet
proc:::exec-success
/execname == "gnome-session"/
{
start = timestamp;
go = 1;
}
io:::start
/go/
{
printf("%10d { -> I/O %d %s %s %s }\n",
(timestamp - start) / 1000000, pid, execname,
args[0]->b_flags & B_READ ? "reads" : "writes",
args[2]->fi_pathname);
}
io:::done
/go/
{
printf("%10d { fi_pathname);
}
io:::start
/go && ((struct buf *)arg0)->b_file != NULL &&
((struct buf *)arg0)->b_file->v_path == NULL/
{
printf("%10s   (vp %p)\n", "", ((struct buf *)arg0)->b_file);
}
io:::start
/go/
{
@apps[execname] = count();
@files[args[2]->fi_pathname] = count();
@appsfiles[execname, args[2]->fi_pathname] = count();
}
proc:::exec
/go/
{
self->parent = execname;
}
proc:::exec-success
/self->parent != NULL/
{
printf("%10d -> %d %s (from %d %s)\n",
(timestamp - start) / 1000000, pid, execname,
curpsinfo->pr_ppid, self->parent);
self->parent = NULL;
}
proc:::exit
/go/
{
printf("%10d pr_flag & PR_IDLE)/
{
printf("%10d [ idle ]\n",
(timestamp - start) / 1000000);
}
END
{
printf("\n  %-72s %s\n", "APPLICATION", "I/Os");
printa("  %-72s %@d\n", @apps);
printf("\n  %-72s %s\n", "FILE", "I/Os");
printa("  %-72s %@d\n", @files);
printf("\n  %-16s %-55s %s\n", "APPLICATION", "FILE", "I/Os");
printa("  %-16s %-55s %@d\n", @appsfiles);
}

```

This script uses a combination of CPU sampling and I/O tracing to determine roughly what's going on over the course of logging in. I ran the above script on on my [Acer Ferrari 3400](http://blogs.sun.com/roller/page/chandan?entry=pictures_solaris_on_ferrari) laptop running [OpenSolaris](http://opensolaris.org) by dropping to the command-line login after a reboot and running:

```
# nohup ./login.d > /var/tmp/login.out &

```

I then logged in, and made sure that the first thing I did was launch a gnome-terminal to `pkill dtrace`. (This could obviously be made quite a bit more precise, but it works as a crude way of indicating the completion of login.) Here is [the output from performing this experiment](http://blogs.sun.com/roller/resources/bmc/login.out). The first column is the millisecond offset from the start of gnome-session. CPU samples are contained within square brackets, launched programs are contained withing curly braces, and I/O is explicitly noted. An I/O summary is provided at the end. A couple of observations:

- Nearly one third of all I/O is to shared object text and read-only data. This is classic death-of-a-thousand-cuts, and it's hard to see that there's an easy way to fix this. But perhaps text could be reordered to save some I/O? More investigation is probably merited here.
    
- Over a quarter of all I/O is from [GConf](http://www.gnome.org/projects/gconf/) -- and many of these are from wandering around an expansive directory hierarchy looking for configuration information. It is well-known that the XML backend is a big performance problem, and a better backend is apparently being worked on. At any rate, solving this problem is clearly in [GConf's future](http://www.gnome.org/projects/gconf/plans.html).
    
- Of the remaining I/O, a bunch is to icon files. [Glynn Foster](http://www.gnome.org/~gman/) pointed out that this looks to be addressed by the [GTK+ gtk-update-icon-cache](http://developer.gnome.org/doc/API/2.2/gtk/gtk-update-icon-cache.html), new in GTK+ 2.6 and contained within [GNOME 2.10](http://www.gnomejournal.org/article/17/gnome-210-desktop-and-development-platform), so this experiment will obviously need to be repeated on GNOME 2.10.
    
- CPU utilization doesn't look like a big issue -- or it's one that is at least dwarfed by the cost of performing I/O. That said, gconfd-2 looks to be a bit piggy in this department as well. We're only using CPU sampling -- and we're using a pretty coarse grained sample -- and gconfd-2 still showed up. More precise investigation into CPU utilization can be performed with the `sched` provider and its `on-cpu` and `off-cpu` probes.
    

And an end-note: you might note many files named "`<unknown>`" in the output. This is due to I/Os being induced by lookups that are going through directories and/or symlinks that haven't been explicitly opened. For these I/Os, my script also gives you the pointer to the vnode structure in the kernel; you can get the path to these by using `::vnode2path` in [MDB](http://opensolaris.org/os/community/mdb/):

```
# mdb -k
Loading modules: [ unix krtld genunix specfs dtrace ufs ip
sctp usba uhci s1394 random nca lofs nfs audiosup sppp ptm ipc ]
> ffffffff8f6851c0::vnode2path
/usr/sfw/lib/libXrender.so.1
>

```

Yes, having to do this sucks a bit, and [it's a known issue](http://bugs.opensolaris.org/bugdatabase/view_bug.do?bug_id=6175313). And rumor has it that [Eric](http://blogs.sun.com/eschrock) even has a workspace with the fix, so stay tuned...

**Update**: Eric pointed me to a prototype with the fix, and I reran the script on a GNOME cold start; [here is the output from that run](http://blogs.sun.com/roller/resources/bmc/login.out-clean). Interestingly, because the symlinks now show up, a little postprocessing reveals that we chased nearly eighty symlinks on startup! From the output, reading many of these took 10 to 20 milliseconds; we might be spending as much as one second of GNOME startup blocked on I/O just to chase symlinks! Ouch! Again, it's not clear how one would fix this; having an app link to `libpango-1.0.so.0` and not `libpango-1.0.so.400.1` is clearly a Good Thing, and having this be a symlink instead of a hardlink is clearly a Good Thing -- but all of that goodness leaves you with a read dependency that's hard to work around. Anyway, be looking for Eric's fix in [OpenSolaris](http://opensolaris.org) and then in an upcoming [Solaris Express](http://www.sun.com/software/solaris/solaris-express/) release; it makes this kind of analysis much easier -- and thanks Eric for the quick prototype!

* * *

Technorati tags: [OpenSolaris](http://technorati.com/tag/OpenSolaris) [Solaris](http://technorati.com/tag/Solaris) [DTrace](http://technorati.com/tag/DTrace) [GNOME](http://technorati.com/tag/GNOME)
