---
title: "Serving up disaster porn with Manta"
date: "2013-07-23"
---

Years ago, [Ben Fried](https://plus.google.com/108690239590381459413/about) liberated me by giving me the words to describe myself: I am a disaster porn addict. The sordid specifics of my addiction would surely shock the unafflicted: there are well-thumbed [NTSB final accident reports](http://www.ntsb.gov/doclib/reports/2001/AAR0101.pdf) hidden under my matress; I prowl the internet late at night for pictures of [dermoid cysts](http://24.media.tumblr.com/tumblr_m4b0u2AAqm1rwyutuo1_500.jpg); and I routinely binge on the vexed [Soviet missions to Mars](https://en.wikipedia.org/wiki/Phobos_1).

In terms of my software work, my predilection for (if not addiction to) systems failure has manifested itself in an acute interest in postmortem debugging and debuggability. You can see this through my career, be it [postmortem memory leak detection](http://illumos.org/books/mdb/kmem-1.html#casestudy-34) in 1999, [postmortem object type identifcation](http://arxiv.org/abs/cs/0309037) in 2002, [postmortem DTrace](http://dtrace.org/guide/chapter37.html) in 2003 or [postmortem debugging of JavaScript](http://dtrace.org/blogs/bmc/2012/05/05/debugging-node-js-memory-leaks/) in 2012.

That said, postmortem debugging does have a dark side -- or at least, a tedious side: the dumps themselves are annoyingly large artifacts to deal with. Small ones are only tens of megabytes, but a core dump from a modestly large VM -- and certainly a crash dump of a running operating system -- will easily be gigabytes or tens of gigabyes in size. This problem is not new, and my career has been pockmarked by centralized dump servers that were constantly running low of space\[1\]. To free up space in the virtual morgue, dumps from resolved issues are inevitably torched on a regular basis. This act strikes me as a a kind of desecration; every dump -- even one whose immediate failure is understood -- is a sacred snapshot of a running system, and we can't know what questions we might have in the future that may be answerable by the past. There have been many times in my career when debugging a new problem has led to my asking new questions of old dumps.

At Joyent, we have historically gotten by on the dump management problem with the traditional centralized dump servers managed with creaky shell scripts -- but it wasn't pretty, and I lived in fear of dumps slipping through the cracks. Ironically, it was the development of [Manta](http://www.joyent.com/products/manta) that made our kludged-together solution entirely untenable: while in the Joyent public cloud we may have the luxury of (broadly) ignoring core dumps that may correspond to errant user code, in Manta, we care about _every_ core dump from _every_ service -- we always want to understand why any component fails. But because it's a distributed service, a single bug could cause many components to fail -- and generate many, many dumps. In the earlier stages of Manta development, we quickly found ourselves buried under an avalanche of hundreds of dumps. Many of them were due to known issues, of course, but I was concerned that a hithertofore unseen issue would remain hidden among the rubble -- and that we would lose our one shot to debug a problem that would return to haunt us in production.

## A sexy solution

Of course, the answer to the Manta dump problem was clear: Manta may have been presenting us with a big data problem, but it also provided us a big data solution. Indeed, Manta is perfectly suited for the dump problem: dumps are big, but they also require computation (if in modest measure) to make any real use of them, and [Manta's compute abstraction of the operating system](http://www.joyent.com/blog/hello-manta-bringing-unix-to-big-data) is perfect for running the debugger. There were just two small wrinkles, the first being that Manta as designed only allowed computation access to an object via a (non-interactive) job, posing a problem for the fundamentally interactive act of debugging. This wasn't a technological limitation _per se_ (after all, the operating system naturally includes interactive elements like the shell and so on), but allowing an interactive shell to be easily created from a Manta job was going to require some new plumbing. Fortunately, some lunchtime moaning on my part managed to convince [Joshua Clulow](https://github.com/jclulow) to write this (if only to quiet my bleating), and the unadulterated awesomeness that is [mlogin](http://blog.sysmgr.org/2013/06/manta-mlogin.html) was born.

The second wrinkle was even smaller: for historical (if not antiquated) reasons, kernel crash dumps don't have just one file, but two -- a crash dump and a "name list" that contained the symbol table. But for over a decade we have also had the symbol table in the dump itself, and it was clear the vestigial appendix that was unix.0 needed to be removed and libkvm modified to dig it out of the dump -- a [straightforward fix](https://github.com/joyent/illumos-joyent/commit/e083a554cd81f87a93a0b654bcbe3ffa29437ef2).

Wrinkles smoothed, we had the necessary foundation in Manta to implement [Thoth](https://github.com/joyent/manta-thoth), a Manta-based system for dump management. Using the [node-manta](https://github.com/joyent/node-manta) SDK to interact with Manta, Thoth allows for uploading dumps to Manta, tracking auxiliary information about each dump, debugging a dump (natch), querying dumps, and (most interestingly) automatically analyzing many dumps in parallel. (If you are in the [SmartOS](http://smartos.org/) or broader [illumos community](http://illumos.org/) and you want to use Thoth, it's open source and [available on GitHub](https://github.com/joyent/manta-thoth).)

While the problem Thoth solves may have but limited appeal, some of the patterns it uses may be more broadly applicable to those building Manta-based systems. As you might imagine, Thoth uploads a dump by calculating a unique hash for the dump on the client\[2\]. Once the hash is calculated, a Manta directory is created that contains the dump itself. For the metadata associated with the dump (machine, application, datacenter and so on), I took a quick-and-dirty approach: Thoth stores the metadata about a dump in a JSON payload that lives beside the dump in the same directory. Here is what a JSON payload might look like:

```
% thoth info 6433c1ccfb41929d
{
  "name": "/thoth/stor/thoth/6433c1ccfb41929dedf3257a8d9160ea",
  "dump": "/thoth/stor/thoth/6433c1ccfb41929dedf3257a8d9160ea/core.svc.startd.70377",
  "pid": "70377",
  "cmd": "svc.startd",
  "psargs": "/lib/svc/bin/svc.startd",
  "platform": "joyent_20130625T221319Z",
  "node": "eb9ca020-77a6-41fd-aabb-8f95305f9aa6",
  "version": "1",
  "time": 1373566304,
  "stack": [
    "libc.so.1`_lwp_kill+0x15()",
    "libc.so.1`raise+0x2b()",
    "libc.so.1`abort+0x10e()",
    "utmpx_postfork+0x44()",
    "fork_common+0x186()",
    "fork_configd+0x8d()",
    "fork_configd_thread+0x2ca()",
    "libc.so.1`_thrp_setup+0x88()",
    "libc.so.1`_lwp_start()"
  ],
  "type": "core",
}

```

To actually debug a dump, one can use the "debug" command, which simply uses mlogin to allow interactive debugging of the dump via [mdb](http://illumos.org/books/mdb/preface.html):

```
% thoth debug 6433c1ccfb41929d
thoth: debugging 6433c1ccfb41929dedf3257a8d9160ea
 * created interactive job -- 8ea7ce47-a58d-4862-a070-a220ae23e7ce
 * waiting for session... - established
thoth: dump info in $THOTH_INFO
Loading modules: [ svc.startd libumem.so.1 libc.so.1 libuutil.so.1 libnvpair.so.1 ld.so.1 ]
>

```

Importantly, this allows you to share a dump with others without moving the dump and without needing to grant ssh access or punch holes in firewalls -- they need only be able to have access to the object. For an open source system like ours, this alone is huge: if we see a panic in (say) ZFS, I would like to enlist (say) [Matt](https://twitter.com/mahrens1) or [George](http://blog.delphix.com/gwilson/) to help debug it -- even if it's to verify a hypothesis that we have developed. Transporting tens of gigs around becomes so painful that we simply don't do it -- and the quality of software suffers.

To query dumps, Thoth runs simple Manta compute jobs on the (small) metadata objects. For example, if you wanted to find all of the dumps from a particular machine, you would run:

```
% thoth ls node=eb9ca020-77a6-41fd-aabb-8f95305f9aa6
thoth: creating job to list
thoth: created job 2e8af859-ffbf-4757-8b54-b7865307d4d9
thoth: waiting for completion of job 2e8af859-ffbf-4757-8b54-b7865307d4d9
DUMP             TYPE  TIME                NODE/CMD         TICKET
0ccf27af56fe18b9 core  2013-07-11T18:11:43 svc.startd       OS-2359
6433c1ccfb41929d core  2013-07-11T18:11:44 svc.startd       OS-2359

```

This kicks off a job that looks at all of the metadata objects in parallel in a map phase, pulls out those that match the specified machine and passes them to a reduce phase that simply serves to coalesce the results into a larger JSON object. We can use the mjob command along with [Trent Mick](https://github.com/trentm)'s excellent [json](https://github.com/trentm/json) tool to see the first phase:

```
% mjob get 2e8af859-ffbf-4757-8b54-b7865307d4d9 | json phases[0].exec
mls /thoth/stor/thoth | awk '{ printf("/thoth/stor/thoth/%s/info.json\n", $1) }' | xargs mcat

```

So this, as it turns out, is a tad fancy: in the context of a job, it runs [mls](https://github.com/joyent/node-manta/blob/master/docs/man/mls.md) on the thoth directory, formats that as an absolute path name to the JSON object, and then uses the venerable [xargs](http://en.wikipedia.org/wiki/Xargs) to send that output as arguments to [mcat](https://github.com/joyent/manta-compute-bin/blob/master/man/mcat.1). mcat interprets its arguments as Manta objects, and runs the next phase of a job in parallel on those objects. We could -- if we wanted -- do the mls on the client and then pass in the results, but it would require an unnecessary roundtrip between server and client for each object; the power of Manta is that you can get the server to do work for you! Now let's take a look at the next phase:

```
% mjob get 2e8af859-ffbf-4757-8b54-b7865307d4d9 | json phases[1].exec
json -c 'node=="eb9ca020-77a6-41fd-aabb-8f95305f9aa6"'

```

This is very simple: it just uses [json](https://github.com/trentm/json)'s -c option to pass the entire JSON blob if the "node" property matches the specified value. Finally, the last phase:

```
% mjob get 2e8af859-ffbf-4757-8b54-b7865307d4d9 | json phases[2].exec
json -g

```

This uses json (again) to aggregate all of the JSON blobs into one array that contains all of them -- and this array is what is returned as the output of the job. Now, as one might imagine, all of this is a slight abuse of Manta, as it's using Manta to implement a database (albeit crudely) -- but Manta is high performing enough that this works well, and it's much simpler than having to spin up infrastructure dedicated to Thoth (which would require sizing a database, making it available, backing it up, etc). Manta loves it quick and dirty!

## Getting kinky

Leveraging Manta's unique strengths, Thoth also supports a notion of _analyzers_, simple shell scripts that run in the context of a Manta job. For example, here is an analyzer that determines if a dump is a duplicate of a [known svc.startd issue](https://github.com/joyent/illumos-joyent/commit/e69eeac446070858d26f79013a333b85c26f7e63):

```
#
# This analyzer only applies to core files
#
if [[ "$THOTH_TYPE" != "core" ]]; then
	exit 0;
fi

#
# This is only relevant for svc.startd
#
if [[ `cat $THOTH_INFO | json cmd` != "svc.startd" ]]; then
	exit 0;
fi

#
# This is only OS-2359 if we have utmpx_postfork in our stack
#
if ( ! echo ::stack | mdb $THOTH_DUMP | grep utmpx_postfork > /dev/null ); then
	exit 0;
fi

#
# We have a winner! Set the ticket.
#
thoth_ticket OS-2359
echo $THOTH_NAME: successfully diagnosed as OS-2359

```

Thoth's "analyze" command can then be used to run this against one dump, any dumps that match a particular specification, or all dumps.

## The money shot

Here's a more involved analyzer that pulls an [SMF FMRI](http://www.illumos.org/man/5/smf) out of the dump and -- looking at the time of the dump -- finds the log (in Manta!) that corresponds to that service at that hour and looks for a string ("PANIC") in that log that corresponds to service failure. This log snippet is then attached to the dump via a custom property ("paniclog") that can then be queried by other analyzers:

```
if [[ "$THOTH_TYPE" != "core" ]]; then
	exit 0
fi

if ( ! pargs -e $THOTH_DUMP | grep -w SMF_FMRI > /dev/null ); then
	exit 0
fi

fmri=`pargs -e $THOTH_DUMP | grep -w SMF_FMRI | cut -d= -f2-`
time=`cat $THOTH_INFO | json time`
node=`cat $THOTH_INFO | json node`

path=`basename $fmri | cut -d: -f1`/`date -d @$time +%Y/%m/%d/%H`/${node}.log

echo === $THOTH_NAME ===

echo log: $path

if ( ! mget /$MANTA_USER/stor/logs/$path > log.out ); then
	echo "  (not found)"
	exit 0
fi

grep -B 10 -A 10 PANIC log.out > panic.out

echo paniclog:
cat panic.out

thoth_set paniclog < panic.out

```

In a distributed system, logs are a critical component of system state -- being able to quickly (and automatically) couple those logs with core dumps has been huge for us in terms of quickly diagnosing issues.

## Pillow talk

Thoth is definitely a quick-and-dirty system -- I was really designing it to satisfy our own immediate needs (and yes, addictions). If the usage patterns changed (if, for example, one were going to have millions of dumps instead of thousands), one would clearly want a proper database fronting Manta. Of course, it must also be said that one would then need to deal with annoying infrastructure issues like sizing that database, making it available, backing it up, etc. And if (when?) this needs to be implemented, my first approach would be to broadly keep the structure as it is, but add a Manta job that iterates over all metadata and adds the necessary rows in the appropriate tables of a centralized reporting database. This allows Manta to store the canonical data, leveraging the _in situ_ compute to populate caching tiers and/or reporting databases.

As an engineer, it's always gratifying to build something that you yourself want to use. Strange as it may sound to some, Thoth epitomizes that for me: it is the dump management and analysis framework that I never dreamed possible -- and thanks to Manta, it was amazingly easy (and fun!) to build. Most importantly, in just the short while that we've had Thoth fully wired up, we have already caught a few issues that would have likely escaped notice before. (Thanks to a dump harvested by Thoth, we caught [a pretty nasty OS bug](https://github.com/joyent/illumos-joyent/commit/b034663e3b02babe0051b61f6782e7bfb5b7b9b4) that had managed to escape detection for many years.) Even though Thoth's purpose is admittedly narrow, it's an example of the kinds of things that you can build on top of Manta. We have many more such ideas in the works -- and we're not the only ones; Manta is inspiring many with their own big data problems -- from [e-commerce companies](http://building.wanelo.com/post/54110156963/a-cost-effective-approach-to-scaling-event-based-data) to to [scientific computing](http://smartos.blueprint.org/home/manta-unleashed) and much in between!

* * *

\[1\] Pouring one out for my dead homie, cores2.central

\[2\] Shout-out to [Robert](http://dtrace.org/blogs/rm)'s excellent [node-ctype](https://github.com/rmustacc/node-ctype) which allows for binary dumps to be easily parsed client-side
