---
title: "agghist, aggzoom and aggpack"
date: "2013-11-11"
---

As I have often remarked, DTrace is a workhorse, not a show horse: the features that we have added to DTrace over the years come not from theoretical notions but rather from actual needs in production environments. This is as true now as it was [a decade ago](http://dtrace.org/blogs/bmc/2013/09/03/happy-10th-birthday-dtrace/), and even in core abstractions, extensive use of DTrace seems to give rise to new ways of thinking about them. In particular, [Brendan](http://dtrace.org/blogs/brendan) recently had a couple of feature ideas for aggregation processing that have turned out to be really interesting...

## `agghist`

When one performs a count() or sum() aggregating action, the result is a table that consists of values and numbers, e.g.:

```
# dtrace -n syscall:::entry'{@[execname] = count()}'
dtrace: description 'syscall:::entry' matched 233 probes
^C

  utmpd                                                             2
  metadata                                                          5
  pickup                                                           14
  ur-agent                                                         18
  epmd                                                             20
  fmd                                                              20
  jsontool.js                                                      33
  vmadmd                                                           33
  master                                                           41
  mdnsd                                                            41
  intrd                                                            45
  devfsadm                                                         52
  bridged                                                          81
  mkdir                                                            83
  ipmgmtd                                                          95
  cat                                                              96
  ls                                                              100
  truss                                                           101
  ipmon                                                           106
  sendmail                                                        111
  dirname                                                         115
  svc.startd                                                      125
  svc.configd                                                     153
  ksh93                                                           164
  zoneadmd                                                        165
  svcprop                                                         258
  sshd                                                            262
  bash                                                            291
  zpool                                                           436
  date                                                            448
  cron                                                            470
  lldpd                                                           584
  dump-minutely-sd                                                611
  haproxy                                                         760
  nscd                                                           1275
  zfs                                                            1966
  zoneadm                                                        2414
  java                                                           2916
  tail                                                           3750
  postgres                                                       6702
  redis-server                                                  21283
  dtrace                                                        28308
  node                                                          39836
  beam.smp                                                      55940

```

Brendan's observation was that it would be neat to (optionally) use the histogram-style aggregation printing with these count()/sum() aggregations to be able to more easily differentiate values. For that, we have the new "`-x agghist`" option:

```
# dtrace -n syscall:::entry'{@[execname] = count()}' -x agghist
dtrace: description 'syscall:::entry' matched 233 probes
^C


              key  ------------- Distribution ------------- count
            utmpd |                                         2
         metadata |                                         5
           pickup |                                         14
             epmd |                                         20
              fmd |                                         23
         ur-agent |                                         25
             init |                                         27
            mdnsd |                                         30
      jsontool.js |                                         33
           vmadmd |                                         36
           master |                                         41
            intrd |                                         45
         devfsadm |                                         52
          bridged |                                         78
            mkdir |                                         83
              cat |                                         96
          ipmgmtd |                                         97
               ls |                                         100
         sendmail |                                         101
            ipmon |                                         102
          dirname |                                         115
       svc.startd |                                         125
            truss |                                         140
      svc.configd |                                         153
            ksh93 |                                         164
             sshd |                                         248
          svcprop |                                         258
             bash |                                         290
         zoneadmd |                                         387
            zpool |                                         436
             date |                                         448
             cron |                                         470
            lldpd |                                         562
 dump-minutely-sd |                                         615
          haproxy |                                         622
             nscd |                                         808
              zfs |                                         1966
          zoneadm |@                                        2414
             java |@                                        3003
             tail |@                                        3607
         postgres |@                                        5728
     redis-server |@@@@@                                    20611
           dtrace |@@@@@@                                   26966
             node |@@@@@@@@@@                               39825
         beam.smp |@@@@@@@@@@@@@                            55942


```

It's obviously the same information, but presented with a quicker visual cue as to the distribution. Now, one may well note that this output has a lot of dead whitespace in it -- read on.

## `aggzoom`

The DTrace histogram-style output came directly from [lockstat](https://www.illumos.org/man/1M/lockstat), which proceeded it by several years. lockstat was important for DTrace in several ways: aside from showing the power of production instrumentation, lockstat pointed to the need for first-class aggregations, for disjoint instrumentation providers and for multiplexed consumers (for which it was repaid by having its guts ripped out, being reimplemented as both a [DTrace provider](http://dtrace.org/guide/chapter18.html) and a DTrace consumer). Due to some combination of my laziness and my reverence for the lockstat-style histogram, I simply lifted the lockstat processing code -- which means I inherited (or stole, depending on your perspective) the decisions [Jeff](http://www.eweek.com/c/a/IT-Management/Suns-ZFS-Creator-to-Quit-Oracle-and-Join-Startup-519195/) had made with regard to histogram rendering. In particular, lockstat determines the height of a bar by taking the bucket's share of the total and multiplying it by the full height of the histogram. That is, if you add up all of the heights of all of the bars, they will add up to the full height of the histogram. The benefit of this is that it quickly tells you relative distribution: if you see a full-height bar, you know that that bucket represents essentially all of the values. But the problem is that if the number of buckets is large and/or the values of those buckets are relatively evenly distributed, the result is a bunch of very short bars (or worse, zero height bars) and dead whitespace. This can become so annoying to DTrace users that Brendan has been known to observe that this is the one improvement over DTrace that he can point to in [SystemTap](http://dtrace.org/blogs/brendan/2011/10/15/using-systemtap/): that instead of dividing the height of the histogram among all of the bars, each bar has a height in proportion to the histogram height that is its value in proportion to the bucket of greatest value. Of course, the shape of the distribution doesn't change -- one simply is automatically scaling the height of the highest bar to the height of the histogram and adjusting all other bars accordingly.

To allow for this behavior, I added "`-x aggzoom`"; running the same example as above with this new option:

```
# dtrace -n syscall:::entry'{@[execname] = count()}' -x agghist -x aggzoom
dtrace: description 'syscall:::entry' matched 233 probes
^C

              key  ------------- Distribution ------------- count
            utmpd |                                         2
         metadata |                                         7
           pickup |                                         14
         ur-agent |                                         20
              fmd |                                         25
             epmd |                                         26
      jsontool.js |                                         33
            mdnsd |                                         39
           master |                                         41
           vmadmd |                                         44
         rsyslogd |                                         57
         devfsadm |                                         66
            mkdir |                                         83
            intrd |                                         90
              cat |                                         96
          bridged |                                         99
               ls |                                         100
          dirname |                                         115
            truss |                                         125
            ipmon |                                         130
         sendmail |                                         131
          ipmgmtd |                                         133
            ksh93 |                                         164
       svc.startd |                                         179
             init |                                         189
             sshd |                                         253
          svcprop |                                         258
             bash |                                         283
            zpool |                                         436
             date |                                         448
             cron |                                         470
 dump-minutely-sd |                                         611
            lldpd |                                         716
      svc.configd |@                                        939
          haproxy |@                                        1097
         zoneadmd |@                                        1455
              zfs |@                                        1966
          zoneadm |@@                                       2414
             nscd |@@                                       2867
             java |@@@                                      4277
             tail |@@@                                      4579
         postgres |@@@@@@                                   9389
     redis-server |@@@@@@@@@@@@@@@@@@                       26177
           dtrace |@@@@@@@@@@@@@@@@@@@@@@@                  34530
             node |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@        48151
         beam.smp |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@   55940

```

It's zoomtastic!

## `aggpack`

If you follow Brendan's blog, you know that he's always experimenting with ways of visually communicating a maximal amount of system data. One recent visualization that has been particularly interesting are his [frequency trails](http://dtrace.org/blogs/brendan/2013/06/19/frequency-trails/) -- a kind of stacked sparkline. Thinking about density of presentation sparked Brendan to observe that he often needs to try to visually correlate multiple quantize()/lquantize() aggregation keys, e.g. this classic DTrace "one-liner" to show the distribution of system call time (in nanoseconds) by system call:

```
# dtrace -n syscall:::entry'{self->ts = timestamp}' \
    -n syscall:::return'/self->ts/{@[probefunc] = \
    quantize(timestamp - self->ts); self->ts = 0}'
dtrace: description 'syscall:::entry' matched 233 probes
dtrace: description 'syscall:::return' matched 233 probes
^C

  sigpending
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 1
            2048 |                                         0

  llseek
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |@@@@@@@@@@@@@@@@@@@@                     1
            1024 |@@@@@@@@@@@@@@@@@@@@                     1
            2048 |                                         0

  fstat
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 1
            4096 |                                         0

  setcontext
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 1
            4096 |                                         0

  fstat64
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |@@@@@@@@@@@@@@@@@@@@                     2
            1024 |@@@@@@@@@@@@@@@@@@@@                     2
            2048 |                                         0

  sigaction
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |@@@@@@@@@@@@@@@@@@@@                     2
            1024 |@@@@@@@@@@@@@@@@@@@@                     2
            2048 |                                         0

  getuid
           value  ------------- Distribution ------------- count
             128 |                                         0
             256 |@@@@@@@@@@@@@@@@@@@@                     1
             512 |                                         0
            1024 |                                         0
            2048 |                                         0
            4096 |@@@@@@@@@@@@@@@@@@@@                     1
            8192 |                                         0

  accept
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@@@@@@@@                     1
            4096 |@@@@@@@@@@@@@@@@@@@@                     1
            8192 |                                         0

  sysconfig
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |@@@@@@@@@@@@@                            2
            1024 |@@@@@@@@@@@@@@@@@@@@                     3
            2048 |@@@@@@@                                  1
            4096 |                                         0

  bind
           value  ------------- Distribution ------------- count
            2048 |                                         0
            4096 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 2
            8192 |                                         0

  lwp_sigmask
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@       11
            1024 |@@@                                      1
            2048 |@@@                                      1
            4096 |                                         0

  recvfrom
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 5
            4096 |                                         0

  setsockopt
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@                               1
            2048 |@@@@@@@@@@                               1
            4096 |@@@@@@@@@@@@@@@@@@@@                     2
            8192 |                                         0

  fcntl
           value  ------------- Distribution ------------- count
             128 |                                         0
             256 |@@@                                      1
             512 |@@@@@@@@@@@@@@@@@@@@                     8
            1024 |@@@@@@@@@@@@@@@                          6
            2048 |@@@                                      1
            4096 |                                         0

  systeminfo
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@@@@@@                          3
            2048 |@@@@@@@@@@@@@@@@@@@@@@@@@                5
            4096 |                                         0

  schedctl
           value  ------------- Distribution ------------- count
            8192 |                                         0
           16384 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 1
           32768 |                                         0

  getsockopt
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@          11
            2048 |@@@@@@@@@                                3
            4096 |                                         0

  stat
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@@@@@@@@@@@@@@@              4
            4096 |                                         0
            8192 |@@@@@@@@@@@@@                            2
           16384 |                                         0

  recvmsg
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@        15
            4096 |@@@@                                     2
            8192 |@@                                       1
           16384 |                                         0

  connect
           value  ------------- Distribution ------------- count
            2048 |                                         0
            4096 |@@@@@@@@@@@@@@@@@@@@@@@@@@@              4
            8192 |                                         0
           16384 |@@@@@@@@@@@@@                            2
           32768 |                                         0

  brk
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@                               2
            2048 |@@@@@@@@@@@@@@@                          3
            4096 |                                         0
            8192 |@@@@@@@@@@                               2
           16384 |                                         0
           32768 |@@@@@                                    1
           65536 |                                         0

  so_socket
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@@@@@@@@@@                   5
            4096 |                                         0
            8192 |@@@@@@@@@                                2
           16384 |@@@@@@@@@                                2
           32768 |                                         0

  mmap
           value  ------------- Distribution ------------- count
           32768 |                                         0
           65536 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 1
          131072 |                                         0

  putmsg
           value  ------------- Distribution ------------- count
           32768 |                                         0
           65536 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 1
          131072 |                                         0

  p_online
           value  ------------- Distribution ------------- count
             128 |                                         0
             256 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  250
             512 |@                                        5
            1024 |                                         0
            2048 |                                         1
            4096 |                                         0

  getpid
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@                               9
            2048 |@@@@@@@@@@@@@@@@@@@@@@@                  21
            4096 |@@                                       2
            8192 |@@@@                                     4
           16384 |                                         0

  close
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@@@@@@@@@@                      13
            2048 |@@@@@@@                                  5
            4096 |@@@@@@@                                  5
            8192 |@@@                                      2
           16384 |@@@@                                     3
           32768 |                                         0

  lseek
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@@@@@                           22
            2048 |@@@@@@@@@@@@@@@@@@@@@@@                  37
            4096 |@@                                       3
            8192 |@                                        2
           16384 |                                         0

  open
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@@@@@@@@@@@@                            6
            4096 |@@@@@@@                                  3
            8192 |@@@@@@@@@@@@@                            6
           16384 |@@                                       1
           32768 |@@@@                                     2
           65536 |                                         0

  lwp_cond_signal
           value  ------------- Distribution ------------- count
            2048 |                                         0
            4096 |@@@@@@                                   2
            8192 |                                         0
           16384 |@@@@@@@@@@@@@@@@@@@@@@@                  8
           32768 |@@@@@@@@@@@                              4
           65536 |                                         0

  sendmsg
           value  ------------- Distribution ------------- count
            8192 |                                         0
           16384 |@@@@                                     1
           32768 |@@@@@@@@@@@@@@@@@@@@@@@@@@@              6
           65536 |@@@@@@@@@                                2
          131072 |                                         0

  gtime
           value  ------------- Distribution ------------- count
             128 |                                         0
             256 |@@@@@@@@@@@@@@@@@@@@                     576
             512 |@@@@@@@@@@@@@@@@                         452
            1024 |@@                                       56
            2048 |@                                        34
            4096 |                                         3
            8192 |                                         8
           16384 |                                         1
           32768 |                                         1
           65536 |                                         0

  read
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |                                         1
            1024 |@@@@@@                                   19
            2048 |@@@@@@@@@@                               34
            4096 |@@@@@                                    15
            8192 |@@@@@@@@@@@@@@@@@@                       61
           16384 |@                                        2
           32768 |                                         0

  stat64
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@                                        2
            4096 |@                                        1
            8192 |@@@                                      5
           16384 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@          52
           32768 |@@@@                                     7
           65536 |                                         0

  send
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |@@                                       2
            4096 |@                                        1
            8192 |@@@@@@@@@@@@@@@@@@                       22
           16384 |@@@@@@@@@@                               12
           32768 |@@@@@                                    6
           65536 |@@@                                      4
          131072 |@@                                       3
          262144 |                                         0

  write
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |@@                                       5
            1024 |@@@@                                     8
            2048 |                                         0
            4096 |@@@                                      6
            8192 |@@@@@@@@@@@                              23
           16384 |@@@@@@@@                                 16
           32768 |@@@@@@@@@@                               22
           65536 |@@                                       4
          131072 |                                         0
          262144 |                                         1
          524288 |                                         0

  doorfs
           value  ------------- Distribution ------------- count
            2048 |                                         0
            4096 |@@@@@@@@@@@@@@@@@@@@                     1
            8192 |                                         0
           16384 |                                         0
           32768 |                                         0
           65536 |                                         0
          131072 |                                         0
          262144 |                                         0
          524288 |                                         0
         1048576 |                                         0
         2097152 |@@@@@@@@@@@@@@@@@@@@                     1
         4194304 |                                         0

  yield
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@@@@@@@@@@@@@@@@@@@@@                    198
            2048 |@                                        5
            4096 |                                         1
            8192 |@@@@@@                                   61
           16384 |@@@@@@@@@@@@                             110
           32768 |                                         2
           65536 |                                         2
          131072 |                                         0
          262144 |                                         1
          524288 |                                         0

  recv
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |@@@@@@@@@@@@                             17
            1024 |@@@@                                     6
            2048 |@@@@@@@@@@@@@                            19
            4096 |@@@@                                     6
            8192 |                                         0
           16384 |                                         0
           32768 |                                         0
           65536 |                                         0
          131072 |@@                                       3
          262144 |@@@@                                     5
          524288 |                                         0
         1048576 |                                         0
         2097152 |                                         0
         4194304 |                                         0
         8388608 |                                         0
        16777216 |@                                        1
        33554432 |                                         0

  nanosleep
           value  ------------- Distribution ------------- count
         4194304 |                                         0
         8388608 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 201
        16777216 |                                         0
        33554432 |                                         0
        67108864 |                                         0
       134217728 |                                         0
       268435456 |                                         0
       536870912 |                                         2
      1073741824 |                                         0

  ioctl
           value  ------------- Distribution ------------- count
             128 |                                         0
             256 |@                                        50
             512 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@   2157
            1024 |                                         17
            2048 |                                         8
            4096 |                                         10
            8192 |                                         13
           16384 |                                         6
           32768 |                                         3
           65536 |                                         7
          131072 |                                         0
          262144 |                                         0
          524288 |                                         0
         1048576 |                                         0
         2097152 |                                         0
         4194304 |                                         0
         8388608 |                                         2
        16777216 |                                         0
        33554432 |                                         0
        67108864 |                                         6
       134217728 |                                         0
       268435456 |                                         1
       536870912 |                                         4
      1073741824 |                                         0

  lwp_cond_wait
           value  ------------- Distribution ------------- count
        16777216 |                                         0
        33554432 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@          46
        67108864 |@@@@                                     6
       134217728 |                                         0
       268435456 |@                                        2
       536870912 |@@@@                                     6
      1073741824 |                                         0

  pollsys
           value  ------------- Distribution ------------- count
             512 |                                         0
            1024 |@                                        2
            2048 |                                         1
            4096 |                                         1
            8192 |                                         1
           16384 |                                         0
           32768 |                                         0
           65536 |@                                        2
          131072 |                                         0
          262144 |@@                                       6
          524288 |@                                        2
         1048576 |@@                                       5
         2097152 |@                                        2
         4194304 |                                         0
         8388608 |@@@                                      8
        16777216 |                                         0
        33554432 |                                         1
        67108864 |@@@@@@@@@@@@@@@@@@                       56
       134217728 |@@@@@@@@@                                28
       268435456 |                                         0
       536870912 |@@@                                      8
      1073741824 |                                         1
      2147483648 |                                         0

  lwp_park
           value  ------------- Distribution ------------- count
            1024 |                                         0
            2048 |                                         6
            4096 |@                                        21
            8192 |@                                        20
           16384 |@@@@@@@@@                                134
           32768 |@@@@@@@@@@                               145
           65536 |@@@@@@@@@@                               153
          131072 |@                                        11
          262144 |                                         3
          524288 |                                         0
         1048576 |                                         0
         2097152 |                                         0
         4194304 |                                         0
         8388608 |@@@@@@                                   89
        16777216 |                                         0
        33554432 |                                         0
        67108864 |                                         0
       134217728 |                                         2
       268435456 |                                         4
       536870912 |@                                        18
      1073741824 |                                         2
      2147483648 |                                         0

  portfs
           value  ------------- Distribution ------------- count
             256 |                                         0
             512 |                                         4
            1024 |@@@                                      41
            2048 |@@@@                                     44
            4096 |@                                        14
            8192 |                                         2
           16384 |                                         1
           32768 |                                         2
           65536 |                                         1
          131072 |                                         3
          262144 |@                                        10
          524288 |                                         3
         1048576 |                                         6
         2097152 |                                         3
         4194304 |                                         0
         8388608 |@@@@@@@@@@@@@@@@@@@@@@@                  272
        16777216 |                                         1
        33554432 |                                         1
        67108864 |                                         4
       134217728 |                                         2
       268435456 |                                         5
       536870912 |@@@@@                                    58
      1073741824 |                                         3
      2147483648 |                                         1
      4294967296 |                                         0


```

For those keeping score, that would be 482 lines of output -- and that was for just a couple of seconds. Brendan's observation was that it would be great to tip these aggregations on their side (so their bars point up-and-down, not left-to-right) -- with the output for each key appearing on one line. This is clearly a great idea, and "`-x aggpack`" was born:

```
# dtrace -n syscall:::entry'{self->ts = vtimestamp}' \
    -n syscall:::return'/self->ts/{@[probefunc] = \
    quantize(vtimestamp - self->ts); self->ts = 0}' -x aggpack
dtrace: description 'syscall:::entry' matched 233 probes
dtrace: description 'syscall:::return' matched 233 probes
^C

              key  min .------------------. max     | count
       sigpending    8 :     X            : 1048576 | 1
     lwp_continue    8 :      X           : 1048576 | 1
           uucopy    8 :      X           : 1048576 | 1
           getuid    8 :  x   x           : 1048576 | 2
         lwp_kill    8 :       X          : 1048576 | 1
        sigaction    8 :   xx x           : 1048576 | 4
           llseek    8 :     x x          : 1048576 | 2
          fstat64    8 :     xx           : 1048576 | 4
       setcontext    8 :      xx          : 1048576 | 2
      lwp_sigmask    8 :   xx _           : 1048576 | 10
       systeminfo    8 :     xx           : 1048576 | 4
        sysconfig    8 :   ____x          : 1048576 | 6
           accept    8 :       xx         : 1048576 | 2
             bind    8 :         X        : 1048576 | 2
            fstat    8 :     x    x       : 1048576 | 2
         recvfrom    8 :        X         : 1048576 | 5
           doorfs    8 :         xx       : 1048576 | 2
              brk    8 :     ___xx        : 1048576 | 8
       lwp_create    8 :           X      : 1048576 | 1
         schedctl    8 :       x   x      : 1048576 | 2
         p_online    8 :   X_  _          : 1048576 | 256
           getpid    8 :      xx_ _       : 1048576 | 28
          recvmsg    8 :       xx_        : 1048576 | 18
             open    8 :         xx x     : 1048576 | 4
            lseek    8 :       xx _ _     : 1048576 | 9
       getsockopt    8 :      _xx         : 1048576 | 43
             stat    8 :        x_x _     : 1048576 | 7
             mmap    8 :             X    : 1048576 | 1
           putmsg    8 :             X    : 1048576 | 1
             recv    8 :     _xxx__       : 1048576 | 65
            gtime    8 :   X_____         : 1048576 | 899
            fcntl    8 :   ___xx          : 1048576 | 208
       setsockopt    8 :      __x__       : 1048576 | 55
          sendmsg    8 :           xxx    : 1048576 | 9
  lwp_cond_signal    8 :       _x__x      : 1048576 | 33
            close    8 :       __xx_      : 1048576 | 65
    lwp_cond_wait    8 :        _xx_      : 1048576 | 69
           munmap    8 :                X : 1048576 | 1
             read    8 :    ___x_x_ _ _   : 1048576 | 137
           stat64    8 :        ___X      : 1048576 | 49
            yield    8 : _    X___        : 1048576 | 1314
        so_socket    8 :        ___x_     : 1048576 | 60
        nanosleep    8 :         _x__     : 1048576 | 77
             send    8 :      ____xx__    : 1048576 | 66
          pollsys    8 :       _ _xx_     : 1048576 | 86
            ioctl    8 :    X_________    : 1048576 | 3259
          connect    8 :         _ _xx_   : 1048576 | 57
           mmap64    8 :        __x _   x : 1048576 | 14
            write    8 :     __ __x_____  : 1048576 | 115
           portfs    8 :     ______x_     : 1048576 | 619
         lwp_park    8 :        _xx_x_    : 1048576 | 654

```

This communicates essentially the same amount of information in just a fraction of the output (55 lines versus 482) -- and makes it (much) easier to quickly compare distributions across aggregation keys. That said, one of the challenges here is that ASCII can be very limiting when trying to come with characters of clearly different height. When I demonstrated a prototype of this at the illumos BOF at [Surge](http://surge.omniti.com/2013), a couple of folks volunteered that I should look into using the Unicode Block Elements for this output. Here's the same enabling, but rendered using these elements:

```
# dtrace -n syscall:::entry'{self->ts = vtimestamp}' \
    -n syscall:::return'/self->ts/{@[probefunc] = \
    quantize(vtimestamp - self->ts); self->ts = 0}' -x aggpack
dtrace: description 'syscall:::entry' matched 233 probes
dtrace: description 'syscall:::return' matched 233 probes
^C


              key  min .-------------------. max     | count
         lwp_self   32 :   █               : 8388608 | 1
        sysconfig   32 : ▃▃▃               : 8388608 | 3
       sigpending   32 :    █              : 8388608 | 1
             pset   32 :  ▆ ▃              : 8388608 | 3
      getsockname   32 :     █             : 8388608 | 1
       systeminfo   32 :   ▅▄              : 8388608 | 5
            fstat   32 :      █            : 8388608 | 1
           llseek   32 :    ▆▃             : 8388608 | 3
          memcntl   32 :      █            : 8388608 | 1
      resolvepath   32 :      █            : 8388608 | 1
           semsys   32 :      █            : 8388608 | 1
       setcontext   32 :     █             : 8388608 | 2
          setpgrp   32 :      █            : 8388608 | 1
          fstat64   32 :   ▃▃▃             : 8388608 | 6
        sigaction   32 : ▃▃▂▃              : 8388608 | 17
           accept   32 :       █           : 8388608 | 1
           access   32 :       █           : 8388608 | 1
           munmap   32 :       █           : 8388608 | 1
        setitimer   32 :    ▅▃▃            : 8388608 | 4
      lwp_sigmask   32 : ▄▃▃▂              : 8388608 | 26
             pipe   32 :        █          : 8388608 | 1
          waitsys   32 :   ▅    ▅          : 8388608 | 2
             stat   32 :     ▃ ▆           : 8388608 | 3
             bind   32 :      ▅▄           : 8388608 | 5
          mmapobj   32 :         █         : 8388608 | 1
           open64   32 :         █         : 8388608 | 1
         p_online   32 : █▁ ▁              : 8388608 | 256
           getpid   32 :   ▁▆▂             : 8388608 | 45
         schedctl   32 :         █         : 8388608 | 2
       getsockopt   32 :    ▃▅▂            : 8388608 | 32
       setsockopt   32 :   ▁▁▃▆            : 8388608 | 31
            fcntl   32 : ▂▁▃▃▂             : 8388608 | 114
          recvmsg   32 :    ▁▄▃▂▁▁         : 8388608 | 17
           putmsg   32 :           █       : 8388608 | 1
             mmap   32 :       ▅   ▅       : 8388608 | 2
           writev   32 :         ▆ ▃       : 8388608 | 3
            lseek   32 :   ▁▃▃▄▁▁          : 8388608 | 76
              brk   32 :  ▃▂▃▃▂▁▁ ▁        : 8388608 | 124
            gtime   32 : ▇▁▁▁▁▁ ▁          : 8388608 | 1405
          sendmsg   32 :        ▂▅▄        : 8388608 | 10
             recv   32 :   ▂▃▃▂▂▁          : 8388608 | 159
            yield   32 :    █▁▁▁           : 8388608 | 409
             open   32 :      ▃▂▄▁▂        : 8388608 | 30
            close   32 :    ▂▃▂▃▂▁         : 8388608 | 60
        so_socket   32 :       ▁▃▆         : 8388608 | 29
  lwp_cond_signal   32 :    ▁▃▂▂▁▃▁        : 8388608 | 59
        nanosleep   32 :       ▂▆▂         : 8388608 | 74
    lwp_cond_wait   32 :      ▂▃▄▂         : 8388608 | 120
          connect   32 :         ▂▅▃       : 8388608 | 24
           stat64   32 :       ▁▂▇▁        : 8388608 | 77
          pollsys   32 :   ▁▁▁ ▁▅▄         : 8388608 | 148
             send   32 :    ▁▂▂▂▂▃▂▁       : 8388608 | 137
             read   32 :   ▁▂▃▂▃▂▁▁▁▁      : 8388608 | 283
            ioctl   32 :  ▇▁▁▁▁▁▁▁▁▁▁      : 8388608 | 3697
            write   32 :   ▁▁▁▂▂▃▂▁▁ ▁▁    : 8388608 | 207
          forksys   32 :                 █ : 8388608 | 1
           portfs   32 :   ▁▂▂▁▁▂▄▁ ▁      : 8388608 | 807
         lwp_park   32 :  ▁ ▁▁▁▃▃▂▃▁       : 8388608 | 919

```

Delicious! (Assuming, of course, that these look right for you -- which they may or may not, depending on how your monospaced font renders the Unicode Block Elements.) After one look at the Unicode Block Elements, it clearly had to be the default behavior -- but if your terminal is rendered in a font that can't display the UTF-8 encodings of these characters (less common) or if they render in a way that is not monospaced despite being in a putatively monospaced font (more common), I also added a "`-x encoding`" option that can be set to "`ascii`" to force the ASCII output.

Returning to our example, if the above is too much dead space for you, you can combine it with `aggzoom`:

```
# dtrace -n syscall:::entry'{self->ts = vtimestamp}' \
    -n syscall:::return'/self->ts/{@[probefunc] = \
    quantize(vtimestamp - self->ts); self->ts = 0}' -x aggpack -x aggzoom
dtrace: description 'syscall:::entry' matched 233 probes
dtrace: description 'syscall:::return' matched 233 probes
^C


              key  min .----------------. max     | count
     lwp_continue   32 :    █           : 1048576 | 1
       sigpending   32 :    █           : 1048576 | 1
           uucopy   32 :    █           : 1048576 | 1
         lwp_kill   32 :     █          : 1048576 | 1
        sigaction   32 : ▄▄ █           : 1048576 | 4
             pset   32 :  █  ▄          : 1048576 | 3
        sysconfig   32 : ███ █          : 1048576 | 4
       setcontext   32 :    ██          : 1048576 | 2
            fstat   32 :      █         : 1048576 | 1
         recvfrom   32 :      █         : 1048576 | 1
       systeminfo   32 :   ▅█           : 1048576 | 5
      lwp_sigmask   32 : ██▂▅           : 1048576 | 14
           accept   32 :     █ █        : 1048576 | 2
          recvmsg   32 :     █ █        : 1048576 | 2
             stat   32 :        █       : 1048576 | 2
         p_online   32 : █▁ ▁           : 1048576 | 256
         schedctl   32 :      █  █      : 1048576 | 2
              brk   32 :    ▄███▄       : 1048576 | 8
       getsockopt   32 :    ▅██         : 1048576 | 21
           getpid   32 :    █▇          : 1048576 | 39
       setsockopt   32 :     ▃█▂        : 1048576 | 16
       lwp_create   32 :          █     : 1048576 | 1
          sendmsg   32 :          █     : 1048576 | 1
            fcntl   32 : ▃▄▃▆█▁         : 1048576 | 60
           writev   32 :           █    : 1048576 | 1
            lseek   32 :    ▆▇█▂        : 1048576 | 65
             recv   32 :  ▁▆▃█▆▅        : 1048576 | 62
           putmsg   32 :           █    : 1048576 | 2
             open   32 :      █ ▅▂▃     : 1048576 | 16
            gtime   32 : █▁▁▁▁  ▁       : 1048576 | 1365
            close   32 :    ▂█▂█▄▃      : 1048576 | 36
  lwp_cond_signal   32 :     ▂▂█▅▅▂     : 1048576 | 18
        so_socket   32 :      ▁▂▁█      : 1048576 | 19
             mmap   32 :     ▆▃ █▄ ▆    : 1048576 | 13
        nanosleep   32 :       ▃█▃      : 1048576 | 34
             read   32 :   ▁▂▅▄█▅▂      : 1048576 | 144
    lwp_cond_wait   32 :       ▃█▂▁     : 1048576 | 74
            yield   32 :    █▁▁▁ ▁      : 1048576 | 1187
             send   32 :     ▁▂▅█▅▄▂    : 1048576 | 50
          connect   32 :      ▂▂ ▅██▃   : 1048576 | 20
           stat64   32 :     ▁▁▁▃█▁     : 1048576 | 80
          pollsys   32 :     ▁▁▁█▆      : 1048576 | 136
            write   32 :  ▂▄ ▃▂▄█▆▃▂ ▂  : 1048576 | 77
            ioctl   32 :  █▂▁▁▁▁▁▁ ▁    : 1048576 | 4587
           mmap64   32 :      ▂▄▄▂▅ ▂ █ : 1048576 | 20
           portfs   32 :  ▁▁▂▂▁▂▂█▂     : 1048576 | 601
         lwp_park   32 :    ▁▁▂▆▇▃█▁▁   : 1048576 | 787

```

Here's another fun one:

```
# dtrace -n BEGIN'{start = timestamp}' \
    -n sched:::on-cpu'{@[execname] = \
    lquantize((timestamp - start) / 1000000000, 0, 30, 1)}' \
    -x aggpack -n tick-1sec'/++i >= 30/{exit(0)}' -q
              key     min .--------------------------------. max      | count
             sshd     < 0 : █                              : >= 30    | 1
            zpool     < 0 : █                              : >= 30    | 8
              zfs     < 0 : █                              : >= 30    | 19
          zoneadm     < 0 : █                              : >= 30    | 19
            utmpd     < 0 :     █                          : >= 30    | 1
            intrd     < 0 :                             █  : >= 30    | 2
             epmd     < 0 :   ▂    ▂    ▂    ▂    ▂    ▂   : >= 30    | 6
            mdnsd     < 0 :   ▂    ▂    ▂    ▂    ▂    ▂   : >= 30    | 6
         sendmail     < 0 :   ▂    ▂    ▂    ▂    ▂    ▂   : >= 30    | 6
         ur-agent     < 0 :   ▂▂         ▂         ▂   ▃▂  : >= 30    | 14
           vmadmd     < 0 :     ▃    ▂    ▂    ▂    ▂    ▃ : >= 30    | 22
         devfsadm     < 0 : ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁ ▁  : >= 30    | 30
          bridged     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 30
          fsflush     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 32
              fmd     < 0 :        ▁▁        ▁▇        ▁▁  : >= 30    | 41
            ipmon     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 51
          ipmgmtd     < 0 :         ▃         ▃         ▃  : >= 30    | 66
            lldpd     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 90
           dtrace     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 111
          haproxy     < 0 : ▂▁▁▁▁▁▁▂▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▂▁▁ : >= 30    | 113
      svc.configd     < 0 :         ▂▁        ▄▁        ▂▁ : >= 30    | 126
       svc.startd     < 0 :         ▃▁        ▃▁        ▃▁ : >= 30    | 150
         beam.smp     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 209
             nscd     < 0 : ▁       ▃▂        ▃▂        ▃▂ : >= 30    | 192
         postgres     < 0 : ▁▁▁▁▁▁▁▂▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 598
     redis-server     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 598
      zpool-zones     < 0 :     ▂  ▁ ▂    ▃  ▁ ▂    ▂  ▁ ▃ : >= 30    | 623
             java     < 0 : ▁▁▁▁▁▂▁▁▁▁▁▁▁▁▁▁▁▁▂▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 1211
             tail     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 1178
             node     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▂▁ : >= 30    | 11195
            sched     < 0 : ▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁ : >= 30    | 22101

```

This (quickly) shows you the second offset for CPU scheduling events from the start of the D script for various applications, and points to some obvious conclusions: there are clearly a bunch of programs running once every ten seconds, a couple more running once every five seconds, and so on.

## `encoding`

Seeing this, why should `aggpack` hog all that Unicode hawtness? While only `aggpack` will use the UTF-8 encodings by default, I also extended the `encoding` option to allow a "`utf8`" setting that forces the UTF-8 encoding of the Unicode Block Elements to be used for all aggregation display. For example, to return to our earlier `agghist` example:

```
# dtrace -n syscall:::entry'{@[execname] = count()}' \
    -x agghist -x aggzoom -x encoding=utf8
dtrace: description 'syscall:::entry' matched 233 probes
dtrace: description 'syscall:::entry' matched 233 probes
^C

              key  ------------- Distribution ------------- count
            utmpd |                                         2
             sshd |                                         10
           pickup |                                         14
              fmd |                                         20
         ur-agent |                                         21
             epmd |                                         22
           vmadmd |                                         31
      jsontool.js |                                         33
            mdnsd |                                         33
           master |                                         41
            intrd |                                         45
         devfsadm |                                         54
          ipmgmtd |                                         55
          bridged |                                         81
            mkdir |                                         83
              cat |                                         96
         sendmail |                                         101
            ipmon |                                         108
          dirname |                                         115
       svc.startd |                                         129
            ksh93 |                                         164
               ls |                                         182
          svcprop |▏                                        258
             bash |▏                                        290
            zpool |▎                                        436
             date |▎                                        448
             cron |▎                                        470
            lldpd |▍                                        595
 dump-minutely-sd |▍                                        609
          haproxy |▍                                        654
      svc.configd |▌                                        907
         zoneadmd |█                                        1554
              zfs |█▎                                       1979
          zoneadm |█▋                                       2414
             nscd |█▋                                       2522
             java |██▏                                      3205
             tail |██▌                                      3775
         postgres |████▏                                    6083
     redis-server |██████████████▋                          21710
           dtrace |███████████████████▎                     28509
             node |█████████████████████████▍               37427
         beam.smp |██████████████████████████████████████   55948

```

To paraphrase [Kent Brockman](http://www.youtube.com/watch?v=Du0d8QxlSYk), I for one welcome our new Unicode Block Element overlords -- and I look forward to toiling in their underground sugar caves!

## Availability

All of these new options have been [integrated into SmartOS](https://github.com/joyent/illumos-joyent/commit/78e24ab6803bbe11ba37642624e1498ede5b239d) (and we aim to get them up into [illumos](http://wiki.illumos.org/display/illumos/illumos+Home)). If you're a Joyent public cloud customer, you already all have these improvements or they will be coming to you when your platform is next upgraded (depending on the compute node that you have landed on). And if you're a DTrace user and like Brendan and have some new way that you'd like to see data visualized (or have any other DTrace feature request), don't hesitate to speak up -- DTrace only improves when those of us who depend on it every day envision a way that it could be even better!
