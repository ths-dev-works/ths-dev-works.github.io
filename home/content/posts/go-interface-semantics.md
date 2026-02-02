---
title: "Go Mechanics: Interface Semantics"
date: 2026-02-02
tags: ["Go", "Interfaces", "Semantics", "Method Sets", "Value vs Pointer"]
description: "Understanding how Go interfaces provide both value and pointer semantic forms, and the method set rules that ensure integrity."
---

In Go, interfaces can store values using either value semantics (storing a copy of the value) or pointer semantics (storing a copy of the value's address). Understanding these semantics is critical for writing consistent, reliable code that maintains integrity as it grows.

This post summarizes William Kennedy's core insights on interface semantics from Ardan Labs, focusing on the language mechanics and method set rules that govern how interfaces work.

## Introduction: Value vs Pointer Semantics

The choice between value and pointer semantics is fundamental to Go programming. Semantic consistency is critical for:
- **Code integrity**: Maintaining predictable behavior
- **Readability**: Helping developers maintain a strong mental model
- **Maintainability**: Minimizing mistakes and unexpected behavior

Interfaces in Go provide both semantic forms, and understanding how they work is essential for proper Go programming.

## Language Mechanics: Interface Storage

An interface can store:
1. **Value semantics**: Its own copy of a value
2. **Pointer semantics**: A copy of the value's address (sharing the original)

### Figure 1: Interface Storage Semantics

```
┌─────────────────────────────────────────────────────────┐
│              INTERFACE STORAGE SEMANTICS                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┐    ┌─────────────────┐             │
│  │ VALUE SEMANTICS │    │POINTER SEMANTICS│             │
│  │                 │    │                 │             │
│  │ entities[0] = u │    │ entities[1] = &u│             │
│  │                 │    │                 │             │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │             │
│  │ │Interface    │ │    │ │Interface    │ │             │
│  │ │Value        │ │    │ │Value        │ │             │
│  │ │             │ │    │ │             │ │             │
│  │ │ ┌─────────┐ │ │    │ │ ┌─────────┐ │ │             │
│  │ │ │Type     │ │ │    │ │ │Type     │ │ │             │
│  │ │ │Info     │ │ │    │ │ │Info     │ │ │             │
│  │ │ └─────────┘ │ │    │ │ └─────────┘ │ │             │
│  │ │             │ │    │ │             │ │             │
│  │ │ ┌─────────┐ │ │    │ │ ┌─────────┐ │ │             │
│  │ │ │Data     │ │ │    │ │ │Data     │ │ │             │
│  │ │ │Copy of  │ │ │    │ │ │Copy of  │ │ │             │
│  │ │ │user     │ │ │    │ │ │address  │ │ │             │
│  │ │ └─────────┘ │ │    │ │ │of user  │ │ │             │
│  │ └─────────────┘ │    │ │ └─────────┘ │ │             │
│  │                 │    │ │             │ │             │
│  │                 │    │ │ ┌─────────┐ │ │             │
│  │                 │    │ │ │Original │ │ │             │
│  │                 │    │ │ │user     │ │ │             │
│  │                 │    │ │ │value    │ │ │             │
│  │                 │    │ │ └─────────┘ │ │             │
│  │                 │    │ └─────────────┘ │             │
│  └─────────────────┘    └─────────────────┘             │
│                                                         │
│  After u.name = "Bill_CHG":                             │
│  - entities[0] still shows "Bill"  (independent copy)   │
│  - entities[1] shows "Bill_CHG"   (shared original)     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Basic Interface Semantics Example

```go
package main

import "fmt"

type printer interface {
    print()
}

type user struct {
    name string
}

func (u user) print() {
    fmt.Println("User Name:", u.name)
}

func main() {
    u := user{"Bill"}

    entities := []printer{
        u,   // Value semantics - stores copy of user
        &u,  // Pointer semantics - stores copy of address
    }

    u.name = "Bill_CHG"

    for _, e := range entities {
        e.print()
    }
}
```

**Output**:
```
User Name: Bill
User Name: Bill_CHG
```

### What's Happening?

- **Index 0**: Uses value semantics - stores a copy of the original `user` value
- **Index 1**: Uses pointer semantics - stores the address of the original `user` value
- **After modification**: Only the pointer semantics version sees the change

This demonstrates the fundamental difference: value semantics create independent copies, while pointer semantics share the original value.

## Method Sets: The Rules of Interface Implementation

Method set rules determine what data can be stored inside an interface based on how the interface methods are implemented. These rules are all about maintaining integrity.

### Method Set Rules

#### Value Receiver Methods (Value Semantics)
When you implement an interface using value receivers:
- ✅ **Can store copies of values** (value semantics)
- ⚠️ **Can store copies of addresses** (pointer semantics) - with caution

#### Pointer Receiver Methods (Pointer Semantics)
When you implement an interface using pointer receivers:
- ✅ **Can store copies of addresses** (pointer semantics)
- ❌ **Cannot store copies of values** (value semantics)

### Figure 2: Method Set Rules

```
┌──────────────────────────────────────────────────────────┐
│                  METHOD SET RULES                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │           VALUE RECEIVER METHODS                    │ │
│  │                                                     │ │
│  │  func (m MyType) Method() {}                        │ │
│  │                                                     │ │
│  │  ┌─────────────────┐    ┌─────────────────┐         │ │
│  │  │ STORE VALUE     │    │ STORE POINTER   │         │ │
│  │  │ ✅ ALLOWED      │    │ ⚠️  ALLOWED     │         │ │
│  │  │                 │    │ (with caution)  │         │ │
│  │  │ i = MyType{}    │    │ i = &MyType{}   │         │ │
│  │  └─────────────────┘    └─────────────────┘         │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │          POINTER RECEIVER METHODS                   │ │
│  │                                                     │ │
│  │  func (m *MyType) Method() {}                       │ │
│  │                                                     │ │
│  │  ┌─────────────────┐    ┌─────────────────┐         │ │
│  │  │ STORE VALUE     │    │ STORE POINTER   │         │ │
│  │  │ ❌ FORBIDDEN    │    │ ✅ ALLOWED      │         │ │
│  │  │                 │    │                 │         │ │
│  │  │ i = MyType{}    │    │ i = &MyType{}   │         │ │
│  │  │ (compile error) │    │                 │         │ │
│  │  └─────────────────┘    └─────────────────┘         │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────│───────────────────┘
```

### Why These Rules Exist

#### Rule 1: Addressability Constraints
Not all values are addressable in Go. Consider this example:

```go
type notifier interface {
    notify()
}

type duration int

func (d *duration) notify() {
    fmt.Println("Sending Notification in", *d)
}

func main() {
    duration(42).notify()  // COMPILE ERROR
}
```

**Error**:
```
cannot call pointer method on duration(42)
cannot take the address of duration(42)
```

**Why?** Literal values like `duration(42)` are constants that only exist at compile time and don't have addresses. Since pointer receiver methods require sharing, they can't work with non-addressable values.

#### Rule 2: Integrity Protection
The method set rules prevent dangerous semantic mixing:

- **Pointer semantics → Value semantics**: ❌ **Dangerous** - You can't safely copy values that are designed to be shared
- **Value semantics → Pointer semantics**: ⚠️ **Possible** - Can be safe but requires conscious decision

This protects against unintended side effects and maintains semantic consistency.

## Interfaces Are Valueless

A crucial concept in Go is that interface values themselves are "valueless" - they have no concrete meaning without the data stored inside them.

### Interface Value Structure

```go
type notifier interface {
    notify()
}

type duration int

func (d duration) notify() {
    fmt.Println("Sending Notification in", d)
}

func main() {
    var n notifier        // n is nil - valueless
    n = duration(42)      // n now has concrete data
    n.notify()
}
```

**Key Points**:
- Interface values start as `nil` (valueless)
- They only become concrete when data is stored inside
- The method set rules determine what data can be stored
- Implementation details (like how interfaces store data internally) are irrelevant to the semantics

### Interface Comparison

When you compare interface values, you're comparing the concrete data inside them, not the interface values themselves:

```go
type errorString struct {
    s string
}

func (e errorString) Error() string {
    return e.s
}

func New(text string) error {
    return errorString{text}  // Value semantics
}

var ErrBadRequest = New("Bad Request")

func main() {
    err := webCall()
    if err == ErrBadRequest {
        fmt.Println("Interface Values MATCH")
    }
}

func webCall() error {
    return New("Bad Request")
}
```

**Output**:
```
Interface Values MATCH
```

**Why they match**: Both interface values contain the same concrete data (`errorString{"Bad Request"}`), so they're equivalent regardless of being different interface variables.

## Practical Implications

### Choosing the Right Semantics

#### Use Value Semantics When:
- The type is small and cheap to copy
- The type represents immutable data
- You want independence between interface values
- The type doesn't need to be modified through the interface

#### Use Pointer Semantics When:
- The type is large and expensive to copy
- The type represents mutable state
- You need to share the same instance across multiple interface values
- The type needs to be modified through the interface

### Consistency is Key

The most important principle is semantic consistency:

```go
// GOOD: Consistent value semantics
type User struct {
    name string
}

func (u User) String() string {  // Value receiver
    return u.name
}

// GOOD: Consistent pointer semantics  
type File struct {
    data []byte
}

func (f *File) Write(data []byte) error {  // Pointer receiver
    f.data = append(f.data, data...)
    return nil
}

// BAD: Mixed semantics without reason
type Config struct {
    settings map[string]string
}

func (c Config) Get(key string) string {     // Value receiver
    return c.settings[key]
}

func (c *Config) Set(key, value string) {   // Pointer receiver
    c.settings[key] = value
}
```

### When to Mix Semantics

Sometimes mixing semantics is necessary, but it should be a conscious decision:

```go
type Counter struct {
    count int
}

// Value semantics for reading (safe, immutable operation)
func (c Counter) Value() int {
    return c.count
}

// Pointer semantics for modification (needs to change state)
func (c *Counter) Increment() {
    c.count++
}
```

## Method Set Rules Summary

### Value Receiver Implementation
```go
type MyType struct{}

func (m MyType) Method() {}  // Value receiver

var i MyInterface
i = MyType{}     // ✅ Value semantics
i = &MyType{}    // ⚠️ Pointer semantics (allowed but be careful)
```

### Pointer Receiver Implementation
```go
type MyType struct{}

func (m *MyType) Method() {}  // Pointer receiver

var i MyInterface
i = &MyType{}    // ✅ Pointer semantics
i = MyType{}     // ❌ Value semantics (compile error)
```

## Key Takeaways

1. **Interfaces support both semantics**: Value (copy) and pointer (share)
2. **Method set rules enforce integrity**: Prevent dangerous semantic mixing
3. **Addressability matters**: Not all values can be shared via pointers
4. **Interfaces are valueless**: Only the data inside matters
5. **Consistency is critical**: Maintain semantic consistency across your codebase
6. **Choose consciously**: Select semantics based on your specific needs
7. **Implementation details don't matter**: Focus on the semantic behavior, not internal representation

## Best Practices

1. **Be consistent**: Use the same semantic throughout your type's interface implementations
2. **Document exceptions**: If you mix semantics, explain why
3. **Prefer value semantics**: For small, immutable types
4. **Use pointer semantics**: For large types or when modification is needed
5. **Test semantic behavior**: Ensure your interfaces behave as expected
6. **Review for consistency**: During code reviews, check semantic consistency

## In Summary

- **Interface semantics determine** how data is stored and shared
- **Method set rules protect** against dangerous semantic mixing
- **Value semantics create** independent copies of data
- **Pointer semantics share** the original data across interface values
- **Consistency maintains** code integrity and readability
- **Interfaces are valueless** containers for concrete data
- **Choose consciously** based on your specific requirements

---

*Based on William Kennedy's ["Interface Semantics"](https://www.ardanlabs.com/blog/2017/07/interface-semantics.html) from Ardan Labs.*
