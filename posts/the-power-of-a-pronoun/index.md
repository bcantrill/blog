---
title: "The Power of a Pronoun"
date: "2013-11-30"
---

Long ago in my technical writing, I sought to eliminate gendered pronouns when
referring to unnamed people or entities. It's not that I'm fooling myself into
believing that eliminating gendered pronouns somehow gets more women into
computer science, but rather that I think that using male pronouns sounds like
a holdover from the era of the
[kitchencomputer](http://en.wikipedia.org/wiki/Honeywell_316#Kitchen_Computer).
As for the mechanics of doing this, I personally dislike randomly substituting
"she" for "he"; my preference is to use the singular they/them, recasting the
sentence as necessary when that sounds too awkward.

I say all of this because of a recent issue in [node.js](http://nodejs.org/),
aJoyent-sponsored project. One of the node.js core contributors,
[Ben Noordhuis](https://github.com/bnoordhuis), [rejected a
pull request](https://github.com/joyent/libuv/pull/1015#issuecomment-29538615)
that eliminated the use of a gendered pronoun
in [libuv](https://github.com/joyent/libuv). Now, this was quickly reversed by
node.js project lead [Isaac Schlueter](https://github.com/isaacs) (that is,
Isaac accepted the patch eliminating the gendered pronoun), but because this
is a Joyent-sponsored project, many made the reasonable inference that Ben is
a Joyent employee -- and have called Joyent to task for tolerating such poor
behavior. (Especially when that poor behavior transcended into the
gobsmackingly inappropriate as [Ben tried to revert
Isaac'scommit](https://github.com/joyent/libuv/commit/804d40ee14dc0f82c482dcc8d1c41c14333fcb48).)

But while Isaac is a Joyent employee, *Ben is not* -- and if he had been, he
wouldn't be as of this morning: to reject a pull request that eliminates a
gendered pronoun on the *principle* that pronouns should in fact be gendered
would constitute a fireable offense for me and for Joyent. On the one hand, it
seems ridiculous (absurd, perhaps) to fire someone over a pronoun -- but to
characterize it that way would be a gross oversimplification: it's not the use
of the gendered pronoun that's at issue (that's just sloppy), but rather the
*insistence* that pronouns should in fact be gendered. To me, that insistence
can only come from one place: that gender -- specifically, masculinity -- is
inextricably linked to software, and that's not an attitude that Joyent
tolerates. This isn't merely a legalistic concern (though that too, certainly),
but also a technical one: we believe that [empathy is a core
engineering
value](http://www.listbox.com/member/archive/182179/2013/05/sort/time_rev/page/1/entry/1:75/20130508172342:8CF96552-B825-11E2-98EE-8CBB5940B2DC/)
-- and
that an engineer that has so little empathy as to not understand why the use
of gendered pronouns is a concern almost certainly makes poor technical
decisions as well.

While we would fire Ben over this, node.js is an open source project and one
doesn't necessarily have the same levers. Indeed, one of the challenges of an
open source project that depends on volunteer effort is [dealing with
assholes](http://www.slideshare.net/dberkholz/assholes-are-killing-your-project),
and fortunately in this regard, node.js is in Isaac's very capable hands. Isaac is
one of the most inclusive, empathetic engineers that I have ever had the
privilege to work with, and I know that he will deal with Ben's unacceptable
behavior accordingly and in the best interests of node.js. But just so you
heard it from us: if this were the act of a Joyent employee, we would -- to
deliberately use a gender-neutral pronoun -- fire them.

