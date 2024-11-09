---
title: "DTrace on Rails"
date: "2006-05-02"
categories: 
  - "solaris"
---

First I need to apologize for having been absent for so long -- I am very much heads-down on a new project. (The details of which will need to wait for another day, I'm afraid -- but suffice it to say that it, like just about everything else I've done at Sun, leverages much of what I've done before it.)

That said, I wanted to briefly emerge to discuss some recent work. A while ago, I blogged about [DTrace and Ruby](http://blogs.sun.com/roller/page/bmc?entry=dtrace_and_ruby), using [Rich Lowe](mailto:richlowe@richlowe.net)'s [prototype DTrace provider](http://richlowe.net/~richlowe/patches/ruby-1.8.2-dtrace.diff). This provider represents a quantum leap for Ruby observability, but it suffers from the fact that we must do work (in particular, we must get the class and method) _even when disabled_. This is undesirable (especially considering that the effect can be quite significant -- up to 2X), and it runs against the DTrace ethos of zero disabled probe effect, but there has been no better solution. Now, however, thanks to Adam's work on [is-enabled probes](http://www.opensolaris.org/jive/thread.jspa?messageID=31314), we can have a Ruby provider that has zero disabled probe effect. (Or essentially zero: I actually measured the probe effect at 0.2% -- very much in the noise.) Having zero disabled probe effect allows us to deploy DTrace on Ruby in production -- which in turn opens up a whole new domain for DTrace: [Ruby on Rails](http://www.rubyonrails.org/). And as I was reminded by [Jason Hoffman](http://joyent.com/team/jasonhoffman)'s recent [Scale with Rails](http://www.scalewithrails.com/downloads/ScaleWithRails-April2006.pdf) presentation (in which he outlines why they picked Solaris generally -- and [ZFS](http://www.opensolaris.org/os/community/zfs/) in particular), this is a hugely important growth area for Solaris. So without further ado, here is a (reasonably) simple script that relies on some details of WEBrick and Rails to yield a system profile for Rails requests:

```
#pragma D option quiet

self string uri;

syscall::read:entry
/execname == "ruby" && self->uri == NULL/
{
        self->fd = arg0;
        self->buf = arg1;
        self->size = arg2;
}

syscall::read:return
/self->uri == NULL && self->buf != NULL && strstr(this->str =
    copyinstr(self->buf, self->size), "GET ") == this->str/
{
        this->head = strtok(this->str, " ");
        self->uri = this->head != NULL ? strtok(NULL, " ") : NULL;
        self->syscalls = 0;
        self->rbcalls = 0;
}

syscall::read:return
/self->buf != NULL/
{
        self->buf = NULL;
}

syscall:::entry
/self->uri != NULL/
{
        @syscalls[probefunc] = count();
}

ruby$1:::function-entry
/self->uri != NULL/
{
        @rbclasses[this->class = copyinstr(arg0)] = count();
        this->sep = strjoin(this->class, "#");
        @rbmethods[strjoin(this->sep, copyinstr(arg1))] = count();
}

pid$1::mysql_send_query:entry
/self->uri != NULL/
{
        @queries[copyinstr(arg1)] = count();
}

syscall::write:entry
/self->uri != NULL && arg0 == self->fd && strstr(this->str =
    copyinstr(arg1, arg2), "HTTP/1.1") == this->str/
{
        self->uri = NULL;
        ncalls++;
}

END
{
        normalize(@syscalls, ncalls);
        trunc(@syscalls, 10);
        printf("Top ten system calls per URI serviced:\n");
        printf("---------------------------------------");
        printf("--------------------------------+------\n");
        printa("  %-68s | %@d\n", @syscalls);

        normalize(@rbclasses, ncalls);
        trunc(@rbclasses, 10);
        printf("\nTop ten Ruby classes called per URI serviced:\n");
        printf("---------------------------------------");
        printf("--------------------------------+------\n");
        printa("  %-68s | %@d\n", @rbclasses);

        normalize(@rbmethods, ncalls);
        trunc(@rbmethods, 10);
        printf("\nTop ten Ruby methods called per URI serviced:\n");
        printf("---------------------------------------");
        printf("--------------------------------+------\n");
        printa("  %-68s | %@d\n", @rbmethods);

        trunc(@queries, 10);
        printf("\nTop ten MySQL queries:\n");
        printf("---------------------------------------");
        printf("--------------------------------+------\n");
        printa("  %-68s | %@d\n", @queries);
}
```

Running the above while horsing around with the Depot application from [Agile Web Development with Rails](http://www.pragmaticprogrammer.com/title/rails/) yields the following:

```
Top ten system calls per URI serviced:
-----------------------------------------------------------------------+------
  setcontext                                                           | 15
  fcntl                                                                | 16
  fstat64                                                              | 16
  open64                                                               | 21
  close                                                                | 25
  llseek                                                               | 27
  lwp_sigmask                                                          | 30
  read                                                                 | 62
  pollsys                                                              | 80
  stat64                                                               | 340

Top ten Ruby classes called per URI serviced:
-----------------------------------------------------------------------+------
  ActionController::CodeGeneration::Source                             | 89
  ActionController::CodeGeneration::CodeGenerator                      | 167
  Fixnum                                                               | 190
  Symbol                                                               | 456
  Class                                                                | 556
  Hash                                                                 | 1000
  String                                                               | 1322
  Array                                                                | 1903
  Object                                                               | 2364
  Module                                                               | 6525

Top ten Ruby methods called per URI serviced:
-----------------------------------------------------------------------+------
  Object#dup                                                           | 235
  String#==                                                            | 250
  Object#is_a?                                                         | 288
  Object#nil?                                                          | 316
  Hash#[]                                                              | 351
  Symbol#to_s                                                          | 368
  Object#send                                                          | 593
  Module#included_modules                                              | 1043
  Array#include?                                                       | 1127
  Module#==                                                            | 5058

Top ten MySQL queries:
-----------------------------------------------------------------------+------
  SELECT * FROM products  LIMIT 0, 10                                  | 2
  SELECT * FROM products WHERE (products.id = '7')  LIMIT 1            | 2
  SELECT count(*) AS count_all FROM products                           | 2
  SHOW FIELDS FROM products                                            | 5
```

While this gives us lots of questions we might want to answer (e.g., "why the hell are we doing 340 stats on every 'effing request!"1), it might be a little easier to look at a view that lets us see requests and the database queries that they induce. Here, for example, is a similar script to do just that:

```
#pragma D option quiet

self string uri;
self string newuri;

BEGIN
{
        start = timestamp;
}

syscall::read:entry
/execname == "ruby" && self->uri == NULL/
{
        self->fd = arg0;
        self->buf = arg1;
        self->size = arg2;
}

syscall::read:return
/self->uri == NULL && self->buf != NULL && (strstr(this->str =
    copyinstr(self->buf, self->size), "GET ") == this->str ||
    strstr(this->str, "POST ") == this->str)/
{
        this->head = strtok(this->str, " ");
        self->newuri = this->head != NULL ? strtok(NULL, " ") : NULL;
}

syscall::read:return
/self->newuri != NULL/
{
        self->uri = self->newuri;
        self->newuri = NULL;
        printf("%3d.%03d => %s\n", (timestamp - start) / 1000000000,
            ((timestamp - start) / 1000000) % 1000, self->uri);
}

pid$1::mysql_send_query:entry
/self->uri != NULL/
{
        printf("%3d.%03d   -> \"%s\"\n", (timestamp - start) / 1000000000,
            ((timestamp - start) / 1000000) % 1000,
            copyinstr(self->query = arg1));
}

pid$1::mysql_send_query:return
/self->query != NULL/
{
        printf("%3d.%03d   <- \"%s\"\n", (timestamp - start) / 1000000000,
             ((timestamp - start) / 1000000) % 1000,
             copyinstr(self->query));
        self->query = NULL;
}

syscall::read:return
/self->buf != NULL/
{
        self->buf = NULL;
}

syscall::write:entry
/self->uri != NULL && arg0 == self->fd && strstr(this->str =
    copyinstr(arg1, arg2), "HTTP/1.1") == this->str/
{
        printf("%3d.%03d <= %s\n", (timestamp - start) / 1000000000,
             ((timestamp - start) / 1000000) % 1000,
             self->uri);
        self->uri = NULL;
}
```

Running the above while clicking around with the Depot app:

```
# ./rsnoop.d `pgrep ruby`
  7.936 => /admin/edit/7
  7.955   -> "SELECT * FROM products WHERE (products.id = '7')  LIMIT 1"
  7.956   <- "SELECT * FROM products WHERE (products.id = '7')  LIMIT 1"
  7.957   -> "SHOW FIELDS FROM products"
  7.957   <- "SHOW FIELDS FROM products"
  7.971 <= /admin/edit/7  20.881 => /admin/update/7
 20.952   -> "SELECT * FROM products WHERE (products.id = '7')  LIMIT 1"
 20.953   <- "SELECT * FROM products WHERE (products.id = '7')  LIMIT 1"
 20.953   -> "SHOW FIELDS FROM products"
 20.953   <- "SHOW FIELDS FROM products"  20.954   -> "BEGIN"
 20.954   <- "BEGIN"  20.955   -> "UPDATE products SET `title` = 'foo bar', `price` = 1.2, ...
 20.955   <- "UPDATE products SET `title` = 'foo bar', `price` = 1.2, ...
 20.989   -> "COMMIT"
 20.989   <- "COMMIT"
 21.001 <= /admin/update/7  21.005 => /admin/show/7
 21.023   -> "SELECT * FROM products WHERE (products.id = '7')  LIMIT 1"
 21.023   <- "SELECT * FROM products WHERE (products.id = '7')  LIMIT 1"
 21.024   -> "SHOW FIELDS FROM products"
 21.024   <- "SHOW FIELDS FROM products"
 21.038 <= /admin/show/7
```

I'm no Rails developer, but it seems like this might be useful... If you want to check this out for yourself, start by getting [Rich's prototype provider](http://richlowe.net/~richlowe/patches/ruby-1.8.2-dtrace.diff). (Using it, you can do everything I've done here, just with higher disabled probe effect.) Meanwhile, I'll work with Rich to get the lower disabled probe effect version out shortly. Happy Railing!

* * *

1 Or if you're as savvy as the commenters on this blog entry, you might be saying to yourself, "why the hell is the 'effing development version running in production?!"
