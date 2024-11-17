---
title: "Whither USENIX?"
date: "2004-07-06"
---

As [I mentioned earlier](http://dtrace.org/blogs/bmc/2004/07/03/beating-the-odds/), I recently returned from [USENIX '04](http://www.usenix.org/events/usenix04/), where we presented the [DTrace paper](http://www.sun.com/bigadmin/content/dtrace/dtrace_usenix.pdf). It was a little shocking to me that our paper was the **only** paper to come exclusively from industry: most papers had no industry connection whatsoever, and the papers that had any authors from industry were typically primarily written by PhD students interning at industry labs. The content of the General Session was thus academic in the strictest sense: it consisted of papers written by graduate students, solving problems in systems sufficiently small to be solvable by a single graduate student working for a reasonably short period of time. The problem is that many of these systems -- to me at least -- are so small as to not be terribly relevant. This is important because relevance is sufficiently vital to USENIX to be embodied in the [Mission Statement](http://www.usenix.org/about/): USENIX "supports and disseminates research with a practical bias." And of course, there is a more pragmatic reason to seek relevance in the General Session: most of the attendees are from industry, and most of them are paying full-freight. Given that relevance is so critical to USENIX, I was a little surprised that -- unlike most industry conferences I have attended -- there was no way to provide feedback on the General Session. How does the Steering Committee know if the research has a "practical bias" if they don't ask the question?

This leads to the more general question: how do we keep the "practical bias" in academic systems research? Before I try to answer that directly, it's worth looking at the way research is conducted by other engineering disciplines. (After all, one of the things that separates systems from the rest of computer science is its relative proximity to engineering.) To me, it's very interesting to look at the [history of mechanical engineering at MIT](http://libraries.mit.edu/archives/mithistory/histories-offices/mecheng.html). In particular, note the programs that no longer exist:

- Marine engineering, stopped in 1913
- Locomotive engineering, stopped in 1918
- Steam turbine engineering, stopped in 1918
- Engine design, stopped in 1925
- Automotive engineering, stopped in 1949

Why did these programs stop? It's certainly not because there weren't engineering problems to solve. (I can't imagine that anyone would argue that [a 1949 V8](http://www.vanpeltsales.com/FH_web/FH_images/FH_engine-pics/Flathead_Engine_complete1949to53.jpg) was the _ne plus ultra_ of internal combustion engines.) This is something of an educated guess (I'm not a mechanical engineer, so I trust [someone](http://blogs.sun.com/barts) will correct me if I'm grossly wrong here), but I bet these programs were stopped because the economics no longer made sense: it became prohibitively expensive to meaningfully contribute to the state-of-the-art. That is, these specialities were so capital and resource intensive, that they could no longer be undertaken by a single graduate student, or even by a single institution. By the time an institution had built a lab and the expertise to contribute meaningfully, the lab would be obsolete and the expertise would have graduated. Moreover, the disciplines were mature enough that there was an established industry that understood that research begat differentiated product, and differentiated product begat profit. Industry was therefore motivated to do its own research -- which is a good thing, because only industry could afford it.

And what has happened to, say, engine design since the formal academic programs stopped? Hard problems are still being solved, but the way those problems are solved has changed. For example, look at [the 2001 program](http://www.ata.it/italiano/manifest/2001/28_novembre/program.htm) for the [Small Engine Technology Conference](http://www.sae.org/events/set/). A roughly typically snippet:

- G.P. BLAIR - The Queen's University of Belfast (United Kingdom)  
    D.O. MACKEY, M.C. ASHE, G.F. CHATFIELD - OPTIMUM Power Technology (USA)  
    Exhaust pipe tuning on a four-stroke engine; experimentation and simulation
    
- G.P. BLAIR, E. CALLENDER The Queen's University of Belfast (United Kingdom)  
    D.O. MACKEY - OPTIMUM Power Technology (USA)  
    Maps of discharge coefficient data for valves, ports and throttles
    
- V. LAKSHMINARASIMHAN, M.S. RAMASAMY, Y. RAMACHANDRA BABU TVS-Suzuki (India)  
    4 stroke gasoline engine performance optimization using statistical techniques
    
- K. RAJASHEKHAR SWAMY, V. HARNE, D.S. GUNJEGAONKAR TVS-Suzuki (India)  
    K.V. GOPALKRISHNAN - Indian Institute of Technology (India)  
    Study and development of lean burn systems on small 4-stroke gasoline engine
    

Note that there's some work exclusively by industry, and some work done in conjunction with academia. (There's some work done exclusively by academia, too -- but it's the exception, and it tends to be purely analytical work.) And here's the [Program Committee](http://www.ata.it/italiano/manifest/2001/28_novembre/foreword.htm) for this conference:

- M. NUTI - Industrial Consultant
- P. COLOMBO - [Dell'Orto](http://www.dellorto.it/)
- C. DOVERI - [EDI Progetti e Sviluppo](http://www.ediprogetti.it/main.htm)
- G. FORASASSI - [Università di Pisa](http://www.unipi.it/)
- R. GENTILI - [Università di Pisa](http://www.unipi.it/)
- G. LASSANSKE - Chair North American Technical Committee
- G. LEVIZZARI - [ATA](http://www.ata.it)
- M. MARCACCI - [Piaggio](http://www.piaggiousa.com/)
- L. MARTORANO - [Università di Pisa](http://www.unipi.it/)
- L. PETRINI - [Aprilia](http://www.apriliausa.com/portale/eng/home.phtml)
- R. RINOLFI - [Centro Ricerche Fiat](http://www.crf.it/uk/home.htm)

Of these, three are clearly academics, and seven are clearly from industry.

Okay, so that's one example of how a traditional engineering discipline conducts joint academic/industrial research. Let's get back to USENIX with a look at the Program Committee for [USENIX '05](http://www.usenix.org/events/usenix05/organizers.html). Note that the mix is exactly the inverse: twelve work for a university and five work for a company. Worse, of those five putatively from industry, **all** of them work in academic labs. In our industry, these labs have a long tradition of being pure research outfits -- they often have little-to-no product responsibilities. (Which, by the way, is just an observation -- it's not meant to be a value judgement.)

Even more alarming, the makeup of the [FREENIX '05](http://www.usenix.org/events/usenix05/cfp/freenix.html) [program committee](http://www.usenix.org/events/usenix05/organizers.html) is almost completely from industry. This leads to the obvious question: is FREENIX becoming some sort of dumping ground for systems research deemed to be too "practically biased" for the academy? I hope not: aside from the obvious problem of confusing research problems with business models, having the General Session become strictly academic and leaving the FREENIX track to become strictly industrial effectively separates the academics from the practitioners. And this, in my opinion, is exactly what we _don't_ need...

So how do we keep the "practical bias" in the academic work presented at USENIX? For starters, industry should be better represented at the Program Committee and on the Steering Committee. In my opinion, this is most easily done by eliminating FREENIX (as such), folding the FREENIX Program Committee into the General Session Program Committee, and then having an interleaved "practical" track within the General Session. That is, alternate between practical sessions and academic ones -- forcing the practitioners to sit through the academic sessions and vice versa.

That may be too radical, but the larger point is that we need to start having an honest conversation: how do we prevent USENIX from becoming [irrelevant](http://www.dowsers.org/)?
