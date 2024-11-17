---
title: "OpenSolaris Sewer Tour"
date: "2005-06-14"
---

Having [OpenSolaris](http://www.opensolaris.org) available today is an odd
phenomenon in some ways. For me, having spent virtually my entire professional
engineering life developing Solaris, opening the source is like having
tourists suddenly flock to your hometown. And as the proud and unabashed local
that I am, I feel compelled to welcome newcomers with a little bit of a
personal tour through the source. But I don't want to lead the kind of bus
tour that you'll get from the [big tour
operators](http://www.solarisinternals.com/si/index.php) (not that you don't
want to take in those sights of course); I'd rather take you on something like
a [sewer
tour](http://www.mtholyoke.edu/courses/rschwart/hist255-s01/mapping-paris/Paris_Sewers_Page3.html)
through the city's underbelly: I want to show you the water mains and the
power grid and the telco infrastructure that make Solaris what it is. Solaris
is the New York City of operating systems: it is what you _don't_ see that
makes what you do see possible -- and like New York City, it's what you don't
see that's _really_ interesting...[^1]

So, with that preamble, let's do a little source spelunking into the core of Solaris. A word of warning: we're going to go deep into the system, and this won't be for everyone. (Or even, perhaps, anyone?) But if you're ready to have your brain bent a bit, grab yourself a cup of coffee, close your office door and read on...

For today, I want to fully explain the context around this somewhat cryptic comment of mine in [`turnstile_block`](https://github.com/illumos/illumos-gate/blob/aaceae985c2e78cadef76bf0b7b50ed887ccb3a6/usr/src/uts/common/os/turnstile.c#L406) in [`usr/src/uts/common/os/turnstile.c`](https://github.com/illumos/illumos-gate/blob/master/usr/src/uts/common/os/turnstile.c):

```
	/*
	 * Follow the blocking chain to its end, willing our priority to
	 * everyone who's in our way.
	 */
	while (t->t_sobj_ops != NULL &&
	    (owner = SOBJ_OWNER(t->t_sobj_ops, t->t_wchan)) != NULL) {
		if (owner == curthread) {
			if (SOBJ_TYPE(sobj_ops) != SOBJ_USER_PI) {
				panic("Deadlock: cycle in blocking chain");
			}

			/*
			 * If the cycle we've encountered ends in mp,
			 * then we know it isn't a 'real' cycle because
			 * we're going to drop mp before we go to sleep.
			 * Moreover, since we've come full circle we know
			 * that we must have willed priority to everyone
			 * in our way.  Therefore, we can break out now.
			 */
			if (t->t_wchan == (void *)mp)
				break;

```

There's quite a story behind this comment -- both in human and engineering
terms. The human side of the story is that this code was added in the last
moments of Solaris 8, in one of the more intense experiences of my engineering
career: a week-long collaboration with [Jeff
Bonwick](http://blogs.sun.com/bonwick) that required so much shared mental
state that he and I both came to call it "the mind-meld." The engineering story
is quite a bit more intricate, and requires a lot more background. It starts
earlier in Solaris 8, when we developed an infrastructure for user-level
priority inheritance -- a mechanism to avoid [priority
inversion](http://www.embedded.com/story/OEG20020321S0023). For those
unfamiliar with the problem of priority inversion, it is this: given three
threads at three different priorities, if the highest priority thread blocks on
a synchronization object held by the lowest priority thread, the middling
priority thread could (in a pure priority preemptive system running on a
uniprocessor) run in perpetuity. This is an inversion, and one mechanism to
solve it is something called [priority
inheritance](http://www.embedded.com/showArticle.jhtml?articleID=20600062).
Under priority inheritance, when one thread is going to block on a lower
priority thread, the higher priority thread _wills_ its priority to the lower
priority thread. That is, the lower priority thread _inherits_ the higher
priority for the duration of the critical section.

Now, in Solaris we have long had priority inheritance for kernel
synchronization primitives -- indeed, this is one of the architectural
differences between SunOS 4.x and Solaris 2.x. And just getting priority
inheritance right for kernel synchronization primitives is nasty: one must
know who owns a lock, and one must know which lock that owner is blocked on
(if any). That is, if a thread is blocking on a lock that itself is owned by a
blocked thread, we need to be able to determine what lock the blocked thread
is blocked on, and which thread owns that lock. In order to avoid missed
wakeups, we must do this _without_ one of the threads in the blocking chain
changing its state. (That is, we don't want to conclude that a thread that
we're blocking on isn't blocked itself, only to have it block before we go to
sleep -- this would create a window for potential priority inversion.) The way
that we traditionally prevent a thread from moving underneath us is to grab
its thread lock (a clever synchronization mechanism that merits its own
lengthy blog entry[\[2\]](#foot2)), but this implies that we're going to be
simultaneously holding two locks of the same type. This is a problem because
-- in any parallel system -- simultaneously holding more than one lock of the
same type is a tricky proposition: there is a possibility of deadlock if the
lock acquisition order is not rigidly defined. This is a particular problem
for thread locks, because both thread locks can be locks on turnstile hash
chains -- and turnstile hash chains can be used by disjoint threads
simultaneously. (A turnstile is the mechanism that we use to put a thread to
sleep.) Jeff solved this particular problem of turnstile deadlocking in an
elegant way; I'll let his comment in
[`turnstile_interlock`](https://github.com/illumos/illumos-gate/blob/aaceae985c2e78cadef76bf0b7b50ed887ccb3a6/usr/src/uts/common/os/turnstile.c#L331)
do the explaining:

```
/*
* When we apply priority inheritance, we must grab the owner's thread lock
* while already holding the waiter's thread lock.  If both thread locks are
* turnstile locks, this can lead to deadlock: while we hold L1 and try to
* grab L2, some unrelated thread may be applying priority inheritance to
* some other blocking chain, holding L2 and trying to grab L1. The most
* obvious solution -- do a lock_try() for the owner lock -- isn't quite
* sufficient because it can cause livelock: each thread may hold one lock,
* try to grab the other, fail, bail out, and try again, looping forever.
* To prevent livelock we must define a winner, i.e. define an arbitrary
* lock ordering on the turnstile locks.  For simplicity we declare that
* virtual address order defines lock order, i.e. if L1 < L2, then the
* correct lock ordering is L1, L2. Thus the thread that holds L1 and
* wants L2 should spin until L2 is available, but the thread that holds
* L2 and can't get L1 on the first try must drop L2 and return failure.
* Moreover, the losing thread must not reacquire L2 until the winning
* thread has had a chance to grab it; to ensure this, the losing thread
* must grab L1 after dropping L2, thus spinning until the winner is done.
* Complicating matters further, note that the owner's thread lock pointer
* can change (i.e. be pointed at a different lock) while we're trying to
* grab it.  If that happens, we must unwind our state and try again.
*
* On success, returns 1 with both locks held.
* On failure, returns 0 with neither lock held.
*/
```

This lock ordering issue is part of what made it difficult to implement
priority inheritance for kernel synchronization objects -- but priority
inheritance for kernel synchronization objects only solves part of the larger
priority inversion problem: in a multithreaded real-time system, one needs
priority inheritance for both kernel-level _and_ user-level synchronization
objects. And _this_ problem -- user-level priority inheritance -- is the
problem that we endeavored to solve in Solaris 8. We assigned an engineer to
solve it, and (with extensive guidance from those of us who best understand
the guts of scheduling and synchronization), the new facility was integrated
in October of 1999.

A few months later -- in December of 1999 -- I was looking at a crash dump
from an operating system panic that a colleague had encountered. It was
immediately clear that this was some sort of defect in our implementation of
user-level priority inheritance, but as I understood the bug, I came to
realize that this was no surface problem: this was a design defect. Here is my
analysis:

```
[ bmc, 12/13/99 ]

The following sequence of events can explain the state in the dump (the arrow
denotes an ordering):

        Thread A (300039c8580)                 Thread B (30003c492a0)
	(executing on CPU 10)                   (executing on CPU 4)
+------------------------------------+ +-------------------------------------+
|                                    | |                                     |
|  Calls lwp_upimutex_lock() on      | |                                     |
|  lock 0xff350000                   | |                                     |
|                                    | |                                     |
|  lwp_upimutex_lock() acquires      | |                                     |
|  upibp->upib_lock                  | |                                     |
|                                    | |                                     |
|  lwp_upimutex_lock(), seeing the   | |                                     |
|  lock held, calls turnstile_block()| |                                     |
|                                    | |                                     |
|  turnstile_block():                | |                                     |
|  - Acquires A's thread lock        | |                                     |
|  - Transitions A into TS_SLEEP     | |                                     |
|  - Drops A's thread lock           | |                                     |
|  - Drops upibp->upib_lock          | |                                     |
|  - Calls swtch()                   | |                                     |
|                                    | |                                     |
|                                    | |                                     |
:                                    : :                                     :

   +----------------------------------------------------------------------+
   | Holder of 0xff350000 releases the lock, explicitly handing it off to |
   | thread A (and thus setting upi_owner to 300039c8580)                 |
   +----------------------------------------------------------------------+

:                                    : :                                     :
|                                    | |                                     |
|  Returns from turnstile_block()    | |                                     |
|                                    | |  Calls lwp_upimutex_lock() on       |
|                                    | |  lock 0xff350000                    |
|                                    | |                                     |
|                                    | |  lwp_upimutex_lock() acquires       |
|                                    | |  upibp->upib_lock                   |
|                                    | |                                     |
|                                    | |  Seeing the lock held (by A), calls |
|                                    | |  turnstile_block()                  |
|  Calls lwp_upimutex_owned() to     | |                                     |
|  check for lock hand-off           | |  turnstile_block():                 |
|                                    | |  - Acquires B's thread lock         |
|  lwp_upimutex_owned() attempts     | |  - Transitions B into TS_SLEEP,     |
|  to acquire upibp->upib_lock       | |    setting B's wchan to upimutex    |
|                                    | |    corresponding to 0xff350000      |
|  upibp->upib_lock is held by B;    | |  - Attempts to promote holder of    |
|  calls into turnstile_block()      | |    0xff350000 (Thread A)            |
|  through mutex_vector_enter()      | |  - Acquires A's thread lock         |
|                                    | |  - Adjusts A's priority             |
|  turnstile_block():                | |  - Drops A's thread lock            |
|                          <--------------+                                  |
|  - Acquires A's thread lock        | |  - Drops B's thread lock            |
|  - Attempts to promote holder of   | |                                     |
|    upibp->upib_lock (Thread B)     | |                                     |
|  - Acquires B's thread lock        | |  - Drops upibp->upib_lock           |
|  - Adjusts B's priority            | |                                     | 
|  - Drops B's thread lock           | |                                     |
|  - Seeing that B's wchan is not    | |                                     |
|    NULL, attempts to continue      | |                                     |
|    priority inheritance            | |                                     |
|  - Calls SOBJ_OWNER() on B's wchan | |                                     |
|  - Seeing that owner of B's wchan  | |                                     |
|    is A, panics with "Deadlock:    | |                                     |
|    cycle in blocking chain"        | |                                     |
|                                    | |                                     |
+------------------------------------+ +-------------------------------------+

As the above sequence implies, the problem is in turnstile_block():

        THREAD_SLEEP(t, &tc->tc_lock);
        t->t_wchan = sobj;
        t->t_sobj_ops = sobj_ops;
        ...
        /*
         * Follow the blocking chain to its end, or until we run out of
         * inversions, willing our priority to everyone who's in our way.
         */
        while (inverted && t->t_sobj_ops != NULL &&
            (owner = SOBJ_OWNER(t->t_sobj_ops, t->t_wchan)) != NULL) {
                ...
        }
(1) --> thread_unlock_nopreempt(t);
        /*
         * At this point, "t" may not be curthread. So, use "curthread", from
         * now on, instead of "t".
         */
        if (SOBJ_TYPE(sobj_ops) == SOBJ_USER_PI) {
(2) -->         mutex_exit(mp);
                ...

We're dropping the thread lock of the blocking thread (at (1)) before we drop
the upibp->upib_lock at (2).  From (1) until (2) we are violating one of
the invariants of SOBJ_USER_PI locks:  when sleeping on a SOBJ_USER_PI lock,
_no_ kernel locks may be held; any held kernel locks can yield a deadlock
panic.
```

Once I understood the problem, it was disconcertingly easy to reproduce: in a
few minutes I was able to bang out a test case that panicked the system in the
same manner as seen in the crash dump.[^3] While I had some ideas
on how to fix this, the late date in the release and the seriousness of the
problem prompted me to call Jeff at home to discuss.[^4] As Jeff
and I discussed the problem, we couldn't seem to come up with a potential
solution that didn't introduce a new problem. Indeed, the more we talked about
the problem the harder it seemed. The essence of the problem is this: for
user-level locks, we normally keep track of the state associated with the lock
(for example, whether or not there's a waiter) at user-level -- and that
information is considered purely advisory by the kernel. (There are several
situations in which the waiters bit can't be trusted, and the kernel knows not
to trust it in these situations.) To implement priority inheritance for
user-level locks, however, one must become much more precise about ownership
-- the ownership must be tracked the same way we track ownership for
kernel-level synchronization primitives. That is, when we're doing the
complicated thread lock dance that I described above, we can't be doing loads
from user-level memory to determine ownership.[^5] Here's the nasty
implication of this: the kernel-level state tracking the ownership of the
user-level lock must itself be protected by a lock, and that (in-kernel) lock
must itself implement priority inheritance to avoid a potential inversion.
This leads us to a deadlock that we did not predict: the in-kernel lock must
be acquired _and_ dropped to _both_ acquire the user-level lock _and_ to drop
it. That is, there are conditions in which a thread owns the in-kernel lock
and wants the user-level lock, and there are conditions in which a thread owns
the user-level lock and wants the in-kernel lock. (The `upib_lock` is the
in-kernel lock that implements this. Knowing this, you might want to pause to
reread my description of the race condition above.)

The above bug captures one manifestation of this, but Jeff and I began to
realize that there must be another manifestation lurking: if one were blocking
on the in-kernel lock when the false deadlock was discovered, the kernel would
clearly panic. But what if one were blocking on the user-level lock when the
false deadlock was discovered? We quickly determined (and a test case
confirmed) that in this case, the attempt to acquire the user-level would
(erroneously) return EDEADLK. That is, in this case, the code saw that the
"deadlock" was induced by a user-level synchronization primitive, and therefore
assumed that it was an application-induced deadlock -- a bug in the
application. So in this failure mode, a correct program would have one of its
calls to
[`pthread_mutex_lock`](http://www.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_lock.html)
erroneously fail -- a failure mode even more serious than a panic because it
could easily lead to the application corrupting its data.

So how to solve these problems? We found this to be a hard problem because we
kept trying to find a way to avoid that in-kernel lock. (I have presented the
in-kernel lock as a natural constraint on the problem, but that was a
conclusion that we only came to with tremendous reluctance.) Whenever one of us
came up with some scheme to avoid the lock, the other would find some window
invalidating the scheme. After exhausting ourselves on the alternatives, we
were forced to the conclusion that the in-kernel lock _was_ a constraint on the
problem -- and our focus switched from avoiding the situation to detecting it.

There are two cases to detect: the panic case and the false deadlock case. The
false deadlock case is actually pretty easy to detect and handle, because we
always find ourselves at the end of the blocking chain -- and we always find
that the lock that we own that induced the deadlock is the in-kernel lock
passed as a parameter to
[`turnstile_block`](https://github.com/illumos/illumos-gate/blob/aaceae985c2e78cadef76bf0b7b50ed887ccb3a6/usr/src/uts/common/os/turnstile.c#L406)
(`mp`). Because we know that we have willed our priority to the entire blocking
chain, we can just detect this and break out.

The panic case is nastier to deal with. To remind, this case is that the
thread owns the user-level synchronization object, and is blocking trying to
acquire the in-kernel lock. We might wish to handle this case in a similar
way, by adding a case to check if the deadlock ends in the current thread and
the last synchronization object in the blocking chain is a user-level
synchronization object, then it's a false deadlock. (That is, handle this case
by a more general handling of the above case.) This is simple, but it's also
wrong: it ignores the possibility of an actual application-level deadlock
(that is, an application bug), in which EDEADLK _must_ be returned. To deal
with this case, we observe that if a blocking chain runs from in-kernel
synchronization objects to user-level synchronization objects, we know that
we're in this case. Since we know that we've caught another thread in code in
which they can't be preempted, we can fix this by busy-waiting until the lock
changes and then restarting the priority inheritance dance. Here's the code to
handle this case:[^6]

```
	/*
	 * We now have the owner's thread lock.  If we are traversing
	 * from non-SOBJ_USER_PI ops to SOBJ_USER_PI ops, then we know
	 * that we have caught the thread while in the TS_SLEEP state,
	 * but holding mp.  We know that this situation is transient
	 * (mp will be dropped before the holder actually sleeps on
	 * the SOBJ_USER_PI sobj), so we will spin waiting for mp to
	 * be dropped.  Then, as in the turnstile_interlock() failure
	 * case, we will restart the priority inheritance dance.
	 */
	if (SOBJ_TYPE(t->t_sobj_ops) != SOBJ_USER_PI &&
	    owner->t_sobj_ops != NULL &&
	    SOBJ_TYPE(owner->t_sobj_ops) == SOBJ_USER_PI) {
		kmutex_t *upi_lock = (kmutex_t *)t->t_wchan;

		ASSERT(IS_UPI(upi_lock));
		ASSERT(SOBJ_TYPE(t->t_sobj_ops) == SOBJ_MUTEX);

		if (t->t_lockp != owner->t_lockp)
			thread_unlock_high(owner);
		thread_unlock_high(t);
		if (loser)
			lock_clear(&turnstile_loser_lock);

		while (mutex_owner(upi_lock) == owner) {
			SMT_PAUSE();
			continue;
		}

		if (loser)
			lock_set(&turnstile_loser_lock);
		t = curthread;
		thread_lock_high(t);
		continue;
	}
```

Once these problems were fixed, we thought we were done. But further stress
testing revealed that an even darker problem lurked -- one that I honestly
wasn't sure that we would be able to solve. I'm actually going to leave the
description of this problem (and its solution) for another day -- but for an
embarrassing reason: when I went into the code to write this blog entry, I
discovered (to my horror) that an addition was made to
[`turnstile.c`](https://github.com/illumos/illumos-gate/blob/master/usr/src/uts/common/os/turnstile.c)
with an apparent disregard for the subtle intricacies of this subsystem.
(Indeed, fear of such a hasty change is exactly why we went to such pains to
comment these intricacies in the first place.) That is, this darker bug has
been reintroduced in a new manifestation. The bug will be virtually impossible
to hit, but there is no question that it's a bug. I'm not going to describe it
here, but if you find it (and you can explain it fully), and you're in the job
market, we'll fly you out for an interview. (I'm not joking.) Everything you
need to know to find the bug is in
[`turnstile.c`](https://github.com/illumos/illumos-gate/blob/master/usr/src/uts/common/os/turnstile.c);
if you can find it, pretty soon you'll be leading your own sewer tour...

* * *

[^1]: New York City owes a particularly large debt to what isn't normally seen: its [Manhattan schist](http://www.nycgovparks.org/sub_your_park/historical_signs/hs_historical_sign.php?id=10763) is an extremely hard variant of granite, without which its buildings' towering heights would be impossible.

[^2]: Calling Joe Eykholt...

[^3]: This is one of the most gratifying feelings in software engineering: analyzing a failure postmortem, discovering that the bug should be easily reproduced, writing a test case testing the hypothesis, and then watching the system blow up just as you predicted. Nothing quite compares to this feeling; it's the software equivalent of the [walk-off home run](http://en.wikipedia.org/wiki/Walk_off_home_run).

[^4]: Jeff was at home because it was a Sunday. And Jeff was up because it was very late on Sunday -- so much so that it was actually early on Monday morning.

[^5]: I won't go into details about why this is the case -- it's left as an exercise to the reader -- but suffice it to say it's one of the many reasons that we have joked about requiring a license to grab thread lock.

[^6]: The code dealing with `turnstile_loser_lock` didn't actually exist when we wrote this case -- that was added to deal with (yet) another problem we discovered as a result of our four day mind-meld. This problem deserves its own blog entry, if only for the great name that Jeff gave it: dueling losers.

