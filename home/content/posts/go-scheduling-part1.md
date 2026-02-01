---
title: "Go Scheduling: Part I - OS Scheduler Fundamentals"
date: 2026-01-31
tags: ["Go", "Scheduling", "Concurrency", "OS", "Performance"]
description: "Understanding the operating system scheduler fundamentals that form the foundation for Go's scheduling system."
---

This post summarizes the operating system scheduler fundamentals that provide the foundation for understanding Go's scheduling system.

## OS Scheduler Basics

### Threads and Execution
- **Threads** are "paths of execution" that run independently with their own state
- Every program creates a Process with an initial Thread
- Scheduling decisions happen at the Thread level, not Process level
- Threads can run concurrently (taking turns) or in parallel (simultaneously on different cores)

### Thread States
1. **Waiting**: Stopped waiting for hardware, OS calls, or synchronization
2. **Runnable**: Wants CPU time to execute instructions  
3. **Executing**: Currently on a core executing machine instructions

### Work Types
- **CPU-Bound**: Never enters Waiting state (constant calculations)
- **IO-Bound**: Causes Waiting states (network, database, system calls)

## Context Switching

### Preemptive Scheduling
- Modern OS schedulers are preemptive (unpredictable Thread selection)
- Never write code based on perceived behavior
- Control synchronization explicitly for determinism

### Performance Impact
- **Context switches are expensive**: ~1000-1500 nanoseconds
- Cost: ~12k-18k instructions lost per switch
- **IO-Bound work**: Context switches help (cores stay busy)
- **CPU-Bound work**: Context switches hurt (pure latency cost)

## Performance Principles

### "Less is More"
- Fewer Runnable Threads = less overhead, more time per Thread
- More Runnable Threads = less time per Thread, less work done
- Balance cores vs. Threads for optimal throughput

### Cache Coherency
- **Cache lines**: 64-byte chunks exchanged between memory and caches
- **False sharing**: Multiple threads accessing same cache line causes performance issues
- Each core gets its own cache line copy (value semantics in hardware)
- Cache invalidation creates memory access latency (~100-300 clock cycles)

### Historical Context
- Traditional approach: Thread pools with ~3 threads per core
- Go eliminates need for manual thread pool management
- OS schedulers make complex balancing decisions constantly

## Key Takeaways

1. **Threads are independent execution paths** with their own state
2. **Context switches are expensive** and impact performance differently based on work type
3. **Fewer threads often perform better** than many threads
4. **Cache coherency is critical** for multithreaded performance
5. **OS schedulers make complex decisions** balancing many factors

This foundation is essential for understanding how Go's scheduler builds upon these OS fundamentals in Part II.

---

*Based on William Kennedy's ["Scheduling In Go : Part I - OS Scheduler"](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html) from Ardan Labs.*
