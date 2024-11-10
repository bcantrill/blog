---
title: "The relative performance of C and Rust"
date: "2018-09-29"
---

My blog post on [falling in love with Rust](http://dtrace.org/blogs/bmc/2018/09/18/falling-in-love-with-rust/) got quite a bit of attention -- with many being surprised by what had surprised me as well: the high performance of my naive Rust versus my (putatively less naive?) C. However, [others viewed it as irresponsible](https://www.reddit.com/r/rust/comments/9gzldv/bryan_cantrill_the_observation_deck_falling_in/e6a4ure/) to report these performance differences, believing that these results would be blown out of proportion or worse. The concern is not entirely misplaced: system benchmarking is one of those areas where -- in Jonathan Swift's words from three centuries ago -- "falsehood flies, and the truth comes limping after it."

There are myriad reasons why benchmarking is so vulnerable to leaving truth behind. First, it's deceptively hard to quantify the performance of a system simply because the results are so difficult to verify: the numbers we get must be validated (or rejected) according to the degree that they comport with our expectations. As a result, if our expectations are incorrect, the results can be wildly wrong. To see this vividly, please watch (or rewatch!) Brendan Gregg's excellent (and hilarious) [lightning talk on benchmarking gone wrong](https://www.youtube.com/watch?v=vm1GJMp0QN4&t=17m49s). Brendan recounts his experience dealing with a particularly flawed approach, and it's a talk that I always show anyone who is endeavoring to benchmark the system: it shows how easy it is to get it totally wrong -- and how important it is to rigorously validate results.

Second, even if one gets an entirely correct result, it's really only correct within the context of the system. As we succumb to the temptation of applying a result more universally than this context merits -- as the asterisks and the qualifiers on a performance number are quietly amputated -- a staid truth is transmogrified into a flying falsehood. Worse, some of that context may have been implicit in that the wrong thing may have been benchmarked: in trying to benchmark one aspect of the system, one may inadvertently merely quantify an otherwise hidden bottleneck.

So take all of this as disclaimer: I am not trying to draw large conclusions about "C vs. Rust" here. To the contrary, I think that it is a reasonable assumption that, for any task, a lower-level language can always be made to outperform a higher-level one. But with that said, a pesky fact remains: I reimplemented a body of C software in Rust, and it performed better for the same task; what's going on? And is there anything broader we _can_ say about these results?

To explore this, I ran some statemap rendering tests on SmartOS on a single-socket Haswell server (Xeon E3-1270 v3) running at 3.50GHz. The C version was compiled with GCC 7.3.0 with `-O2` level optimizations; the Rust version was compiled with 1.29.0 with `--release`. All of the tests were run bound to a processor set containing a single core; all were bound to one logical CPU within that core, with the other logical CPU forced to be idle. [cpustat](https://illumos.org/man/1m/cpustat) was used to gather CPU performance counter data, with one number denoting one run with `pic0` programmed to that CPU performance counter. The [input file](https://us-east.manta.joyent.com/bcantrill/public/statemap/pg-zfs.out.gz) (~30MB compressed) contains 3.5M state changes, and in the default config will generate a [~6MB SVG](https://us-east.manta.joyent.com/bcantrill/public/statemap/pg-zfs.svg).

Here are the results for a subset of the counters relating to the cache performance:

<table cellpadding="5" width="100%"><tbody><tr><td><b>Counter</b></td><td align="right"><b>statemap-gcc</b></td><td align="right"><b>statemap-rust</b></td><td align="center"><b>%Δ</b></td></tr><tr bgcolor="#d8dfe5"><td align="left">cpu_clk_unhalted.thread_p</td><td align="right">32,166,437,125</td><td align="right">23,127,271,226</td><td align="center">-28.1%</td></tr><tr bgcolor=""><td align="left">inst_retired.any_p</td><td align="right">49,110,875,829</td><td align="right">48,752,136,699</td><td align="center">-0.7%</td></tr><tr bgcolor="#d8dfe5"><td align="left">cpu_clk_unhalted.ref_p</td><td align="right">918,870,673</td><td align="right">660,493,684</td><td align="center">-28.1%</td></tr><tr bgcolor=""><td align="left">mem_uops_retired.stlb_miss_loads</td><td align="right">8,651,386</td><td align="right">2,353,178</td><td align="center">-72.8%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_uops_retired.stlb_miss_stores</td><td align="right">268,802</td><td align="right">1,000,684</td><td align="center">272.3%</td></tr><tr bgcolor=""><td align="left">mem_uops_retired.lock_loads</td><td align="right">7,791,528</td><td align="right">51,737</td><td align="center">-99.3%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_uops_retired.split_loads</td><td align="right">107,969</td><td align="right">52,745,125</td><td align="center">48752.1%</td></tr><tr bgcolor=""><td align="left">mem_uops_retired.split_stores</td><td align="right">196,934</td><td align="right">41,814,301</td><td align="center">21132.6%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_uops_retired.all_loads</td><td align="right">11,977,544,999</td><td align="right">9,035,048,050</td><td align="center">-24.6%</td></tr><tr bgcolor=""><td align="left">mem_uops_retired.all_stores</td><td align="right">3,911,589,945</td><td align="right">6,627,038,769</td><td align="center">69.4%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_load_uops_retired.l1_hit</td><td align="right">9,337,365,435</td><td align="right">8,756,546,174</td><td align="center">-6.2%</td></tr><tr bgcolor=""><td align="left">mem_load_uops_retired.l2_hit</td><td align="right">1,205,703,362</td><td align="right">70,967,580</td><td align="center">-94.1%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_load_uops_retired.l3_hit</td><td align="right">66,771,301</td><td align="right">33,323,740</td><td align="center">-50.1%</td></tr><tr bgcolor=""><td align="left">mem_load_uops_retired.l1_miss</td><td align="right">1,276,311,911</td><td align="right">105,524,579</td><td align="center">-91.7%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_load_uops_retired.l2_miss</td><td align="right">69,671,774</td><td align="right">34,616,966</td><td align="center">-50.3%</td></tr><tr bgcolor=""><td align="left">mem_load_uops_retired.l3_miss</td><td align="right">2,544,750</td><td align="right">1,364,435</td><td align="center">-46.4%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_load_uops_retired.hit_lfb</td><td align="right">1,393,631,815</td><td align="right">157,897,686</td><td align="center">-88.7%</td></tr><tr bgcolor=""><td align="left">mem_load_uops_l3_hit_retired.xsnp_miss</td><td align="right">435</td><td align="right">526</td><td align="center">20.9%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_load_uops_l3_hit_retired.xsnp_hit</td><td align="right">1,269</td><td align="right">740</td><td align="center">-41.7%</td></tr><tr bgcolor=""><td align="left">mem_load_uops_l3_hit_retired.xsnp_hitm</td><td align="right">820</td><td align="right">517</td><td align="center">-37.0%</td></tr><tr bgcolor="#d8dfe5"><td align="left">mem_load_uops_l3_hit_retired.xsnp_none</td><td align="right">67,846,758</td><td align="right">33,376,449</td><td align="center">-50.8%</td></tr><tr bgcolor=""><td align="left">mem_load_uops_l3_miss_retired.local_dram</td><td align="right">2,543,699</td><td align="right">1,301,381</td><td align="center">-48.8%</td></tr></tbody></table>

So the Rust version is issuing a remarkably similar number of instructions (within less than one percent!), but with a decidedly different mix: just three quarters of the loads of the C version and (interestingly) many more stores. The [cycles per instruction](https://en.wikipedia.org/wiki/Cycles_per_instruction) (CPI) drops from 0.65 to 0.47, indicating much better memory behavior -- and indeed the L1 misses, L2 misses and L3 misses are all way down. The L1 hits as an absolute number are actually quite high relative to the loads, giving Rust a 96.9% L1 hit rate versus the C version's 77.9% hit rate. Rust also lives much better in the L2, where it has half the L2 misses of the C version.

Okay, so Rust has better memory behavior than C? Well, not so fast. In terms of what this thing is actually _doing_: the core of statemap processing is coalescing a large number of state transitions in the raw data into a smaller number of rectangles for the resulting SVG. When presented with a new state transition, it picks the "best" two adjacent rectangles to coalesce based on a variety of properties. As a result, this code spends all of its time constantly updating an efficient data structure to be able to make this decision. For the C version, this is a binary search tree (an AVL tree), but Rust (interestingly) doesn't offer a binary search tree -- and it is instead implemented with a [BTreeSet](https://doc.rust-lang.org/std/collections/struct.BTreeSet.html), which implements a [B-tree](https://en.wikipedia.org/wiki/B-tree). B-trees are common when dealing with on-disk state, where the cost of loading a node contained in a disk block is much, much less than the cost of searching that node for a desired datum, but they are less common as a replacement for an in-memory BST. Rust makes the (compelling) argument that, given the modern memory hierarchy, the cost of getting a line from memory is far greater than the cost of reading it out of a cache -- and B-trees make sense as a replacement for BSTs, albeit with a much smaller value for B. (Cache lines are somewhere between 64 and 512 bytes; disk blocks start at 512 bytes and can be much larger.)

Could the performance difference that we're seeing simply be Rust's data structure being -- per its design goals -- more cache efficient? To explore this a little, I varied the value of the number of rectangles in the statemap, as this will affect both the size of the tree (more rectangles will be a larger tree, leading to a bigger working set) and the number of deletions (more rectangles will result in fewer deletions, leading to less compute time).

The results were pretty interesting:

![](http://dtrace.org/resources/bmc/statemap-perf.png)

A couple of things to note here: first, there are 3.5M state transitions in the input data; as soon as the number of rectangles exceeds the number of states, there is no reason for any coalescing, and some operations (namely, deleting from the tree of rectangles) go away entirely. So that explains the flatline at roughly 3.5M rectangles.

Also not surprisingly, the worst performance for both approaches occurs when the number of rectangles is set at more or less half the number of state transitions: the tree is huge (and therefore has relatively poorer cache performance for either approach) and each new state requires a deletion (so the computational cost is also high).

So far, this seems consistent with the BTreeSet simply being a more efficient data structure. But what is up with that lumpy Rust performance?! In particular there are some strange spikes; e.g., zooming in on the rectangle range up to 100,000 rectangles:

![](http://dtrace.org/resources/bmc/statemap-perf-lumpy-wat.png)

Just from eyeballing it, they seem to appear at roughly logarithmic frequency with respect to the number of rectangles. My first thought was perhaps some strange interference relationship with respect to the B-tree and the cache size or stride, but this is definitely a domain where an ounce of data is worth much more than a pound of hypotheses!

Fortunately, because Rust is static (and we have things like, say, symbols and stack traces!), we can actually just use DTrace to explore this. Take this simple D script, `rustprof.d`:

```
#pragma D option quiet

profile-4987hz
/pid == $target && arg1 != 0/
{
        @[usym(arg1)] = count();
}

END
{
        trunc(@, 10);
        printa("%10@d %A\n", @);
}

```

I ran this against two runs: one at a peak (e.g., 770,000 rectangle) and then another at the adjacent trough (e.g., 840,000 rectangles), demangling the resulting names by sending the the output through [rustfilt](https://github.com/luser/rustfilt). Results for 770,000 rectangles:

```
# dtrace -s ./rustprof.d -c "./statemap --dry-run -c 770000 ./pg-zfs.out" | rustfilt
3943472 records processed, 769999 rectangles
      1043 statemap`<alloc::collections::btree::map::BTreeMap<K, V>>::remove
      1180 statemap`<std::collections::hash::map::DefaultHasher as core::hash::Hasher>::finish
      1208 libc.so.1`memmove
      1253 statemap`<serde_json::read::StrRead<'a> as serde_json::read::Read<'a>>::parse_str
      1320 statemap`<std::collections::hash::map::HashMap<K, V, S>>::remove
      1695 libc.so.1`memcpy
      2558 statemap`statemap::statemap::Statemap::ingest
      4123 statemap`<std::collections::hash::map::HashMap<K, V, S>>::insert
      4503 statemap`<std::collections::hash::map::HashMap<K, V, S>>::get
     26640 statemap`alloc::collections::btree::search::search_tree

```

And now the same thing, but against the adjacent valley of better performance at 840,000 rectangles:

```
# dtrace -s ./rustprof.d -c "./statemap --dry-run -c 840000 ./pg-zfs.out" | rustfilt
3943472 records processed, 839999 rectangles
       971 statemap`<std::collections::hash::map::DefaultHasher as core::hash::Hasher>::write
      1071 statemap`<alloc::collections::btree::map::BTreeMap<K, V>>::remove
      1158 statemap`<std::collections::hash::map::DefaultHasher as core::hash::Hasher>::finish
      1228 libc.so.1`memmove
      1348 statemap`<serde_json::read::StrRead<'a> as serde_json::read::Read<'a>>::parse_str
      1628 libc.so.1`memcpy
      2524 statemap`statemap::statemap::Statemap::ingest
      2948 statemap`<std::collections::hash::map::HashMap<K, V, S>>::insert
      4125 statemap`<std::collections::hash::map::HashMap<K, V, S>>::get
     26359 statemap`alloc::collections::btree::search::search_tree

```

The samples in `btree::search::search_tree` are roughly the same -- but the poorly performing one has many more samples in `HashMap<K, V, S>::insert` (4123 vs. 2948). What is going on? The [HashMap](https://doc.rust-lang.org/std/collections/struct.HashMap.html) implementation in Rust uses [Robin Hood hashing](https://en.wikipedia.org/wiki/Hash_table#Robin_Hood_hashing) and linear probing -- which means that hash maps must be resized when they hit a certain load factor. (By default, the [hash map load factor is 90.9%](https://github.com/rust-lang/rust/blob/70073ec61d0d56bca45b9bd40659bb75799cd273/src/libstd/collections/hash/map.rs#L43-L63).) And note that [I am using hash maps to effectively implement a doubly linked list](https://www.youtube.com/watch?v=aWbGPMxs0AM#t=56m55s): I will have a number of hash maps that -- between them -- will contain the specified number of rectangles. Given that we only see this at particular sizes (and given that the distance between peaks increases exponentially with respect to the number of rectangles), it seems entirely plausible that at some numbers of rectangles, the hash maps will grow large enough to induce quite a bit more probing, but not _quite_ large enough to be resized.

To explore this hypothesis, it would be great to vary the hash map load factor, but unfortunately [the load factor isn't currently dynamic](https://github.com/rust-lang/rust/blob/70073ec61d0d56bca45b9bd40659bb75799cd273/src/libstd/collections/hash/map.rs#L121-L124). Even then, we could explore this by using [with_capacity](https://doc.rust-lang.org/std/collections/struct.HashMap.html#method.with_capacity) to preallocate our hash maps, but the statemap code doesn't necessarily know how much to preallocate because the rectangles themselves are spread across many hash maps.

Another option is to _replace_ our use of HashMap with a different data structure -- and in particular, we can use a [BTreeMap](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html) in its place. If the load factor isn't the issue (that is, if there is something else going on for which the additional compute time in `HashMap<K, V, S>::insert` is merely symptomatic), we would expect a BTreeMap-based implementation to have a similar issue at the same points.

With Rust, conducting this experiment is absurdly easy:

```diff
diff --git a/src/statemap.rs b/src/statemap.rs
index a44dc73..5b7073d 100644
--- a/src/statemap.rs
+++ b/src/statemap.rs
@@ -109,7 +109,7 @@ struct StatemapEntity {
     last: Option,                      // last start time
     start: Option,                     // current start time
     state: Option,                     // current state
-    rects: HashMap<u64, RefCell>,      // rectangles for this entity
+    rects: BTreeMap<u64, RefCell>,     // rectangles for this entity
 }

 #[derive(Debug)]
@@ -151,6 +151,7 @@ use std::str;
 use std::error::Error;
 use std::fmt;
 use std::collections::HashMap;
+use std::collections::BTreeMap;
 use std::collections::BTreeSet;
 use std::str::FromStr;
 use std::cell::RefCell;
@@ -306,7 +307,7 @@ impl StatemapEntity {
             description: None,
             last: None,
             state: None,
-            rects: HashMap::new(),
+            rects: BTreeMap::new(),
             id: id,
         }
     }

```

That's it: because the two (by convention) have the same interface, there is nothing else that needs to be done! And the results, with the new implementation in light blue:

![](http://dtrace.org/resources/bmc/statemap-perf-hashmap.png)

Our lumps are gone! In general, the BTreeMap-based implementation performs a little worse than the HashMap-based implementation, but without as much variance. Which isn't to say that this is devoid of strange artifacts! It's especially interesting to look at the variation at lower levels of rectangles, when the two implementations seem to alternate in the pole position:

![](http://dtrace.org/resources/bmc/statemap-perf-watwat.png)

I don't know what happens to the BTreeMap-based implementation at about ~2,350 rectangles (where it degrades by nearly 10% but then recovers when the number of rectangles hits ~2,700 or so), but at this point, the effects are only academic for my purposes: for statemaps, the default number of rectangles is 25,000. That said, I'm sure that digging there would yield interesting discoveries!

So, where does all of this leave us? Certainly, Rust's foundational data structures perform very well. Indeed, it might be tempting to conclude that, because a significant fraction of the delta here is the difference in data structures (i.e., BST vs. B-tree), the difference in language (i.e., C vs. Rust) doesn't matter at all.

But that would be overlooking something important: part of the reason that using a BST (and in particular, an AVL tree) was easy for me is because we have [an AVL tree implementation](https://illumos.org/man/3LIB/libavl) built as an _intrusive data structure_. This is a pattern we use a bunch in C: the data structure is embedded in a larger, containing structure -- and it is the caller's responsibility to allocate, free and lock this structure. That is, implementing a library as an intrusive data structure _completely sidesteps both allocation and locking_. This allows for an entirely robust arbitrarily embeddable library, and it also makes it really easy for a single data structure to be in many different data structures simultaneously. For example, take ZFS's [zio structure](https://github.com/illumos/illumos-gate/blob/42e00f035d368f958a26818f8991759a087b374d/usr/src/uts/common/fs/zfs/sys/zio.h#L389-L469), in which a single contiguous chunk of memory is on (at least) two different lists and three different AVL trees! (And if that leaves you wondering how anything could possibly be so complicated, see [George Wilson's recent talk explaining the ZIO pipeline](https://www.youtube.com/watch?v=qkA5HhfzsvM).)

Implementing a B-tree this way, however, would be a mess. The value of a B-tree is in the contiguity of nodes -- that is, it is the allocation that is a core part of the win of the data structure. I'm sure it isn't impossible to implement an intrusive B-tree in C, but it would require so much more caller cooperation (and therefore a more complicated and more error-prone interface) that I do imagine that it would have you questioning life choices quite a bit along the way. (After all, a B-tree is a win -- but it's a constant-time win.)

Contrast this to Rust: [intrusive data structures are possible in Rust](https://github.com/Amanieu/intrusive-rs), but they are essentially an anti-pattern. Rust really, really wants you to have complete orthogonality of purpose in your software. This leads you to having multiple disjoint data structures with very clear trees of ownership -- where before you might have had a single more complicated data structure with graphs of multiple ownership. This clear separation of concerns in turn allows for these implementations to be both broadly used and carefully optimized. For an in-depth example of the artful implementation that Rust allows, see [Alexis Beingessner's excellent blog entry on the BTreeMap implementation](http://cglab.ca/~abeinges/blah/rust-btree-case/).

All of this adds up to the existential win of Rust: **powerful abstractions without sacrificing performance**. Does this mean that Rust will always outperform C? No, of course not. But it does mean that you shouldn't be surprised when it does -- and that if you care about performance and you are implementing new software, it is probably past time to give Rust a very serious look!

**Update:** Several have asked if Clang would result in materially better performance; my apologies for not having mentioned that when I did [my initial analysis](https://github.com/joyent/statemap/issues/42), I had included Clang and knew that (at the default rectangles of 25,000), it improved things a little but not enough to approach the performance of the Rust implementation. But for completeness sake, I cranked the Clang-compiled binary at the same rectangle points:

![](http://dtrace.org/resources/bmc/statemap-perf-clang.png)

Despite its improvement over GCC, I don't think that the Clang results invalidate any of my analysis -- but apologies again for not including them in the original post!
