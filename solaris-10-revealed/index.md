---
title: "Solaris 10 Revealed"
date: "2005-01-25"
categories: 
  - "solaris"
---

Or some of it, anyway. If you haven't yet seen it, today we [open sourced some of Solaris](http://opensolaris.org). This isn't actually the formal launch of OpenSolaris -- we're still busily working away on that -- but we wanted to reveal enough of the juicy bits of the Solaris source for the world to realize that we're serious about OpenSolaris. I view OpenSolaris as an important milestone in the history of operating systems -- doubly so because it is Solaris 10 that we are open sourcing, an operating system that I believe to be an important historical milestone in its own right -- so it is an honor for me to report that the source code that we decided to release today is that for [DTrace](http://www.sun.com/bigadmin/content/dtrace).

But enough with the pomp; let's talk source. I assume that most people who download the source today will download it, check to see that it's actually source code,1 and then say to themselves, "now what?" To help answer that question, I thought I would take a moment to describe the layout of the source, and point you to some interesting tidbits therein.

As with any Unix variant with Bell labs roots, you'll find all of the source under the "`usr/src`" directory. Under `usr/src`, you'll find four directories:

- `usr/src/cmd` contains commands. Soon, this directory will be populated with its 400+ commands, but for now it just contains subdirectories for each of the following DTrace consumers: `dtrace(1M)`, `intrstat(1M)`, `lockstat(1M)` and `plockstat(1M)`. This directory additionally contains a subdirectory for the `isaexec` command, as the DTrace consumers are all `isaexec(3C)`'d.
    
- `usr/src/lib` contains libraries. Soon, this directory will be populated with its 150+ libraries, but for now it just contains the libraries upon which the DTrace consumers depend. Each of these libraries is in an eponymous subdirectory:
    
    - `libdtrace(3LIB)` is the library that does much of the heavy lifting for a DTrace consumer. The D compiler lives here, along with all of the infrastructure to process outbound data from the kernel. The kernel/user interface is discussed in detail in `<sys/dtrace.h>`.
        
    - `libctf` is a library that is able to interpret the Compact C Type Format (CTF2). CTF is much more compact than the traditional type stabs, and we use it to represent the kernel's types. (CTF is what allows `::print` to work in `mdb`. If you've never done it, try "`echo '::print -at vnode\_t' | mdb -k`" as `root` on a Solaris 9 or Solaris 10 machine.) DTrace is very dependent on `libctf` and hence the reason that we're including it now. Note that much of the source for `libctf` is in `usr/src/common`; see below.
        
    - `libproc` is a library that exports interfaces for process control via /proc. (The Solaris `/proc` is vastly different from the Linux `/proc` in that it is used primarily as a means of process control and process information -- not simply as a means of system information; see `proc(4)` for details.) Many, many Solaris utilities link with `libproc` including `pcred(1)`, `pfiles(1)`, `pflags(1)`, `pldd(1)`, `plimit(1)`, `pmap(1)`, `ppgsz(1)`, `ppriv(1)`, `prctl(1)`, `preap(1)`, `prstat(1)`, `prun(1)`, `pstack(1)`, `pstop(1)`, `ptime(1)`, `ptree(1)` and `pwdx(1)`. Thanks to the powerful interfaces in `libproc`, many of these utilities are quite short in terms of lines of code. These interfaces aren't yet public, but you can get a taste for them by looking at `/usr/include/libproc.h` -- or by reading the source, of course!
        
    
- `usr/src/uts` is the main event -- the kernel. ("uts" stands for "Unix Time-Sharing System", and is another artifact from Bell Labs.) The subdirectories here are roughly what you might expect:
    
    - `common` contains common code
        
    - `i86pc` contains code specific to the PC machine architecture
        
    - `intel` contains code specific to the x86 instruction set architecture
        
    - `sparc` contains code specific to the SPARC instruction set architecture
        
    - `sun4` contains code specific to the sun4u machine architecture, but general to all platform architectures within that machine architecture
    
    The difference between instruction set architecture and machine architecture is a bit fuzzy, especially when there is a one-to-one relationship between the two. (And in case you're not yet confused, platform architectures add another perplexing degree of freedom within machine architectures.) All of this made more sense when there was a one-to-many relationship between instruction sets and machine architectures, but `sun4m`, `sun4d` and `sun4c` have been EOL'd and the source for these machine architectures has been removed. This layout may seem confusing, but I'm here to describe the source layout -- not defend it -- so moving on...
    
    In terms of DTrace, most of the excitement is in `usr/src/uts/common/dtrace`, `uts/src/uts/intel/dtrace` and `usr/src/uts/sparc/dtrace`. In `usr/src/uts/common/dtrace`, you'll find the meat of the in-kernel DTrace component in the 13,000+ lines of `dtrace.c`. In this directory you will additionally find the source for common providers like `profile(7D)` and `systrace(7D)` along with the common components for the `lockstat(7D)` provider. In the ISA-specific directories, you'll find the ISA-specific halves to these providers, along with wholly ISA-specific providers like `fbt(7D)`. You will also find the ISA-specific components to DTrace itself in `dtrace\_asm.s` and `dtrace\_isa.c`.

So that's the basic layout of the source but...now what? If you're like me, you don't have the time or interest to understand something big and foreign -- and you mainly look at source code to get a flavor for things. Maybe you just want to grep for "XXX" (you'll find two -- but we on the DTrace team are responsible for neither one) or look for curse words (you'll find none -- at least none yet) or just search for revealingly frank words like "hack", "kludge", "vile", "sleaze", "ugly", "gross", "mess", etc. (of which you will regrettably find at least one example of each).

But in the interest of leaving you with more than just whiffs of our dirty laundry, let me guide you to some interesting tidbits that require little or no DTrace knowledge to appreciate. I'm not claiming that these tidbits are particularly novel or even particularly core to DTrace; I'm only claiming that they're interesting for one reason or another. For example, check out this comment in `dtrace.c`:

```
/*
 * We want to have a name for the minor.  In order to do this,
 * we need to walk the minor list from the devinfo.  We want
 * to be sure that we don't infinitely walk a circular list,
 * so we check for circularity by sending a scout pointer
 * ahead two elements for every element that we iterate over;
 * if the list is circular, these will ultimately point to the
 * same element.  You may recognize this little trick as the
 * answer to a stupid interview question -- one that always
 * seems to be asked by those who had to have it laboriously
 * explained to them, and who can't even concisely describe
 * the conditions under which one would be forced to resort to
 * this technique.  Needless to say, those conditions are
 * found here -- and probably only here.  Is this is the only
 * use of this infamous trick in shipping, production code?
 * If it isn't, it probably should be...
 */

```

This is the code that executes the `ddi\_pathname` function in D. A critical constraint on executing D is that any programming errors must be caught and handled gracefully. (We call this the "safety constraint" because failure to abide by it will induce fatal system failure.) While many programming environments recover from memory-related errors, we in DTrace must additionally guard against infinite iteration -- a much harder problem. (In fact, this problem is so impossibly hard that its name has become synonymous with undecidability: this is the [halting problem](http://en.wikipedia.org/wiki/Halting_problem) that Turing proved impossible to solve in 1936.) We skirt the halting problem in DTrace by not allowing programmer-specified iteration whatsoever: DIF doesn't allow backwards branches. But some D functions -- like `ddi\_pathname()` -- require iteration over untrusted data structures to function. When iterating over an untrusted list, our fear is circularity (be it innocent or pernicious), and the easiest way for us to determine this circularity is to use the interview question described above.

I wrote this code a while ago; now that I read it again I actually _can_ imagine some uses for this in other production code -- but I would imagine it would all be of the assertion variety. (That is, again the data structure is effectively untrusted.) Any other use still strikes me as busted (prove me wrong?), and I still have disdain for those that ask it as an interview question (apologies if this includes you, gentle reader).

Here's another interesting tidbit, also in `dtrace.c`:

```
	switch (v) {
	case DIF_VAR_ARGS:
		ASSERT(mstate->dtms_present & DTRACE_MSTATE_ARGS);
		if (i >= sizeof (mstate->dtms_arg) /
		    sizeof (mstate->dtms_arg[0])) {
			int aframes = mstate->dtms_probe->dtpr_aframes + 2;
			dtrace_provider_t *pv;
			uint64_t val;
			pv = mstate->dtms_probe->dtpr_provider;

			if (pv->dtpv_pops.dtps_getargval != NULL)
				val = pv->dtpv_pops.dtps_getargval(pv->dtpv_arg,
				    mstate->dtms_probe->dtpr_id,
				    mstate->dtms_probe->dtpr_arg, i, aframes);
			else
				val = dtrace_getarg(i, aframes);

			/*
			 * This is regrettably required to keep the compiler
			 * from tail-optimizing the call to dtrace_getarg().
			 * The condition always evaluates to true, but the
			 * compiler has no way of figuring that out a priori.
			 * (None of this would be necessary if the compiler
			 * could be relied upon to _always_ tail-optimize
			 * the call to dtrace_getarg() -- but it can't.)
			 */
			if (mstate->dtms_probe != NULL)
				return (val);
			ASSERT(0);
			...

```

This is the code that retrieves an argument due to a reference to `args\[`_n_`\]` (or `arg`_n_). The clause above will only be executed if _n_ is equal to or greater than five -- in which case we need to go fishing in the stack frame for the argument. And here's where things get a bit gross: in order to be able to find the right stack frame, we must know exactly how many stack frames have been artificially pushed in the process of getting into DTrace. This includes frames that the provider may have pushed (tracked in the probe as the `dtpr\_aframes` variable) and the frames that DTrace itself has pushed (rather bogusly represented by the constant "2", above: one for `dtrace\_probe()` and one for `dtrace\_dif\_emulate()`). The problem is that if the call to `dtrace\_getarg()` is tail-optimized, our calculation is incorrect. We therefore have to trick the compiler by having an expression after the call that the compiler is forced to evaluate after the call. We do this by having an expression that always evaluates to true, but dereferences through a pointer. Because `dtrace\_getarg()` is in another object file, no amount of alias disambiguation is going to figure out that it doesn't modify `dtms\_probe`; the compiler doesn't tail-optimize the above call, the stack frame calculation is correct, and the arguments are correctly fished out of the (true) caller's stack frame.

There's an interesting footnote to the above code: recently, we ran a research tool that performs static analysis of code on the source for the Solaris kernel. The tool was actually pretty good, and found all sorts of interesting issues. Among other things, the tool flagged the above code, observing that `dtms\_probe` is never `NULL`. (The tool may be clever enough to determine that, but it obviously can't be clever enough to know that we're trying to outsmart the compiler here.) While this might give us pause, it needn't: the tool might warn about it, but no compiler could safely avoid evaluating the expression -- because `dtrace\_getarg()` is not in the same object file, it cannot be absolutely certain that `dtrace\_getarg()` does not store to `dtms\_probe`.

As long as we're going through `dtrace\_getarg()`, though, it may be interesting to look at a routine that implements this on SPARC.3 This routine, found in `usr/src/uts/sparc/dtrace/dtrace\_asm.s`, fishes a specific argument out of a specified register window -- _without_ causing a window spill trap. Here's the function:

```
#if defined(lint)
/*ARGSUSED*/
int
dtrace_fish(int aframes, int reg, uintptr_t *regval)
{
return (0);
}
#else   /* lint */
	ENTRY(dtrace_fish)
	rd      %pc, %g5
	ba      0f
	add     %g5, 12, %g5
	mov     %l0, %g4
	mov     %l1, %g4
	mov     %l2, %g4
	mov     %l3, %g4
	mov     %l4, %g4
	mov     %l5, %g4
	mov     %l6, %g4
	mov     %l7, %g4
	mov     %i0, %g4
	mov     %i1, %g4
	mov     %i2, %g4
	mov     %i3, %g4
	mov     %i4, %g4
	mov     %i5, %g4
	mov     %i6, %g4
	mov     %i7, %g4
0:
	sub     %o1, 16, %o1            ! Can only retrieve %l's and %i's
	sll     %o1, 2, %o1             ! Multiply by instruction size
	add     %g5, %o1, %g5           ! %g5 now contains the instr. to pick
	rdpr    %ver, %g4
	and     %g4, VER_MAXWIN, %g4

	!
	! First we need to see if the frame that we're fishing in is still
	! contained in the register windows.
	!
	rdpr    %canrestore, %g2
	cmp     %g2, %o0
	bl      %icc, 2f
	rdpr    %cwp, %g1
	sub     %g1, %o0, %g3
	brgez,a,pt %g3, 0f
	wrpr    %g3, %cwp

	!
	! CWP minus the number of frames is negative; we must perform the
	! arithmetic modulo MAXWIN.
	!
	add     %g4, %g3, %g3
	inc     %g3
	wrpr    %g3, %cwp
0:
	jmp     %g5
	ba      1f
1:
	wrpr    %g1, %cwp
	stn     %g4, [%o2]
	retl
	clr     %o0                     ! Success; return 0.
2:
	!
	! The frame that we're looking for has been flushed to the stack; the
	! caller will be forced to
	!
	retl
	add     %g2, 1, %o0             ! Failure; return deepest frame + 1
	SET_SIZE(dtrace_fish)
#endif

```

First, apologies for the paucity of comments in the above. The lack of comments is particularly unfortunate because the function is somewhat subtle, as it uses some dicey register window manipulation plus an odd SPARC technique known as "instruction picking": the `jmp` with the the `ba` in the delay slot picks one of the instructions out of the table that follows the `ba 0f`, thus allowing the caller to specify any register to fish out of the window without requiring any compares.4 If you're interested in the details of the register window manipulation logic in this function, you should consult the [SPARC V9 Architecture Manual](http://www.sparc.com/standards/SPARCV9.pdf).

That about does it for tidbits, at least for now. As you browse the DTrace source, you may well find yourself asking "does it need to be this complicated?" The short answer is, in most cases, "regrettably yes." If you're looking for some in-depth discussion on the specific issues that complicate specific features, I would direct you to functions like `dtrace\_hres\_tick()` (in `usr/src/uts/common/os/dtrace\_subr.c`), `dtrace\_buffer\_reserve()` (in `usr/src/uts/common/dtrace/dtrace.c`) and `dt\_consume\_begin()` (in `usr/src/lib/libdtrace/common/dt\_consume.c`). These functions are good examples of how seemingly simple DTrace features like timestamps, ring buffers and `BEGIN`/`END` probes can lead to much more complexity than one might guess.

Finally, I suppose there's an outside chance that you might actually want to understand how DTrace works -- perhaps even to modify it yourself. If this describes you, you should first heed this advice from `usr/src/uts/common/dtrace/dtrace.c`:

```
/*
 * DTrace - Dynamic Tracing for Solaris
 *
 * This is the implementation of the Solaris Dynamic Tracing framework
 * (DTrace).  The user-visible interface to DTrace is described at length in
 * the "Solaris Dynamic Tracing Guide".  The interfaces between the libdtrace
 * library, the in-kernel DTrace framework, and the DTrace providers are
 * described in the block comments in the <sys/dtrace.h> header file.  The
 * internal architecture of DTrace is described in the block comments in the
 * <sys/dtrace_impl.h> header file.  The comments contained within the DTrace
 * implementation very much assume mastery of all of these sources; if one has
 * an unanswered question about the implementation, one should consult them
 * first.
 * ...

```

This is important advice, because we (by design) put many of the implementation comments in a stock header file, `<sys/dtrace\_impl.h>`. We did this because we believe in the Unix idea that the system implementation should be described as much as possible in its publicly available header files.5 Discussing the comments in `<sys/dtrace\_impl.h>` evokes a somewhat amusing anecdote; take this comment describing the implementation of speculative tracing:

```
/*
 * DTrace Speculations
 *
 * Speculations have a per-CPU buffer and a global state.  Once a speculation
 * buffer has been committed or discarded, it cannot be reused until all CPUs
 * have taken the same action (commit or discard) on their respective
 * speculative buffer.  However, because DTrace probes may execute in arbitrary
 * context, other CPUs cannot simply be cross-called at probe firing time to
 * perform the necessary commit or discard.  The speculation states thus
 * optimize for the case that a speculative buffer is only active on one CPU at
 * the time of a commit() or discard() -- for if this is the case, other CPUs
 * need not take action, and the speculation is immediately available for
 * reuse.  If the speculation is active on multiple CPUs, it must be
 * asynchronously cleaned -- potentially leading to a higher rate of dirty
 * speculative drops.  The speculation states are as follows:
 *
 *  DTRACESPEC_INACTIVE       <= Initial state; inactive speculation
 *  DTRACESPEC_ACTIVE         <= Allocated, but not yet speculatively traced to
 *  DTRACESPEC_ACTIVEONE      <= Speculatively traced to on one CPU
 *  DTRACESPEC_ACTIVEMANY     <= Speculatively traced to on more than one CPU
 *  DTRACESPEC_COMMITTING     <= Currently being committed on one CPU
 *  DTRACESPEC_COMMITTINGMANY <= Currently being committed on many CPUs
 *  DTRACESPEC_DISCARDING     <= Currently being discarded on many CPUs
 *
 * The state transition diagram is as follows:
 *
 *     +----------------------------------------------------------+
 *     |                                                          |
 *     |                      +------------+                      |
 *     |  +-------------------| COMMITTING |<-----------------+   |
 *     |  |                   +------------+                  |   |
 *     |  | copied spec.            ^             commit() on |   | discard() on
 *     |  | into principal          |              active CPU |   | active CPU
 *     |  |                         | commit()                |   |
 *     V  V                         |                         |   |
 * +----------+                 +--------+                +-----------+
 * | INACTIVE |---------------->| ACTIVE |--------------->| ACTIVEONE |
 * +----------+  speculation()  +--------+  speculate()   +-----------+
 *     ^  ^                         |                         |   |
 *     |  |                         | discard()               |   |
 *     |  | asynchronously          |            discard() on |   | speculate()
 *     |  | cleaned                 V            inactive CPU |   | on inactive
 *     |  |                   +------------+                  |   | CPU
 *     |  +-------------------| DISCARDING |<-----------------+   |
 *     |                      +------------+                      |
 *     | asynchronously             ^                             |
 *     | copied spec.               |       discard()             |
 *     | into principal             +------------------------+    |
 *     |                                                     |    V
 *  +----------------+             commit()              +------------+
 *  | COMMITTINGMANY |<----------------------------------| ACTIVEMANY |
 *  +----------------+                                   +------------+
 */

```

In writing up this comment, I became elated that I was able to render the state transition diagram as a planar graph. In fact, I was so (excessively) proud that I showed the state transition diagram to my wife, explaining that I wanted to have it tattooed on my back.6 But my initial cut of this had a typo: instead of saying "`discard() on active CPU`," that edge was labelled "`dicard() on active CPU`." Of course, my wife saw this instantly, and -- without a missing a beat -- responded "please don't tattoo 'dicard' on your back." Pride goeth before a fall, but it's especially painful when it goeth mere _seconds_ before...

Anyway, if you've already read all three of these (the [Solaris Dynamic Tracing Guide](http://docs.sun.com/app/docs/doc/817-6223), `<sys/dtrace.h>` and `<sys/dtrace\_impl.h>`) then you're ready to start reading the source for purposes of understanding it. If you run into source that you don't understand (and certainly if you believe that you've found a bug), please post to [the DTrace forum](http://forum.sun.com/forum.jsp?forum=211). Not only will one of us answer your question, but there's a good chance that we'll update the comments as well; if you can't understand it, we probably haven't been sufficiently clear in our comments. (If you haven't already inferred it, source readability is very important to us.)

Well, that should be enough to get oriented. If it isn't obvious, we're very excited to be making the source to Solaris available. And hopefully this hors d'oeuvre of DTrace source will hold your appetite until we serve the main course of Solaris source. Bon appetit!

* * *

1 It's unclear what passes for convincing in this regard. Perhaps people just want to be sure that it's not just a bunch of files filled with the results of "`while true; do banner all work and no play makes jack a dull boy ; done`"?

2 The reason that it's CTF and not CCTF is a kind of strange inside joke: the format is so compact, even its acronym is compressed. Yes, this is about as unfunny as the [Language H](http://hopl.murdoch.edu.au/showlanguage.prx?exp=2154&language=Language%20H) humor that we never seem to get sick of...

3 Please don't infer from this that I'm a SPARC bigot; both of my laptops, my desktop and my build machine are all AMD64 boxes running the 64-bit kernel. It's only that the SPARC version of this particular operation happens to be interesting, not that it's interesting because it happens to be SPARC...

4 This brings up a fable of sorts: once, many years ago, there was a system for dynamic instrumentation of SPARC. Unlike DTrace, however, this system was both aggressive and naïve in its instrumentation and, as a result of this unfortunate combination of attributes, couldn't guarantee safety. In particular, the technique used by the function here -- using a DCTI couple to effect instruction picking -- was incorrectly instrumented by the system. When confronted with this in front of a large-ish and highly-technical audience at Sun, the author of said system (who was interviewing for a job at Sun at the time) responded (in a confident, patronizing tone) that the SPARC architecture didn't allow such a construct. The members of the audience gasped, winced and/or snickered: to not deal correctly with this construct was bad enough, but to simply pretend it didn't exist (and to be a prick about it on top of it all) was beyond the pale; the author didn't get the job offer, and the whole episode entered local lore. The moral of the story is this: don't assume that someone asking you a question is an idiot -- especially if the question is about the intricacies of SPARC DCTI couples.

5 It's unclear if this was really a deliberate philosophy or more an accident of history, but it's certainly a Solaris philosophy at any rate. Perhaps its transition from Unix accident to Solaris philosophy was marked by [Jeff](http://blogs.sun.com/bonwick)'s "Riemann sum" comment (and accompanying ASCII art diagram) in `<sys/kstat.h>`?

6 I'm pretty sure that I was joking...

Technorati tags: [DTrace](http://technorati.com/tag/DTrace) [OpenSolaris](http://technorati.com/tag/OpenSolaris) [Solaris](http://technorati.com/tag/Solaris)
