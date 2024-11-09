---
title: "Log/linear quantizations in DTrace"
date: "2011-02-09"
---

As [I alluded to when I first joined Joyent](http://dtrace.org/blogs/bmc/2010/07/30/hello-joyent/), we are using DTrace as a foundation to tackle the cloud observability problem. And as you may have seen, [we're making some serious headway](http://dtrace.org/blogs/brendan/2011/01/24/cloud-analytics-first-video/) on this problem. Among the gritty details of scalably instrumenting latency is a dirty little problem around data aggregation: when we instrument the system to record latency, we are (of course) using DTrace to aggregate that data _in situ_, but with what granularity? For some operations, visibility into microsecond resolution is critical, while for other (longer) operations, millisecond resolution is sufficient - and some classes of operations may take hundreds or thousands (!) of milliseconds to complete. If one makes the wrong choice as the resolution of aggregation, one either ends up clogging the system with unnecessarily fine-grained data, or discarding valuable information in overly coarse-grained data. One might think that one could infer the interesting latency range from the nature of the operation, but this is unfortunately not the case: I/O operations (a typically-millisecond resolution operation) can take but microseconds, and CPU scheduling operations (a typically-microsecond resolution operation) can take hundreds of milliseconds. Indeed, the problem is that one often does not know what order of magnitude of resolution is interesting for a particular operation until one has a feel for the data for the specific operation -- one needs to instrument the system to best know how to instrument the system!

We had similar problems at [Fishworks](http://dtrace.org/blogs/bmc/2008/11/10/fishworks-now-it-can-be-told/), of course, and my solution there was wholly dissatisfying: when instrumenting the latency of operations, I put in place not one but many (five) `lquantize()`'d aggregations that (linearly) covered disjoint latency ranges at different orders of magnitude. As we approached this problem again at Joyent, [Dave](http://dtrace.org/blogs/dap/) and [Robert](http://dtrace.org/blogs/rm/) both balked at the clumsiness of the multiple aggregation solution (with any olfactory reaction presumably heightened by the fact that they were going to be writing the pungent code to deal with this). As we thought about the problem in the abstract, this really seemed like something that DTrace itself should be providing: an aggregation that is neither logarithmic (i.e., `quantize()`) nor linear (i.e., `lquantize()`), but rather something in between...

After playing around with some ideas and exploring some dead-ends, we came to an aggregating action that logarithmically aggregates by order of magnitude, but then linearly aggregates within an order of magnitude. The resulting aggregating action -- dubbed "log/linear quantization" and denoted with the new `llquantize()` aggregating action -- is paramaterized with a factor, a low magnitude, a high magnitude and the number of linear steps per magnitude. With the caveat that these bits are still hot from the oven, the patch for `llquantize()` (which should patch cleanly against illumos) is [here](http://dtrace.org/resources/bmc/dtrace-llquantize.patch).

To understand the problem we're solving, it's easiest to see `llquantize()` in action:

```
# cat > llquant.d
tick-1ms
{
	/*
	 * A log-linear quantization with factor of 10 ranging from magnitude
	 * 0 to magnitude 6 (inclusive) with twenty steps per magnitude
	 */
	@ = llquantize(i++, 10, 0, 6, 20);
}

tick-1ms
/i == 1500/
{
	exit(0);
}
^D
# dtrace -V
dtrace: Sun D 1.7
# dtrace -qs ./llquant.d

           value  ------------- Distribution ------------- count
             < 1 |                                         1
               1 |                                         1
               2 |                                         1
               3 |                                         1
               4 |                                         1
               5 |                                         1
               6 |                                         1
               7 |                                         1
               8 |                                         1
               9 |                                         1
              10 |                                         5
              15 |                                         5
              20 |                                         5
              25 |                                         5
              30 |                                         5
              35 |                                         5
              40 |                                         5
              45 |                                         5
              50 |                                         5
              55 |                                         5
              60 |                                         5
              65 |                                         5
              70 |                                         5
              75 |                                         5
              80 |                                         5
              85 |                                         5
              90 |                                         5
              95 |                                         5
             100 |@                                        50
             150 |@                                        50
             200 |@                                        50
             250 |@                                        50
             300 |@                                        50
             350 |@                                        50
             400 |@                                        50
             450 |@                                        50
             500 |@                                        50
             550 |@                                        50
             600 |@                                        50
             650 |@                                        50
             700 |@                                        50
             750 |@                                        50
             800 |@                                        50
             850 |@                                        50
             900 |@                                        50
             950 |@                                        50
            1000 |@@@@@@@@@@@@@                            500
            1500 |                                         0
```

As you can see in the example, at each order of magnitude (determined by the factor -- 10, in this case), we have 20 steps (or the factor at orders of magnitude less than the number of steps): up to 10, the buckets each span 1 value; from 10 through 100, each spans 5 values; from 100 to 1000, each spans 50; from 1000 to 10000 each spans 500, and so on. The result is not necessarily easy on the eye (and indeed, it seems unlikely that llquantize() will find much use on the command line), but is easy on the system: it generates data that has the resolution of linearly quantized data with the efficiency of logarithmically quantized data.

Of course, you needn't appreciate the root system here to savor the fruit: if you are a Joyent customer, you will soon be benefitting from `llquantize()` without ever having to wade into the details, as we bring [real-time analytics](http://joyeur.com/2011/01/24/executive-speaker-series-bryan-cantrill-and-brendan-gregg-on-cloud-analytics/) to the cloud. Stay tuned!
