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
  tests/threads/mlfqs-fair-2 and tests/threads/mlfqs-fair-20 are passing since it doesn't check if the nice value is set during the test so all the test is checking if the round robin schedular  is fair.

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

### ALGORITHMS 

> A2: Briefly describe what happens in a call to timer_sleep(),
> including the effects of the timer interrupt handler.

> A3: What steps are taken to minimize the amount of time spent in
> the timer interrupt handler?

### SYNCHRONIZATION 

> A4: How are race conditions avoided when multiple threads call
> timer_sleep() simultaneously?

> A5: How are race conditions avoided when a timer interrupt occurs
> during a call to timer_sleep()?

### RATIONALE 

> A6: Why did you choose this design?  In what ways is it superior to
> another design you considered?

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

> B5: Describe the sequence of events when lock_release() is called
> on a lock that a higher-priority thread is waiting for.

### SYNCHRONIZATION 

> B6: Describe a potential race in thread_set_priority() and explain
> how your implementation avoids it.  Can you use a lock to avoid
> this race?

 - I disabled interrupts so it doesn't recursively call sema down waiting for the lock. The base priority has to be modified, recalculate the priority, if the thread should yield the current thread if it is greater, etc in order atomically. If an interrupts splices into the execution it could crash so interrupts are disabled.
### RATIONALE 

> B7: Why did you choose this design?  In what ways is it superior to
> another design you considered?
It implements priority scheduling with donation which before pintos uses a First Come First serve approach. The design prevents priority inversion by donating priority to lower priority threads preventing starvation of higher priority threads. The wait on lock field allow donation to happen in nest chains. 

			   SURVEY QUESTIONS
			   ================

Answering these questions is optional, but it will help us improve the
course in future quarters.  Feel free to tell us anything you
want--these questions are just to spur your thoughts.  You may also
choose to respond anonymously in the course evaluations at the end of
the quarter.

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
