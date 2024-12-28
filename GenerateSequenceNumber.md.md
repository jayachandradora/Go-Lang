# Generate Sequence Number

Let's break down the given Java code and convert it to Go language while maintaining the logic. I'll also explain the functionality and share a similar use case in Go.

### Problem Understanding:
The task is to generate numbers in sequence, with each thread responsible for printing numbers at certain intervals. Specifically, there are three threads, and the sequence of numbers should be printed in a cyclic order among them. The logic ensures that the sequence prints numbers in a round-robin manner.

In Java:
1. Three threads are created, each responsible for printing numbers.
2. A `NumbersGenerator` class holds the state (`number`), which tracks the sequence and synchronizes the printing of numbers using `wait()` and `notifyAll()`.
3. Each thread prints numbers corresponding to its turn based on the modulo of `number % numberOfThreads`.

### Go Lang Conversion:

In Go, we can use **goroutines** and **channels** for concurrent execution. Go has a simpler concurrency model with `goroutines`, and we can use `sync.Mutex` or `sync.Cond` to manage synchronization between threads (goroutines).

### Converted Go Code:

```go
package main

import (
	"fmt"
	"sync"
)

const (
	TotalNumberInSequence = 20
	TotalNumberOfThreads  = 3
)

type NumbersGenerator struct {
	mu              sync.Mutex
	number          int
	numberOfThreads int
	totalNumbers    int
	cond            *sync.Cond
}

func NewNumbersGenerator(numberOfThreads, totalNumbers int) *NumbersGenerator {
	mu := sync.Mutex{}
	cond := sync.NewCond(&mu)
	return &NumbersGenerator{
		mu:              mu,
		numberOfThreads: numberOfThreads,
		totalNumbers:    totalNumbers,
		cond:            cond,
	}
}

func (ng *NumbersGenerator) printNumbers(threadIndex int) {
	ng.mu.Lock()
	defer ng.mu.Unlock()

	for ng.number < ng.totalNumbers {
		// Check if it's this thread's turn to print
		for ng.number%ng.numberOfThreads != threadIndex {
			ng.cond.Wait() // Wait until it's this thread's turn
		}

		// Print the number and increment
		fmt.Printf("Thread-%d: %d\n", threadIndex+1, ng.number)
		ng.number++

		// Notify other threads that it's their turn
		ng.cond.Broadcast()
	}
}

func main() {
	// Create the number generator
	numbersGenerator := NewNumbersGenerator(TotalNumberOfThreads, TotalNumberInSequence)

	// Create and start goroutines for each thread
	var wg sync.WaitGroup

	for i := 0; i < TotalNumberOfThreads; i++ {
		wg.Add(1)
		go func(threadIndex int) {
			defer wg.Done()
			numbersGenerator.printNumbers(threadIndex)
		}(i)
	}

	// Wait for all goroutines to finish
	wg.Wait()
}
```

### Explanation of Go Code:

1. **NumbersGenerator struct**: 
   - This struct holds the state (`number`), the number of threads (`numberOfThreads`), the total numbers to be printed (`totalNumbers`), and a condition variable (`cond`) to coordinate goroutines.
   - `sync.Mutex` is used to protect the state (`number`) from concurrent modification, ensuring only one goroutine can modify it at a time.

2. **NewNumbersGenerator function**:
   - This function creates and returns a `NumbersGenerator` instance, initializing the mutex and condition variable.

3. **printNumbers function**:
   - Each goroutine (thread) calls this method to print numbers.
   - The goroutine first locks the mutex to ensure exclusive access to the `number`.
   - The `for` loop ensures that the sequence of numbers is printed until the total count is reached.
   - The goroutine waits until it's its turn to print the number. This is done by checking `ng.number % ng.numberOfThreads == threadIndex`, where `threadIndex` is the index of the current goroutine.
   - If it’s not this thread’s turn, it calls `ng.cond.Wait()`, which releases the mutex and blocks until notified.
   - Once the goroutine prints the number, it calls `ng.cond.Broadcast()` to notify all waiting goroutines that it's their turn.

4. **Main function**:
   - A `sync.WaitGroup` is used to ensure that the main function waits for all goroutines to complete.
   - Three goroutines are started, each with a different `threadIndex` (0, 1, 2), corresponding to the three threads.
   - The program waits for all goroutines to complete using `wg.Wait()`.

---

### Similar Use Case in Go:

Let’s say we want to modify the program to **print numbers in reverse order** by using goroutines.

#### Go Code Example: Printing Numbers in Reverse Order

```go
package main

import (
	"fmt"
	"sync"
)

const (
	TotalNumberInSequence = 20
	TotalNumberOfThreads  = 3
)

type NumbersGenerator struct {
	mu              sync.Mutex
	number          int
	numberOfThreads int
	totalNumbers    int
	cond            *sync.Cond
}

func NewNumbersGenerator(numberOfThreads, totalNumbers int) *NumbersGenerator {
	mu := sync.Mutex{}
	cond := sync.NewCond(&mu)
	return &NumbersGenerator{
		mu:              mu,
		numberOfThreads: numberOfThreads,
		totalNumbers:    totalNumbers,
		cond:            cond,
	}
}

func (ng *NumbersGenerator) printNumbers(threadIndex int) {
	ng.mu.Lock()
	defer ng.mu.Unlock()

	for ng.number < ng.totalNumbers {
		// Check if it's this thread's turn to print
		for ng.number%ng.numberOfThreads != threadIndex {
			ng.cond.Wait() // Wait until it's this thread's turn
		}

		// Print the number and increment
		fmt.Printf("Thread-%d: %d\n", threadIndex+1, ng.number)
		ng.number++

		// Notify other threads that it's their turn
		ng.cond.Broadcast()
	}
}

func main() {
	// Create the number generator
	numbersGenerator := NewNumbersGenerator(TotalNumberOfThreads, TotalNumberInSequence)

	// Create and start goroutines for each thread
	var wg sync.WaitGroup

	for i := 0; i < TotalNumberOfThreads; i++ {
		wg.Add(1)
		go func(threadIndex int) {
			defer wg.Done()
			numbersGenerator.printNumbers(threadIndex)
		}(i)
	}

	// Wait for all goroutines to finish
	wg.Wait()
}
```

#### Explanation:
- This program is very similar to the original one, but now the **logic** can be adjusted to print numbers in reverse order. However, the primary logic remains the same for controlling concurrency. The modification of number printing order could be achieved based on how the sequence is generated or with conditions.

### Summary:
- **Goroutines**: We use goroutines in Go to execute tasks concurrently, similar to threads in Java.
- **Condition Variables**: In Go, the `sync.Cond` type is used for waiting and notifying between goroutines, similar to `wait()` and `notifyAll()` in Java.
- **Mutex**: `sync.Mutex` is used to ensure that only one goroutine can access the shared `number` state at any given time.
- **`sync.WaitGroup`**: This is used to wait for all goroutines to complete before exiting the program.

This Go implementation provides a robust and efficient way to handle concurrency and synchronization, similar to the original Java example.
