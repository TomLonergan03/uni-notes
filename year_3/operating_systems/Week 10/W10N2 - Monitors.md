Monitors address the key usability issues with [[W10N1 - Semaphores|semaphores]]. Monitors are a programming language construct which controls access to shared data. Every method automatically acquires a lock on entry and releases it on exit. 

Monitors also have a condition wait method, which allows a thread inside the monitor to release the monitor until a condition is met (e.g. if a producer thread enters a monitor and finds the queue is full, it cannot simply sleep because then no consumer can get in to empty the queue).
```Java
Monitor producer_consumer {
	buffer resources[N];
	condition not_full, not_empty;

	void produce(resource x) {
		if (/*resources is full*/)
			cond_wait(not_full);
		// insert X into resources
		cond_signal(not_empty);
	}
	
	void consume(resource *x) {
		if (/*resources is empty*/)
			cond_wait(not_empty);
		*x = /* get resource from resources */;
		cond_signal(not_full);
	}
}
```
A `cond_wait`ing thread is placed in a queue for whatever condition is being waited on.
