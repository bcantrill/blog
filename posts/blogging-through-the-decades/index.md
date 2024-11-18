---
title: "Blogging through the decades"
date: "2024-11-16"
---

Over the summer, I hit an anniversary of sorts:  I've been blogging for
two decades (!).
For reasons I'll get to, I've been reflecting back on
my history of writing in general and blogging in particular.

In 2004, blogging wasn't exactly new (it had been around in one form or
another for as long as a decade), but it also wasn't something that I
was
actively engaged in.  And while there was some blogging infrastructure
at Sun, it was desultory and oriented around Java. Similarly,
while there were also vectors for engineers to directly engage with
customers, they were limited to proprietary forums, with very specific
topics (e.g., support issues) and generally inaccessible and undiscoverable.
All of that changed in the spring, when 
Sun rolled out 
[a new policy on discourse](https://www.tbray.org/ongoing/When/200x/2004/05/02/Policy) and then -- a few months later --
[blogs.sun.com](https://www.tbray.org/ongoing/When/200x/2004/06/06/BSC).
As [Tim Bray retold](https://www.tbray.org/ongoing/When/200x/2004/05/02/PolicyMaking),
the disposition with respect to blogging was
purposeful:  employees weren't merely *allowed* to blog,
they were *actively encouraged* to do so -- and provided all of the infrastructure
to make it easy.  The message was clear, and it was explicit: "We trust you."

That trust -- Sun's finest quality! -- was not new for the company; I came to work at Sun *because* I had seen an
engineer (Jeff Bonwick)
talk about their work (on
[comp.unix.solaris](https://groups.google.com/g/comp.unix.solaris/c/e2C6JIxPHfA/m/rq7XBHY4P74J)!), and I needed no convincing on the merits of technologist
transparency.
But wasn't blogging just a little too... trendy?
I quickly got over myself:
we had a lot to talk about -- DTrace had integrated in the fall of 2003 --
and blogging felt like it was
worth a shot. On June 14, 2004,
[I took the plunge]({{< relref "posts/some-prehistory" >}}).
As it would be 
[with Twitter Spaces many years later]({{< relref "posts/twitter-spaces-a-few-weeks-in" >}}),
it was immediately obvious that any reticence I might have had was misplaced --
that blogging was larger than the sum of its parts.
As I would describe to folks internally, the beauty of blogging is that
you needn't be regular and you also needn't be on topic: you can write
*only* when you have something to say -- and you can say whatever you want.

Blogging at Sun was emblematic of a larger trend:
the company that had invented open systems was in the process of 
reimagining itself in a new era of transparency and collaboration,
as exemplified with the announced
intention of open sourcing Solaris.
While it had been announced earlier in 2004, open sourcing a complicated
proprietary system like Solaris is
[deceptively tricky](https://www.youtube.com/watch?v=Zpnncakrelk#t=11m12s),
and it was taking us a while to get all of the 
details right.
Blogging was a perfect complement to our open sourcing activity;
I blogged when
[DTrace went first in the chute]({{< relref "posts/solaris-10-revealed" >}}) --
and when we ultimately 
open sourced the rest of the operating system later in 2005,
we launched it by 
[encouraging engineers to blog about it]({{< relref "posts/sifting-through-the-blogs" >}}).
When ZFS integrated into the system later that year, we marked the
occasion with
[another flurry of blog entries]({{< relref "posts/welcome-to-zfs" >}}). Thanks
to blogging, we created an extraordinary amount of dense, technical content -- and
all in the voice of the technologists themselves.

The years went by,
and as
[Tim Bray reflected back in 2007](https://www.tbray.org/ongoing/When/200x/2007/04/27/BSC),
blogging at Sun was essentially without downside.
While I absolutely agree, it's hard for me not to be a little bittersweet about
that fifteen years later:  there was *so* much great content that 
the only downside -- such as it was -- was that it was shackled to a
listing corporate vessel.
And when that vessel sank, much of the content went to a watery grave
(or would have, had it not been for the heroics of [the Internet Archive](https://web.archive.org/)!).

Speaking for myself,
when it came time to say
[good-bye to Sun]({{< relref "posts/good-bye-sun" >}}), I didn't want
to leave my writing behind to its own fate. Fortunately, Sun had had the foresight to make
clear to us that we ourselves owned our own writing,
and I was able to easily export my data out of Sun to re-host
it elsewhere.
Even though I was 
[going to work for a cloud provider]({{< relref "posts/hello-joyent" >}})
I had not wanted to co-mingle my future job with my present one,
so I did as one did:  I created a WordPress blog on a
third-party site.
This WordPress site had the curse of being *just* good enough, and even after being
at Joyent, moving it was never quite seemed worth it.

I continued to blog, though now with a cadence that was reduced due to
the presence of microblogging (viz.
our first two kids -- 
born in 
[2004]({{< relref "posts/tobin-cormac-gaffikin-cantrill" >}})
and
[2007]({{< relref "posts/alexander-morgan-gaffikin-cantrill" >}})
-- were announced with blog posts;
our third -- born in 
[2012](https://x.com/bcantrill/status/200764623824240640) --
with a tweet).
Having a new vector for hot takes meant that blogging could be reserved
for long-form writing (or, occasionally,
[long-form hot takes]({{< relref "posts/unikernels-are-unfit-for-production" >}}));
when we released something new at Joyent -- like
[KVM]({{< relref "posts/kvm-on-illumos" >}}),
[Manta]({{< relref "posts/manta-from-revelation-to-product" >}}), or
[Triton]({{< relref "posts/triton-docker-and-the-best-of-all-worlds" >}}) --
I used blogging to tell some of the human side of the story.

Years passed. 
[Joyent was bought by Samsung]({{< relref "posts/samsung-acquires-joyent" >}}).
[I fell in love with Rust]({{< relref "posts/falling-in-love-with-rust" >}}).
[I left Joyent]({{< relref "posts/ex-joyeur" >}}) and
[started Oxide]({{< relref "posts/the-soul-of-a-new-computer-company" >}}).
At Oxide, long-form writing remained important as ever, and I used 
blogging to not just talk about our technologies
(like [Hubris and Humility]({{< relref "posts/hubris-and-humility" >}}))
but also to talk about why we were building the company the way
were building it,
from specifics like
[our approach to compensation]({{< relref "posts/compensation-as-a-reflection-of-values" >}}) to our more general thoughts on
[engineering a culture]({{< relref "posts/engineering-a-culture" >}}).

While blogging continued to be important, 
that hosted WordPress lifeboat that I scrambled aboard in 2010 was becoming
*really* problematic; it would constantly spring new leaks and was rife with security
vulnerabilities -- and holy moly was it slow!  I knew I needed to move my blog
to a better spot, but could never *quite* prioritize it...

Into this stasis, I was approached by [Piotr Sarna](https://bio.sarna.dev/) and
[Cynthia Dunlop](https://www.linkedin.com/in/cynthiadunlop/), who were working on a
new book, 
[Writing for Developers](https://www.manning.com/books/writing-for-developers). They had included references to two of my
pieces in the book 
([Rust after the honeymoon]({{< relref "posts/rust-after-the-honeymoon" >}}) and
[The relative performance of C and Rust]({{< relref "posts/the-relative-performance-of-c-and-rust" >}})),
and they wanted me to write the foreword.  Excited that they were taking
on this topic and honored that they would ask me to write it, I agreed.
In preparation for the launch of the book, Cynthia sent me some thoughtful
questions about my own blogging, wondering if I might be willing to blog
my answers.
Anyone who has seen the [Oxide hiring process](https://rfd.shared.oxide.computer/rfd/0003) knows that I
love reflective questions, and Cynthia asked me some great ones:
What blog entry was I most proud of?  Which one was most difficult to write?
What impact of blogging have I found surprising? What advice would I have
for those getting started with blogging?
Of course, answering these questions required me to go back and read 
my older blog entries -- and it is with this that I (finally) hit my breaking
point with WordPress:  if reading a single blog entry was painful, trying to
navigate through ~150 of them over twenty years was untenable.
The lifeboat had served its duty, but it was now time to beach it and 
turn it into kindling: to answer Cynthia's questions, I had to first find
the blog a new home.

A *lot* had changed since 2004, and even since my WordPress lifeboat in 2010.
For my blog, I wanted to get away from systems that feel like unnecessarily complicated
content management systems. Fortunately,
[static site generators](https://en.wikipedia.org/wiki/Static_site_generator)
represent the approach that I wanted
from a blog: blazingly fast, clean, readable on mobile,
and (most importantly for someone who was fleeing captive content
management for the second time), entirely git-backed.
There were some good options to choose from, but 
I landed on 
[Hugo](https://gohugo.io/), using the 
[hugo-blog-awesome theme](https://github.com/hugo-sid/hugo-blog-awesome) --
and Will Boyd's excellent [wordpress-export-to-markdown](https://github.com/lonekorean/wordpress-export-to-markdown) for facilitating the migration.

In addition to the rise of static site generators,
there of course had been another big change:
[we shipped the Oxide cloud computer]({{< relref "posts/cloud-computer" >}})!
To allow customers to try out the Oxide rack without needing to first
buy one, we stood up an Oxide rack in a datacenter environment,
spinning up a 
[silo](https://docs.oxide.computer/guides/operator/silo-management) for
folks who want to kick the tires.
And increasingly, we use this production rack for our own infrastructure at Oxide.
All of this gave me an opportunity for a special homecoming for my blog:
it could run on the system that we have [built from scratch](https://oxide-and-friends.transistor.fm/episodes/tales-from-the-bringup-lab-2021-12-06)!
A blog may be the simplest possible use case for the Oxide rack,
but it was still really fun to *use* the product -- and (huge
credit to the folks building
[the frontend of the computer](https://oxide-and-friends.transistor.fm/episodes/the-frontend-of-the-computer)) *so* damned easy.  I especially loved how easy it
was to do [VPC management](https://docs.oxide.computer/guides/configuring-guest-networking), which quickly becomes complicated.

All of this to say: the text you are reading now, dear reader, is being
served from a web server running on an instance in a production Oxide rack!

Having now decades of blogging readily accessible, it would be easy
to take on Cynthia's questions, right?  Well, not so fast.
It's indeed easy to [read all of my writing](/), but the more I read, the more
ambiguity there was.
My decades of blogging were teeming with life:
birth ([Fishworks]({{< relref "posts/fishworks-now-it-can-be-told" >}}),
[Manta]({{< relref "posts/manta-from-revelation-to-product" >}}),
[Oxide]({{< relref "posts/the-soul-of-a-new-computer-company" >}}),
[illumos]({{< relref "posts/opensolaris-and-the-power-to-fork" >}})),
love ([JavaScript]({{< relref "posts/on-modalities-and-misadventures" >}}),
[systems]({{< relref "posts/reflections-on-systems-we-love" >}}),
[Rust]({{< relref "posts/falling-in-love-with-rust" >}}),
[*The Soul of a New Machine*]({{< relref "posts/reflecting-on-the-soul-of-a-new-machine" >}}))
and death ([Sun]({{< relref "posts/good-bye-sun" >}}),
[Solaris]({{< relref "posts/the-sudden-death-and-eternal-life-of-solaris" >}})).
And the death, sadly, was not merely metaphorical;
over the years, I memorialized the passing of three people who still had a lot
of life in front of them:
[John Birrell]({{< relref "posts/john-birrell" >}}),
[Khaled Bichara]({{< relref "posts/rip-khaled-bichara" >}}),
and -- tragically recently --
[Charles Beeler]({{< relref "posts/remembering-charles-beeler" >}}).

I have written some [very dense technical 
pieces]({{< relref "posts/opensolaris-sewer-tour" >}}) and
taken on some [scalding hot topics]({{< relref "posts/the-power-of-a-pronoun" >}});
I have walked through
[history's graveyard]({{< relref "posts/revisiting-the-intel-432" >}}),
[pondered the quality of software]({{< relref "posts/software-immaculate-fetid-and-grimy" >}}), and
[reflected on its economics]({{< relref "posts/the-economics-of-software" >}}).
The throughline of two decades of writing,
especially for the pieces that have enduring resonance:
they reflect the humanity in our endeavor.
There is an old adage to "speak from the heart, not from the book"; looking
back, I am a bit surprised at
the degree that this was true in my own writing.
And while I don't have neat answers to Cynthia's questions, it *does* leave
me with easy advice for would-be bloggers:  write from the heart, even if you
think no one is reading it; if nothing else, your future self will thank you
for it!

