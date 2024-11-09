---
title: "Debugging node.js memory leaks"
date: "2012-05-05"
---

Part of the value of dynamic and interpreted environments is that they handle the complexities of dynamic memory allocation. In particular, one needn't explicitly free memory that is no longer in use: objects that are no longer referenceable are found automatically and destroyed via garbage collection. While garbage collection simplifies the program's relationship with memory, it not mean the end of all memory-based pathologies: if an application retains a reference to an object that is ultimately rooted in a global scope, that object won't be considered garbage and the memory associated with it will not be returned to the system. If enough such objects build up, allocation will ultimately fail (memory is, after all, finite) and the program will (usually) fail along with it. While this is not strictly -- from the native code perspective, anyway -- a memory leak (the application has not leaked memory so much as neglected to unreference a semantically unused object), the effect is nonetheless the same and the same nomenclature is used.

While all garbage collected environments create the potential to create such leaks, it can be particularly easy in JavaScript: closures create implicit references to variables within their scopes -- references that might not be immediately obvious to the programmer. And node.js adds a new dimension of peril with its strictly asynchronous interface with the system: if backpressure from slow upstream services (I/O, networking, database services, etc.) isn't carefully passed to downstream consumers, memory will begin to fill with the intermediate state. (That is, what one gains in concurrency of operations one may pay for in memory.) And of course, node.js is on the server -- where the long-running nature of services means that the effect of a memory leak is much more likely to be felt and to affect service. Take all of these together, and you can easily see why virtually anyone who has stood up node.js in production will identify memory leaks as their most significant open issue.

The state of the art for node.js memory leak detection -- as concisely described by [Felix](https://twitter.com/#!/felixge) in his [node memory leak tutorial](https://github.com/felixge/node-memory-leak-tutorial) -- is to use the v8-profiler and [Danny Coates'](https://twitter.com/#!/blinkenbyte) [node-inspector](https://github.com/dannycoates/node-inspector). While this method is a lot better than nothing, it's very much oriented to the developer in development. This is a debilitating shortcoming because memory leaks are often observed only after days (hours, if you're unlucky) of running in production. At Joyent, we have long wanted to tackle the problem of providing the necessary tooling to identify node.js memory leaks in production. When we are so lucky as to have these in native code (that is, node.js add-ons), we can use [libumem and ::findleaks](http://dtrace.org/blogs/ahl/2004/07/13/number-11-of-20-libumem/) -- a technology that we have used to find thousands of production memory leaks over the years. But this is (unfortunately) not the common case: the common case is a leak in JavaScript -- be it node.js or application code -- for which our existing production techniques are useless.

So for node.js leaks in production, one has been left with essentially a Gedanken experiment: consider your load and peruse source code looking for a leak. Given how diabolical these leaks can be, it's amazing to me that anyone ever finds these leaks. (And it's less surprising that they take an excruciating amount of effort over a long period of time.) That's been the (sad) state of things for quite some time, but recently, this issue boiled over for us: [Voxer](http://voxer.com), a Joyent customer and [long in the vanguard of node.js development](http://dtrace.org/blogs/bmc/2010/08/11/the-node-js-demographic/), was running into a nasty memory leak: a leak sufficiently acute that the service in question was having to be restarted on a regular basis, but not so much that it was reproducible in development. With the urgency high on their side, we again kicked around this seemingly hopeless problem, focussing on the concrete data that we had: core dumps exhibiting the high memory usage, obtained at our request via gcore(1) before service restart. Could we do anything with those?

As an aside, a few months ago, Joyent's [Dave Pacheco](https://twitter.com/#!/dapsays) added some unbelievable [postmortem debugging support for node](http://dtrace.org/blogs/dap/2012/01/13/playing-with-nodev8-postmortem-debugging/). (If postmortem debugging is new to you, you might be interested in checking out [my presentation on debugging production systems](http://www.infoq.com/presentations/Debugging-Production-Systems) from QCon San Francisco. I had the privilege of demonstrating Dave's work in that presentation -- and if you listen very closely at 43:26, you can hear the audience gasp when I demonstrate the amazing ::jsprint.) But Dave hasn't built the infrastructure for walking the data structures of the V8 heap from an MDB dmod -- and it was clear that doing so would be brittle and error-prone.

As Dave, [Brendan](https://twitter.com/#!/brendangregg) and I were kicking around increasingly desperate ideas (including running strings(1) on the dump -- an idea not totally without merit -- and some wild visualization ideas), a much simpler idea collectively occurred to us: given that we understood via Dave's MDB V8 support how a given object is laid out, and given that an object needed quite a bit of self-consistency with referenced but otherwise orthogonal structures, what about just iterating over all of the anonymous memory in the core and looking for objects? That is, iterate over every single address, and see if that address could be interpreted as an object. On the one hand, it was terribly brute force -- but given the level of consistency required of an object in V8, it seemed that it wouldn't pick up too many false positives. The more we discussed it, the more plausible it became, but with Dave fully occupied (on another saucy project we have cooking at Joyent -- more on that later), I roped up and headed into his MDB support for V8...

The result -- [::findjsobjects](https://github.com/joyent/illumos-joyent/commit/e6446071ecdbdc60eda295956dbf3e1778663264) -- takes only a few minutes to run on large dumps, but provides some important new light on the problem. The output of ::findjsobjects consists of representative objects, the number of instances of that object and the number of properties on the object -- followed by the constructor and first few properties of the objects. For example, here is the dump on a gcore of a (non-pathological) Joyent node-based facility:

```
> ::findjsobjects -v
findjsobjects:         elapsed time (seconds) => 20
findjsobjects:                   heap objects => 6079488
findjsobjects:             JavaScript objects => 4097
findjsobjects:              processed objects => 1734
findjsobjects:                 unique objects => 161
OBJECT   #OBJECTS #PROPS CONSTRUCTOR: PROPS
fc4671fd        1      1 Object: flags
fe68f981        1      1 Object: showVersion
fe8f64d9        1      1 Object: EventEmitter
fe690429        1      1 Object: Credentials
fc465fa1        1      1 Object: lib
fc46300d        1      1 Object: free
fc4efbb9        1      1 Object: close
fc46c2f9        1      1 Object: push
fc46bb21        1      1 Object: uncaughtException
fe8ea871        1      1 Object: _idleTimeout
fe8f3ed1        1      1 Object: _makeLong
fc4e7c95        1      1 Object: types
fc46bae9        1      1 Object: allowHalfOpen
...
fc45e249       12      4 Object: type, min, max, default
fc4f2889       12      4 Object: index, fields, methods, name
fd2b8ded       13      4 Object: enumerable, writable, configurable, value
fc0f68a5       14      1 SlowBuffer: length
fe7bac79       18      3 Object: path, fn, keys
fc0e9d21       20      5 Object: _onTimeout, _idleTimeout, _idlePrev, ...
fc45facd       21      4 NativeModule: loaded, id, exports, filename
fc45f571       23      8 Module: paths, loaded, id, parent, exports, ...
fc4607f9       35      1 Object: constructor
fc0f86c9       56      3 Buffer: length, offset, parent
fc0fc92d       57      2 Arguments: length, callee
fe696f59       91      3 Object: index, fields, name
fc4f3785       91      4 Object: fields, name, methodIndex, classIndex
fe697289      246      2 Object: domain, name
fc0f87d9      308      1 Buffer:

```

Now, any one of those objects can be printed with ::jsprint. For example, let's take fc45e249 from the above output:

```
> fc45e249::jsprint
{
    type: number,
    min: 10,
    max: 1000,
    default: 300,
}

```

Note that that's only a _representative_ object -- there are (in the above case) 12 objects that have that same property signature. ::findjsobjects can get you all of them when you specify the address of the reference object:

```
> fc45e249::findjsobjects
fc45e249
fc46fd31
fc467ae5
fc45ecb5
fc45ec99
fc45ec11
fc45ebb5
fc45eb41
fc45eb25
fc45e3d1
fc45e3b5
fc45e399

```

And because MDB is the debugger Unix was meant to have, that output can be piped to ::jsprint:

```
> fc45e249::findjsobjects | ::jsprint
{
    type: number,
    min: 10,
    max: 1000,
    default: 300,
}
{
    type: number,
    min: 0,
    max: 5000,
    default: 5000,
}
{
    type: number,
    min: 0,
    max: 5000,
    default: undefined,
}
...

```

Okay, fine -- but where are these objects referenced? ::findjsobjects has an option for that:

```
> fc45e249::findjsobjects -r
fc45e249 referred to by fc45e1e9.height

```

This tells us (or tries to) who is referencing that first (representative) object. Printing that out (with the "-a" option to show the addresses of the objects):

```
> fc45e1e9::jsprint -a
fc45e1e9: {
    ymax: fe78e061: undefined,
    hues: fe78e061: undefined,
    height: fc45e249: {
        type: fe78e361: number,
        min: 14: 10,
        max: 7d0: 1000,
        default: 258: 300,
    },
    selected: fc45e3fd: {
        type: fe7a2465: array,
        default: fc45e439: [...],
    },
    ...

```

So if we want to learn where all of these objects are referenced, we can again use a pipeline within MDB:

```
> fc45e249::findjsobjects | ::findjsobjects -r
fc45e249 referred to by fc45e1e9.height
fc46fd31 referred to by fc46b159.timeout
fc467ae5 is not referred to by a known object.
fc45ecb5 referred to by fc45eadd.ymax
fc45ec99 is not referred to by a known object.
fc45ec11 referred to by fc45eadd.nbuckets
fc45ebb5 referred to by fc45eadd.height
fc45eb41 referred to by fc45eadd.ymin
fc45eb25 referred to by fc45eadd.width
fc45e3d1 referred to by fc45e1e9.nbuckets
fc45e3b5 referred to by fc45e1e9.ymin
fc45e399 referred to by fc45e1e9.width

```

Of course, the proof of a debugger is in the debugging; would ::findjsobjects actually be of use on the Voxer dumps that served as its motivation? Here is the (elided) output from running it on a big Voxer dump:

```
> ::findjsobjects -v
findjsobjects:         elapsed time (seconds) => 292
findjsobjects:                   heap objects => 8624128
findjsobjects:             JavaScript objects => 112501
findjsobjects:              processed objects => 100424
findjsobjects:                 unique objects => 241
OBJECT   #OBJECTS #PROPS CONSTRUCTOR: PROPS
fe806139        1      1 Object: Queue
fc424131        1      1 Object: Credentials
fc424091        1      1 Object: version
fc4e3281        1      1 Object: message
fc404f6d        1      1 Object: uncaughtException
...
fafcb229     1007     23 ClientRequest: outputEncodings, _headerSent, ...
fafc5e75     1034      5 Timing: req_start, res_end, res_bytes, req_end, ...
fafcbecd     1037      3 Object: aborted, data, end
 8045475     1060      1 Object:
fb0cee9d     1220      9 HTTPParser: socket, incoming, onHeadersComplete, ...
fafc58d5     1271     25 Socket: _connectQueue, bytesRead, _httpMessage, ...
fafc4335     1311     16 ServerResponse: outputEncodings, statusCode, ...
fafc4245     1673      1 Object: slab
fafc44d5     1702      5 Object: search, query, path, href, pathname
fafc440d     1784     14 Client: buffered_writes, name, timing, ...
fafc41c5     1796      3 Object: error, timeout, close
fafc4469     1811      3 Object: address, family, port
fafc42a1     2197      2 Object: connection, host
fbe10625     2389      2 Arguments: callee, length
fafc4251     2759     15 IncomingMessage: statusCode, httpVersionMajor, ...
fafc42ad     3652      0 Object:
fafc6785    11746      1 Object: oncomplete
fb7abc29    15155      1 Object: buffer
fb7a6081    15155      3 Object: , oncomplete, cb
fb121439    15378      3 Buffer: offset, parent, length

```

This immediately confirmed a hunch that Matt had had that this was a buffer leak. And for [Isaac](https://twitter.com/#!/izs) -- who had been working this issue from the Gedanken side and was already zeroing in on certain subsystems -- this data was surprising in as much as it was so confirming: he was already on the right path. In short order, he [nailed it](https://github.com/joyent/node/commit/d1effbb338fe5916e5a25fc3d3587016d0725761), and the fix is in [node 0.6.17](http://blog.nodejs.org/2012/05/04/version-0-6-17-stable/).

The fix was low risk, so Voxer redeployed with it immediately -- and for the first time in quite some time, memory utilization was flat. This was a huge win -- and was the reason for Matt's [tantalizing tweet](https://twitter.com/#!/mranney/status/198125651495096321). The advantages of this approach are that it requires absolutely no modification to one's node.js programs -- no special flags and no different options. And it operates purely postmortem. Thanks to help from gcore(1), core dumps can be taken over time for a single process, and those dumps can then be analyzed off-line.

Even with ::findjsobjects, debugging node.js memory leaks is still tough. And there are certainly lots of improvements to be made here -- there are currently some objects that we do not know how to correctly interpret, and we know that we know that we can improve our algorithm for finding object references -- but this shines a bright new light into what had previously been a black hole!

If you want to play around with this, you'll need [SmartOS](http://smartos.org) or [your favorite illumos distro](http://wiki.illumos.org/display/illumos/Distributions) (which, it must be said, you can get simply by provisioning on [the Joyent cloud](http://www.joyentcloud.com/)). You'll need an updated v8.so -- which you can either build yourself from [illumos-joyent](https://github.com/joyent/illumos-joyent) or you can [download a binary](http://dtrace.org/blogs/bmc/files/2012/05/v8.so_.tar_.gz). From there, follow [Dave's instructions](http://dtrace.org/blogs/dap/2012/01/13/playing-with-nodev8-postmortem-debugging/). Don't hesitate to ping [me](https://twitter.com/#!/bcantrill) if you get stuck or have requests for enhancement -- and here's to flat memory usage on your production node.js service!
