---
title: "The DIRT on JSConf.eu and Surge"
date: "2010-10-02"
---

I just got back from an exciting (if exhausting) conference double-header: [JSConf.eu](http://jsconf.eu/2010/) and [Surge](http://omniti.com/surge/2010). These conferences made for an interesting pairing, together encompassing much of the enthusiasm and challenges in today's systems. It started in Berlin with JSConf.eu, a conference that reflects both JavaScript's energy and diversity. [Ryan](http://twitter.com/ryah) introduced Node.js to the world at JSConf.eu a year ago, and enthusiasm for Node at the conference was running high; I would estimate that at least half of the attendees were implementing server-side JavaScript in one fashion or another. Presentations from the server-side were epitomized by [Felix](http://twitter.com/felixge) and [Philip](http://www.gnegg.ch/), each of whom presented real things done with Node -- expanding on the successes, limitations and futures of what they had developed. The surge of interest in the server-side is of course changing the complexion of not only JSconf but of JavaScript itself. a theme that [Chris Williams](http://twitter.com/voodootikigod) hit with fervor in his closing plenary, in which he cautioned of the dangers for a technology and a community upon whom a wave of popularity is breaking. Having been been near the epicenter of Java's irrational exuberance in the mid-1990s ([UltraJava](http://bwrc.eecs.berkeley.edu/CIC/announce/1996/java-procs.html), anyone?), I certainly share some of Chris's concerns -- and I think that Ryan's [talk on the challenges in Node](http://nodejs.org/jsconf-eu-2010.pdf) reflects a humility that will become essential in the coming year. That said, I am also looking forward to seeing the JavaScript and Node communities expand to include more of the [systems vets that have differentiated Node to date](http://dtrace.org/blogs/bmc/2010/08/11/the-node-js-demographic/).

Speaking of systems vets, a highlight of JSConf.eu was hanging out with [Erik Corry](http://twitter.com/erikcorry) from the V8 team. JavaScript and Node owe a tremendous debt of gratitude to V8, without whom we would have neither the high-performance VM itself, nor the broader [attention to JavaScript performance](http://arewefastyet.com/) so essential to the success of JavaScript on the server-side. Erik and I brainstormed ideas for deeper support of DTrace within V8, which is, of course, an excruciatingly hard problem: as I have observed in the past, magic does not layer well -- and DTrace and a sophisticated VM like V8 are both conjuring deep magic that frequently operate at cross purposes. Still, it was an exciting series of conversations, and we at Joyent certainly look forward to taking a swing at this problem.

After JSConf.eu wrapped up, I spent a few days brainstorming and exploring Berlin (and its Trabants and Veyrons) with [Mark](http://twitter.com/mmayo), [Pedro](http://twitter.com/kusor) and [Filip](http://twitter.com/mamash). I then headed to the inaugural [Surge Conference](http://omniti.com/surge/2010) in Baltimore, and given my experience there, I would put the call out to the systems tribe: Surge promises to be the premier conference for systems practitioners. We systems practitioners have suffered acutely at the hands of academic computer science's foolish obsession with conferences as publishing vehicle, having seen conference after conference of ours becoming casualties on their bloody path to tenure. Surge, by contrast, was everything a systems conference should be: exciting talks on real systems by those who designed, built and debugged them -- and a terrific hallway track besides.

To wit: I was joined at Surge by Joyent VP of Operations Ryan Nelson; for us, Surge more than paid for itself during [Rod Cope](http://twitter.com/RodCope)'s presentation on his ten lessons for Hadoop deployment. Rod's lessons were of the ilk that only come the hard way; his was a presentation fired in a kiln of unspeakable pain. For example, he noted that his cluster had been plagued by a wide variety of seemingly random low-level problems: lost interrupts, networking errors, I/O errors, etc. Acting apparently operating on a tip in a Broadcom forum, Rod disabled C-states on all of his Dell R710s and 2970s -- and the problems all lifted (and have been absent for four months). Given that we have recently lived through same (and came to the same conclusion, albeit more recently and therefore much more tentatively), I [tweeted](http://twitter.com/bcantrill/status/26090857201) what Rod had just reported, knowing that Ryan was in [John Allspaw](http://twitter.com/allspaw)'s concurrent sesssion and would likely see the tweet. Sure enough, I saw him retweet it minutes later -- and he practically ran in after the end of Rod's talk to see if I was kidding. (When it comes to firmare- and BIOS-level issues, I assure you: I never kid.) Not only was Rod's confirmation of our experience tremendously valuable, it goes to the kind of technologist at Surge: this is a conference with practitioners who are comfortable not only at lofty architectural heights, but also at the grittiest of systems' depths.

Beyond the many thought-provoking sessions, I had great conversations with many people, including [Paul](http://twitter.com/pquerna), [Rod](http://twitter.com/RodCope), [Justin](http://twitter.com/justinsheehy), [Chris](http://twitter.com/skeptomai), [Geir](http://twitter.com/geirmagnusson), [Stephen](http://twitter.com/sogrady), [Wez](http://twitter.com/wezfurlong), [Mark](http://twitter.com/markimbriaco), [Joe](http://twitter.com/williamsjoe), [Kate](http://twitter.com/katemats), [Tom](http://twitter.com/tomdyninc), [Theo](http://twitter.com/postwait) (natch) -- and a reunion (if sodden) with surprise stowaway [Ted Leung](http://twitter.com/twleung).

As for my formal role at Surge, I had the honor of giving one of the opening keynotes. In preparing for it, I tried to put it all together: what larger trends are responsible for the tremendous interest in Node? What are the implications of these trends for other elements of the systems we build? From [Node Knockout](http://nodeknockout.com/), it is clear that a major use case for Node is web-based real-time applications. At the moment these are primarily games -- but as practitioners like [Matt Ranney](http://twitter.com/maranney) can attest, games are by no means the only real-time domain in which Node is finding traction. In part based on my experience developing the backend for the [Node knockout leaderboard](http://dtrace.org/blogs/bmc/2010/08/30/dtrace-node-js-and-the-robinson-projection/), the argument that I made in the keynote is that these web-based real-time applications have a kind of orthogonal data consistency: instead of being on the [ACID](http://en.wikipedia.org/wiki/ACID)/[BASE](http://en.wikipedia.org/wiki/Eventual_consistency) axis, the "consistency" of the data is its temporality. (That is, if the data is late, it's effectively inconsistent.) This is not necessarily new (market-facing trading systems have long had these properties), but I believe we're about to see these systems brought to a much larger audience -- and developed by a broader swath of practitioners. And of course -- like AJAX before it -- this trend was begging for an acronym. Struggling to come up with something, I went back to what I was trying to describe: these are real-time systems -- but they are notable because they are data-intensive. A ha: "data-intensive real-time" -- DIRT! I dig DIRT! I don't know if the acronym will stick or not, but I did note that it was mentioned in each of [Theo](http://twitter.com/postwait)'s and [Justin](http://twitter.com/justinsheehy)'s talks -- and this was just minutes after it was coined. (Video of this keynote will be available soon; slides are [here](http://dtrace.org/resources/bmc/DIRT.pdf).)

Beyond the keynote, I also gave a talk on the perils of building enterprise systems from commodity components. This blog entry has gone on long enough, so I won't go into depth on this topic here, saying only this: [fear the octopus](http://www.youtube.com/watch?v=p9A-oxUMAy8)!

Thanks to the organizers of both JSconf and Surge for two great conferences. I arrive home thoroughly exhausted (JSconf parties longer, but Surge parties harder; pick your poison), but also exhilarated and excited about the future. Thanks again -- and see you next year!