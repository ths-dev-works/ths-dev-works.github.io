---
title: "Go Scheduling: Part III - Concurrency"
date: 2026-02-01
tags: ["Go", "Scheduling", "Concurrency", "Performance", "Workloads"]
description: "Understanding when and how to use concurrency effectively in Go with different workload types."
---

This post explores concurrency in Go, focusing on when to use it and how different workload types affect performance decisions.

## What is Concurrency

### Definition
- **Concurrency**: "Out of order" execution of instructions that would normally run sequentially
- **Parallelism**: Executing two or more instructions simultaneously 
- **Key Insight**: Concurrency is not the same as parallelism

### Concurrency vs Parallelism
```
┌────────────────────────────────────────────────────────────┐
│                Concurrency vs Parallelism                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Parallelism (2 Cores):                                    │
│  ┌─────────────────────────┐       ┌─────────────────────┐ │
│  │ P1 (Core 1)             │       │ P2 (Core 2)         │ │
│  │ ┌─────┐                 │       │ ┌─────┐             │ │
│  │ │ G1* │ ← Executing     │       │ │ G2* │ ← Executing │ │
│  │ └─────┘                 │       │ └─────┘             │ │
│  └─────────────────────────┘       └─────────────────────┘ │
│                                                            │
│  Concurrency (Single Core):                                │
│  ┌─────────────────────────┐                               │
│  │ P1 (Core 1)             │                               │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                               │
│  │ │ G1* │ │ G2  │ │ G3  │ │ ← Taking turns sharing        │
│  │ └─────┘ └─────┘ └─────┘ │   the same core               │
│  └─────────────────────────┘                               │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### When to Use Concurrency
- **Value**: Must provide enough performance gain to justify complexity cost
- **Feasibility**: Out of order execution must be possible and make sense
- **Testing**: Start with sequential solution, then consider concurrency

## Workload Types

### CPU-Bound Workloads
**Definition**: Work that never creates waiting states - constant calculations
- **Examples**: Mathematical computations, calculating Pi, data processing
- **Characteristics**: 
  - No natural waiting states
  - Requires parallelism for effective concurrency
  - More goroutines than cores can slow down execution
  - Context switches create "Stop The World" events

### IO-Bound Workloads
**Definition**: Work that causes natural waiting states
- **Examples**: Network requests, file I/O, system calls, synchronization events
- **Characteristics**:
  - Natural waiting states allow efficient sharing
  - Doesn't require parallelism for effective concurrency
  - More goroutines than cores can speed up execution
  - Context switches don't create "Stop The World" events

### Workload Decision Matrix
```
┌─────────────────────────────────────────────────────────────┐
│                    Workload Analysis                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  CPU-Bound:                                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ • Need parallelism (goroutines ≤ cores)                │ │
│  │ • Context switches are expensive                       │ │
│  │ • Too many goroutines = slower performance             │ │
│  │ • Example: Mathematical calculations                   │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  IO-Bound:                                                  │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ • Don't need parallelism (goroutines > cores OK)       │ │
│  │ • Context switches are efficient                       │ │
│  │ • More goroutines = better performance                 │ │
│  │ • Example: File reading, network calls                 │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Practical Examples

### CPU-Bound: Adding Numbers
**Sequential Version** (5 lines):
```go
func add(numbers []int) int {
    var v int
    for _, n := range numbers {
        v += n
    }
    return v
}
```

**Concurrent Version** (26 lines):
```go
func addConcurrent(goroutines int, numbers []int) int {
    var v int64
    totalNumbers := len(numbers)
    lastGoroutine := goroutines - 1
    stride := totalNumbers / goroutines

    var wg sync.WaitGroup
    wg.Add(goroutines)

    for g := 0; g < goroutines; g++ {
        go func(g int) {
            start := g * stride
            end := start + stride
            if g == lastGoroutine {
                end = totalNumbers
            }

            var lv int
            for _, n := range numbers[start:end] {
                lv += n
            }

            atomic.AddInt64(&v, int64(lv))
            wg.Done()
        }(g)
    }

    wg.Wait()
    return int(v)
}
```

**Analysis**:
- **Workload Type**: CPU-Bound (pure math)
- **Concurrency Benefit**: Yes (can split work)
- **Parallelism Required**: Yes (goroutines ≤ cores)
- **Complexity Cost**: High (5→26 lines)

### CPU-Bound: Bubble Sort
**Sequential Version**: Standard bubble sort algorithm
**Concurrent Version**: Sorts chunks concurrently, then re-sorts entire list

**Analysis**:
- **Workload Type**: CPU-Bound
- **Concurrency Benefit**: No (expensive to combine results)
- **Problem**: After concurrent sorting, still need full sort
- **Result**: Adds complexity without performance benefit

### IO-Bound: Reading Files
**Sequential Version**:
```go
func find(topic string, docs []string) int {
    var found int
    for _, doc := range docs {
        items, err := read(doc)
        if err != nil {
            continue
        }
        for _, item := range items {
            if strings.Contains(item.Description, topic) {
                found++
            }
        }
    }
    return found
}
```

**Concurrent Version**: Uses worker pool with channel
```go
func findConcurrent(goroutines int, topic string, docs []string) int {
    var found int64

    ch := make(chan string, len(docs))
    for _, doc := range docs {
        ch <- doc
    }
    close(ch)

    var wg sync.WaitGroup
    wg.Add(goroutines)

    for g := 0; g < goroutines; g++ {
        go func() {
            var lFound int64
            for doc := range ch {
                items, err := read(doc)
                if err != nil {
                    continue
                }
                for _, item := range items {
                    if strings.Contains(item.Description, topic) {
                        lFound++
                    }
                }
            }
            atomic.AddInt64(&found, lFound)
            wg.Done()
        }()
    }

    wg.Wait()
    return int(found)
}
```

**Performance Results**:
- **Without Parallelism**: ~87-88% faster than sequential
- **With Parallelism**: No additional performance gain
- **Key Insight**: Concurrency alone provides major benefits for IO-bound work

## Key Takeaways

### Decision Framework
1. **Start Sequential**: Always begin with working sequential solution
2. **Identify Workload**: Determine if CPU-Bound or IO-Bound
3. **Assess Concurrency**: Can work be split and combined efficiently?
4. **Consider Parallelism**: Does it need multiple cores?

### Workload Guidelines
- **CPU-Bound**: 
  - Use concurrency only if work can be efficiently split
  - Limit goroutines to number of cores
  - Beware of context switching overhead

- **IO-Bound**:
  - Concurrency almost always beneficial
  - Can use more goroutines than cores
  - Context switches are efficient due to natural waiting

### Red Flags
- **Expensive Result Combination**: Like bubble sort's re-sorting requirement
- **No Natural Waiting States**: CPU-bound work without parallelism
- **Complexity vs Benefit**: 5→26 lines of code for minimal gain

### Success Indicators
- **Natural Waiting States**: File I/O, network calls, synchronization
- **Efficient Work Splitting**: Clear boundaries for concurrent processing
- **Simple Result Combination**: Easy to merge concurrent results

## Conclusion

Effective concurrency in Go requires understanding your workload type:

- **IO-Bound workloads** benefit greatly from concurrency without needing parallelism
- **CPU-Bound workloads** need parallelism and careful goroutine management
- **Some algorithms** (like bubble sort) aren't suitable for concurrency despite being CPU-bound

The key is identifying when "out of order" execution provides real value that justifies the complexity cost. Always start sequential, analyze your workload, and make informed decisions about concurrency and parallelism.

---

*Based on William Kennedy's ["Scheduling In Go : Part III - Concurrency"](https://www.ardanlabs.com/blog/2018/12/scheduling-in-go-part3.html) from Ardan Labs.*
