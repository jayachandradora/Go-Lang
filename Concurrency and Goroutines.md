# Concurrency and Goroutines

Concurrency and goroutines are core concepts in Go that allow you to write efficient programs that can run multiple tasks simultaneously. Here's a comprehensive guide to all types of concurrency examples using goroutines in Go.

### 1. **Basic Goroutine Example**

A **goroutine** is a lightweight thread managed by the Go runtime. You create a goroutine by using the `go` keyword followed by a function call.

#### Example 1: Basic Goroutine

```go
package main

import (
	"fmt"
	"time"
)

func sayHello() {
	fmt.Println("Hello, Goroutine!")
}

func main() {
	// Start a goroutine
	go sayHello()

	// Sleep for a short time to allow the goroutine to execute
	time.Sleep(1 * time.Second)

	// Main function continues executing
	fmt.Println("Main function finished.")
}
```

- **Explanation**:
  - The `sayHello` function is executed in a goroutine, running concurrently with the main function.
  - `time.Sleep(1 * time.Second)` ensures the main function waits long enough for the goroutine to finish executing.

---

### 2. **Goroutines with Wait Groups**

A **`sync.WaitGroup`** is used to wait for a collection of goroutines to finish executing.

#### Example 2: Goroutines with WaitGroup

```go
package main

import (
	"fmt"
	"sync"
)

func printMessage(msg string) {
	fmt.Println(msg)
}

func main() {
	var wg sync.WaitGroup

	// Launching multiple goroutines
	wg.Add(3)
	go func() {
		defer wg.Done()
		printMessage("Hello from Goroutine 1")
	}()
	go func() {
		defer wg.Done()
		printMessage("Hello from Goroutine 2")
	}()
	go func() {
		defer wg.Done()
		printMessage("Hello from Goroutine 3")
	}()

	// Wait for all goroutines to finish
	wg.Wait()

	fmt.Println("All Goroutines completed.")
}
```

- **Explanation**:
  - `wg.Add(3)` increments the wait group counter to 3 (for the 3 goroutines).
  - `wg.Done()` is called when each goroutine completes, signaling the wait group.
  - `wg.Wait()` blocks the main function until all goroutines have called `Done()`.

---

### 3. **Channels for Communication Between Goroutines**

Go uses **channels** for goroutines to communicate with each other. A channel is a way to send and receive values between goroutines.

#### Example 3: Goroutines and Channels

```go
package main

import (
	"fmt"
)

func sendData(ch chan string) {
	ch <- "Data from Goroutine"
}

func main() {
	// Create a channel of type string
	ch := make(chan string)

	// Start a goroutine to send data
	go sendData(ch)

	// Receive data from the channel
	data := <-ch
	fmt.Println(data)

	// The main function finishes
	fmt.Println("Main function finished.")
}
```

- **Explanation**:
  - A channel `ch` is created with `make(chan string)`.
  - The goroutine sends a string to the channel.
  - The main function receives the data using `<-ch` and prints it.

---

### 4. **Buffered Channels**

Buffered channels allow sending multiple messages into a channel without blocking, as long as there is space in the buffer.

#### Example 4: Buffered Channels

```go
package main

import (
	"fmt"
)

func sendData(ch chan string) {
	ch <- "Data from Goroutine"
}

func main() {
	// Create a buffered channel with a capacity of 2
	ch := make(chan string, 2)

	// Start a goroutine to send data
	go sendData(ch)

	// Send another piece of data directly in the main goroutine
	ch <- "Data from Main"

	// Receive data from the channel
	fmt.Println(<-ch)
	fmt.Println(<-ch)

	// The main function finishes
	fmt.Println("Main function finished.")
}
```

- **Explanation**:
  - The channel is buffered with a size of 2, allowing two pieces of data to be sent without blocking.
  - Data is sent from both the main function and the goroutine.

---

### 5. **Select Statement**

The `select` statement provides a way to wait on multiple channels. It is like a `switch` statement, but for channels.

#### Example 5: Using `select` with Channels

```go
package main

import (
	"fmt"
	"time"
)

func sendData(ch chan string) {
	ch <- "Data from Goroutine"
}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	// Launch two goroutines
	go sendData(ch1)
	go sendData(ch2)

	// Using select to wait on multiple channels
	select {
	case msg1 := <-ch1:
		fmt.Println("Received from ch1:", msg1)
	case msg2 := <-ch2:
		fmt.Println("Received from ch2:", msg2)
	case <-time.After(1 * time.Second):
		fmt.Println("Timeout!")
	}

	// The main function finishes
	fmt.Println("Main function finished.")
}
```

- **Explanation**:
  - `select` waits on multiple channels. It will execute the case for the first channel that is ready to send or receive data.
  - If none of the channels are ready within the timeout (1 second in this case), the `time.After(1 * time.Second)` case will trigger.

---

### 6. **Goroutines and Mutexes (Data Race Prevention)**

When multiple goroutines access shared resources, you might encounter a **data race**. To prevent this, Go provides a `sync.Mutex` to lock resources.

#### Example 6: Goroutines with Mutexes

```go
package main

import (
	"fmt"
	"sync"
)

var counter int
var mu sync.Mutex

func increment() {
	mu.Lock() // Lock the mutex to prevent data race
	defer mu.Unlock()
	counter++
}

func main() {
	var wg sync.WaitGroup

	// Launch multiple goroutines
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			increment()
		}()
	}

	// Wait for all goroutines to complete
	wg.Wait()

	// Print the final counter value
	fmt.Println("Final counter value:", counter)
}
```

- **Explanation**:
  - `mu.Lock()` locks the mutex before modifying the shared resource (`counter`), and `mu.Unlock()` unlocks it after the modification.
  - This prevents the race condition when multiple goroutines increment the counter.

---

### 7. **Goroutines with Error Handling and Channels**

We can use channels to return results and errors from goroutines.

#### Example 7: Goroutines with Error Handling

```go
package main

import (
	"errors"
	"fmt"
)

func processData(id int, ch chan<- string, errCh chan<- error) {
	if id%2 == 0 {
		ch <- fmt.Sprintf("Data processed by Goroutine %d", id)
	} else {
		errCh <- errors.New(fmt.Sprintf("Error in Goroutine %d", id))
	}
}

func main() {
	dataCh := make(chan string)
	errCh := make(chan error)

	// Launch multiple goroutines
	for i := 1; i <= 5; i++ {
		go processData(i, dataCh, errCh)
	}

	// Receive results or errors from channels
	for i := 0; i < 5; i++ {
		select {
		case msg := <-dataCh:
			fmt.Println(msg)
		case err := <-errCh:
			fmt.Println("Error:", err)
		}
	}

	// The main function finishes
	fmt.Println("Main function finished.")
}
```

- **Explanation**:
  - Two channels (`dataCh` and `errCh`) are used to receive either data or errors.
  - The `select` statement listens for messages from either channel and prints the result accordingly.

---

### 8. **Worker Pool Pattern**

A worker pool allows a set of workers to handle tasks concurrently, and you can control the number of workers.

#### Example 8: Worker Pool

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d\n", id, job)
		results <- job * 2 // Process the job and send result back
	}
}

func main() {
	// Channels for jobs and results
	jobs := make(chan int, 5)
	results := make(chan int, 5)

	var wg sync.WaitGroup

	// Launch 3 workers
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go worker(i, jobs, results, &wg)
	}

	// Send jobs to the workers
	for j := 1; j <= 5; j++ {
		jobs <- j
	}
	close(jobs) // Close the jobs channel after sending all jobs

	// Wait for all workers to finish
	wg.Wait()
	close(results)

	// Collect results
	for result := range results {
		fmt.Println("Result:", result)
	}

	fmt.Println("Main function finished.")
}
```

- **Explanation**:
  - A worker pool is created with 3 workers that process jobs from the `jobs` channel.
  - The `results` channel collects the processed data.
  - The program ensures all workers finish before closing

 the results channel.

---

### Summary of Concurrency Patterns in Go

1. **Goroutines**: Launching concurrent tasks using the `go` keyword.
2. **Wait Groups**: Waiting for multiple goroutines to finish.
3. **Channels**: Communication between goroutines.
4. **Buffered Channels**: Channels with capacity to hold multiple messages.
5. **Select**: Waiting on multiple channels.
6. **Mutexes**: Preventing data races by locking shared resources.
7. **Error Handling in Goroutines**: Using channels to handle errors in goroutines.
8. **Worker Pool**: Managing a pool of workers to process jobs concurrently.

Go's concurrency model is powerful and simple to use, making it a great language for building high-performance and scalable systems.
