The processing model defines how the DBMS executes a query plan. Each processing model has different trade-offs for different workloads (e.g. lots of single record changes vs few large operations). There are three main approaches:
- Iterator model
- Vectorised (batch) model
- Materialisation model
# Iterator model
The **iterator model** has each query plan operator maintain an internal state and implement three functions:
- `open()` initialises the operators internal state
- `next()` returns the next result tuple or a null marker if there are no more tuples
- `close()` cleans up all allocated resources

This allows any iterator to be input into any other as they all implement the same interface.

The entire plan is started by calling `open()` on the root operator, which is then forwarded through the plan by the operators themselves. Control then returns to the query processor. The root is requested to produce its `next()` record, which is again forwarded through the plan as needed, and as soon as the next result record is produced control returns to the query processor again.

This approach is used in almost every DBMS.

The query processor would use the following routine to evaluate a query plan $q$:
```python
q.open()
r = q.next()
while r != EOF:
	emit(r)
	r = q.next()
q.close()
```
This makes it easy to control the number of output records (e.g. `LIMIT`).

The iterator operation allows for tuple pipelining as each tuple is processed through as many operators as possible before retrieving the next tuple. This reduces memory requirements and response time as each chunk of input propagates to the output immediately. Some operators may block until children emit all of their tuples, e.g. sorting, hash join, grouping and duplicate elimination over unsorted inputs, subqueries, with this data typically being buffered on disk.
## Example operators
An example implementation for selection $\sigma_p$:
```python
def open():
	child.open()

def close():
	child.close()

def next():
	while True:
		r = child.next()
		if r == EOF:
			return EOF
		if p(r):
			return r
```

And one for a heap scan (which would be a leaf of the query plan):
```python
def open():
	self.heap = open heap file for this relation
	self.current_page = heap.first_page()
	self.current_slot = current_page.first_slot()

def close():
	self.heap.close()

def next():
	if current_page == NULL:
		return EOF
	current = tuple at (self.current_page, self.current_slot) # tuple to be returned
	self.current_slot = self.current_slot.advance() # advance slot for next call
	if self.current_slot == NULL: # advance to next page, first slot
		self.current_page = self.current_page.advance()
		if self.current_page != NULL:
			self.current_slot = self.current_page.first_slot()
	return current
```

For a [[W4N3 - Joins#Nested loop joins|nested loop join]]:
```python
def open():
	self.left_child.open()
	self.right_child.open()
	self.r = left_child.next()

def close():
	self.left_child.close()
	self.right_child.close()

def next():
	while self.r != EOF:
		while True
			s = right_child.next()
			if s == EOF:
				break
			if p(r, s):
				return r, s
		# reset inner join input
		self.right_child.close()
		self.right_child.open()
		self.r = self.left_child.next()
	return EOF
```

For a 2-pass [[W4N1 - Sorting#External sorting|external merge sort]]:
```
def open():
	child.open()
	generate sorted runs on disk by calling child.next()
	load first page of each run into input buffer

def next():
	output = min tuple across all buffers
	if min tuple was the last one in a buffer:
		fetch next page from that run
	return output

def close():
	child.close()
	deallocate runs files
```
## Pros and cons
Pros:
- Simple interface
- Easy combination of operators
- Provides first results of query quickly
Cons:
- Next is called for every single tuple and operator
- Virtual calls via function pointers which degrades branch prediction performance in modern CPUs
- Poor code locality and complex bookkeeping as each operator must maintain state to know where to resume
# Vectorisation model
The **vectorisation model** is similar to the iterator model, but each operator emits a batch of tuples instead of a single tuple. An operators internal loop processes multiple tuples at once, possibly using vectorised (SIMD) instructions to process batches of tuples. Vectorisation is ideal for analytical workflows.
# Materialisation model
The **materialisation model** has each operator process its input all at once then emit an output as a single result. This is bottom-up plan processing as operators have data pushed to them instead of pulled up by them. It leads to better code and data locality.

It is good for transactional workloads which typically only access a small number of tuples at a time, but is not good for analytical workflows with large intermediate results.