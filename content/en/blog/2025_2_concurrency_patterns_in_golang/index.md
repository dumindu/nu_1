---
title: Concurrency Patterns in Golang
slug: "concurrency-patterns-in-golang"
publishDate: 2025-11-26T00:00:00+08:00
---

## Essentials

### 1. OS Threads Vs Goroutines

- OS Threads: Managed by the kernel, fixed stack size 1MB (But Modern Linux uses NPTL/ Native POSIX Thread Library, 256KB or less), context switching between OS threads requires a costly transition from user space to kernel space, forcing the operating system to interrupt its core work.
- Goroutines: Managed by the Go runtime scheduler in user space, ~2KB dynamic stack size, can multiplex thousands of goroutines onto a very few OS threads, context switching is lightning-fast as it's managed in user space. Concurrently executing functions.

### 2. Synchronization Patterns

- General
  - Locking
    - Ensures mutual exclusion using primitives like Mutexes, RWMutexes, Semaphores, and critical sections. Only one thread accesses a critical section of shared data at a time. CPUs leverage sophisticated OS schedulers to put waiting threads to sleep and switch tasks efficiently.
    - Risk of deadlock and livelock; a thread holding a lock can be preempted by the OS scheduler, potentially blocking all other waiting threads for an unpredictable duration.

  - Lock-Free
    - Avoids blocking the entire system if one thread is delayed, using low-level hardware-intrinsic atomic operations like Compare-And-Swap (CAS) or Load-Link/Store-Conditional (LL/SC). Threads optimistically retry operations in a "CAS loop" if a conflict occurs.
    - Difficult to design and implement correctly; susceptible to issues like the ABA problem. Can perform worse than locks under heavy contention due to wasted work from retries; involves complex memory management strategies.

- Go: Hybrid Approach

  > Idiomatic Go emphasizes "share memory by communicating" via channels, using the CSP Communicating Sequential Processes) model to abstract away the complexity of low-level lock management from developers. Locking synchronization is managed entirely in user space to ensure channels are safe.

  - Lock-Based Primitives
    - Mutexes & RWMutexes: Go provides the standard `sync.Mutex` (mutual exclusion) and `sync.RWMutex` (reader/writer lock) for protecting critical sections of complex data structures.
    - Channels (`chan`): The preferred idiomatic tool for concurrency management, often used to implement simple binary or counting semaphores via their buffer capacity.
    - Semaphores (`golang.org/x/sync/semaphore`): For formal, advanced use cases requiring features like context cancellation or weighted acquisition.
    - Wait Groups (`sync.WaitGroup`): Used when the main goroutine needs to block until a collection of other goroutines have all completed their tasks.
    - Condition Variables (`sync.Cond`): Used to coordinate goroutines that need to wait for a shared state to become true (Ex. a buffer going from empty to having items).
    - Once (`sync.Once`): Guarantees that a function is executed only once across the entire program runtime, typically used for lazy initialization (singletons).
    - Concurrent Map (`sync.Map`): A specialized map optimized for a cache-like access pattern (write once, read many times), offering better performance than a standard map with an RWMutex in specific scenarios.
    - Pool (`sync.Pool`): A mechanism to temporarily store and reuse allocated objects to reduce garbage collection overhead, managing access concurrently.
    - Error Group (`golang.org/x/sync/errgroup`): Simplifies running multiple goroutines that return errors. If any error occurs, the context is canceled, and the error is returned.
    - Single Flight (`golang.org/x/sync/singleflight`): Prevents duplicate function calls with the same key from running simultaneously; only one execution proceeds, and all callers receive that single result.

  - Lock-Free Primitives (`sync/atomic`): This package offers functions like `AddInt64` and `CompareAndSwapInt64` for performing operations on basic types (`int32`, `int64`, `uint32`, `uint64`, `bool`, `pointer`). These high-throughput operations use direct CPU instructions and are advised only for special low-level needs due to their inherent implementation complexity.

---

Let's look at some concurrency patterns in Go.

## Fan-Out / Fan-In

```text
💡 Merger is optional, if all workers use the same channel                 

                       +--------------+                                   
            +--------> |   WORKER 1   | --------+                         
            |          +--------------+         |                         
            |                                   |                         
            |          +--------------+         |                         
  input ----+--------> |   WORKER 2   | --------+----> merger ----> output
            |          +--------------+         |                         
            |                                   |                         
            |          +--------------+         |                         
            +--------> |   WORKER 3   | --------+                         
                       +--------------+                                   
                                                                          
            ↑                                   ↑                         
         Fan-Out                              Fan-In                       
```

Example: Pizza Delivery in a Restaurant,

- A single order taker (input channel) collects many pizza orders. Several cooks (multiple goroutines) grab orders and work on each parallelly. A single waiter (output channel) gathers every finished pizza and passes them to customers.

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

const minWait, maxWait = 0.1, 1.0 // Workers wait randomly 0.1 - 1 second

func workerFunc(input <-chan int, output chan<- int) func() {
    return func() { // Return as a func() to be called by wg.Go()
        for v := range input {
            randomNumber := rand.Float64()*(maxWait-minWait) + minWait
            time.Sleep(time.Duration(randomNumber * float64(time.Second)))
            output <- v
        }
    }
}

func main() {
    input := make(chan int)
    output := make(chan int)

    var wg sync.WaitGroup

    // Start workers
    workerCount := 3
    for i := 0; i < workerCount; i++ {
        wg.Go(workerFunc(input, output))
    }

    // Producer
    go func() {
        defer close(input)
        for i := 1; i < 6; i++ { // 1-5 orders
            input <- i
        }
    }()

    // Close output when all workers are done
    go func() {
        wg.Wait()
        close(output)
    }()

    // Aggregator
    for order := range output {
        fmt.Println("Delivered order", order)
    }
}
```

## Pipelining (Sequential Stage Pattern)

```text
              +--------------+       +--------------+       +--------------+               
  input ----> |   WORKER 1   | ----> |   WORKER 2   | ----> |   WORKER 3   | ----> output  
              +--------------+       +--------------+       +--------------+               
                                                                                           
          ↑                      ↑                      ↑                      ↑           
    Stage 1 Chan           Stage 2 Chan           Stage 3 Chan           Stage 4 Chan      
```

Example: Assembly Line in a Car Factory,

- Unfinished cars move sequentially from one specialized stage to the next step-by-step (chassis -> add engine -> paint -> add tires -> safety test). All stages run concurrently, allowing work to pass immediately from one stage's output to the next stage's input.

```go
package main

import (
    "fmt"
)

func stage1(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for v := range in {
            out <- v * 2 // 1–5 -> 2–10
        }
    }()
    return out
}

func stage2(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for v := range in {
            out <- v * 5 // 2–10 -> 10–50
        }
    }()
    return out
}

func main() {
    // Producer
    stage1Ch := make(chan int)
    go func() {
        defer close(stage1Ch)
        for i := 1; i < 6; i++ { // 1-5
            stage1Ch <- i
        }
    }()

    // Pipeline
    stage2Ch := stage1(stage1Ch)
    stage3Ch := stage2(stage2Ch)

    // Sink
    for final := range stage3Ch {
        fmt.Println("Final output", final)
    }
}
```

## Publish/Subscribe (Pub/Sub)

```text
                                                          +----------------+                       
                                               +--------> |  SUBSCRIBER 1  |   ← Chan 1/ Worker 1  
                                      TOPIC 1  |          +----------------+                       
                                     +---------|                                                   
                                     |         |          +----------------+                       
                                     |         +--------> |  SUBSCRIBER 2  |   ← Chan 2/ Worker 2  
                +------------+       |                    +----------------+                       
  MESSAGE ----> |   BROKER   | ----> |                                                             
                +------------+       |                    +----------------+                       
                                     |         +--------> |  SUBSCRIBER 3  |   ← Chan 3/ Worker 3  
                                     | TOPIC 2 |          +----------------+                       
                                     +---------|                                                   
                                               |          +----------------+                       
                                               +--------> |  SUBSCRIBER 4  |   ← Chan 4/ Worker 4  
                                                          +----------------+                       
```

Example: Return Management in an Online Store,

- When a customer creates a return, the Return System publishes a single event like “ReturnCreated” to the central event broker. Several services subscribe to this same event: Billing subscribes to decide if a refund is needed, Warehouse subscribes to expect the incoming package, Inventory subscribes to prepare to restock, and Notifications subscribes to email the customer. All these subscribers react in parallel to the same “ReturnCreated” event, and the Return System doesn’t call any of them directly. It only publishes a message to the broker.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Message struct {
    Topic   string
    Content string
}

type subscribeReq struct {
    Topic string
    Ch    chan Message
}

// Broker implements the Active Object pattern using channels
type Broker struct {
    subscribers map[string][]chan Message

    // Activation Queues for different types of requests
    publishCh   chan Message
    subscribeCh chan subscribeReq
    doneCh      chan struct{}

    wg sync.WaitGroup
}

func NewBroker() *Broker {
    broker := &Broker{
        subscribers: make(map[string][]chan Message),
        publishCh:   make(chan Message),
        subscribeCh: make(chan subscribeReq),
        doneCh:      make(chan struct{}),
    }

    // Active Object pattern: starts a single central goroutine to manage all internal state
    broker.wg.Go(broker.loop)

    return broker
}

func (b *Broker) loop() {
    for {
        select {
        case msg := <-b.publishCh:
            subs := append([]chan Message(nil), b.subscribers[msg.Topic]...) // Copy on Read to operate outside the loop context safely
            for _, ch := range subs {
                select {
                case ch <- msg:
                    // Successfully sent
                default:
                    // Subscriber is full/slow, drop message and continue to the next subscriber
                    fmt.Printf("Warning: Dropping message for topic %s (consumer channel full/slow).\n", msg.Topic)
                }
            }
        case req := <-b.subscribeCh:
            b.subscribers[req.Topic] = append(b.subscribers[req.Topic], req.Ch)
        case <-b.doneCh:
            for topic, chans := range b.subscribers {
                for _, ch := range chans {
                    close(ch)
                }
                delete(b.subscribers, topic)
            }
            return
        }
    }
}

func (b *Broker) Subscribe(topic string) <-chan Message {
    ch := make(chan Message, 10) // Buffered
    b.subscribeCh <- subscribeReq{Topic: topic, Ch: ch}
    return ch
}

func (b *Broker) Publish(msg Message) {
    b.publishCh <- msg
}

func (b *Broker) Close() {
    close(b.doneCh)
    b.wg.Wait()
}

func main() {
    warehouseFunc := func(input <-chan Message) func() {
        return func() { // Return as a func() to be called by wg.Go()
            for m := range input {
                fmt.Println("Informed to warehouse", m.Content)
            }
        }
    }

    customerFunc := func(input <-chan Message) func() {
        return func() { // Return as a func() to be called by wg.Go()
            for m := range input {
                fmt.Println("Informed to customer", m.Content)
            }
        }
    }

    broker := NewBroker()

    // Subscribe to topics
    warehouseCh := broker.Subscribe("return-created")
    customerCh := broker.Subscribe("return-created")

    var wg sync.WaitGroup

    // Subscriber workers
    wg.Go(warehouseFunc(warehouseCh))
    wg.Go(customerFunc(customerCh))

    // Publish a message
    broker.Publish(Message{Topic: "return-created", Content: `{"orderID": "1", "itemID": "1:1"}`})

    // In this example, we just wait 3 seconds to consume messages in the workers.
    time.Sleep(3 * time.Second)

    // Shut down broker: closes all subscriber channels
    broker.Close()
    wg.Wait()
}
```
