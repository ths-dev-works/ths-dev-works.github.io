---
title: "Go Scheduling: Part II - Go Scheduler"
date: 2026-02-01
tags: ["Go", "Scheduling", "Concurrency", "Goroutines", "Performance"]
description: "Understanding Go's scheduler mechanics and how it builds upon OS scheduler fundamentals."
---

This post summarizes Go's scheduler mechanics and how it provides efficient scheduling for goroutines.

## Go Scheduler Architecture

### Core Components
- **Logical Processors (P)**: One per virtual core (handles hyper-threading)
- **OS Threads (M)**: One per P, managed by OS but attached to P
- **Goroutines (G)**: Application-level coroutines, context-switched on M
- **Run Queues**: Local (LRQ) per P, Global (GRQ) for unassigned goroutines

### G-M-P Model Visualization
```
┌──────────────────────────────────────────────────────────────┐
│                    Go Scheduler Architecture                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  P1 (Logical Processor)           P2 (Logical Processor)     │
│  ┌─────────────────────────┐       ┌───────────────────────┐ │
│  │ LRQ (Local Run Queue)   │       │ LRQ (Local Run Queue) │ │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │       │ ┌─────┐ ┌─────┐ ┌───┐ │ │
│  │ │ G1  │ │ G2  │ │ G3  │ │       │ │ G4  │ │ G5  │ │G6 │ │ │
│  │ └─────┘ └─────┘ └─────┘ │       │ └─────┘ └─────┘ └───┘ │ │
│  └─────────────────────────┘       └───────────────────────┘ │
│           │                               │                  │
│           ▼                               ▼                  │
│  ┌─────────────────────────┐       ┌───────────────────────┐ │
│  │ M1 (OS Thread)          │       │ M2 (OS Thread)        │ │
│  │ Currently Executing G1  │       │ Currently Executing G4│ │
│  └─────────────────────────┘       └───────────────────────┘ │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                   GRQ (Global Run Queue)                     │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐     │
│  │ G7  │ │ G8  │ │ G9  │ │ G10 │ │ G11 │ │ G12 │ │ G13 │     │
│  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘     │
└──────────────────────────────────────────────────────────────┘
```

### Initialization
- Go program gets P for each virtual core (`runtime.NumCPU()`)
- Each P gets an OS Thread (M)
- Initial goroutine starts execution
- All components work together for efficient scheduling

## Scheduler Characteristics

### Cooperating vs Preemptive
- **OS Scheduler**: Preemptive (kernel-level, unpredictable)
- **Go Scheduler**: Cooperating (user-space, but feels preemptive)
- Go scheduler runs in user space above kernel
- Non-deterministic behavior like OS scheduler
- Developer doesn't control scheduling decisions

### Goroutine States
Same three states as OS Threads:
1. **Waiting**: Stopped waiting for OS calls or synchronization
2. **Runnable**: Wants time on M to execute instructions
3. **Executing**: Currently on M executing instructions

## Context Switching

### Scheduling Events
Four events allow scheduler decisions:
1. **`go` keyword**: Creating new goroutines
2. **Garbage Collection**: GC goroutines need execution time
3. **System Calls**: May cause goroutine to block M
4. **Synchronization**: Atomic, mutex, channel operations

### Function Call Importance
- Context switching happens at function call safe points
- Tight loops without function calls cause scheduler latencies
- Go 1.12+ adds non-cooperative preemption for tight loops

## System Call Handling

### Asynchronous System Calls
- **Network Poller**: Handles async network calls efficiently
- Uses kqueue (MacOS), epoll (Linux), IOCP (Windows)
- Goroutine moves to network poller, M stays available
- No extra M needed for network operations
- Reduces OS scheduling load

### Async System Call Flow
```
┌─────────────────────────────────────────────────────────────┐
│                Asynchronous System Call Flow                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Before Network Call:                                       │
│  ┌─────────────────────────┐                                │
│  │ P1                      │                                │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                                │
│  │ │ G1* │ │ G2  │ │ G3  │ │  ← G1 executing on M1          │
│  │ └─────┘ └─────┘ └─────┘ │                                │
│  └─────────────────────────┘                                │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐                                │
│  │ M1 (OS Thread)          │                                │
│  │ Currently Executing G1  │                                │
│  └─────────────────────────┘                                │
│                                                             │
│  Network Call Made:                                         │
│  ┌─────────────────────────┐       ┌─────────────────────┐  │
│  │ P1                      │       │ Network Poller      │  │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │       │ ┌─────┐             │  │
│  │ │ G2* │ │ G3  │ │ G4  │ │       │ │ G1  │ ← G1 moved  │  │
│  │ └─────┘ └─────┘ └─────┘ │       │ └─────┘    to poller│  │
│  └─────────────────────────┘       └─────────────────────┘  │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐                                │
│  │ M1 (OS Thread)          │                                │
│  │ Now Executing G2        │                                │
│  └─────────────────────────┘                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Synchronous System Calls
- **File I/O**: Cannot be async, blocks M
- **CGO**: May block M when calling C functions
- **Scheduler Response**: Detaches M from P, brings new M
- Blocked goroutine moves back to LRQ when call completes
- Original M cached for future use

### Sync System Call Flow
```
┌─────────────────────────────────────────────────────────────┐
│                Synchronous System Call Flow                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Before File I/O Call:                                      │
│  ┌─────────────────────────┐                                │
│  │ P1                      │                                │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                                │
│  │ │ G1* │ │ G2  │ │ G3  │ │  ← G1 executing on M1          │
│  │ └─────┘ └─────┘ └─────┘ │                                │
│  └─────────────────────────┘                                │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐                                │
│  │ M1 (OS Thread)          │                                │
│  │ Currently Executing G1  │                                │
│  └─────────────────────────┘                                │
│                                                             │
│  File I/O Call Blocks M1:                                   │
│  ┌─────────────────────────┐                                │
│  │ P1                      │                                │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                                │
│  │ │ G2* │ │ G3  │ │ G4  │ │  ← G2 now executing on M2      │
│  │ └─────┘ └─────┘ └─────┘ │                                │
│  └─────────────────────────┘                                │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐       ┌─────────────────────┐  │
│  │ M2 (OS Thread)          │       │ M1 (Blocked)        │  │
│  │ Now Executing G2        │       │ G1 doing file I/O   │  │
│  └─────────────────────────┘       └─────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Work Stealing

### Purpose
- Prevents M from going idle (avoids OS context switches)
- Balances goroutines across all P's
- Keeps work efficiently distributed

### Work Stealing Visualization
```
┌─────────────────────────────────────────────────────────────┐
│                    Work Stealing Example                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Initial State:                                             │
│  ┌─────────────────────────┐       ┌──────────────────────┐ │
│  │ P1                      │       │ P2                   │ │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │       │ ┌─────┐ ┌─────┐ ┌───┐│ │
│  │ │ G1* │ │ G2  │ │ G3  │ │       │ │ G4* │ │ G5  │ │G6 ││ │
│  │ └─────┘ └─────┘ └─────┘ │       │ └─────┘ └─────┘ └───┘│ │
│  └─────────────────────────┘       └──────────────────────┘ │
│                                                             │
│  GRQ: ┌─────┐                                               │
│       │ G7  │                                               │
│       └─────┘                                               │
│                                                             │
│  P1 Finishes Work:                                          │
│  ┌─────────────────────────┐       ┌──────────────────────┐ │
│  │ P1 (Empty)              │       │ P2                   │ │
│  │                         │       │ ┌─────┐ ┌─────┐ ┌───┐│ │
│  │                         │       │ │ G4* │ │ G5  │ │G6 ││ │
│  │                         │       │ └─────┘ └─────┘ └───┘│ │
│  └─────────────────────────┘       └──────────────────────┘ │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐                                │
│  │ P1 Steals from P2:      │                                │
│  │ ┌─────┐ ┌─────┐         │                                │
│  │ │ G5* │ │ G6  │         │  ← Half stolen from P2         │
│  │ └─────┘ └─────┘         │                                │
│  └─────────────────────────┘                                │
│                                                             │
│  Final Balanced State:                                      │
│  ┌─────────────────────────┐       ┌─────────────────────┐  │
│  │ P1                      │       │ P2                  │  │
│  │ ┌─────┐ ┌─────┐         │       │ ┌─────┐             │  │
│  │ │ G5* │ │ G6  │         │       │ │ G4* │             │  │
│  │ └─────┘ └─────┘         │       │ └─────┘             │  │
│  └─────────────────────────┘       └─────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Stealing Rules
1. Check global runnable queue (1/61 of the time)
2. Check local queue
3. Try to steal from other P's (take half)
4. Check global queue again
5. Poll network

### Benefits
- Ms stay busy ("spinning")
- Better cache locality
- Reduced OS scheduling overhead

## Performance Advantages

### Context Switch Efficiency
- **OS Thread context switch**: ~1000-1500 nanoseconds (~12k-18k instructions)
- **Goroutine context switch**: ~200 nanoseconds (~2.4k instructions)
- **5-6x improvement** in context switching cost

### Performance Comparison Visualization
```
┌──────────────────────────────────────────────────────────────┐
│                Performance Comparison                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  OS Thread Context Switch:                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Time: ~1000-1500 nanoseconds                            │ │
│  │ Lost Instructions: ~12k-18k                             │ │
│  │ Cache Misses: High (bouncing between cores)             │ │
│  │ OS Overhead: High                                       │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  Goroutine Context Switch:                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Time: ~200 nanoseconds                                  │ │
│  │ Lost Instructions: ~2.4k                                │ │
│  │ Cache Misses: Low (same core/thread)                    │ │
│  │ OS Overhead: Minimal                                    │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  Performance Gain: 5-6x faster context switching             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Key Innovation
Go turns **IO/Blocking work into CPU-bound work** at OS level:
- OS Thread never enters waiting state
- All context switching happens at application level
- Better cache-line efficiency
- Reduced NUMA latency

### Practical Impact
- Same OS Thread and Core used for all processing
- No instruction loss to OS context switches
- More work done with fewer threads
- Reduced load on OS and hardware

## Key Takeaways

1. **Go scheduler is cooperating but feels preemptive** - non-deterministic like OS scheduler
2. **Goroutines are application-level threads** with same three states as OS threads
3. **Context switching is much cheaper** than OS thread switching (~200ns vs ~1000ns)
4. **Network poller handles async calls efficiently** without blocking threads
5. **Work stealing prevents idle cores** and balances load across processors
6. **Go turns blocking work into CPU-bound work** at OS level for major efficiency gains

This design allows Go to execute more work over time by using fewer threads more efficiently, reducing OS scheduling load while maintaining high concurrency.

---

*Based on William Kennedy's ["Scheduling In Go : Part II - Go Scheduler"](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html) from Ardan Labs.*
