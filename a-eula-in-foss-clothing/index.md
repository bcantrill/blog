---
title: "A EULA in FOSS clothing?"
date: "2018-12-17"
---

There was a tremendous amount of reaction to and discussion about my [blog entry on the midlife crisis in open source](http://dtrace.org/blogs/bmc/2018/12/14/open-source-confronts-its-midlife-crisis/). As part of this discussion on HN, Jay Kreps of Confluent took the time to write a [detailed response](https://news.ycombinator.com/item?id=18687498#18689179) -- which he shortly thereafter elevated into [a blog entry](https://medium.com/@jaykreps/a-quick-comment-on-bryan-cantrills-blog-on-licensing-8dccee41d9e6).

Let me be clear that I hold Jay in high regard, as both a software engineer and an entrepreneur -- and I appreciate the time he took to write a thoughtful response. That said, there are aspects of his response that I found troubling enough to closely re-read the [Confluent Community License](https://www.confluent.io/confluent-community-license) -- and that in turn has led me to a deeply disturbing realization about what is potentially going on here.

Here is what Jay said that I found troubling:

> The book analogy is not accurate; for starters, copyright does not apply to physical books and intangibles like software or digital books in the same way.

Now, what Jay said is true to a degree in that (as with many different kind of expression), software has code specific to it; this can be found in [17 U.S.C. § 117](https://www.copyright.gov/title17/92chap1.html#117). But the fact that Jay also made reference to digital books was odd; digital books really have nothing to do with software (or not any more so than any other kind of creative expression). That said, digital books and proprietary software _do_ actually share one thing in common, though it's horrifying: in both cases **their creators have maintained that you don't actually own the copy you paid for**. That is, unlike a book, you don't actually _buy_ a copy of a digital book, you merely acquire a license to use their book under their terms. But how do they do this? Because when you access the digital book, you click "agree" on a license -- an End User License Agreement (EULA) -- that makes clear that you don't _actually_ own anything. The exact language varies; take (for example) [VMware's end user license agreement](https://www.vmware.com/download/eula/universal_eula.html):

> **2.1 General License Grant.** VMware grants to You a non-exclusive, non-transferable (except as set forth in Section 12.1 (Transfers; Assignment) license to use the Software and the Documentation during the period of the license and within the Territory, solely for Your internal business operations, and subject to the provisions of the Product Guide. Unless otherwise indicated in the Order, licenses granted to You will be perpetual, will be for use of object code only, and will commence on either delivery of the physical media or the date You are notified of availability for electronic download.

That's a bit wordy and oblique; in this regard, [Microsoft's Windows 10 license](https://www.microsoft.com/en-us/Useterms/Retail/Windows/10/UseTerms_Retail_Windows_10_English.htm) is refreshingly blunt:

> (2)(a) **License.** The software is licensed, not sold. Under this agreement, we grant you the right to install and run one instance of the software on your device (the licensed device), for use by one person at a time, so long as you comply with all the terms of this agreement.

That's pretty concise: "The software is licensed, not sold." So why do this at all? EULAs are an attempt to get out of copyright law -- where the copyright owner is quite limited in the rights afforded to them as to how the content is consumed -- and into contract law, where there are many fewer such limits. And EULAs have accordingly historically restricted (or tried to restrict) all sorts of uses like [benchmarking](https://en.wikipedia.org/wiki/David_DeWitt), reverse engineering, running with competitive products (or, say, being used by a competitor to make competitive products), and so on.

Given the onerous restrictions, it is not surprising that [EULAs are very controversial](https://www.eff.org/wp/dangerous-terms-users-guide-eulas/). They are also legally dubious: when you are forced to click through or (as it used to be back in the day) forced to unwrap a sealed envelope on which the EULA is printed to get to the actual media, it's unclear how much you are actually "agreeing" to -- and it may be considered a [contract of adhesion](https://en.wikipedia.org/wiki/Standard_form_contract). And this is just one of [many legal objections to EULAs](http://thewaronbullshit.com/2007/09/05/eula/).

Suffice it to say, EULAs have long been considered open source poison, so with Jay's frightening reference to EULA'd content, I went back to the [Confluent Community License](https://www.confluent.io/confluent-community-license) -- and proceeded to kick myself for having missed it all on my first quick read. First, there's this:

> This Confluent Community License Agreement Version 1.0 (the “Agreement”) sets forth the terms on which Confluent, Inc. (“Confluent”) makes available certain software made available by Confluent under this Agreement (the “Software”). BY INSTALLING, DOWNLOADING, ACCESSING, USING OR DISTRIBUTING ANY OF THE SOFTWARE, YOU AGREE TO THE TERMS AND CONDITIONS OF THIS AGREEMENT. IF YOU DO NOT AGREE TO SUCH TERMS AND CONDITIONS, YOU MUST NOT USE THE SOFTWARE. IF YOU ARE RECEIVING THE SOFTWARE ON BEHALF OF A LEGAL ENTITY, YOU REPRESENT AND WARRANT THAT YOU HAVE THE ACTUAL AUTHORITY TO AGREE TO THE TERMS AND CONDITIONS OF THIS AGREEMENT ON BEHALF OF SUCH ENTITY.

You will notice that this looks nothing like any traditional source-based license -- but it _is_ exactly the kind of boilerplate that you find on EULAs, terms-of-service agreements, and other contracts that are being rammed down your throat. And then there's this:

> **1.1 License.** Subject to the terms and conditions of this Agreement, Confluent hereby grants to Licensee a non-exclusive, royalty-free, worldwide, non-transferable, non-sublicenseable license during the term of this Agreement to: (a) use the Software; (b) prepare modifications and derivative works of the Software; (c) distribute the Software (including without limitation in source code or object code form); and (d) reproduce copies of the Software (the “License”).

On the one hand looks like the opening of open source licenses like (say) the [Apache Public License](https://www.apache.org/licenses/LICENSE-2.0) (albeit missing important words like "perpetual" and "irrevocable"), but the next two sentences are the difference that are the focus of the license:

> Licensee is not granted the right to, and Licensee shall not, exercise the License for an Excluded Purpose. For purposes of this Agreement, “Excluded Purpose” means making available any software-as-a-service, platform-as-a-service, infrastructure-as-a-service or other similar online service that competes with Confluent products or services that provide the Software.

But how can you later tell me that I can't use _my_ copy of the software because it competes with a service that Confluent started to offer? Or is that copy not in fact mine? This is answered in section 3:

> Confluent will retain all right, title, and interest in the Software, and all intellectual property rights therein.

Okay, so my copy of the software _isn't_ mine at all. On the one hand, this is (literally) proprietary software boilerplate -- but I was given the source code and the right to modify it; how do I not own my copy? Again, proprietary software is built on the notion that -- unlike the book you bought at the bookstore -- you don't own anything: rather, you license the copy that is in fact owned by the software company. And again, as it stands, this is dubious, and [courts have ruled against "licensed, not sold" software](https://en.wikipedia.org/wiki/Software_license#Ownership_vs._licensing). But how can a license explicitly allow me to _modify_ the software and at the same time tell me that I don't own the copy that I just modified?! And to be clear: I'm not asking who owns the copyright (that part is clear, as it is for open source) -- I'm asking **who owns the copy of the work that I have modified?** How can one argue that I don't own the copy of the software that I downloaded, modified and built myself?!

This prompts the following questions, which I [also asked Jay via Twitter](https://twitter.com/bcantrill/status/1074260959898693632):

1. If I git clone software covered under the Confluent Community License, who owns that copy of the software?
    
2. Do you consider the Confluent Community License to be a contract?
    
3. Do you consider the Confluent Community License to be a EULA?
    

To Confluent: please answer these questions, and put the answers in your FAQ. Again, I think it's fine for you to be an open core company; just make this software proprietary and be done with it. (And don't let yourself be troubled about the fact that it was once open source; there is [ample precedent for reproprietarizing software](https://www.youtube.com/watch?v=Zpnncakrelk#t=17m21s).) What I object to (and what I think others object to) is trying to be at once open and proprietary; you must pick one.

To GitHub: Assuming that this is in fact a EULA, I think it is perilous to allow EULAs to sit in public repositories. It's one thing to have one click through to accept a license (though again, that itself is dubious), but to say that a git clone is an implicit acceptance of a contract that happens to be sitting somewhere in the repository beggars belief. With efforts like [choosealicense.com](https://choosealicense.com/), GitHub has been a model in guiding projects with respect to licensing; it would be helpful for GitHub's counsel to weigh in on their view of this new strain of source-available proprietary software and the degree to which it comes into conflict with [GitHub's own terms of service](https://help.github.com/articles/github-terms-of-service/#5-license-grant-to-other-users).

To foundations concerned with software liberties, including the Apache Foundation, the Linux Foundation, the Free Software Foundation, the Electronic Frontier Foundation, the Open Source Initiative, and the Software Freedom Conservancy: the open source community needs your legal review on this! I don't think I'm being too alarmist when I say that this is potentially a dangerous new precedent being set; it would be very helpful to have your lawyers offer their perspectives on this, even if they disagree with one another. We seem to be in some terrible new era of frankenlicenses, where the worst of proprietary licenses are bolted on to the goodwill created by open source licenses; we need your legal voices before these creatures destroy the village!
