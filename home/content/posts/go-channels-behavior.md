---
title: "Go Mechanics: The Behavior of Channels"
date: 2026-02-02
tags: ["Go", "Concurrency", "Channels", "Goroutines", "Signaling"]
description: "Understanding Go channels as signaling mechanisms for orchestrating goroutines and writing better concurrent code."
---

In Go, channels are one of the most powerful features for concurrent programming. However, many developers make the mistake of thinking about channels as data structures or queues. The key to mastering channels is to think of them as **signaling mechanisms** that allow goroutines to communicate about events.

This post summarizes William Kennedy's core insights on channel behavior from Ardan Labs, focusing on the three fundamental attributes of signaling: Guarantee of Delivery, State, and With/Without Data.

## The Signaling Philosophy

### From Data Structure to Signaling Mechanism
When working with channels, forget about their internal structure and focus on their behavior. A channel allows one goroutine to signal another goroutine about a particular event. This signaling mindset will help you write better, more precise concurrent code.

### The Three Signaling Attributes
1. **Guarantee Of Delivery**: Do you need confirmation that a signal was received?
2. **State**: What is the current state of the channel (nil, open, closed)?
3. **With or Without Data**: Are you signaling with data or just the signal itself?

## Guarantee Of Delivery

The fundamental question: "Do I need a guarantee that the signal sent by a particular goroutine has been received?"

Consider this basic example:
```go
go func() {
    p := <-ch // Receive
}()

ch <- "paper" // Send
```

Does the sending goroutine need to know that "paper" was received before continuing? Your answer determines which channel type to use.

### Figure 1: Guarantee Of Delivery

```
┌─────────────────────────────────────────────────────────┐
│                  GUARANTEE OF DELIVERY                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Does the sender need to know the signal was received?  │
│                                                         │
│  ┌─────────────────┐    ┌─────────────────┐             │
│  │   YES           │    │   NO            │             │
│  │                 │    │                 │             │
│  │ Unbuffered      │    │ Buffered (>1)   │             │
│  │ Channel         │    │ Channel         │             │
│  │                 │    │                 │             │
│  │ Receive         │    │ Send            │             │
│  │ happens BEFORE  │    │ happens BEFORE  │             │
│  │ Send completes  │    │ Receive         │             │
│  │                 │    │                 │             │
│  │ 100% Guarantee  │    │ No Guarantee    │             │
│  │                 │    │                 │             │
│  └─────────────────┘    └─────────────────┘             │
│                                                         │
│  ┌─────────────────┐                                    │
│  │   BUFFERED = 1  │                                    │
│  │                 │                                    │
│  │ Delayed         │                                    │
│  │ Guarantee       │                                    │
│  │                 │                                    │
│  │ Previous signal │                                    │
│  │ guaranteed      │                                    │
│  │ received        │                                    │
│  │                 │                                    │
│  └─────────────────┘                                    │
└─────────────────────────────────────────────────────────┘
```

### Channel Types and Guarantees

#### Unbuffered Channels - Guaranteed Delivery
```go
ch := make(chan string)
```
- **Guarantee**: Signal sent has been received
- **Mechanism**: Receive happens BEFORE Send completes
- **Use case**: When you must know the signal was received

#### Buffered Channels (>1) - No Guarantee
```go
ch := make(chan string, 10)
```
- **Guarantee**: No guarantee of reception
- **Mechanism**: Send happens BEFORE Receive
- **Use case**: When you don't need immediate confirmation

#### Buffered Channels (=1) - Delayed Guarantee
```go
ch := make(chan string, 1)
```
- **Guarantee**: Previous signal was received
- **Mechanism**: Receive of first signal happens BEFORE second Send completes
- **Use case**: When you need one-signal lag guarantee

## Channel States

Channels exist in three possible states, each with distinct behaviors:

### Figure 2: Channel States

```
┌──────────────────────────────────────────────────────────┐
│                    CHANNEL STATES                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌───────────┐ │
│  │      NIL        │  │      OPEN       │  │  CLOSED   │ │
│  │                 │  │                 │  │           │ │
│  │ var ch chan T   │  │ ch := make()    │  │ close(ch) │ │
│  │ ch = nil        │  │                 │  │           │ │
│  │                 │  │                 │  │           │ │
│  │ Send & Receive  │  │ Normal Send &   │  │ No more   │ │
│  │ BLOCK forever   │  │ Receive         │  │ Sends     │ │
│  │                 │  │                 │  │           │ │
│  │                 │  │                 │  │ Receives  │ │
│  │                 │  │                 │  │ still work│ │
│  │                 │  │                 │  │           │ │
│  └─────────────────┘  └─────────────────┘  └───────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Nil Channel
```go
var ch chan string  // Zero value
ch = nil            // Explicit nil
```
- **Behavior**: Any Send or Receive blocks forever
- **Use case**: Turn off signaling, rate limiting

### Open Channel
```go
ch := make(chan string)
```
- **Behavior**: Normal Send and Receive operations
- **Use case**: Active communication between goroutines

### Closed Channel
```go
close(ch)
```
- **Behavior**: No more Sends allowed, Receives still work
- **Use case**: Signal completion, no more data coming

## Signaling With Data

When you signal with data, you're typically:
- Asking a goroutine to start a new task
- Getting a result back from a goroutine

### Figure 3: Signaling With Data

```
┌───────────────────────────────────────────────────────────┐
│                SIGNALING WITH DATA                        │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────┐ │
│  │  UNBUFFERED     │  │ BUFFERED (>1)   │  │ BUFFERED   │ │
│  │                 │  │                 │  │  (=1)      │ │
│  │ ch <- "paper"   │  │ ch <- "paper"   │  │ ch <- "p"  │ │
│  │                 │  │                 │  │            │ │
│  │ Receive         │  │ Send            │  │ Send of    │ │
│  │ happens BEFORE  │  │ happens BEFORE  │  │ first sig  │ │
│  │ Send completes  │  │ Receive         │  │ happens    │ │
│  │                 │  │                 │  │ BEFORE     │ │
│  │ GUARANTEE       │  │ NO GUARANTEE    │  │ second     │ │
│  │                 │  │                 │  │ Send       │ │
│  │                 │  │                 │  │            │ │
│  │                 │  │                 │  │ DELAYED    │ │
│  │                 │  │                 │  │ GUARANTEE  │ │
│  └─────────────────┘  └─────────────────┘  └────────────┘ │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Scenario 1: Wait For Task (Guaranteed Delivery)
Manager hires employee and waits until ready to give them work:

```go
func waitForTask() {
    ch := make(chan string)

    go func() {
        p := <-ch  // Employee waits for paper
        
        // Employee performs work here
        // Employee is done and free to go
    }()

    time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
    ch <- "paper"  // Manager sends work when ready
}
```

**Key Points**:
- Employee blocks waiting for work
- Manager gets guarantee employee received the paper
- Unknown latency for both parties

### Scenario 2: Wait For Result (Guaranteed Delivery)
Manager hires employee to work immediately and waits for result:

```go
func waitForResult() {
    ch := make(chan string)

    go func() {
        time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        ch <- "paper"  // Employee sends result
        
        // Employee is done and free to go
    }()

    p := <-ch  // Manager waits for result
}
```

**Key Points**:
- Employee starts work immediately
- Manager blocks waiting for result
- Employee gets guarantee manager received the result

### Scenario 3: Fan Out (No Guarantee)
Manager hires team, each works independently:

```go
func fanOut() {
    emps := 20
    ch := make(chan string, emps)  // Buffered for all results

    for e := 0; e < emps; e++ {
        go func() {
            time.Sleep(time.Duration(rand.Intn(200)) * time.Millisecond)
            ch <- "paper"  // Send doesn't block
        }()
    }

    for emps > 0 {
        p := <-ch
        fmt.Println(p)
        emps--
    }
}
```

**Key Points**:
- No guarantee when results will be received
- Employees don't block when sending results
- Buffer size must be calculated based on constraints

### Scenario 4: Drop (No Guarantee)
Manager discards work when employee is at capacity:

```go
func selectDrop() {
    const cap = 5
    ch := make(chan string, cap)

    go func() {
        for p := range ch {
            fmt.Println("employee : received :", p)
        }
    }()

    const work = 20
    for w := 0; w < work; w++ {
        select {
        case ch <- "paper":
            fmt.Println("manager : send ack")
        default:
            fmt.Println("manager : drop")
        }
    }

    close(ch)
}
```

**Key Points**:
- Work is dropped when buffer is full
- No back pressure on work submission
- Uses `select` with `default` for non-blocking sends

### Scenario 5: Wait For Tasks (Delayed Guarantee)
Manager sends multiple tasks to single employee with buffer of 1:

```go
func waitForTasks() {
    ch := make(chan string, 1)

    go func() {
        for p := range ch {
            fmt.Println("employee : working :", p)
        }
    }()

    const work = 10
    for w := 0; w < work; w++ {
        ch <- "paper"
    }

    close(ch)
}
```

**Key Points**:
- Buffer of 1 provides delayed guarantee
- Previous task guaranteed to be received before next send
- Reduces latency while maintaining guarantees

## Signaling Without Data

When you signal without data, you're typically:
- Telling a goroutine to stop what it's doing
- Reporting completion with no result
- Signaling shutdown

### Figure 4: Signaling Without Data

```
┌───────────────────────────────────────────────────────┐
│             SIGNALING WITHOUT DATA                    │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────────┐  ┌─────────────────┐             │
│  │   UNBUFFERED    │  │   BUFFERED      │             │
│  │                 │  │                 │             │
│  │ close(ch)       │  │ close(ch)       │             │
│  │                 │  │                 │             │
│  │ Close happens   │  │ Close happens   │             │
│  │ BEFORE Receive  │  │ BEFORE Receive  │             │
│  │                 │  │                 │             │
│  │ Perfect for     │  │ Code smell for  │             │
│  │ cancellation    │  │ cancellation    │             │
│  │                 │  │                 │             │
│  └─────────────────┘  └─────────────────┘             │
│                                                       │
│  Use context package for cancellation                 │
│  Uses unbuffered channel internally                   │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### The Close Operation
```go
close(ch)
```

**Benefits**:
- Single goroutine can signal many goroutines at once
- Perfect for cancellation and shutdown scenarios
- Any receive on closed channel doesn't block

### Best Practice: Use Context Package
```go
// Preferred approach
ctx, cancel := context.WithCancel(context.Background())
go func() {
    select {
    case <-ctx.Done():
        // Handle cancellation
    }
}()
cancel()  // Signal without data
```

### Manual Cancellation Channel
If you implement your own cancellation:
```go
ch := make(chan struct{})  // Zero-space signaling channel
close(ch)  // Signal cancellation
```

**Important**: Use `chan struct{}` for signaling-only channels - it's the zero-space, idiomatic choice.

### Scenario 6: Context-based Cancellation
Using context package for timeout-based cancellation:

```go
func withTimeout() {
    duration := 50 * time.Millisecond

    ctx, cancel := context.WithTimeout(context.Background(), duration)
    defer cancel()

    ch := make(chan string, 1)

    go func() {
        time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
        ch <- "paper"
    }()

    select {
    case p := <-ch:
        fmt.Println("work complete", p)

    case <-ctx.Done():
        fmt.Println("moving on")
    }
}
```

**Key Points**:
- Context uses unbuffered channel internally for cancellation
- Buffered channel of 1 prevents goroutine leaks
- `select` waits on either work completion or timeout
- Always call `cancel()` in defer for cleanup

## Buffer Size Guidelines

### The "No Random Numbers" Rule
Buffer sizes must never be arbitrary. They must be calculated based on well-defined constraints.

### Key Questions for Buffer Sizing
1. **Do I have a well-defined amount of work to be completed?**
2. **How much outstanding work puts me at capacity?**
3. **If my goroutine can't keep up, can I discard new work?**
4. **What level of risk am I willing to accept if my program terminates unexpectedly?**

If these questions don't make sense for your use case, using a buffer larger than 1 is probably wrong.

## Language Mechanics Summary

### Unbuffered Channels
- **Behavior**: Receive happens before Send
- **Benefit**: 100% guarantee signal was received
- **Cost**: Unknown latency on when signal will be received

### Buffered Channels
- **Behavior**: Send happens before Receive
- **Benefit**: Reduce blocking latency between signaling
- **Cost**: No guarantee when signal was received
- **Note**: Larger buffer = less guarantee

### Closing Channels
- **Behavior**: Close happens before Receive (like Buffered)
- **Use**: Signaling without data, perfect for cancellations

### Nil Channels
- **Behavior**: Send and Receive block
- **Use**: Turn off signaling, rate limiting, temporary stoppages

## Design Philosophy

### When to Use Buffered Channels > 1
Only allowed if:
- You can answer the buffer sizing questions clearly
- You have measurements showing the need
- You know exactly what happens when sending goroutine blocks

### Code Smells to Avoid
- Random buffer sizes
- Buffered channels for cancellation (use `chan struct{}` instead)
- Using channels for simple mutex-style synchronization
- Thinking of channels as data structures rather than signals

## Performance Considerations

### Cost/Benefit Analysis
- **Guaranteed delivery** = unknown latency cost
- **No guarantee** = potential data loss on program termination
- **Buffer size** = trade-off between throughput and memory usage

### Memory Impact
- Each buffered element consumes memory
- Larger buffers = more memory pressure
- Consider what happens on program crash

## Key Takeaways

1. **Think signals, not data**: Channels are for signaling between goroutines
2. **Choose guarantees wisely**: Unbuffered for guarantees, buffered for throughput
3. **State matters**: nil, open, and closed channels have different behaviors
4. **Buffer with purpose**: Never use random buffer sizes
5. **Signal without data**: Use `close()` or `context` for cancellation
6. **Question your design**: Are you really using channels for the right reasons?

## In Summary

- **Channels orchestrate goroutines** through signaling, not data sharing
- **Three attributes define behavior**: Guarantee, State, and Data/No-Data
- **Unbuffered channels provide guarantees** at the cost of unknown latency
- **Buffered channels provide throughput** at the cost of guarantees
- **Closing channels signals completion** and enables cancellation patterns
- **Buffer sizes must be calculated** based on well-defined constraints
- **Think in terms of signaling** to write better concurrent Go programs

---

*Based on William Kennedy's ["The Behavior Of Channels"](https://www.ardanlabs.com/blog/2017/10/the-behavior-of-channels.html) from Ardan Labs.*
