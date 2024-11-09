---
title: "Rust after the honeymoon"
date: "2020-10-11"
---

Two years ago, I had a blog entry describing [falling in love with Rust](http://dtrace.org/blogs/bmc/2018/09/18/falling-in-love-with-rust/). Of course, a relationship with a technology is like any other relationship: as novelty and infatuation wears off, it can get on a longer term (and often more realistic and subdued) footing -- or it can begin to fray. So well one might ask: how is Rust after the honeymoon?

By way of answering that, I should note that about a year ago (and a year into my relationship with Rust) [we started Oxide](http://dtrace.org/blogs/bmc/2019/12/02/the-soul-of-a-new-computer-company/). On the one hand, the name was no accident -- we saw Rust playing a large role in our future. But on the other, we hadn't yet started to build in earnest, so it was really more pointed question than assertion: where might Rust fit in a stack that stretches from the bowels of firmware through a hypervisor and control plane and into the lofty heights of REST APIs?

The short answer from [an Oxide perspective](https://www.youtube.com/watch?v=vvZA9n3e5pc) is that Rust has proven to be a really good fit -- remarkably good, honestly -- at more or less all layers of the stack. You can expect much, much more to come from Oxide on this (we intend to open source more or less everything we're building), but for a teaser of the scope, you can see it in the work of Oxide engineers: see [Cliff's blog](http://cliffle.com/blog/), [Adam and Dave's talk on Dropshot](https://twitter.com/ahl/status/1308421852931461126), [Jess on using Dropshot within Oxide](https://blog.jessfraz.com/post/the-automated-cio/), [Laura on Rust macros](https://www.labbott.name/blog/2020/03/30/rust-macros.html), and [Steve Klabnik on why he joined Oxide](https://steveklabnik.com/writing/today-is-my-first-day-at-oxide-computer-company). (Requisite aside: [we're hiring](https://oxide.computer/careers/)!)

So Rust is going really well for us at Oxide, but for the moment I want to focus on more personal things -- reasons that I _personally_ have enjoyed implementing in Rust. These run the gamut: some are tiny but beautiful details that allow me to indulge in the pleasure of the craft; some are much more profound features that represent important advances in the state of the art; and some are bodies of software developed by the Rust community, notable as much for their reflection of who is attracted to Rust (and why) as for the artifacts themselves. It should also be said that I stand by absolutely everything [I said two years ago](http://dtrace.org/blogs/bmc/2018/09/18/falling-in-love-with-rust/); this is not as a replacement for that list, but rather a supplement to it. Finally, this list is highly incomplete; there's a lot to love about Rust and this shouldn't be thought of as any way exhaustive!

### 1\. `no\_std`

When developing for embedded systems -- and especially for the flotilla of microcontrollers that surround a host CPU on the kinds of servers we're building at Oxide -- memory use is critical. Historically, C has been the best fit for these applications just because it so lean: by providing essentially nothing other than the portable assembler that is the language itself, it avoids the implicit assumptions (and girth) of a complicated runtime. But the nothing that C provides reflects history more than minimalism; it is not an elegant nothing, but rather an ill-considered nothing that leaves those who build embedded systems building effectively everything themselves -- and in a language that does little to help them write correct software.

Meanwhile, having been generally designed around modern machines with seemingly limitless resources, higher-level languages and environments are simply too full-featured to fit into (say) tens of kilobytes or into the (highly) constrained environment of a microcontroller. And even where one could cajole these other languages into the embedded use case, it has generally been as a reimplementation, leaving developers on a fork that isn't necessarily benefiting from development in the underlying language.

Rust has taken a different approach: a rich, default standard library but _also_ a first-class mechanism for programs to opt out of that standard library. By marking themselves as [no\_std](https://rust-embedded.github.io/book/intro/no-std.html), programs confine themselves to the functionality found in [libcore](https://doc.rust-lang.org/core/). This functionality, in turn, makes no system assumptions -- and in particular, performs no heap allocations. This is not easy for a system to do; it requires extraordinary discipline by those developing it (who must constantly differentiate between core functionality and standard functionality) and a broad empathy with the constraints of embedded software. Rust is blessed with both, and the upshot is remarkable: a safe, powerful language that can operate in the highly constrained environment of a microcontroller -- with binaries every bit as small as those generated by C. This makes `no\_std` -- as [Cliff has called it](http://cliffle.com/blog/m4vga-in-rust/#on-no-std) -- the killer feature of embedded Rust, without real precedence or analogue.

### 2\. `{:#x?}`

Two years ago, I mentioned that I love `format!`, and in particular the `{:?}` format specifier. What took me longer to discover was `{:#?}`, which formats a structure but also pretty-prints it (i.e., with newlines and indentation). This can be coupled with `{:#x}` to yield `{:#x?}` which pretty-prints a structure _in hex_. So this:

```
    println!("dumping {:#x?}", region);

```

Becomes this:

```
dumping Region {
    daddr: Some(
        0x4db8,
    ),
    base: 0x10000,
    size: 0x8000,
    attr: RegionAttr {
        read: true,
        write: false,
        execute: true,
        device: false,
        dma: false,
    },
    task: Task(
        0x0,
    ),
}

```

My fingers now type `{:#x?}` by default, and hot damn is it ever nice!

### 3\. Integer literal syntax

Okay, another small one: I love the Rust integer literal syntax! In hardware-facing systems, we are often expressing things in terms of masks that ultimately map to binary. It is beyond me why C thought to introduce octal and hexadecimal but not binary in their literal syntax; Rust addresses this gap with the same "`0b`" prefix as found in [some non-standard C compiler extensions](https://gcc.gnu.org/onlinedocs/gcc/Binary-constants.html). Additionally, Rust allows for integer literals to be arbitrarily intra-delimited with an underscore character. Taken together, this allows for a mask consisting of bits 8 through 10 and bit 12 (say) to be expressed as `0b0000\_1011\_1000\_0000` -- which to me is clearer as to its intent and less error prone than (say) `0xb80` or `0b101110000000`.

And as long as we're on the subject of integer literals: I also love that the types (and the suffix that denotes a literal's type) explicitly encode bit width and signedness. Instead of dealing with the implicit signedness and width of `char`, `short`, `long` and `long long`, we have `u8`, `u16`, `u32`, `u64`, etc. Much clearer!

### 4\. DWARF support

Debugging software -- and more generally, the debuggability of software systems -- is in my marrow; it may come as no surprise that one of the things that I personally have been working on is the debugger for a _de novo_ Rust operating system that we're developing. To be useful, debuggers need help from the compiler in the way of type information -- but this information has been historically excruciating to extract, especially in production systems. (Or as [Robert phrased it concisely years ago](http://dtrace.org/blogs/rm/2013/11/14/userland-ctf-in-dtrace/): "the compiler is the enemy.") And while [DWARF](https://en.wikipedia.org/wiki/DWARF) is the _de facto_ standard, it is only as good as the compiler's willingness to supply it.

Given how much debuggability can (sadly) lag development, I wasn't really sure what I would find with respect to Rust, but I have been delighted to discover thorough DWARF support. This is especially important for Rust because it (rightfully) makes extensive use of inlining; without DWARF support to make sense of this inlining, it can be hard to make any sense of the generated assembly. I have been able to use the DWARF information to build some pretty powerful Rust-based tooling -- with much promise on the horizon. (You can see an early study for this work in [Tockilator](https://github.com/oxidecomputer/tockilator).)

### 5\. Gimli and Goblin

Lest I sound like I am heaping too much praise on DWARF, let me be clear that DWARF is historically acutely painful to deal with. The specification (to the degree that one can call it that) is an elaborate mess, and the format itself seems to go out of its way to inflict pain on those who would consume it. Fortunately, the [Gimli crate](https://github.com/gimli-rs/gimli) that consumes DWARF is really good, having made it easy to build DWARF-based tooling. (I have found that whenever I am frustrated with Gimli, I am, in fact, frustrated with some strange pedantry of DWARF -- which Gimli rightfully refuses to paper over.)

In addition to Gimli, I have also enjoyed using [Goblin](https://github.com/m4b/goblin) to consume ELF. ELF -- in stark contrast to DWARF -- is tight and crisp (and the traditional C-based tooling for ELF is quite good), but it was nice nonetheless that Goblin makes it so easy to zing through an ELF binary.

### 6\. Data-bearing enums

Enums -- that is, the "sum" class of algebraic types -- are core to Rust, and give it the beautiful error handling that I described falling in love with two years ago. Algebraic types allow much more than just beautiful error handling, e.g. Rust's ubiquitous [`Option` type](https://doc.rust-lang.org/std/option/), which allows for sentinel values to be eliminated from one's code -- and with it some significant fraction of defects. But it's one thing to use these constructs, and another to begin to develop algebraic types for one's own code, and I have found the ability for enums to optionally bear data to be incredibly useful. In particular, when parsing a protocol, one is often taking a stream of bytes and turning it into one of several different kinds of things; it is really, really nice to have the type system guide how software should consume the protocol. For example, here's an enum that I defined when parsing data from ARM's [Embedded Trace Macrocell signal protocol](https://developer.arm.com/documentation/ihi0014/q/ETMv3-Signal-Protocol/Packet-types?lang=en):

```
#[derive(Copy, Clone, Debug)]
pub enum ETM3Header {
    BranchAddress { addr: u8, c: bool },
    ASync,
    CycleCount,
    ISync,
    Trigger,
    OutOfOrder { tag: u8, size: u8 },
    StoreFailed,
    ISyncCycleCount,
    OutOfOrderPlaceholder { a: bool, tag: u8 },
    VMID,
    NormalData { a: bool, size: u8 },
    Timestamp { r: bool },
    DataSuppressed,
    Ignore,
    ValueNotTraced { a: bool },
    ContextID,
    ExceptionExit,
    ExceptionEntry,
    PHeaderFormat1 { e: u8, n: u8 },
    PHeaderFormat2 { e0: bool, e1: bool },
}

```

That variants can have wildly different types (and that some can bear data while others don't -- and some can be structured, while others are tuples) allows for the type definition to closely match the specification, and helps higher-level software consume the protocol correctly.

### 7\. Ternary operations

In C, the [ternary operator](https://en.wikipedia.org/wiki/%3F:) allows for a terse conditional expression that can be used as an rvalue, e.g.:

```
	x = is_foo ? foo : bar;

```

This is equivalent to:

```
	if (is_foo) {
		x = foo;
	} else {
		x = bar;
	}

```

This construct is particularly valuable when not actually assigning to an lvalue, but when (for example) returning a value or passing a parameter. And indeed, I would estimate that a plurality -- if not a majority -- of my lifetime-use of the ternary operator has been in [arguments to printf](https://github.com/illumos/illumos-gate/blob/4d503977b10eb9d89d0aa5114bb17fd9861d2177/usr/src/cmd/mdb/common/modules/genunix/kmem.c#L2279-L2281).

While Rust has no ternary operator _per se_, it is [expression-oriented](https://en.wikipedia.org/wiki/Expression-oriented_programming_language): statements have values. So the above example becomes:

```
	x = if is_foo { foo } else { bar };

```

That's a bit more verbose than its C equivalent (though I personally like its explicitness), but it really starts to shine when things can marginally more complicated: nested ternary operators get gnarly in C, but they are easy to follow as simple nested if-then-else statements in Rust. And (of course) `match` is an expression as well -- and I found that I often use `match` where I would have used a ternary operator in C, with the added benefit that I am forced to deal with every case. As a concrete example, take this code that is printing a slice of little-endian bytes as an 8-bit, 16-bit, or 32-bit quantity depending on a `size` parameter:

```
    print!("{:0width$x} ",
        match size {
            1 => line[i - offs] as u32,
            2 => u16::from_le_bytes(slice.try_into().unwrap()) as u32,
            4 => u32::from_le_bytes(slice.try_into().unwrap()) as u32,
            _ => {
                panic!("invalid size");
            }
        },
        width = size * 2
    );

```

For me, this is all of the power of the ternary operator, but without its pitfalls!

An interesting footnote on this: Rust once _had_ the C-like ternary operator, [but removed it](https://github.com/rust-lang/rust/issues/1698), as the additional syntax [didn't carry its weight](https://github.com/rust-lang/rfcs/issues/1362#issuecomment-156116312). This pruning in Rust's early days -- the idea that syntax should carry its weight by bringing unique expressive power -- has kept Rust from the fate of languages that suffered from debilitating addictions to new syntax and concomitant complexity overdose; when [there is more than one way to do it](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it) for absolutely everything, a language becomes so baroque as to become [write-only](https://en.wikipedia.org/wiki/Write-only_language)!

### 8\. `paste!`

This is a small detail, but one that took me a little while to find. As I described in my blog entry two years ago, I have historically made heavy use of the C preprocessor. One (arcane) example of this is the [`##` token concatenation operator](https://en.wikipedia.org/wiki/C_preprocessor#Token_concatenation), which I have needed only rarely -- but found essential in those moments. (Here's a [concrete example](https://github.com/illumos/illumos-gate/blob/28de4f3c3209c81f9a96e2019d44a0b9adcb74cb/usr/src/uts/common/dtrace/dtrace.c#L419-L454).) As part of a macro that I was developing, I found that I needed the equivalent for Rust, and was delighted to find David Tolnay's [paste crate](https://github.com/dtolnay/paste). `paste!` was exactly what I needed -- and more testament to both the singular power of Rust's macro system and David's knack for build singularly useful things with it!

### 9\. `unsafe`

A great strength of Rust is its safety -- but something I _also_ appreciate about it is the escape hatch offered via [`unsafe`](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html), whereby certain actions are permitted that are otherwise disallowed. It should go without saying that one should not use `unsafe` without good reason -- but such good reasons can and do exist, and I appreciate that Rust trusts the programmer enough to allow them to take their safety into their own hands. Speaking personally, most of my own uses of `unsafe` have boiled down to accesses to register blocks on a microcontroller: on the one hand, unsafe because they dereference arbitrary memory -- but on the other, safe by inspection. That said, the one time I had to write unsafe code that _actually_ felt dangerous (namely, [in dealing with an outrageously unsafe C library](https://github.com/capstone-rust/capstone-rs/issues/84)), I was definitely in a heightened state of alert! Indeed, my extreme caution around unsafe code reflects how much Rust has changed my disposition: after nearly three decades working in C, I thought I appreciated its level of unsafety, but the truth is I had just become numb to it; to implement in Rust is to eat the fruit from the tree of knowledge of unsafe programs -- and to go back to unsafe code is to realize that you were naked all along!

### 10\. Multi-platform support

When Steve Klabnik joined Oxide, we got not only an important new addition to the team, but a new platform as well: Steve is using Windows as his daily driver, in part because of his own personal dedication to keeping Rust multi-platform. While I'm not sure that anything could drive me personally to use Windows (aside: MS-DOS robbed me of my childhood), I do strongly believe in platform heterogeneity. I love that Rust forces the programmer to really think about implicitly platform-specific issues: Rust refuses to paper over the cracks in computing's foundation for sake of expediency. If this can feel unnecessarily pedantic (can't I just have a timestamp?!), it is in multi-platform support where this shines: software that I wrote just... worked on Windows. (And where it didn't, it was despite Rust's best efforts: when a standard library gives you first-class support to abstract the path separator, you have no one to blame but yourself if you hard-code your own!)

Making and keeping Rust multi-platform is hard work for everyone involved; but as someone who is currently writing Rust for multiple operating systems (Linux, [illumos](https://twitter.com/papertigerss/status/1314210195178881027) and -- thanks to Steve -- Windows) and multiple ISAs (e.g., x86-64, ARM Thumb-2), I very much appreciate that this is valued by the Rust community!

### 11\. `anyhow!` + `RUST\_BACKTRACE`

In my original piece, I praised the error handling of Rust, and that is certainly truer than ever: I simply cannot imagine going back to a world without algebraic types for error handling. The challenge that remained was that there were [several conflicting crates](https://blog.yoshuawuyts.com/error-handling-survey/) building different error types and supporting routines, resulting in some confusion as to best practice. All of this left me -- like many -- simply rolling my own via `Box<dyn Error>`, which works well enough, but it doesn't really help a thorny question: when an error emerges deep within a stack of composed software, where did it _actually_ come from?

Enter David Tolnay (again!) and his handy [`anyhow!` crate](https://docs.rs/anyhow/), which pulls together best practices and ties that into the [improvements in the `std::error::Error` trait](https://github.com/rust-lang/rfcs/blob/master/text/2504-fix-error.md) to yield a crate that is powerful without being imposing. Now, when an error emerges from within a stack of software, we can get a crisp chain of causality, e.g.:

```
readmem failed: A core architecture specific error occurred

Caused by:
    0: Failed to read register CSW at address 0x00000000
    1: Didn't receive any answer during batch processing: [Read(AccessPort(0), 0)]

```

And we can set `RUST\_BACKTRACE` to get a full backtrace where an error actually originates -- which is especially useful when a failure emerges from a surprising place, like this one from a Drop implementation in [probe-rs](https://github.com/probe-rs/probe-rs):

```
Stack backtrace:
   0: probe_rs::probe::daplink::DAPLink::process_batch
   1: probe_rs::probe::daplink::DAPLink::batch_add
   2: ::read_register
   3: probe_rs::architecture::arm::communication_interface::ArmCommunicationInterface::read_ap_register
   4: probe_rs::architecture::arm::memory::adi_v5_memory_interface::ADIMemoryInterface::read_word_32
   5: <probe_rs::architecture::arm::memory::adi_v5_memory_interface::ADIMemoryInterface as probe_rs::memory::MemoryInterface>::read_word_32
   6: ::get_available_breakpoint_units
   7: <core::iter::adapters::ResultShunt<I> as core::iter::traits::iterator::Iterator>::next
   8: <alloc::vec::Vec as alloc::vec::SpecFromIter>::from_iter
   9: ::drop
  10: core::ptr::drop_in_place
  11: main
  12: std::sys_common::backtrace::__rust_begin_short_backtrace
  13: std::rt::lang_start::{{closure}}
  14: core::ops::function::impls::<impl core::ops::function::FnOnce<A> for &F>::call_once
  15: main
  16: __libc_start_main
  17: _start) })

```

### 12\. `asm!`

When writing software at the hardware/software interface, there is inevitably some degree of direct machine interaction that must be done via assembly. Historically, I have done this via [dedicated .s files](https://github.com/illumos/illumos-gate/blob/master/usr/src/uts/intel/dtrace/dtrace_asm.s) -- which are inconvenient, but explicit.

Over the years, compilers added the capacity to drop assembly into C, but the verb here is apt: the resulting assembly was often dropped on its surrounding C like a Looney Tunes anvil, with the interface between the two often being ill-defined, compiler-dependent or both. Rust took this approach at first too, but it suffered from all of the historical problems of inline assembly -- which in Rust's case meant being highly dependent on LLVM implementation details. This in turn meant that it was unlikely to ever become stabilized, which would relegate those who need inline assembly to forever be on nightly Rust.

Fortunately, Amanieu d'Antras took on this gritty problem, and landed [a new `asm!` syntax](https://blog.rust-lang.org/inside-rust/2020/06/08/new-inline-asm.html). The new syntax is a pleasure to work with, and frankly Rust has now leapfrogged C in terms of ease and robustness of integrating inline assembly!

### 13\. String continuations

Okay, this is another tiny one, but meaningful for me and one that took me too long to discover. So first, something to know about me: I am an eighty column purist. For me, this has nothing to do with punchcards or whatnot, but rather with [type readability](https://visualdesignfordh.files.wordpress.com/2014/06/type-readability.pdf), which tends to result in 50-100 characters per line -- and generally about 70 or so. (I would redirect rebuttals to your bookshelf, where most any line of most any page of most any book will show this to be more or less correct.) So I personally embrace the "hard 80", and have found that the rework that that can sometimes require results in more readable, better factored code. There is, however, one annoying exception to this: when programmatically printing a string that is itself long, one is left with much less horizontal real estate to work with! In C, this is a snap: string literals without intervening tokens are automatically concatenated, so the single literal can be made by multiple literals across multiple lines. But in Rust, string literals can span multiple lines (generally a feature!), so splitting the line will also embed the newline and any leading whitespace. e.g.:

```
    println!(
        "...government of the {p}, by the {p}, for the {p},
        shall not perish from the earth.",
        p = "people"
    );

```

Results in a newline and some leading whitespace that represent the structure of the program, not the desired structure of the string:

```
...government of the people, by the people, for the people,
        shall not perish from the earth.

```

I have historically worked around this by using the `concat!` macro to concatenate two (or more) static strings, which works well enough, but looks pretty clunky, e.g.:

```
    println!(
        concat!(
            "...government of the {p}, by the {p}, for the {p}, ",
            "shall not perish from the earth."
        ),
        p = "people"
    );

```

As it turns out, I was really overthinking it, though it took an embarrassingly long time to discover: Rust has support for continuation of string literals! If a line containing a string literal ends in a backslash, the literal continues on the next line, _with the newline and any leading whitespace elided_. This is one of those really nice things that Rust lets us have; the above example becomes:

```
    println!(
        "...government of the {p}, by the {p}, for the {p}, \
        shall not perish from the earth.",
        p = "people"
    );

```

So much cleaner!

### 14\. `\--pretty=expanded` and `cargo expand`

In C -- especially C that makes heavy use of the preprocessor -- the `\-E` option can be invaluable: it stops the compilation after the preprocessing phase and dumps the result to standard output. Rust, as it turns out has an equivalent in the `\--pretty=expanded` unstable compiler option. The output out of this can be a little hard on the eyes, so you want to send it through `rustfmt` -- but the result can be really enlightening as to how things actually work. Take, for example, the following program:

```
fn main() {
    println!("{} has been quite a year!", 2020);
}

```

Here is the `\--pretty=expanded` output:

```
$ rustc -Z unstable-options --pretty=expanded year.rs | rustfmt --emit stdout
#![feature(prelude_import)]
#![no_std]
#[prelude_import]
use std::prelude::v1::*;
#[macro_use]
extern crate std;
fn main() {
    {
        ::std::io::_print(::core::fmt::Arguments::new_v1(
            &["", " has been quite a year!\n"],
            &match (&2020,) {
                (arg0,) => [::core::fmt::ArgumentV1::new(
                    arg0,
                    ::core::fmt::Display::fmt,
                )],
            },
        ));
    };
}

```

As an aside, [`format\_args!`](https://doc.rust-lang.org/std/macro.format_args.html) is really magical -- and a subject that really merits its own blog post from someone with more expertise on the subject. (Yes, this is the Rust blogging equivalent of Chekhov's gun!)

With so many great David Tolnay crates, it's fitting we end on one final piece of software from him: [cargo expand](https://github.com/dtolnay/cargo-expand) is a pleasant wrapper around `\--pretty=expanded` that (among other things) allows you to only dump a particular function.

## The perfect marriage?

All of this is not to say that Rust is perfect; there are certainly some minor annoyances (rustfmt: [looking at you](https://github.com/rust-lang/rustfmt/issues/4306)!), and some forthcoming features that I eagerly await (e.g., safe transmutes, const generics). And in case it needs to be said: just because Rust makes it easier to write robust software doesn't mean that it makes it impossible to write shoddy software!

Dwelling on the imperfections, though, would be a mistake. When getting into a long-term relationship with anything -- be it a person, or a company, or a technology -- it can be tempting to look at its surface characteristics: does this person, or company or technology have _attributes_ that I do or don't like? And those are important, but they can be overemphasized: because things change over time, we sometimes look too much at what things _are_ rather than _what guides them_. And in this regard, my relationship with Rust feels particularly sound: it feels like my values and [Rust's values](https://www.infoq.com/presentations/rust-tradeoffs/) are a good fit for one another -- and that my growing relationship with Rust will be one of the most important of my career!
