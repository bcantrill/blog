---
title: "DTrace and PHP, demonstrated"
date: "2005-08-05"
categories: 
  - "solaris"
---

If you were at [my presentation at OSCON yesterday](/resources/bmc/dtrace_oscon.pdf), I apologize for the brisk pace -- there was a ton of material to cover, and forty-five minutes isn't much time. Now that I've got a little more time, I'd like to get into the details of the PHP demonstration that I did during the presentation. Here is the (very) simple PHP program that I was using:

```
<?php
function blastoff()
{
        echo "Blastoff!\\n\\n";
}

function one()
{
        echo "One...\\n";
        blastoff();
}

function two()
{
        echo "Two...\\n";
        one();
}

function three()
{
        echo "Three...\\n";
        two();
}

function launch()
{
        three();
}

while (1)
{
        launch();
        sleep(1);
}
?>

```

Running this in a window just repeats this output:

```
% php ./blastoff.php
php ./blastoff.php
Content-type: text/html
X-Powered-By: PHP/5.1.0-dev

Three...
Two...
One...
Blastoff!

Three...
Two...
One...
Blastoff!
...

```

Now, because I have specified [Wez Furlong's](http://netevil.org/) new [dtrace.so](http://blogs.sun.com/roller/comments/bmc/Weblog/dtrace_and_php#comments) as an extension in my [php.ini](http://www.mcs.vuw.ac.nz/technical/software/PHP/configuration.html) file, when I run the above, two probes show up:

```
# dtrace -l | grep php
4   php806       dtrace.so          php_dtrace_execute function-return
5   php806       dtrace.so          php_dtrace_execute function-entry

```

The `function-entry` and `function-return` probes have three arguments:

- `arg0` is the name of the function (a pointer to a string within PHP)
    
- `arg1` is the name of the file containing the call site (also a pointer to a string within PHP)
    
- `arg2` is the line number of the call site a pointer to a string within PHP)
    

So for starters, let's just get an idea of which functions are being called in my PHP program:

```
# dtrace -n function-entry'{printf("called %s() in %s at line %d\n", \
copyinstr(arg0), copyinstr(arg1), arg2)}' -q
called launch() in /export/php/blastoff.php at line 32
called three() in /export/php/blastoff.php at line 27
called two() in /export/php/blastoff.php at line 22
called one() in /export/php/blastoff.php at line 16
called blastoff() in /export/php/blastoff.php at line 10
called launch() in /export/php/blastoff.php at line 32
called three() in /export/php/blastoff.php at line 27
called two() in /export/php/blastoff.php at line 22
called one() in /export/php/blastoff.php at line 16
called blastoff() in /export/php/blastoff.php at line 10
^C

```

If you're new to DTrace, note that you have the power to trace an arbitrary D expression in your action. For example, instead of printing out the file and line number of the call site, we could trace the `walltimestamp`:

```
# dtrace -n function-entry'{printf("called %s() at %Y\n", \
copyinstr(arg0), walltimestamp)}' -q
called launch() at 2005 Aug  5 08:08:24
called three() at 2005 Aug  5 08:08:24
called two() at 2005 Aug  5 08:08:24
called one() at 2005 Aug  5 08:08:24
called blastoff() at 2005 Aug  5 08:08:24
called launch() at 2005 Aug  5 08:08:25
called three() at 2005 Aug  5 08:08:25
called two() at 2005 Aug  5 08:08:25
called one() at 2005 Aug  5 08:08:25
called blastoff() at 2005 Aug  5 08:08:25
^C

```

Note, too, that (unless I direct it not to) this will aggregate across PHP instances. So, for example:

```
#!/usr/sbin/dtrace -s

#pragma D option quiet

php*:::function-entry
{
        @bypid[pid] = count();
        @byfunc[copyinstr(arg0)] = count();
        @bypidandfunc[pid, copyinstr(arg0)] = count();
}

END
{
        printf("By pid:\n\n");
        printa("  %-40d %@d\n", @bypid);

        printf("\nBy function:\n\n");
        printa("  %-40s %@d\n", @byfunc);

        printf("\nBy pid and function:\n\n");
        printa("  %-9d %-30s %@d\n", @bypidandfunc);
}

```

If I run three instances of `blastoff.php` and then the above script, I see the following after I `^C`:

```
By pid:

  806                                      30
  875                                      30
  889                                      30

By function:

  launch                                   18
  three                                    18
  two                                      18
  one                                      18
  blastoff                                 18

By pid and function:

  875       two                            6
  875       three                          6
  875       launch                         6
  875       blastoff                       6
  889       blastoff                       6
  806       launch                         6
  889       one                            6
  806       three                          6
  889       two                            6
  806       two                            6
  889       three                          6
  806       one                            6
  889       launch                         6
  806       blastoff                       6
  875       one                            6

```

The point is that DTrace allows you to aggregate _across PHP instances_, allowing you to understand not just what a particular PHP is doing, but what PHP is doing more generally.

If we're interested in better understanding the code flow in a particular PHP instance, we can write a script that uses a thread-local variable to follow the code flow:

```
#pragma D option quiet

self int indent;

php$target:::function-entry
/copyinstr(arg0) == "launch"/
{
        self->follow = 1;
}

php$target:::function-entry
/self->follow/
{
        printf("%*s", self->indent, "");
        printf("-> %s\n", copyinstr(arg0));
        self->indent += 2;
}

php$target:::function-return
/self->follow/
{
        self->indent -= 2;
        printf("%*s", self->indent, "");
        printf("follow = 0;
        exit(0);
}

```

Running the above requires giving the script a particular PHP process; assuming the above script is named `blast.d`, it might look like this:

```
# dtrace -s ./blast.d -p `pgrep -n php`
-> launch
  -> three
    -> two
      -> one
        -> blastoff
        <- blastoff
      <- one
    <- two
  <- three
<- launch

```

This shows us all of the PHP functions that were called from `launch()`, but it doesn't tell the full story -- we know that our PHP functions are calling into native code to do some of their work. To add this to the mix, we'll write a slightly longer script:

```
#pragma D option quiet

self int indent;

php$target:::function-entry
/copyinstr(arg0) == "launch"/
{
        self->follow = 1;
}

php$target:::function-entry
/self->follow/
{
        printf("%\*s", self->indent, "");
        printf("-> %-20s %\*sphp\\n", copyinstr(arg0),
            46 - self->indent, "");
        self->indent += 2;
}

php$target:::function-return
/self->follow/
{
        self->indent -= 2;
        printf("%\*s", self->indent, "");
        printf("indent, "");
}

pid$target:libc.so.1::entry
/self->follow/
{
        printf("%\*s", self->indent, "");
        printf("-> %-20s %\*s%s\\n", probefunc, 46 - self->indent, "",
            probemod);
        self->indent += 2;
}

pid$target:libc.so.1::return
/self->follow/
{
        self->indent -= 2;
        printf("%\*s", self->indent, "");
        printf("indent, "",
            probemod);
}

php$target:::function-return
/copyinstr(arg0) == "launch"/
{
        self->follow = 0;
        exit(0);
}

```

Running the above yields the following output:

```
-> launch                                          php
  -> three                                         php
    -> write                                       libc.so.1
      -> _save_nv_regs                             libc.so.1
       _write                                    libc.so.1
      <- _write                                    libc.so.1
     two                                         php
      -> write                                     libc.so.1
        -> _save_nv_regs                           libc.so.1
         _write                                  libc.so.1
        <- _write                                  libc.so.1
       one                                       php
        -> write                                   libc.so.1
          -> _save_nv_regs                         libc.so.1
           _write                                libc.so.1
          <- _write                                libc.so.1
         blastoff                                php
          -> write                                 libc.so.1
            -> _save_nv_regs                       libc.so.1
             _write                              libc.so.1
            <- _write                              libc.so.1
          <- write                                 libc.so.1
        <- blastoff                                php
      <- one                                       php
    <- two                                         php
  <- three                                         php
<- launch                                          php

```

This shows us what's _really_ going on -- or at least more of what's really going on: our PHP functions are inducing work from libc. This is much more information, but of course, we're still only seeing what's going on at user-level. To add the kernel into the mix, add the following to the script:

```
fbt:genunix::entry
/self->follow/
{
        printf("%\*s", self->indent, "");
        printf("-> %-20s %\*skernel\\n", probefunc, 46 - self->indent, "");
        self->indent += 2;
}

fbt:genunix::return
/self->follow/
{
        self->indent -= 2;
        printf("%\*s", self->indent, "");
        printf("indent, "");
}

```

Running this new script generates _much_ more output:

```
-> launch                                          php
  -> three                                         php
    -> write                                       libc.so.1
      -> _save_nv_regs                             libc.so.1
       _write                                    libc.so.1
        -> syscall_mstate                          kernel
          -> gethrtime_unscaled                    kernel
          <- gethrtime_unscaled                    kernel
         syscall_entry                           kernel
          -> pre_syscall                           kernel
           copyin_args32                         kernel
            -> copyin_nowatch                      kernel
              -> watch_disable_addr                kernel
              <- watch_disable_addr                kernel
            <- copyin_nowatch                      kernel
          <- copyin_args32                         kernel
         write32                                 kernel
          -> write                                 kernel
            -> getf                                kernel
              -> set_active_fd                     kernel
              <- set_active_fd                     kernel
             releasef                            kernel
              -> cv_broadcast                      kernel
              <- cv_broadcast                      kernel
            <- releasef                            kernel
          <- write                                 kernel
         syscall_mstate                          kernel
          -> gethrtime_unscaled                    kernel
          <- gethrtime_unscaled                    kernel
        <- syscall_mstate                          kernel
      <- _write                                    libc.so.1
     two                                         php
      -> write                                     libc.so.1
        -> _save_nv_regs                           libc.so.1
         _write                                  libc.so.1
          -> syscall_mstate                        kernel
            -> gethrtime_unscaled                  kernel
            <- gethrtime_unscaled                  kernel
          <- syscall_mstate                        kernel
          ...

```

Now we're seeing _everything_ that our PHP program is doing, from the PHP through the native code, into the kernel and back. So using DTrace on PHP has a number of unique advantages: you can look at the entire system, and you can look at the entire stack -- and you can do it all in production. Thanks again to Wez for putting together the PHP provider -- and if you're a PHP developer, bon appetit!
