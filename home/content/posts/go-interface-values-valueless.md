---
title: "Go Mechanics: Interface Values Are Valueless"
date: 2026-02-02
tags: ["Go", "Interfaces", "Polymorphism", "Data-Oriented Design", "Behavior"]
description: "Understanding how Go interfaces are valueless and how they enable polymorphism through concrete data behavior."
---

In Go, interfaces represent a fundamental shift from implementation-focused thinking to behavior-focused design. The critical insight is that interface values are "valueless" - they have no concrete meaning without the data stored inside them.

This post summarizes William Kennedy's core insights on interface valuelessness from Ardan Labs, focusing on how this concept enables proper polymorphism and data-oriented design.

## Introduction: Focus on Behavior, Not Implementation

When designing with interfaces, most developers focus on implementation details. However, the real power comes from understanding the relationship interfaces have with concrete data.

**Key insight**: Interface values are valueless - they only become meaningful when concrete data is stored inside them.

## Data-Oriented Design Principles

### First Law of Data-Oriented Design
> "If you don't understand the data you are working with, you don't understand the problem you are trying to solve."

Every problem you solve is a data transformation problem:
- **Input** → **Transformation** → **Output**
- Functions are smaller data transformations
- Algorithms are based on concrete data
- Mechanical sympathy comes from understanding concrete data

### Second Law of Data-Oriented Design
> "When the data is changing, your problem is changing. When the problem is changing, then the algorithms you wrote need to change."

**The challenge**: How to allow algorithms to remain small and precise while handling data changes without cascading changes throughout the codebase?

**The solution**: Interfaces provide the mechanism for behavioral decoupling.

## Concrete Data: The Foundation

### Concrete Types with `struct`

```go
type file struct {
    name string
}

type pipe struct {
    name string
}
```

These are **concrete types** - they define real data that:
- Can be stored in memory
- Can be sent over networks
- Can be written to files
- Can be manipulated directly

```go
func main() {
    var f file  // Concrete value of type file
    var p pipe  // Concrete value of type pipe
    
    fmt.Println(f, p)  // { } { }
}
```

Both `f` and `p` are **real values** with concrete state.

## Interfaces Are Valueless

### Interface Types Define Behavior

```go
type reader interface {
    read(b []byte) (int, error)
}
```

An interface type:
- Only declares a method set of behavior
- Has nothing concrete about it
- Creates **valueless** values

```go
var r reader  // r is valueless!
```

**Critical concepts**:
- There is nothing real about variable `r`
- There is nothing concrete about variable `r`
- The variable `r` is **valueless**

### Figure 1: Concrete vs Interface Types

```
┌──────────────────────────────────────────────────────────┐
│              CONCRETE vs INTERFACE TYPES                 │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              CONCRETE TYPES                         │ │
│  │                                                     │ │
│  │  type file struct {                                 │ │
│  │      name string                                    │ │
│  │  }                                                  │ │
│  │                                                     │ │
│  │  var f file  // f is REAL                           │ │
│  │  ┌─────────┐                                        │ │
│  │  │ name: ""│                                        │ │
│  │  └─────────┘                                        │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              INTERFACE TYPES                        │ │
│  │                                                     │ │
│  │  type reader interface {                            │ │
│  │      read(b []byte) (int, error)                    │ │
│  │  }                                                  │ │
│  │                                                     │ │
│  │  var r reader  // r is VALUELESS                    │ │
│  │  ┌─────────┐                                        │ │
│  │  │   ?     │  ← No concrete data yet                │ │
│  │  └─────────┘                                        │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Polymorphism Through Behavior

### The Power of Valueless Interfaces

```go
func retrieve(r reader) error {
    data := make([]byte, 100)
    
    len, err := r.read(data)
    if err != nil {
        return err
    }
    
    fmt.Println(string(data[:len]))
    return nil
}
```

**Tom Kurtz's definition of polymorphism**:
> "Polymorphism means that you write a certain program and it behaves differently depending on the data that it operates on."

**Key insight**: The function declaration is NOT saying:
- "Pass me a value of type reader" (impossible - reader values don't exist)

**It IS saying**:
- "Pass me any piece of concrete data (value or pointer) that implements the reader contract"
- "Pass me concrete data that exhibits the read behavior"

### Figure 2: Polymorphic Function Behavior

```
┌──────────────────────────────────────────────────────────┐
│              POLYMORPHIC FUNCTION                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  func retrieve(r reader) error {                         │
│      // r is valueless - accepts ANY concrete data       │
│      // that implements the reader interface             │
│  }                                                       │
│                                                          │
│  ┌─────────────────┐    ┌─────────────────┐              │
│  │   file VALUE    │    │   pipe VALUE    │              │
│  │                 │    │                 │              │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │              │
│  │ │ name: "data"│ │    │ │ name: "cfg" │ │              │
│  │ └─────────────┘ │    │ └─────────────┘ │              │
│  └─────────────────┘    └─────────────────┘              │
│           │                       │                      │
│           │ retrieve(f)           │ retrieve(p)          │
│           ▼                       ▼                      │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              retrieve() FUNCTION                    │ │
│  │                                                     │ │
│  │  r.read() behaves differently based on:             │ │
│  │  • file.read()  → returns RSS data                  │ │
│  │  • pipe.read()  → returns JSON data                 │ │
│  │                                                     │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Giving Data Behavior

### Methods Enable Behavior

```go
type reader interface {
    read(b []byte) (int, error)
}

type file struct {
    name string
}

func (file) read(b []byte) (int, error) {
    s := "<rss><channel><title>Going Go</title></channel></rss>"
    copy(b, s)
    return len(s), nil
}

type pipe struct {
    name string
}

func (pipe) read(b []byte) (int, error) {
    s := `{"name": "bill", "title": "developer"}`
    copy(b, s)
    return len(s), nil
}
```

**Key points**:
- Methods give concrete data behavior
- Value receivers allow both values and pointers to be passed
- Each type now implements the `reader` interface
- Behavior is tied to the concrete data type

### Complete Polymorphic Example

```go
func main() {
    f := file{"data.json"}
    p := pipe{"cfg_service"}
    
    retrieve(f)  // Calls file.read() - RSS data
    retrieve(p)  // Calls pipe.read() - JSON data
}
```

## Interface Value Assignments

### Interface-to-Interface Assignments

```go
type Reader interface {
    Read()
}

type Writer interface {
    Write()
}

type ReadWriter interface {
    Reader
    Writer
}

type system struct {
    Host string
}

func (*system) Read()  { /* ... */ }
func (*system) Write() { /* ... */ }
```

### Figure 3: Interface Value Assignments

```
┌─────────────────────────────────────────────────────────┐
│           INTERFACE VALUE ASSIGNMENTS                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  var rw ReadWriter = &system{"127.0.0.1"}               │
│  var r Reader = rw                                      │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │    rw (ReadWriter)      │    r (Reader)            │ │
│  │                         │                          │ │
│  │ ┌──────────────────┐    │ ┌─────────────────┐      │ │
│  │ │ Interface Value  │    │ │ Interface Value │      │ │
│  │ │                  │    │ │                 │      │ │
│  │ │ ┌─────────────┐  │    │ │ ┌─────────────┐ │      │ │
│  │ │ │Type:        │  │    │ │ │Type:        │ │      │ │
│  │ │ │ReadWriter   │  │    │ │ │Reader       │ │      │ │
│  │ │ └─────────────┘  │    │ │ └─────────────┘ │      │ │
│  │ │ ┌─────────────┐  │    │ │ ┌─────────────┐ │      │ │
│  │ │ │Data:        │  │    │ │ │Data:        │ │      │ │
│  │ │ │&system{...} │◄─┼────┼─┤ │&system{...} │ │      │ │
│  │ │ └─────────────┘  │    │ │ └─────────────┘ │      │ │
│  │ └──────────────────┘    │ └─────────────────┘      │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  Assignment copies the CONCRETE DATA, not the interface │
│  values. Both rw and r store the same concrete data.    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Key insight**: Interface assignments copy the concrete data, not the interface values. Both `rw` and `r` store the same concrete data (`&system{...}`).

```go
func main() {
    var rw ReadWriter = &system{"127.0.0.1"}
    var r Reader = rw
    fmt.Println(rw, r)  // &{127.0.0.1} &{127.0.0.1}
}
```

## Key Takeaways

1. **Interface values are valueless** - they have no meaning without concrete data
2. **Polymorphism is data-driven** - concrete data changes code behavior
3. **Interfaces define behavior contracts** - not concrete implementations
4. **Methods give data behavior** - enabling polymorphism
5. **Interface assignments copy concrete data** - not interface values
6. **Focus on behavior, not implementation** - for better design
7. **Data-oriented design** - understand your data to understand your problem

## Design Principles

### When to Use Interfaces

- **Decoupling**: When you need to decouple from concrete implementations
- **Testing**: When you need to mock behavior
- **Extensibility**: When multiple types need to exhibit the same behavior
- **API design**: When you want to define contracts, not implementations

### Interface Design Guidelines

1. **Focus on behavior**: What should the data be able to do?
2. **Keep interfaces small**: Single responsibility principle
3. **Design around concrete data**: Start with the data, then define behavior
4. **Accept interfaces, return structs**: For maximum flexibility

### Common Pitfalls to Avoid

1. **Over-abstracting**: Don't create interfaces prematurely
2. **Interface pollution**: Don't create interfaces for every concrete type
3. **Ignoring semantics**: Be consistent with value vs pointer semantics
4. **Implementation focus**: Don't design around implementation details

## In Summary

- **Interface values are valueless** containers for concrete data
- **Polymorphism emerges** from concrete data exhibiting behavior
- **Data-oriented design** focuses on understanding concrete data first
- **Behavior contracts** enable precise decoupling without generalization
- **Interface assignments** copy concrete data, not interface values
- **Methods provide** the mechanism for data to exhibit behavior
- **Design focus** should be on behavior relationships, not implementation details

---

*Based on William Kennedy's ["Interface Values Are Valueless"](https://www.ardanlabs.com/blog/2018/03/interface-values-are-valueless.html) from Ardan Labs.*
