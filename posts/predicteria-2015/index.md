---
title: "Predicteria 2015"
date: "2015-01-06"
---

Fifteen years ago, I initiated a time-honored tradition among my colleagues in kernel development at Sun: shortly after the first of every year, we would get together at [our favorite local restaurant](http://www.theosteriaavanti.com/) to form predictions for the coming year. We made one-year, three-year and six-year predictions for both our technologies and more broadly for the industry. We did this for nine years running -- from 2000 to 2008 inclusive -- and came to know the annual ritual as a play on the restaurant name: Predicteria.

I have always been in interested in our past notions of the future ([hoverboards and self-lacing shoes](http://www.newsweek.com/everything-back-future-ii-got-hilariously-wrong-about-2015-according-293272) FTW!), and looking back now at nearly a decade of our predictions has led me to an inescapable (and perhaps obvious) conclusion: **predictions tell you more about the present than the future**. That is, predictions reflect the zeitgeist of the day -- both in substance and in tone: in good years, people predict halcyon days; in bad ones, the apocalypse. And when a particular company or technology happened to be in the news or otherwise on the collective mind, predictions tended to be centered around it: it was often the case that several people would predict that a certain company would be acquired or that a certain technology would flourish -- or perish. (Let the record reflect that [the demise of Itanium](http://www.itworld.com/article/2702259/data-center/hp-to-replace-itanium-with-x86-in-its-nonstop-server.html) was accurately predicted many times over.)

Which is not to say that we never made gutsy predictions; in 2006, a colleague made a one-year prediction that "GOOG embarrassed by revelation of unauthorized US government spying at Gmail." The timing may have been off, but the concern was disturbingly prescient. Sometimes the predictions were right, but for the wrong reasons: in 2003, one of my three-year predictions was that "Apple develops new 'must-have' gadget called the iPhone, a digital camera/MP3 player/cell phone." This turned out to be stunningly accurate, even down to the timing (and it was by far my most accurate big prediction over the years), but if you can't tell by the snide tone, I thought that such a thing would be [Glass](http://en.wikipedia.org/wiki/Google_Glass)-like in its ludicrousness; I had not an inkling as to its transformative power. (And indeed, when the iPhone did in fact emerge a few years later, several at Predicteria predicted that it would be a disappointing flop.)

But accurate predictions were the exception, not the rule; our predictions were usually wrong -- often wildly so. Evergreen wildly wrong predictions included: the rise of [carbon nanotube-based memory](http://en.wikipedia.org/wiki/Nano-RAM), the relevance of [quantum computing](http://en.wikipedia.org/wiki/Anti-gravity), and the death of tape, disk or volatile DRAM (each predicted several times over). We were also wrong by our omissions: as a group, we entirely failed to predict cloud computing -- or even the rise of hardware-based virtualization.

I give all of this as a backdrop to some predictions for the coming year. If my experience taught me anything, it's that these predictions may very well be right on trajectory, but wrong on timing -- and that they may well capture current thinking more than they meaningfully predict the future. They also may be (rightfully) criticized for, as they say, talking our book -- but we have made our bets based on where we think things are going, not vice versa. And finally, I apologize that these are somewhat milquetoast predictions; I'm afraid that practical concerns muffle the gutsy predictions that name names and boldly predict their fates!

Without further ado, looking forward to 2015:

- **2015 is the year of the container.** If you've read our [2014 in review](https://www.joyent.com/blog/2014-in-review-docker-rising), you won't be at all surprised to read this and I don't really think that it's controversial. Thanks to Docker, the world is finally figuring out that OS-based virtualization is actually highly disruptive (better performance _and_ lower cost!), and I think that this realization will become mainstream in 2015. I don't think that Docker will necessarily be the only substrate, but I believe that its momentum is such that it will continue to dominate in 2015.
    
- **The impedance mismatch between containers in development and containers in production will be a wellspring of innovation.** Currently, containers have a ton of developer enthusiasm -- but limited production deployments, in part because of production concerns around security, persistence and network virtualization. All of these problems have multiple solutions -- and there will be still more in 2015 ([not least from us](https://www.joyent.com/blog/making-joyent-the-best-place-to-run-docker)). It's hard to predict winners (or if winners will even emerge in 2015), but it's a sure bet that there will be (many) players tackling the problems in interesting ways.
    
- **The on-prem versus public cloud debate will be increasingly seen as a false dichotomy.** As [I outlined at GigaOm Structure](https://gigaom.com/2014/06/18/the-future-of-computing-may-be-about-balancing-anarchy-and-control/) in June, the future is heterogenous: the public versus on-prem distinction will be be a rent-versus-buy decision based on [economics, risk management and latency](http://www.slideshare.net/bcantrill/structure2014-cantrill/6). Entities will not have one answer here, but many: one is not going to vanquish the other. That said, we also believe that choosing one versus the other shouldn't involve having to change the underlying technology substrate -- a major factor for us when we [open sourced SmartDataCenter](https://www.joyent.com/blog/sdc-and-manta-are-now-open-source) in November.
    
- **The internet-of-things broadens beyond mere trend.** I continue to believe that IoT is an absolute monster of a shift -- one that will have profound changes for nearly every economic endeavor. As [Marc Andreessen famously observed](http://www.wsj.com/articles/SB10001424053111903480904576512250915629460), "software is eating the world" -- and ubiquitous computing will increasingly be the buffet. At Velocity 2014, I described [architecting for the deluge of data](http://www.slideshare.net/bcantrill/velocity2014), and I think that the arrival of machine-generated data (and the analytics to turn that data into business decisions) will be a defining characteristic of the next decade(s) -- part of why we [open sourced Manta](https://www.joyent.com/blog/sdc-and-manta-are-now-open-source) when we opened SmartDataCenter. To what degree this happens in 2015 is unclear, but at some point we will stop thinking of IoT as a "trend" and realize that it is simply part of the rising tide of the information society.
    
- **Wearable computing fizzles.** If wearable computing ever has a time, it isn't now: devices out there today are gimmicky, brittle, and -- to put it euphemistically -- [overly flashy](http://www.urbandictionary.com/define.php?term=Glasshole). These devices will continue to enjoy some life among the one-percenter Sharper Image set, but they won't actually make the leap to broad consumer devices. (An exception may be the [smart onesie](http://www.engadget.com/2014/01/07/intel-smart-baby-onesie/); speaking from my own experience, first-time parents are suckers for anything that seems to relate to infant safety.)
    
- **Security concerns become focused on intrusion and fraud detection.** If the [Target breach](http://krebsonsecurity.com/2014/05/the-target-breach-by-the-numbers/) didn't wake people up, the [Sony Pictures fracas](http://en.wikipedia.org/wiki/Sony_Pictures_Entertainment_hack) sure as hell did. As we head into 2015, it is being more broadly recognized that security is not a public vs. on-prem issue (after all, the biggest breaches have been on-prem) -- and that disgruntled insiders may represent the greatest single threat to an entity. Security (and specifically, intrusion and fraud detection) will be viewed increasingly as a real-time big data problem -- one that could save a business from an existential threat.
    
- **Cryptocurrencies begin their long journey to laughingstock.** In 2015, cryptocurrencies will stop being viewed as having "one bad year" -- and start being viewed for what they are: modern-day Beanie Babies for ultra-libertarian math nerds. It's stunning to me how many otherwise intelligent people fell for cryptocurrency, having forgotten (or failed to understand in the first place) what a currency actually is: a medium for exchange and a store of value. As such, currencies that experience acute inflation or (worse) acute deflation tautologically fail. We actually have a lot of (willfully ignored) macroeconomic history that tells us this; central banking was only invented after we had had disastrous experiences with every other way of managing currency (viz. the [Panic of 1837](http://en.wikipedia.org/wiki/Panic_of_1837)). And this isn't just like, [my opinion, man](https://www.youtube.com/watch?v=pWdd6_ZxX8c): no one purchasing or selling goods is going to opt into a currency that introduces volatility risk in an orthogonal (and often low margin) commercial transaction. (Indeed, in the you-can't-make-this-up department, [Bitcoin has become too volatile to be reliably used for ransom](http://www.nytimes.com/2015/01/04/opinion/sunday/how-my-mom-got-hacked.html?_r=1).) Of course, proponents are now saying that ["the value isn't in the currency but in the blockchain application stack"](http://joel.mn/post/103546215249/the-blockchain-application-stack) -- which is like saying the value of Beanie Babies is in their soft and cuddly nature: yes the value of [Punchers the Lobster](http://beaniepedia.com/beanies/original/punchers-the-lobster/) is non-zero, but it doesn't mean that [it's actually worth 2,600 clams](http://www.smartcollecting.com/news-print.asp?ID=21).
    

Right or wrong, these predictions point to an exciting 2015. And if nothing else you can rely on my for a candid self-assessment of my predictions -- you'll just need to wait fifteen years or so!