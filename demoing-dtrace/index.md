---
title: "Demo’ing DTrace"
date: "2004-08-05"
---

With my explanation of a [demo gone wrong](/2004/07/26/demo-perils), several people have asked me the more general question: how does one demo [DTrace](http://www.sun.com/bigadmin/content/dtrace)? This question doesn't have a single answer, especially given that DTrace is best demonstrated with _ad hoc_ queries of the system. Indeed, the best demos are when someone in the audience shouts out their own question that they want to see answered: answering such a question instantly with DTrace nearly always blows away the questioner -- who has presumably suffered in the past trying to answer similar questions. Of course, saying "why, there are many ways to demo to DTrace!" is a useless answer to the question of how one demos DTrace; while there are many ways that one _can_ demo DTrace, it's useful to get a flavor of how a typical demo might go. So with the substantial caveat that this is not _the_ way to demo DTrace, but merely _a_ way, let me walk you through an example demo.

It all starts by running [dtrace](http://docs.sun.com/db/doc/816-5166/6mbb1kq0f?a=view)(1M) without any arguments:

```
# dtrace
Usage: dtrace [-32|-64] [-aACeEFGHlqSvVwZ] [-b bufsz] [-c cmd] [-D name[=def]]
        [-I path] [-o output] [-p pid] [-s script] [-U name]
        [-x opt[=val]] [-X a|c|s|t]

        [-P provider [[ predicate ] action ]]
        [-m [ provider: ] module [[ predicate ] action ]]
        [-f [[ provider: ] module: ] func [[ predicate ] action ]]
        [-n [[[ provider: ] module: ] func: ] name [[ predicate ] action ]]
        [-i probe-id [[ predicate ] action ]] [ args ... ]

        predicate -> '/' D-expression '/'
           action -> '{' D-statements '}'

        -32 generate 32-bit D programs and ELF files
        -64 generate 64-bit D programs and ELF files

        -a  claim anonymous tracing state
        -A  generate driver.conf(4) directives for anonymous tracing
        -b  set trace buffer size
        -c  run specified command and exit upon its completion
        -C  run cpp(1) preprocessor on script files
        -D  define symbol when invoking preprocessor
        -e  exit after compiling request but prior to enabling probes
        -E  exit after enabling probes but prior to tracing data
        -f  enable or list probes matching the specified function name
        -F  coalesce trace output by function
        -G  generate an ELF file containing embedded dtrace program
        -H  print included files when invoking preprocessor
        -i  enable or list probes matching the specified probe id
        -I  add include directory to preprocessor search path
        -l  list probes matching specified criteria
        -m  enable or list probes matching the specified module name
        -n  enable or list probes matching the specified probe name
        -o  set output file
        -p  grab specified process-ID and cache its symbol tables
        -P  enable or list probes matching the specified provider name
        -q  set quiet mode (only output explicitly traced data)
        -s  enable or list probes according to the specified D script
        -S  print D compiler intermediate code
        -U  undefine symbol when invoking preprocessor
        -v  set verbose mode (report program stability attributes)
        -V  report DTrace API version
        -w  permit destructive actions
        -x  enable or modify compiler and tracing options
        -X  specify ISO C conformance settings for preprocessor
        -Z  permit probe descriptions that match zero probes

```

I usually just point out this just gives your typical Unix-ish help message -- implicitly making the point that dipping into DTrace doesn't require much more than typing its name on the command line. From the output of the usage message, I highlight the "-l" option, noting that it lists all the probes in the system. I then run with that option -- being sure to pipe the output to [more](http://docs.sun.com/db/doc/816-5165/6mbb0m9jd?a=view)(1):

```
# dtrace -l | more
   ID   PROVIDER            MODULE                          FUNCTION NAME
    1     dtrace                                                     BEGIN
    2     dtrace                                                     END
    3     dtrace                                                     ERROR
    4   lockstat           genunix                       mutex_enter adaptive-acquire
    5   lockstat           genunix                       mutex_enter adaptive-block
    6   lockstat           genunix                       mutex_enter adaptive-spin
    7   lockstat           genunix                        mutex_exit adaptive-release
    8   lockstat           genunix                     mutex_destroy adaptive-release
    9   lockstat           genunix                    mutex_tryenter adaptive-acquire
   10   lockstat           genunix                          lock_set spin-acquire
   11   lockstat           genunix                          lock_set spin-spin
   12   lockstat           genunix                      lock_set_spl spin-acquire
   13   lockstat           genunix                      lock_set_spl spin-spin
   14   lockstat           genunix                          lock_try spin-acquire
   15   lockstat           genunix                        lock_clear spin-release
   16   lockstat           genunix                   lock_clear_splx spin-release
   17   lockstat           genunix                      CLOCK_UNLOCK spin-release
   18   lockstat           genunix                          rw_enter rw-acquire
   19   lockstat           genunix                          rw_enter rw-block
   20   lockstat           genunix                           rw_exit rw-release
   21   lockstat           genunix                       rw_tryenter rw-acquire
   22   lockstat           genunix                     rw_tryupgrade rw-upgrade
   23   lockstat           genunix                      rw_downgrade rw-downgrade
   24   lockstat           genunix                       thread_lock thread-spin
   25   lockstat           genunix                  thread_lock_high thread-spin
   26   fasttrap                                            fasttrap fasttrap
   27    syscall                                               nosys entry
   28    syscall                                               nosys return
   29    syscall                                               rexit entry
   30    syscall                                               rexit return
--More--

```

I point out that each of these rows is a point of instrumentation. I then explain each of the columns, explaining that the `PROVIDER` column denotes the DTrace notion of [instrumentation providers](http://docs.sun.com/db/doc/817-6223/6mlkidlef?a=view) which allows for disjoint instrumentation techniques. If the audience is a particularly savvy one, I may point out that the [`lockstat` provider](http://docs.sun.com/db/doc/817-6223/6mlkidllf?a=view) is a recasting of the instrumentation technique used by [lockstat](http://docs.sun.com/db/doc/816-5166/6mbb1kq58?a=view)(1M), and that `lockstat` has been made to be a DTrace consumer. I mention that the `MODULE` column contains the name of the kernel module (in the case of a kernel probe) or shared object name (in the case of a user-level probe), that the `FUNCTION` column contains the function that the probe is in, and that the `NAME` column contains the name of the probe. I explain that a each probe is uniquely identified by the probe _tuple_: provider, module, function and name.

Scrolling down through the output, I point out the probes from [`syscall` provider](http://docs.sun.com/db/doc/817-6223/6mlkidln3?a=view), describing its ability to instrument every system call entry and return. (For some audiences, I may pause here to remind them of the definition of a [system call](http://docs.sun.com/db/doc/816-5167/6mbb2jaet?a=view).)

I then continue to scroll down my probe listing until I get to output that looks something like this:

```
 ...
 1268        fbt              unix                      dvma_release entry
 1269        fbt              unix                      dvma_release return
 1270        fbt              unix                        softlevel1 entry
 1271        fbt              unix                        softlevel1 return
 1272        fbt              unix                        freq_notsc entry
 1273        fbt              unix                        freq_notsc return
 1274        fbt              unix                         psm_unmap entry
 1275        fbt              unix                         psm_unmap return
 1276        fbt              unix             cpu_visibility_online entry
 1277        fbt              unix             cpu_visibility_online return
 1278        fbt              unix                         setbackdq entry
 1279        fbt              unix                         setbackdq return
 1280        fbt              unix                       lgrp_choose entry
 1281        fbt              unix                       lgrp_choose return
 1282        fbt              unix              cpu_intr_swtch_enter entry
 1283        fbt              unix              cpu_intr_swtch_enter return
 1284        fbt              unix                      page_upgrade entry
 1285        fbt              unix                      page_upgrade return
 1286        fbt              unix                      page_lock_es entry
 1287        fbt              unix                      page_lock_es return
 1288        fbt              unix               lgrp_shm_policy_set entry
 1289        fbt              unix               lgrp_shm_policy_set return
 1290        fbt              unix                        segkmem_gc entry
 1291        fbt              unix                        segkmem_gc return
 1292        fbt              unix                      disp_anywork entry
 1293        fbt              unix                      disp_anywork return
 1294        fbt              unix                   prgetprxregsize entry
 1295        fbt              unix                   prgetprxregsize return
 1296        fbt              unix                       cpuid_pass1 entry
 1297        fbt              unix                       cpuid_pass1 return
 1298        fbt              unix                       cpuid_pass2 entry
 1299        fbt              unix                       cpuid_pass2 return
 1300        fbt              unix                       cpuid_pass3 entry
 1301        fbt              unix                       cpuid_pass3 return
 1302        fbt              unix                       cpuid_pass4 entry
 1303        fbt              unix                       cpuid_pass4 return
 1304        fbt              unix                 release_bootstrap entry
 1305        fbt              unix                 release_bootstrap return
 1306        fbt              unix          i_ddi_rnumber_to_regspec entry
 1307        fbt              unix          i_ddi_rnumber_to_regspec return
 1308        fbt              unix                    page_mem_avail entry
 1309        fbt              unix                    page_mem_avail return
 1310        fbt              unix                      page_pp_lock entry
 1311        fbt              unix                      page_pp_lock return
 ...

```

I pause here to explain that the [`fbt` provider](http://docs.sun.com/db/doc/817-6223/6mlkidlmf?a=view) can instrument every function entry and return in the kernel. Making the observation that there are quite a few functions in the kernel, I then quit out of the output, and re-run the command -- this time piping the output through [`wc`](http://docs.sun.com/db/doc/816-5165/6mbb0m9r8?a=view). Usually, the output is something like this:

```
# dtrace -l | wc -l
   32521

```

Anyone who has dealt with [traditional instrumentation frameworks](http://docs.sun.com/db/doc/816-5174/6mbb98ujq?a=view) will typically express some shock at this -- that there are (for example) 32,520 points of instrumentation on an optimized, stock system. Occasionally, someone who has perhaps heard of DTrace may pipe up: "Is that where that 30,000 number comes from that I've seen in in the [press](http://www.theregister.co.uk/2004/02/15/sun_starts_solaris_10_salutations/)?" The quick answer is that this is indeed where that number comes from -- but the longer answer is that this really only the beginning because with the [`pid` provider](http://docs.sun.com/db/doc/817-6223/6mlkidloa?a=view), DTrace can instrument _every instruction in every running app_. For perhaps obvious reasons, we create these `pid` probes lazily -- we don't want to clog up the system with millions of unused probes.

After having listed all probes and counted them, my next invocation of `dtrace` is to list the probes yet again -- but this time listing only probes that match the probe tuple "`syscall:::entry"`. I explain that this means I want to match probes from the [`syscall` provider](http://docs.sun.com/db/doc/817-6223/6mlkidln3?a=view) named `entry` -- and that I don't care about the module and function. This output is simply a shorter version of the previous listing:

```
# dtrace -l -n syscall:::entry
   ID   PROVIDER            MODULE                          FUNCTION NAME
   27    syscall                                               nosys entry
   29    syscall                                               rexit entry
   31    syscall                                             forkall entry
   33    syscall                                                read entry
   35    syscall                                               write entry
   37    syscall                                                open entry
   39    syscall                                               close entry
   41    syscall                                                wait entry
...

```

I then explain that to _enable_ these probes (instead of just listing them), I need to merely not include the "`-l`":

```
# dtrace -n syscall:::entry
dtrace: description 'syscall:::entry' matched 226 probes
CPU     ID                    FUNCTION:NAME
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    261                  sysconfig:entry
  0    261                  sysconfig:entry
  0    195                  sigaction:entry
  0    195                  sigaction:entry
  0    125                      ioctl:entry
  0    261                  sysconfig:entry
  0     61                        brk:entry
  0     61                        brk:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    403                    fstat64:entry
  0     61                        brk:entry
  0     61                        brk:entry
  0    403                    fstat64:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0     35                      write:entry
  0    125                      ioctl:entry
  0    403                    fstat64:entry
  0     35                      write:entry
  0    125                      ioctl:entry
  0    125                      ioctl:entry
  0    339                    pollsys:entry
  0     33                       read:entry
...

```

I let this run on the screen for a while, explaining that we've gone from an optimized, uninstrumented system call table to an instrumented one -- and that we're now seeing every system call as it happens on the box. I explain that (on the one hand) this is novel information: in Solaris, we haven't had a way to get system call information _for the entire system_. (I often remind the audience that this may look similar to [`truss`](http://docs.sun.com/db/doc/816-5165/6mbb0m9q3?a=view) , but it's really quite different: `truss` operates only on a single process, and has an enormous probe effect from constantly stopping and running the target process.)

But while this information may be novel (at least for Solaris), it's actually not that valuable -- merely knowing that we executed these systems calls is unlikely to be terrible helpful. Rather, we may like to know which _applications_ are inducing these system calls. To do this, we add a clause in the DTrace control language, D. The clause we need to add is quite simple; we're going to use the [`trace`](http://docs.sun.com/db/doc/817-6223/6mlkidlir?a=view) action to record the current application name, which is stored in [`execname`](http://docs.sun.com/db/doc/817-6223/6mlkidlg1?a=view) variable. So I `^C` the previous enabling, and run the following:

```
# dtrace -n syscall:::entry'{trace(execname)}'
dtrace: description 'syscall:::entry' matched 226 probes
...
  0    125                      ioctl:entry   xterm
  0    125                      ioctl:entry   xterm
  0    339                    pollsys:entry   xterm
  0    339                    pollsys:entry   xterm
  0    313                lwp_sigmask:entry   Xsun
  0    313                lwp_sigmask:entry   Xsun
  0    313                lwp_sigmask:entry   Xsun
  0    313                lwp_sigmask:entry   Xsun
  0     33                       read:entry   Xsun
  0    233                     writev:entry   Xsun
  0    125                      ioctl:entry   xterm
  0     33                       read:entry   xterm
  0    339                    pollsys:entry   xterm
  0    339                    pollsys:entry   xterm
  0    125                      ioctl:entry   xterm
  0    125                      ioctl:entry   xterm
  0    339                    pollsys:entry   xterm
  0    125                      ioctl:entry   xterm
  0    125                      ioctl:entry   xterm
  0    339                    pollsys:entry   xterm
  0    339                    pollsys:entry   xterm
  0    313                lwp_sigmask:entry   Xsun
  0    313                lwp_sigmask:entry   Xsun
  0    313                lwp_sigmask:entry   Xsun
  0    313                lwp_sigmask:entry   Xsun
  0    339                    pollsys:entry   Xsun
  0     35                      write:entry   dtrace
  0    125                      ioctl:entry   xterm
  0    125                      ioctl:entry   xterm
  0    339                    pollsys:entry   xterm
  0     33                       read:entry   xterm
  0    125                      ioctl:entry   xterm
  0     35                      write:entry   xterm
  0    125                      ioctl:entry   xterm
  0    339                    pollsys:entry   xterm
  0    125                      ioctl:entry   xterm
  0     33                       read:entry   xterm
  0     35                      write:entry   xterm
  0    125                      ioctl:entry   xterm
  0    125                      ioctl:entry   xterm
  0    339                    pollsys:entry   xterm
  0    339                    pollsys:entry   xterm
  0    313                lwp_sigmask:entry   Xsun
...

```

This is clearly more useful: we can see which _apps_ are performing system calls. But seeing this also leads to a very important observation, one that some heckler in a savvy audience may have already made by now: what we're seeing in the above output is the system calls to write all of this gunk out to the terminal. That is, if we were to take this output and [`cut`](http://docs.sun.com/db/doc/816-5165/6mbb0m9dg?a=view) it, [`sort`](http://docs.sun.com/db/doc/816-5165/6mbb0m9og?a=view) it, [`uniq`](http://docs.sun.com/db/doc/816-5165/6mbb0m9qf?a=view) it, and `sort` it again, we would come to the conclusion that `dtrace`, `xterm` and `Xsun` were the problem -- regardless of what was being executed on the machine.

This is a serious problem, and the DTrace solution represents a substantial difference from virtually all earler work in this domain: in DTrace, the aggregation of data is a first-class citizen. Instead of aggregating data postmortem with traditional Unix tools, DTrace allows data to be aggregated _in situ_. This aggregation can reduce the data flow by up to a factor of the number of data points -- a tremendous savings of both space and time.

So to recast the example in terms of aggregations, we want to aggregate on the application name, with our aggregating action being to count:

```
# dtrace -n syscall:::entry'{@[execname] = count()}'
dtrace: description 'syscall:::entry' matched 226 probes

```

When we run the above, we don't see any output -- just that we matched some number of probes. I explain that behind the scenes, we're just keeping a table of application name, and the count of system calls. Whenever we `^C` the above, we get the answer:

```
^C

  utmpd                                                             4
  automountd                                                        8
  inetd                                                             9
  svc.configd                                                       9
  syslogd                                                          16
  FvwmAuto                                                         39
  in.routed                                                        48
  svc.startd                                                       97
  MozillaFirebird- 125
  sendmail                                                        183
  fvwm2                                                           621
  dhcpagent                                                      1192
  dtrace                                                         2195
  xclock                                                         2894
  vim                                                            7760
  xterm                                                         11768
  Xsun                                                          24916
#

```

I often point out that in DTrace -- as in good cocktail conversation -- the answer to one question typically provokes the next question. Looking at the above output, we may have all sorts of questions. For example, we may well ask: what the hell is [`dhcpagent`](http://docs.sun.com/db/doc/817-0690/6mgflnt8b?a=view) doing? To answer this question, we can add a [predicate](http://docs.sun.com/db/doc/817-6223/6mlkidlel?a=view) to our clause:

```
# dtrace -n syscall:::entry'/execname == "dhcpagent"/{@[probefunc] = count()}'

```

The predicate is contained between the forward slashes; the expression in the curly braces is only evaluated if the predicate evaluates to a non-zero value. And note that we have changed the tuple on which we are aggregating: instead of aggregating on `execname` (which, thanks to the predicate, will always be "`dhcpagent`"), we're aggregating on the `probefunc`. For the `syscall` provider, the `probefunc` is the name of the system call.

Running the above for ten seconds or so, and then `^C`'ing yields the following (at least on my laptop):

```
^C

  pollsys                                                           3
  close                                                             3
  open                                                              3
  lwp_sigmask                                                       6
  fstat64                                                           6
  read                                                             12
  ioctl                                                            12
  llseek                                                           15

```

Now, we may be interested in drilling down more into one of these system calls. Let's say we want to know what `dhcpagent` is doing when it performs an [open](http://docs.sun.com/db/doc/816-5167/6mbb2jahi?a=view). We can effect this by narrowing our enabling a bit: instead of enabling _all_ `syscall` probes named `entry`, we want to enable only those probes that are also from the function named `open`. We want to keep our predicate that the `execname` is `dhcpagent`, but this time we don't want to aggregate on the `probefunc` (which, thanks to the narrowed enabling, will always be "`open`"); we want to aggregate on the name of the file that is being opened -- which is the first argument to the `open` system call. To do this, we'll aggregate on `arg0`. For [slightly arcane reasons](http://docs.sun.com/db/doc/817-6223/6mlkidlot?a=view), we need to use the [`copyinstr`](http://docs.sun.com/db/doc/817-6223/6mlkidlj1?a=view) action to effect this. The enabling:

```
# dtrace -n syscall::open:entry'/execname == "dhcpagent"/{@[copyinstr(arg0)] = count()}'
dtrace: description 'syscall::open:entry' matched 1 probe

```

Again, running the above for about ten seconds:

```
^C

  /etc/default/dhcpagent                                            4

```

This tells us that there were four calls to the `open` system call from `dhcpagent` -- and that all four were to the same file. Just on the face of it, this is suspicious: why is `dhcpagent` opening the file "`/etc/default/dhcpagent`" with such frequency? Hasn't whoever wrote this code ever heard of [`stat`](http://docs.sun.com/db/doc/816-5167/6mbb2jaj4?a=view)?[^1] To answer the former question, we're going to change our enabling to aggregate on `dhcpagent`'s stack backtrace when it performs an `open`. This is done with the [`ustack`](http://docs.sun.com/db/doc/817-6223/6mlkidlir?a=view) action. The new enabling:

```
# dtrace -n syscall::open:entry'/execname == "dhcpagent"/{@[ustack()] = count()}'
dtrace: description 'syscall::open:entry' matched 1 probe

```

Running the above for ten seconds or so and then `^C`'ing yields the following:

```
^C


              libc.so.1`__open+0xc
              libc.so.1`_endopen+0x92
              libc.so.1`fopen+0x26
              libcmd.so.1`defopen+0x40
              dhcpagent`df_get_string+0x27
              dhcpagent`df_get_int+0x1c
              dhcpagent`dhcp_requesting+0x248
              libinetutil.so.1`iu_expire_timers+0x8a
              libinetutil.so.1`iu_handle_events+0x95
              dhcpagent`main+0x300
              805314a
                4

```

This tells us that all four of the `open` calls were from the above stack trace.

From here we have many options, and our course of action would depend on how we wanted to investigate this. Up until this point, we have been instrumenting only the system call layer -- the shim layer between applications and the kernel. We may wish to now instrument the application itself. To do this, we first need to know how many `dhcpagent` processes are contributing to the output we're seeing above. There are lots of ways to do this; a simple way is to aggregate on `pid`:

```
# dtrace -n syscall:::entry'/execname == "dhcpagent"/{@[pid] = count()}'
dtrace: description 'syscall:::entry' matched 226 probes
^C

       43               80

```

This indicates that in the time between when we started the above, and the time that we `^C`'d, we saw 80 system calls from pid 43 -- and that is was the only `dhcpagent` process that made any system calls. Now that we have the pid that we're intrested in, we can instrument the application. For starters, let's just instrument the `defopen` function in `libcmd.so.1`. (We know that it's being called because we saw it in the stack backtrace of the `open` system call.) To do this:

```
# dtrace -n pid43::defopen:entry
dtrace: description 'pid43::defopen:entry' matched 1 probe
CPU     ID                    FUNCTION:NAME
  0  32622                    defopen:entry
  0  32622                    defopen:entry
  0  32622                    defopen:entry
  0  32622                    defopen:entry
^C

```

With this we have instrumented the _running app_. When we `^C`'d, the app went back to being its usual optimized self. (Or at least, as optimized as `dhcpagent` ever is.)

If we wanted to trace both entry to and return from the `defopen` function, we could add another enabling to the mix:

```
# dtrace -n pid43::defopen:entry,pid43::defopen:return
dtrace: description 'pid43::defopen:entry,pid43::defopen:return' matched 2 probes
CPU     ID                    FUNCTION:NAME
  0  32622                    defopen:entry
  0  32623                   defopen:return
  0  32622                    defopen:entry
  0  32623                   defopen:return
  0  32622                    defopen:entry
  0  32623                   defopen:return
  0  32622                    defopen:entry
  0  32623                   defopen:return
^C

```

But we may want to know more than this: we may want to know _everything called_ from `defopen`. Answering this is a bit more sophisticated -- and it's a little too much for the command line. To answer this question, we head to an editor and type in the following [DTrace script](http://docs.sun.com/db/doc/817-6223/6mlkidlkk?a=view):

```
#!/usr/sbin/dtrace -s

pid43::defopen:entry
{
        self->follow = 1;
}

pid43:::entry,
pid43:::return
/self->follow/
{}

pid43::defopen:return
/self->follow/
{
        self->follow = 0;
        exit(0);
}

```

So what is this script doing? We're enabling `defopen` again, but this time we're setting a [thread-local variable](http://docs.sun.com/db/doc/817-6223/6mlkidlft?a=view) -- which we named "`follow`" to 1. We then enabled _every_ function entry and return in the process, with the predicate that we're only interested if our thread-local variable is non-zero. This has the effect of only tracing function entry and return if the function call was induced by `defopen`. We're only interested in capturing one of these traces, which is the reason why we explicitly [exit](http://docs.sun.com/db/doc/817-6223/6mlkidliv?a=view) in the `pid43::defopen:return` clause.

Running the above is going to instrument the hell out of `dhcpagent`. We're going to see that we matched many, many probes. Then we'll see a bunch of output as `defopen` is called. Finally, we'll be dropped back onto the shell prompt as we hit the `exit` action. By the time we see the shell again, we have restored the `dhcpagent` process to its uninstrumented self. Running the above, assuming it's in `dhcp.d`:

```
# chmod +x dhcp.d
# ./dhcp.d
dtrace: script './dhcp.d' matched 10909 probes
CPU     ID                    FUNCTION:NAME
  0  32622                    defopen:entry
  0  33328              _get_thr_data:entry
  0  37823            thr_getspecific:entry
  0  43272           thr_getspecific:return
  0  38784             _get_thr_data:return
  0  37176                      fopen:entry
  0  37161                   _findiop:entry
  0  37525      pthread_rwlock_wrlock:entry
  0  37524             rw_wrlock_impl:entry
  0  37513               rw_read_held:entry
  0  42963              rw_read_held:return
  0  37514              rw_write_held:entry
  0  42964             rw_write_held:return
  0  37518                rwlock_lock:entry
  0  37789                      sigon:entry
  0  43238                     sigon:return
  0  42968               rwlock_lock:return
  0  42974            rw_wrlock_impl:return
  0  42975     pthread_rwlock_wrlock:return
  0  37174                     getiop:entry
  0  37674              mutex_trylock:entry
  0  43123             mutex_trylock:return
  0  37676               mutex_unlock:entry
  0  43125              mutex_unlock:return
  0  42625                    getiop:return
  0  37174                     getiop:entry
  0  37674              mutex_trylock:entry
  0  43123             mutex_trylock:return
  0  37676               mutex_unlock:entry
  0  43125              mutex_unlock:return
  0  42625                    getiop:return
  0  37174                     getiop:entry
  0  37674              mutex_trylock:entry
  0  43123             mutex_trylock:return
  0  37676               mutex_unlock:entry
  0  43125              mutex_unlock:return
  0  42625                    getiop:return
  0  37174                     getiop:entry
  0  37674              mutex_trylock:entry
  0  43123             mutex_trylock:return
  0  35744                     memset:entry
  0  41198                    memset:return
  0  37676               mutex_unlock:entry
  0  43125              mutex_unlock:return
  0  42625                    getiop:return
  0  37531      pthread_rwlock_unlock:entry
  0  37514              rw_write_held:entry
  0  37680                 mutex_held:entry
  0  43129                mutex_held:return
  0  42964             rw_write_held:return
  0  37789                      sigon:entry
  0  43238                     sigon:return
  0  42981     pthread_rwlock_unlock:return
  0  42612                  _findiop:return
  0  37123                   _endopen:entry
  0  37309                      _open:entry
  0  37951                     __open:entry
  0  43397                    __open:return
  0  42760                     _open:return
  0  37152                  _flockget:entry
  0  37670                 mutex_lock:entry
  0  37669            mutex_lock_impl:entry
  0  43118           mutex_lock_impl:return
  0  43119                mutex_lock:return
  0  42603                 _flockget:return
  0  37676               mutex_unlock:entry
  0  43125              mutex_unlock:return
  0  42574                  _endopen:return
  0  42627                     fopen:return
  0  32623                   defopen:return
  0  32623                   defopen:return

#

```

This is interesting, but a little hard to sift through. Go back to `dhcp.d`, and add the following line:

```
#pragma D option flowindent

```

This sets a simple option that indents a code flow: probes named `entry` cause indentation to increase two spaces, and probes named `return` cause indentation to decrease two spaces. Running the new `dhcp.d`:

```
# ./dhcp.d
dtrace: script './dhcp.d' matched 10909 probes
CPU FUNCTION
  0  -> defopen
  0    -> _get_thr_data
  0      -> thr_getspecific
  0      <- thr_getspecific
  0    <- _get_thr_data
  0    -> fopen
  0      -> _findiop
  0        -> pthread_rwlock_wrlock
  0          -> rw_wrlock_impl
  0            -> rw_read_held
  0            <- rw_read_held
  0            -> rw_write_held
  0            <- rw_write_held
  0            -> rwlock_lock
  0              -> sigon
  0              <- sigon
  0            <- rwlock_lock
  0          <- rw_wrlock_impl
  0        <- pthread_rwlock_wrlock
  0        -> getiop
  0          -> mutex_trylock
  0          <- mutex_trylock
  0          -> mutex_unlock
  0          <- mutex_unlock
  0        <- getiop
  0        -> getiop
  0          -> mutex_trylock
  0          <- mutex_trylock
  0          -> mutex_unlock
  0          <- mutex_unlock
  0        <- getiop
  0        -> getiop
  0          -> mutex_trylock
  0          <- mutex_trylock
  0          -> mutex_unlock
  0          <- mutex_unlock
  0        <- getiop
  0        -> getiop
  0          -> mutex_trylock
  0          <- mutex_trylock
  0          -> memset
  0          <- memset
  0          -> mutex_unlock
  0          <- mutex_unlock
  0        <- getiop
  0        -> pthread_rwlock_unlock
  0          -> rw_write_held
  0            -> mutex_held
  0            <- mutex_held
  0          <- rw_write_held
  0          -> sigon
  0          <- sigon
  0        <- pthread_rwlock_unlock
  0      <- _findiop
  0      -> _endopen
  0        -> _open
  0          -> __open
  0          <- __open
  0        <- _open
  0        -> _flockget
  0          -> mutex_lock
  0            -> mutex_lock_impl
  0            <- mutex_lock_impl
  0          <- mutex_lock
  0        <- _flockget
  0        -> mutex_unlock
  0        <- mutex_unlock
  0      <- _endopen
  0    <- fopen
  0   | defopen:return
  0  <- defopen

#

```

Now it's much easier to see what exactly is going on. One thing I may be interested in doing is tracing a timestamp at each of these points -- to get an idea of how much time we're spending in each of these functions. To do this, copy `dhcp.d` to `dhcptime.d`, and modify it to look like this:

```
#!/usr/sbin/dtrace -s

#pragma D option flowindent

pid43::defopen:entry
{
        self->start = vtimestamp;
}

pid43:::entry,
pid43:::return
/self->start/
{
        trace(vtimestamp - self->start);
}

pid43::defopen:return
/self->start/
{
        self->start = 0;
        exit(0);
}

```

Running `dhcptime.d`:

```
# ./dhcptime.d
dtrace: script './dhcptime.d' matched 10909 probes
CPU FUNCTION
  0  -> defopen                                               0
  0    -> _get_thr_data                                    2276
  0      -> thr_getspecific                                6214
  0      <- thr_getspecific                                8445
  0    <- _get_thr_data                                   10512
  0    -> fopen                                           14296
  0      -> _findiop                                      34498
  0        -> pthread_rwlock_wrlock                       37822
  0          -> rw_wrlock_impl                            39265
  0            -> rw_read_held                            42514
  0            <- rw_read_held                            61148
  0            -> rw_write_held                           64573
  0            <- rw_write_held                           66630
  0            -> rwlock_lock                             69802
  0              -> sigon                                 89495
  0              <- sigon                                 92150
  0            <- rwlock_lock                             94463
  0          <- rw_wrlock_impl                            96615
  0        <- pthread_rwlock_wrlock                      114732
  0        -> getiop                                     117720
  0          -> mutex_trylock                            121825
  0          <- mutex_trylock                            141821
  0          -> mutex_unlock                             144477
  0          <- mutex_unlock                             146819
  0        <- getiop                                     149181
  0        -> getiop                                     150408
  0          -> mutex_trylock                            168143
  0          <- mutex_trylock                            169478
  0          -> mutex_unlock                             170679
  0          <- mutex_unlock                             171952
  0        <- getiop                                     173196
  0        -> getiop                                     174371
  0          -> mutex_trylock                            175493
  0          <- mutex_trylock                            176733
  0          -> mutex_unlock                             177897
  0          <- mutex_unlock                             179068
  0        <- getiop                                     180403
  0        -> getiop                                     181585
  0          -> mutex_trylock                            182721
  0          <- mutex_trylock                            184085
  0          -> memset                                   185393
  0          <- memset                                   186869
  0          -> mutex_unlock                             188002
  0          <- mutex_unlock                             189275
  0        <- getiop                                     190758
  0        -> pthread_rwlock_unlock                      211137
  0          -> rw_write_held                            212521
  0            -> mutex_held                             214849
  0            <- mutex_held                             217061
  0          <- rw_write_held                            218361
  0          -> sigon                                    237062
  0          <- sigon                                    238458
  0        <- pthread_rwlock_unlock                      241227
  0      <- _findiop                                     243451
  0      -> _endopen                                     263709
  0        -> _open                                      267861
  0          -> __open                                   271268
  0          <- __open                                   334013
  0        <- _open                                      336267
  0        -> _flockget                                  338543
  0          -> mutex_lock                               340668
  0            -> mutex_lock_impl                        342788
  0            <- mutex_lock_impl                        345111
  0          <- mutex_lock                               363767
  0        <- _flockget                                  365602
  0        -> mutex_unlock                               366861
  0        <- mutex_unlock                               368156
  0      <- _endopen                                     370564
  0    <- fopen                                          389523
  0   | defopen:return                                   391539
  0  <- defopen

#

```

The times in the above are all in nanoseconds. That is, the call to `_get_thr_data` was 2,276 nanoseconds after we called `defopen`, the first call to `thr_getspecific` was 6,214 nanoseconds, and so on. Note the big jump in `__open` above: we spent over sixty microseconds (334,013 - 271,268 > 60,000) in that one function. What happened there? We went into the kernel of course. (Remember, we found `defopen` by aggregating on the user stack from the `open` system call.) We may wish to know _everything_ -- kernel _and_ user -- that happens from `defopen`. To do this, go back to the original `dhcp.d` and change this:

```
pid43:::entry,
pid43:::return
/self->follow/
{}

```

To this:

```
pid43:::entry,
pid43:::return,
fbt:::
/self->follow/
{
        printf("  (%s)", probeprov);
}

```

The "`fbt:::`" line enables _every_ probe from the `fbt` provider. Recall that this provider makes available a probe upon entry to and return from every function in the kernel. I've added an additional line that records the name of the current provider (the "`probeprov`" variable) to make clear if a probe is in the kernel or is in user-space. This script is really going to instrument the snot out of the system; running the new `dhcp.d`:

```
# ./dhcp.d
dtrace: script './dhcp.d' matched 42092 probes
CPU FUNCTION
  0  -> defopen                                 (pid43)
  0    -> _get_thr_data                         (pid43)
  0      -> thr_getspecific                     (pid43)
  0      <- thr_getspecific                     (pid43)
  0    <- _get_thr_data                         (pid43)
  0    -> fopen                                 (pid43)
  0      -> _findiop                            (pid43)
  0        -> pthread_rwlock_wrlock             (pid43)
  0          -> rw_wrlock_impl                  (pid43)
  0            -> rw_read_held                  (pid43)
  0            <- rw_read_held                  (pid43)
  0            -> rw_write_held                 (pid43)
  0            <- rw_write_held                 (pid43)
  0            -> rwlock_lock                   (pid43)
  0              -> sigon                       (pid43)
  0              <- sigon                       (pid43)
  0            <- rwlock_lock                   (pid43)
  0          <- rw_wrlock_impl                  (pid43)
  0        <- pthread_rwlock_wrlock             (pid43)
  0        -> getiop                            (pid43)
  0          -> mutex_trylock                   (pid43)
  0          <- mutex_trylock                   (pid43)
  0          -> mutex_unlock                    (pid43)
  0          <- mutex_unlock                    (pid43)
  0        <- getiop                            (pid43)
  0        -> getiop                            (pid43)
  0          -> mutex_trylock                   (pid43)
  0          <- mutex_trylock                   (pid43)
  0          -> mutex_unlock                    (pid43)
  0          <- mutex_unlock                    (pid43)
  0        <- getiop                            (pid43)
  0        -> getiop                            (pid43)
  0          -> mutex_trylock                   (pid43)
  0          <- mutex_trylock                   (pid43)
  0          -> mutex_unlock                    (pid43)
  0          <- mutex_unlock                    (pid43)
  0        <- getiop                            (pid43)
  0        -> getiop                            (pid43)
  0          -> mutex_trylock                   (pid43)
  0          <- mutex_trylock                   (pid43)
  0          -> memset                          (pid43)
  0          <- memset                          (pid43)
  0          -> mutex_unlock                    (pid43)
  0          <- mutex_unlock                    (pid43)
  0        <- getiop                            (pid43)
  0        -> pthread_rwlock_unlock             (pid43)
  0          -> rw_write_held                   (pid43)
  0            -> mutex_held                    (pid43)
  0            <- mutex_held                    (pid43)
  0          <- rw_write_held                   (pid43)
  0          -> sigon                           (pid43)
  0          <- sigon                           (pid43)
  0        <- pthread_rwlock_unlock             (pid43)
  0      <- _findiop                            (pid43)
  0      -> _endopen                            (pid43)
  0        -> _open                             (pid43)
  0          -> __open                          (pid43)
  0            -> syscall_mstate                (fbt)
  0              -> tsc_gethrtimeunscaled       (fbt)
  0              <- tsc_gethrtimeunscaled       (fbt)
  0            <- syscall_mstate                (fbt)
  0            -> open                          (fbt)
  0              -> copen                       (fbt)
  0                -> falloc                    (fbt)
  0                  -> ufalloc                 (fbt)
  0                    -> ufalloc_file          (fbt)
  0                      -> fd_find             (fbt)
  0                      <- fd_find             (fbt)
  0                      -> fd_reserve          (fbt)
  0                      <- fd_reserve          (fbt)
  0                    <- ufalloc_file          (fbt)
  0                  <- ufalloc                 (fbt)
  0                  -> kmem_cache_alloc        (fbt)
  0                  <- kmem_cache_alloc        (fbt)
  0                  -> crhold                  (fbt)
  0                  <- crhold                  (fbt)
  0                <- falloc                    (fbt)
  0                -> vn_openat                 (fbt)
  0                  -> lookupnameat            (fbt)
  0                    -> pn_get_buf            (fbt)
  0                    <- pn_get_buf            (fbt)
  0                    -> lookuppnat            (fbt)
  0                      -> lookuppnvp          (fbt)
  0                        -> pn_fixslash       (fbt)
  0                        <- pn_fixslash       (fbt)
  0                        -> pn_getcomponent   (fbt)
  0                        <- pn_getcomponent   (fbt)
  0                        -> fop_lookup        (fbt)
  0                          -> ufs_lookup      (fbt)
  0                            -> dnlc_lookup   (fbt)
  0                            <- dnlc_lookup   (fbt)
  0                            -> ufs_iaccess   (fbt)
  0                              -> crgetuid    (fbt)
  0                              <- crgetuid    (fbt)
  0                            <- ufs_iaccess   (fbt)
  0                          <- ufs_lookup      (fbt)
  0                        <- fop_lookup        (fbt)
  0                        -> vn_mountedvfs     (fbt)
  0                        <- vn_mountedvfs     (fbt)
  0                        -> vn_rele           (fbt)
  0                        <- vn_rele           (fbt)
  0                        -> pn_getcomponent   (fbt)
  0                        <- pn_getcomponent   (fbt)
  0                        -> fop_lookup        (fbt)
  0                          -> ufs_lookup      (fbt)
  0                            -> dnlc_lookup   (fbt)
  0                            <- dnlc_lookup   (fbt)
  0                            -> ufs_iaccess   (fbt)
  0                              -> crgetuid    (fbt)
  0                              <- crgetuid    (fbt)
  0                            <- ufs_iaccess   (fbt)
  0                          <- ufs_lookup      (fbt)
  0                        <- fop_lookup        (fbt)
  0                        -> vn_mountedvfs     (fbt)
  0                        <- vn_mountedvfs     (fbt)
  0                        -> vn_rele           (fbt)
  0                        <- vn_rele           (fbt)
  0                        -> pn_getcomponent   (fbt)
  0                        <- pn_getcomponent   (fbt)
  0                        -> fop_lookup        (fbt)
  0                          -> ufs_lookup      (fbt)
  0                            -> dnlc_lookup   (fbt)
  0                            <- dnlc_lookup   (fbt)
  0                            -> ufs_iaccess   (fbt)
  0                              -> crgetuid    (fbt)
  0                              <- crgetuid    (fbt)
  0                            <- ufs_iaccess   (fbt)
  0                          <- ufs_lookup      (fbt)
  0                        <- fop_lookup        (fbt)
  0                        -> vn_mountedvfs     (fbt)
  0                        <- vn_mountedvfs     (fbt)
  0                        -> vn_rele           (fbt)
  0                        <- vn_rele           (fbt)
  0                        -> pn_setlast        (fbt)
  0                        <- pn_setlast        (fbt)
  0                        -> vn_path           (fbt)
  0                        <- vn_path           (fbt)
  0                      <- lookuppnvp          (fbt)
  0                    <- lookuppnat            (fbt)
  0                  <- lookupnameat            (fbt)
  0                  -> fop_getattr             (fbt)
  0                    -> ufs_getattr           (fbt)
  0                    <- ufs_getattr           (fbt)
  0                  <- fop_getattr             (fbt)
  0                  -> fop_access              (fbt)
  0                    -> ufs_access            (fbt)
  0                      -> ufs_iaccess         (fbt)
  0                        -> crgetuid          (fbt)
  0                        <- crgetuid          (fbt)
  0                      <- ufs_iaccess         (fbt)
  0                    <- ufs_access            (fbt)
  0                  <- fop_access              (fbt)
  0                  -> fop_open                (fbt)
  0                    -> ufs_open              (fbt)
  0                    <- ufs_open              (fbt)
  0                    -> vn_rele               (fbt)
  0                    <- vn_rele               (fbt)
  0                  <- fop_open                (fbt)
  0                <- vn_openat                 (fbt)
  0                -> setf                      (fbt)
  0                  -> cv_broadcast            (fbt)
  0                  <- cv_broadcast            (fbt)
  0                <- setf                      (fbt)
  0              <- copen                       (fbt)
  0            <- open                          (fbt)
  0            -> syscall_mstate                (fbt)
  0              -> tsc_gethrtimeunscaled       (fbt)
  0              <- tsc_gethrtimeunscaled       (fbt)
  0            <- syscall_mstate                (fbt)
  0          <- __open                          (pid43)
  0        <- _open                             (pid43)
  0        -> _flockget                         (pid43)
  0          -> mutex_lock                      (pid43)
  0            -> mutex_lock_impl               (pid43)
  0            <- mutex_lock_impl               (pid43)
  0          <- mutex_lock                      (pid43)
  0        <- _flockget                         (pid43)
  0        -> mutex_unlock                      (pid43)
  0        <- mutex_unlock                      (pid43)
  0      <- _endopen                            (pid43)
  0    <- fopen                                 (pid43)
  0   | defopen:return                          (pid43)
  0  <- defopen

#

```

As the transition from `pid43` to `fbt` in the above indicates, this script shows us flow control _across_ the user/kernel boundary. (At the risk of sounding too much like [Ron Popeil](http://www.theregister.co.uk/2004/07/08/dtrace_user_take/), the above output is the clearest display of code flow across a protection boundary from _any_ tool -- research or otherwise.)

By this point in the demo, the audience is usually convinced that DTrace can instrument huge tracts of the system that were never before observable; the focus usually shifts from "can this do anything?" to "can this do anything for **me**?" The short answer is "yes" (of course), but a longer answer is merited. Unfortunately, this blog entry has already gone on long enough however; I'll save the longer answer for another day...

[^1]: For whatever it's worth, a bug has been filed on this issue. Once it's fixed, I will sadly have to find some other sub-optimal software with which to demonstrate DTrace. The good news is that such software isn't terribly [hard to find](http://openoffice.org).
