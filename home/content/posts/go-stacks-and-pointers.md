---
title: "Go Mechanics: Stacks, Pointers, and Frame Boundaries"
date: 2026-01-31
tags: ["Go", "Internals", "Memory"]
---

In Go, "Mechanical Sympathy" starts with understanding where your data lives. Here's a comprehensive summary of Bill Kennedy's core principles on Stacks and Pointers from the Ardan Labs series.

## Frame Boundaries: The Foundation

Every function executes within **stack frames** - private memory spaces that provide isolation and context. When a function is called, Go transitions between frames, copying data "by value" across the boundary.

### Key Concepts:
- **Stack Frames**: Each function gets its own memory sandbox
- **Pass-by-Value**: Data is copied between frames (WYSIWYG - What You See Is What You Get)
- **Frame Boundaries**: Functions have direct access only to their own frame memory

## Value Semantics vs. Pointer Semantics

### 1. Value Semantics (Isolation)
When you pass data by value, Go creates a **copy**. The function operates on its own version, ensuring complete isolation.

```go
func increment(inc int) {
    inc++ // Only affects the copy
}
```

### 2. Pointer Semantics (Sharing)
A pointer allows a function to "reach across" frame boundaries into another function's frame. This is **sharing**.

```go
func increment(inc *int) {
    *inc++ // Affects the original value
}
```

## The Stack Mechanics

### Stack Allocation
- Each goroutine starts with ~2KB of stack space
- Frames are taken down the stack (implementation detail)
- Memory below the active frame is invalid
- Memory from the active frame and above is valid

### Function Call Process
1. **Frame Creation**: New stack frame is allocated
2. **Data Transfer**: Values are copied across frame boundary
3. **Execution**: Function operates within its frame
4. **Return**: Frame becomes invalid (but memory isn't cleaned)

### Stack Self-Cleaning
- When a function returns, its frame becomes invalid
- Memory is left untouched (no cleanup cost)
- Stack memory is wiped clean on each new function call
- All values are initialized to zero values

## Stack Visualizations

### Figure 1: Single Frame (main function)
```
┌─────────────────────┐ ← Valid memory
│    main frame       │
│                     │
│  count = 10         │
│  (addr: 0x10429fa4) │
└─────────────────────┘
┌─────────────────────┐ ← Invalid memory
│                     │
└─────────────────────┘
```

### Figure 2: Function Call (main calls increment)
```
┌─────────────────────┐ ← Valid memory
│    main frame       │
│                     │
│  count = 10         │
│  (addr: 0x10429fa4) │
├─────────────────────┤
│  increment frame    │
│                     │
│  inc = 10           │
│  (addr: 0x10429f98) │
└─────────────────────┘
┌─────────────────────┐ ← Invalid memory
│                     │
└─────────────────────┘
```

### Figure 3: After increment execution
```
┌─────────────────────┐ ← Valid memory
│    main frame       │
│                     │
│  count = 10         │
│  (addr: 0x10429fa4) │
├─────────────────────┤
│  increment frame    │
│                     │
│  inc = 11           │
│  (addr: 0x10429f98) │
└─────────────────────┘
┌─────────────────────┐ ← Invalid memory
│                     │
└─────────────────────┘
```

### Figure 4: After function return
```
┌─────────────────────┐ ← Valid memory (main is active)
│    main frame       │
│                     │
│  count = 10         │
│  (addr: 0x10429fa4) │
├─────────────────────┤ ← Invalid memory (increment frame)
│  increment frame    │  (left untouched)
│                     │
│  inc = 11           │
│  (addr: 0x10429f98) │
└─────────────────────┘
┌─────────────────────┐ ← Invalid memory
│                     │
└─────────────────────┘
```

### Figure 5: Pointer Sharing (passing address)
```
┌─────────────────────┐ ← Valid memory
│    main frame       │
│                     │
│  count = 10         │
│  (addr: 0x10429fa4) │
├─────────────────────┤
│  increment frame    │
│                     │
│  inc = 0x10429fa4   │ ← Points to count
│  (addr: 0x10429f98) │
└─────────────────────┘
┌─────────────────────┐ ← Invalid memory
│                     │
└─────────────────────┘
```

### Figure 6: After pointer dereference
```
┌─────────────────────┐ ← Valid memory
│    main frame       │
│                     │
│  count = 11         │ ← Modified via pointer
│  (addr: 0x10429fa4) │
├─────────────────────┤
│  increment frame    │
│                     │
│  inc = 0x10429fa4   │ ← Still points to count
│  (addr: 0x10429f98) │
└─────────────────────┘
┌─────────────────────┐ ← Invalid memory
│                     │
└─────────────────────┘
```

## Pointer Types and Mechanics

### Pointer Type Rules
For every type declared, you get a free pointer type:
- `int` → `*int`
- `User` → `*User`
- All pointers are 4-8 bytes (address size)

### Pointer Variables Are Not Special
- They're regular variables with memory allocation
- They store address values
- The `*` character serves dual purposes: type declaration and dereferencing operator

## The Golden Rule of Pointers

> **If the word "share" doesn't come out of your mouth, you don't need a pointer.**

Pointers serve ONE purpose: to share a value with a function so it can read/write to that value even though it doesn't exist directly inside its own frame.

## Memory Access Patterns

### Direct Access (Value Semantics)
```go
count := 10
increment(count)      // Pass value (copy)
// count remains 10
```

### Indirect Access (Pointer Semantics)
```go
count := 10
increment(&count)     // Pass address (share)
// count becomes 11
```

## Key Takeaways

1. **Go is 100% Pass-by-Value**: Even pointers are copied (you copy the address)
2. **The Stack is Self-Cleaning**: No GC involvement for stack memory
3. **Pointers are for Sharing**: If you aren't sharing data across boundaries, to stay with values
4. **Frame Boundaries Enforce Isolation**: Functions can only directly access their own frame
5. **Indirect Memory Access**: Pointers enable cross-frame memory access
6. **Type Safety**: Pointer types ensure only compatible addresses can be shared

## In Summary

- **Functions execute within the scope of frame boundaries** that provide an individual memory space for each respective function
- **When a function is called**, there is a transition that takes place between two frames
- **The benefit of passing data "by value" is readability**
- **The stack is important** because it provides the physical memory space for the frame boundaries that are given to each individual function
- **All stack memory below the active frame is invalid** but memory from the active frame and above is valid
- **Making a function call** means the goroutine needs to frame a new section of memory on the stack
- **It's during each function call, when the frame is taken**, that the stack memory for that frame is wiped clean
- **Pointers serve one purpose**, to share a value with a function so the function can read and write to that value even though the value does not exist directly inside its own frame
- **For every type that is declared**, either by you or the language itself, you get for free a compliment pointer type you can use for sharing
- **The pointer variable allows indirect memory access** outside of the function's frame that is using it
- **Pointer variables are not special** because they are variables like any other variable. They have a memory allocation and they hold a value

---

*This summary is based on Bill Kennedy's ["Language Mechanics On Stacks And Pointers"](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html) from Ardan Labs, part of a four-part series covering pointers, stacks, heaps, escape analysis, and value/pointer semantics in Go.*
