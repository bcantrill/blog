---
title: "On the beauty in Beautiful Code"
date: "2007-07-28"
categories: 
  - "solaris"
---

I finished [_Beautiful Code_](http://www.oreilly.com/catalog/9780596510046/) this week, and have been reflecting on the book and its development. In particular, I have thought back to some of the authors' discussion, in which some advocated a different title. Many of us were very strongly in favor of the working title of "Beautiful Code", and I weighed in with my specific views on the matter:

```
Date: Tue, 19 Sep 2006 16:18:22 -0700
From: Bryan Cantrill
To: [Beautiful Code Co-Authors]
Subject: Re: [Beautifulcode] reminder: outlines due by end of September
Probably pointless to pile on here, but I'm for "Beautiful Code", if
only because it appropriately expresses the nature of our craft.  We
suffer -- tremendously -- from a bias from traditional engineering that
writing code is like digging a ditch:  that it is a mundane activity best
left to day labor -- and certainly beneath the Gentleman Engineer.  This
belief is profoundly wrong because software is not like a dam or a
superhighway or a power plant:  in software, the blueprints _are_ the
thing; the abstraction _is_ the machine.

It is long past time that our discipline stood on its feet, stepped out
from the shadow of sooty 19th century disciplines, and embraced our
unique confluence between mathematics and engineering.  In short,
"Beautiful Code" is long, long overdue.
```

Now, I don't disagree with my reasoning from last September (though I think that the Norma Rae-esque tone was probably taking it a bit too far), but having now read the book, I stand by the title for a very different reason: this book is so widely varied -- there are so many divergent ideas here -- that _only_ the most subjective possible title could encompass them. That is, any term less subjective than "beautiful" would be willfully ignorant of the disagreements (if implicit) among the authors about what constitutes ideal software.

To give an idea of what I'm talking about, here is the breakdown of languages and their representations in chapters:

<table width="40%" align="center"><tbody><tr><td><strong>Language</strong></td><td><strong>Chapters</strong></td></tr><tr><td>C</td><td>11</td></tr><tr><td>Java</td><td>5</td></tr><tr><td>Scheme/Lisp</td><td>3</td></tr><tr><td>C++</td><td>2</td></tr><tr><td>Fortran</td><td>2</td></tr><tr><td>Perl</td><td>2</td></tr><tr><td>Python</td><td>2</td></tr><tr><td>Ruby</td><td>2</td></tr><tr><td>C#</td><td>1</td></tr><tr><td>JavaScript</td><td>1</td></tr><tr><td>Haskell</td><td>1</td></tr><tr><td>VisualBASIC</td><td>1</td></tr></tbody></table>

(I'm only counting each chapter once, so for the very few chapters that included two languages, I took whatever appeared more frequently. Also, note that some chapters were about the implementation of one language feature in a different language -- so for example, while there are two additional chapters on Python, both pertain more to the C-based implementation of those features than to their actual design or use in Python.)

Now, one could argue (and it would be an interesting argument) about how much choice of language matters in software or, for that matter, in thought. And indeed, in some (if not many) of these chapters, the language of implementation is completely orthogonal to the idea being discussed. But I believe that language _does_ ultimately affect thought, and it's hard to imagine how one could have a sense of beauty that is so uncritical as to equally accommodate all of these languages.

More specifically: read say, R. Kent Dybvig's chapter on the implementation of `syntax-case` in Scheme and William Otte and Douglas Schmidt's chapter on implementing a distributed logging service using an object-oriented C++ framework. It seems unlikely to me that one person will come away saying that both are beautiful to them. (And I'm not talking new-agey "beautiful to someone" kind of beautiful -- I'm talking the "I want to write code like that" kind of beautiful.) This is not meant to be a value judgement on either of these chapters -- just the observation that their definitions of beauty are (in my opinion, anyway) so wildly divergent as to be nearly mutually exclusive. And that's why the title is perfect: both of these chapters _are_ beautiful to their authors, and we _can_ come away saying "Hey, if it's beautiful to you, then great."

So I continue to strongly recommend _Beautiful Code_, but perhaps not in the way that O'Reilly might intend: you should read this book not because it's cover-to-cover perfection, but rather to hone your _own_ sense of beauty. To that end, this is a book best read concurrently with one's peers: discussing (and arguing about) what is beautiful, what isn't beautiful, and why will help you discover and refine your own voice in your code. And doing this will enable you to write the most important code of all: code that is, if nothing else, beautiful to you.
