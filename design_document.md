			+--------------------+
			|        CS 140      |
			| PROJECT 1: THREADS |
			|   DESIGN DOCUMENT  |
			+--------------------+
				   
### GROUP  

**Team Name:** Broken Pipe

 - Daniel Davis <ddavis44@tntech.edu>
 - Dow Cox <dwcox42@tntech.edu>
 - Gerardo Ramirez <gramirez42@tntech.edu>

### PRELIMINARIES 

> If you have any preliminary comments on your submission, notes for the
> TAs, or extra credit, please give them here.

 - mlfqs-fair passing
  tests/threads/mlfqs-fair-2 and tests/threads/mlfqs-fair-20 are passing since it doesn't check if the nice value is set during the test so all the test is checking if the round robin schedular is fair ie thread time in the round is about equal to the average.

<!--
> Please cite any offline or online sources you consulted while
> preparing your submission, other than the Pintos documentation, course
> text, lecture notes, and course staff.
-->

<!-- geroado-->
<!-- geroado-->
<!-- geroado-->
<!-- geroado-->
			     ALARM CLOCK
			     ===========

### DATA STRUCTURES 

> A1: Copy here the declaration of each new or changed `struct` or
> `struct` member, global or static variable, `typedef`, or
> `enumeration`.  Identify the purpose of each in 25 words or less.

In `thread.h`, added to `struct thread`:
```c
int64_t wake_tick; /* Tick at which thread should wake up from sleep. */
```

In `timer.c`, added global variable:
```c
static struct list sleep_list; /* List of sleeping threads, sorted by wake_tick in ascending order. */
```

### ALGORITHMS 

> A2: Briefly describe what happens in a call to timer_sleep(),
> including the effects of the timer interrupt handler.

When `timer_sleep(ticks)` is called:

1. If `ticks <= 0`, the function returns immediately.
2. The current thread's `wake_tick` is set to `timer_ticks() + ticks`.
3. Interrupts are disabled to safely modify the shared sleep list.
4. The thread is inserted into `sleep_list` in sorted order by `wake_tick` using `list_insert_ordered()`.
5. `thread_block()` is called, which sets the thread's status to `THREAD_BLOCKED` and switches to the next ready thread.
6. When the thread resumes, the original interrupt level is restored.

In `timer_interrupt()`:

1. The global `ticks` counter is incremented.
2. `thread_tick()` is called for scheduler bookkeeping.
3. The handler iterates through the front of `sleep_list`. Since the list is sorted, it checks if the front thread's `wake_tick <= ticks`. If so, the thread is removed and unblocked via `thread_unblock()`. This continues until a thread is found whose `wake_tick` has not yet arrived.

> A3: What steps are taken to minimize the amount of time spent in
> the timer interrupt handler?

The sleep list is maintained in sorted order by `wake_tick`. The interrupt handler only checks threads at the front of the list. Once it encounters a thread whose `wake_tick` is greater than the current tick count, it stops immediately. This makes the wake-up operation O(k) where k is the number of threads waking up at this tick, rather than O(n) for all sleeping threads.

The O(n) insertion cost is paid in `timer_sleep()`, which runs in thread context rather than interrupt context, making this an acceptable trade-off.

### SYNCHRONIZATION 

> A4: How are race conditions avoided when multiple threads call
> timer_sleep() simultaneously?

Interrupts are disabled before modifying the shared `sleep_list`:
```c
enum intr_level old_level = intr_disable();
list_insert_ordered(&sleep_list, &cur->elem, wake_tick_less, NULL);
thread_block();
intr_set_level(old_level);
```

Since PintOS runs on a single-core system, disabling interrupts ensures atomicity. Only one thread can execute at a time, and with interrupts disabled, no preemption can occur during the critical section.

> A5: How are race conditions avoided when a timer interrupt occurs
> during a call to timer_sleep()?

The same interrupt-disabling mechanism prevents this race. Interrupts are disabled before the thread is added to `sleep_list` and before `thread_block()` is called. This ensures the timer interrupt handler cannot fire during list manipulation.

Without this protection, the interrupt handler could call `thread_unblock()` on a thread that hasn't yet called `thread_block()`, causing undefined behavior.

Note: Disabling interrupts is appropriate here because we are coordinating data shared between a kernel thread and an interrupt handler. Since interrupt handlers cannot sleep, they cannot acquire locks, making interrupt disabling the correct synchronization mechanism for this specific case.

### RATIONALE 

> A6: Why did you choose this design?  In what ways is it superior to
> another design you considered?

1. **Eliminates busy-waiting**: The original implementation used a while loop with `thread_yield()`, wasting CPU cycles. This design truly blocks the thread, allowing other threads to use the CPU productively.

2. **Direct use of thread_block/thread_unblock**: We initially considered using semaphores (allocating a semaphore for each sleeping thread and calling `sema_down()`/`sema_up()`). However, this approach caused issues when `sema_up()` was called from the interrupt handler. Using `thread_block()` and `thread_unblock()` directly is simpler and specifically designed for this use case.

3. **Efficient interrupt handler**: The sorted list ensures the interrupt handler runs in O(k) time where k is the number of waking threads, rather than scanning all sleeping threads.

4. **Minimal memory overhead**: By adding only `wake_tick` to `struct thread` and reusing the existing `elem` field for the sleep list, no additional memory allocation is required.

5. **Appropriate synchronization**: Interrupts are disabled only when necessary—specifically when coordinating the shared `sleep_list` between the kernel thread and interrupt handler. This is the one case where disabling interrupts is the correct approach, as interrupt handlers cannot use locks.


<!-- Dow and Dianel-->
<!-- Dow and Dianel-->
<!-- Dow and Dianel-->
<!-- Dow and Dianel-->
<!-- Dow and Dianel-->
			 PRIORITY SCHEDULING
			 ===================

### DATA STRUCTURES 

> B1: Copy here the declaration of each new or changed `struct` or
> `struct` member, global or static variable, `typedef`, or
> enumeration.  Identify the purpose of each in 25 words or less.

```c
struct thread
{
   //added propeties
   int base_priority;
   struct lock *wait_on_lock; /*lock that the tread is waiting for */4%E6j
   struct list donations; /*list of donations*/
   struct list_elem donation_elem; /*Donation list Element */
};
```

> B2: Explain the data structure used to track priority donation.
> Use ASCII art to diagram a nested donation.  (Alternately, submit a
> .png file.)
 - base_priority
  Allows for the priority to be returned to the thread after the donations are done.
 - wait_on_lock
  Allows for nested chains of lock holders to have priority donation
 - donations list and donation_elem  
  These two allows for multiple priority donation. The list of threads that have donated priority to the thread that is waiting on. The list element that allows this thread to be inserted into another thread's donations list.

![nested donation diagram](images/nested%20donation%20diagram.png)
### ALGORITHMS 

> B3: How do you ensure that the highest priority thread waiting for
> a lock, semaphore, or condition variable wakes up first?

> B4: Describe the sequence of events when a call to lock_acquire()
> causes a priority donation.  How is nested donation handled?

 - Once lock acquire is called the, there is a check to see if the hold is held. If it is held, then the thread sets the lock to wait on lock. The running thread inserts its self sorted into the holder thread's donations for multiple donations. Nested donation follows the pointer of wait_on_lock then holder then wait_on_lock etc until it reaches a null wait_on_lock, syncing the priority to the highest one. The thread blocks with sema_down(lock), the yields until the lock becomes available. Once it gets the lock wait_on_lock is set to null, become the lock holder, and continues with the donated donated until the lock is released.

> B5: Describe the sequence of events when lock_release() is called
> on a lock that a higher-priority thread is waiting for.

 - When lock released it removed the donations for that lock, recalculates the highest priority of the chain of threads holding locks, and releases the lock with sema_up(lock).
  
### SYNCHRONIZATION 

> B6: Describe a potential race in thread_set_priority() and explain
> how your implementation avoids it.  Can you use a lock to avoid
> this race?

 - I disabled interrupts so it doesn't recursively call sema down waiting for the lock. The base priority has to be modified, recalculate the priority, if the thread should yield the current thread if it is greater, etc in order atomically. If an interrupts splices into the execution it could crash so interrupts are disabled.
### RATIONALE 

> B7: Why did you choose this design?  In what ways is it superior to
> another design you considered?

 - It implements priority scheduling with donation which before pintos uses a First Come First serve approach. The design prevents priority inversion by donating priority to lower priority threads preventing starvation of higher priority threads. The wait on lock field allow donation to happen in nest chains. 

			   SURVEY QUESTIONS
			   ================

Answering these questions is optional, but it will help us improve the course in future quarters.  Feel free to tell us anything you want--these questions are just to spur your thoughts.  You may also choose to respond anonymously in the course evaluations at the end of the quarter.

> In your opinion, was this assignment, or any one of the three problems
> in it, too easy or too hard?  Did it take too long or too little time?

> Did you find that working on a particular part of the assignment gave
> you greater insight into some aspect of OS design?

> Is there some particular fact or hint we should give students in
> future quarters to help them solve the problems?  Conversely, did you
> find any of our guidance to be misleading?

> Do you have any suggestions for the TAs to more effectively assist
> students, either for future quarters or the remaining projects?

> Any other comments?
