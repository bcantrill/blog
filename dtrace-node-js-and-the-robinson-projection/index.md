---
title: "DTrace, node.js and the Robinson Projection"
date: "2010-08-30"
---

When I joined Joyent, [I mentioned](http://dtrace.org/blogs/bmc/2010/07/30/hello-joyent/) that I was seeking to apply DTrace to the cloud, and that I was particularly excited about the development of node.js -- leaving it implict that the intersection of the two technologies would be naturally interesting, As it turns out, we have had an early opportunity to show the potential here: as you might have seen, the [Node Knockout](http://nodeknockout.com/) programming contest was held over the weekend; when I first joined Joyent (but four weeks ago!), [Ryan](http://twitter.com/ryah) was very interested in potentially using DTrace to provide a leaderboard for the competition. I got to work, adding [USDT probes to node.js](http://github.com/bcantrill/node-dtrace). To be fair, this still has some disabled overhead (namely, getting into and out of the node addon that has the true USDT probe), but it's sufficiently modest to deploy DTrace-enabled node's in production.

And thanks to incredibly strong work by Joyent engineers, we were able to make available a new [node.js service](http://nodeknockout.posterous.com/countdown-to-knockout-post-11-deploying-to-jo) that allocated a container per user. This service allowed us to make available a DTrace-enabled node to contestants -- and then observe all of that from the global zone.

For example of the DTrace provider for node.js, here's a simple enabling to print out HTTP requests as zones handle them (running on one of the Node Knockout machines):

```
# dtrace -n 'node*:::http-server-request{printf("%s: %s of %s\n", \
    zonename, args[0]->method, args[0]->url)}' -q
nodelay: GET of /poll6759.479651377309
nodelay: GET of /poll6148.392275444794
nodebodies: GET of /latest/
nodebodies: GET of /latest/
nodebodies: GET of /count/
nodebodies: GET of /count/
nodelay: GET of /poll8973.863890386003
nodelay: GET of /poll2097.9667574643568
awesometown: GET of /graphs/4c7a650eba12e9c41d000005.js
awesometown: POST of /graphs/4c7a650eba12e9c41d000005/appendValue
awesometown: GET of /graphs/4c7acd5ca121636840000002.js
awesometown: GET of /graphs/4c7a650eba12e9c41d000005.js
awesometown: GET of /graphs/4c7a650eba12e9c41d000005.js
awesometown: GET of /graphs/4c7a650eba12e9c41d000005.js
awesometown: GET of /graphs/4c7b2408546a64b81f000001.js
awesometown: POST of /faye
awesometown: POST of /faye
...

```

I added probes around both HTTP request and HTTP response; treating the file descriptor as a token that describes that uniquely describes that request while it is pending (an assumption that would only be invalid in the presence of [HTTP pipelining](http://en.wikipedia.org/wiki/HTTP_pipelining)), allows one to actually determine the latency for requests:

```
# cat http.d
#pragma D option quiet

http-server-request
{
        ts[this->fd = args[1]->fd] = timestamp;
        vts[this->fd] = vtimestamp;
}

http-server-response
/this->ts = ts[this->fd = args[0]->fd]/
{
        @t[zonename] = quantize(timestamp - this->ts);
        @v[zonename] = quantize(vtimestamp - vts[this->fd]);
        ts[this->fd] = 0;
        vts[this->fd] = 0;
}

tick-1sec
{
        printf("Wall time:\n");
        printa(@t);

        printf("CPU time:\n");
        printa(@v);
}

```

This script makes the distinction between wall time and CPU time; for wall-time, you can see the effect of long-polling, e.g. (the values are nanoseconds):

```
    nodelay
           value  ------------- Distribution ------------- count
           32768 |                                         0
           65536 |                                         4
          131072 |@@@@@                                    52
          262144 |@@@@@@@@@@@@@@@@@@                       183
          524288 |@@@@@                                    55
         1048576 |@@@                                      27
         2097152 |@                                        9
         4194304 |                                         5
         8388608 |@                                        8
        16777216 |@                                        6
        33554432 |@                                        9
        67108864 |@                                        7
       134217728 |@                                        12
       268435456 |@                                        11
       536870912 |                                         1
      1073741824 |                                         4
      2147483648 |                                         1
      4294967296 |                                         5
      8589934592 |                                         0
     17179869184 |                                         1
     34359738368 |                                         1
     68719476736 |                                         0

```

You can also look at the CPU time to see those that are doing more actual work. For example, one zone with interesting CPU time outliiers:

```
  nodebodies
           value  ------------- Distribution ------------- count
         4194304 |                                         0
         8388608 |@@@@@@@@@@@@                             57
        16777216 |@@@@                                     21
        33554432 |@@@@                                     18
        67108864 |@@@@@@@                                  34
       134217728 |@@@@@@@@@@@                              54
       268435456 |                                         0
       536870912 |                                         0
      1073741824 |                                         0
      2147483648 |                                         0
      4294967296 |@                                        3
      8589934592 |@                                        4
     17179869184 |                                         0

```

Note that because node has a single thread do all processing, we cannot assume that the requests themselves are inducing the work -- only that CPU work was done between request and response. Still, this data would probably be interesting to the [nodebodies](http://nodebodies.no.de) team...

I also added probes around connection establishment; so here's a simple way of looking at new connections by zone:

```
# dtrace -n 'node*:::net-server-connection{@[zonename] = count()}'
dtrace: description 'node*:::net-server-connection' matched 44 probes
^C

  explorer-sox                                                      1
  nodebodies                                                        1
  anansi                                                           69
  nodelay                                                         102
  awesometown                                                     146

```

Or if we wanted to see which IP addresses were connecting to, say, our good friends at [awesometown](http://awesometown.no.de) (with actual addresses in the output elided):

```
# dtrace -n 'node*:::net-server-connection \
    /zonename == "awesometown"/{@[args[0]->remoteAddress] = count()}'
dtrace: description 'node*:::net-server-connection' matched 44 probes
  XXX.XXX.XXX.XXX                                                   1
  XX.XXX.XX.XXX                                                     1
  XX.XXX.XXX.XXX                                                    1
  XX.XXX.XXX.XX                                                     1
  XXX.XXX.XX.XXX                                                    1
  XXX.XXX.XX.XX                                                     2
  XXX.XXX.XXX.XX                                                    8

```

Ryan saw the DTrace support I had added, and had a great idea: what if we took the IPs of incoming connections and geolocated them, throwing them on a world map and coloring them by team name? This was an idea that was just too exciting not to take a swing at, so we got to work. For the backend, the machinery was begging to itself be written in node, so I did a [libdtrace addon](http://github.com/bcantrill/node-libdtrace) for node and started building a scalable backend for processing the DTrace data from the different Node Knockout machines. Meanwhile, [Joni](http://twitter.com/jahoni) came up with some mockups that had everyone drooling, and [Mark](http://twitter.com/mmayo) contacted [Brian](http://twitter.com/brianleroux) from [Nitobi](http://nitobi.com/) about working on the front-end. Brian and crew were as excited about it as we were, and they put front-end engineer extraordinaire [Yohei](http://twitter.com/yoheis) on the case -- who worked with [Rob](http://twitter.com/rob_ellis) on the Joyent side to pull it all together. Among Rob's other feats, he managed to implement in JavaScript the logic for plotting longitude and latitude in the beautiful [Robinson projection](http://en.wikipedia.org/wiki/Robinson_projection) -- which is a brutally complicated transformation. It was an incredible team, and we were pulling it off in such a short period of time and with such a firm deadline that we often felt like contestants ourselves!

The result -- which it must be said works best in Safari and Chrome -- is at [http://leaderboard.no.de](http://leaderboard.no.de). In keeping with both the spirit of node and DTrace, the leaderboard is updated in real-time; from the time you connect to one of the Joyent-hostest (no.de) contestants, you should see yourself show up in the map in no more than 700 milliseconds (plus your network latwork latency). For crowded areas like the Bay Area, it can be hard to see yourself -- but try moving to Cameroon for best results. It's fun to watch as certain contestants go viral (try both hovering over a particular data point and clicking on the team name in the leaderboard) -- and you can know which continent you're cursing at in [http://saber-tooth-moose-lion.no.de](http://saber-tooth-moose-lion.no.de) (now known to the world as [Swarmation](http://swarmation.com/)).

Enjoy both the leaderboard and the terrific Node Knockout entries (be sure to vote for your favorites!) -- and know that we've only scratched the surface of what DTrace and node.js can do together!
