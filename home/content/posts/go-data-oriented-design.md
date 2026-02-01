---
title: "Go Mechanics: Data-Oriented Design and Semantics"
date: 2026-02-01
tags: ["Go", "Design", "Semantics", "Architecture"]
---

In Go, understanding data-oriented design and choosing between value and pointer semantics is fundamental to writing maintainable, performant software. Here's a comprehensive summary of Bill Kennedy's design philosophy on data and semantics from Ardan Labs.

## Design Philosophies: The Foundation

### Value vs. Pointer Semantics
> "Value semantics keep values on the stack, which reduces pressure on the Garbage Collector (GC). However, value semantics require various copies of any given value to be stored, tracked and maintained. Pointer semantics place values on the heap, which can put pressure on the GC. However, pointer semantics are efficient because only one value needs to be stored, tracked and maintained." - Bill Kennedy

A consistent use of value/pointer semantics for a given data type is critical for maintaining integrity and readability throughout your software.

### Mental Models and Code Scale
> "Let's imagine a project that's going to end up with a million lines of code or more. The probability of those projects being successful in the United States these days is very low - well under 50%." - Tom Love (inventor of Objective C)

Consider that a box of copy paper holds ~100k lines of code. For what percentage of that code could you maintain a mental model? Realistically, a single developer can maintain about 10k lines of code. At million-line scale, you need 100 developers coordinated and in constant communication.

### Debugging and Mental Models
> "The hardest bugs are those where your mental model of the situation is just wrong, so you can't see the problem at all" - Brian Kernighan

Debuggers should be used when you've lost your mental model, not as the first reaction to bugs. Production debugging relies on logs, which require understanding the code's data transformations.

### Readability Through Predictability
> "C is the best balance I've ever seen between power and expressiveness. You can do almost anything you want to do by programming fairly straightforwardly and you will have a very good mental model of what's going to happen on the machine" - Brian Kernighan

This applies equally to Go. Maintaining a clear mental model drives integrity, readability, and simplicity—the cornerstones of maintainable software.

## Data-Driven Design Principles

### Data is Everything
> "If you don't understand the data, you don't understand the problem. This is because all problems are unique and specific to the data you are working with." - Bill Kennedy

Every problem is a data transformation problem. Every function takes input data and produces output data. Your mental model of software is understanding these data transformations.

### Less is More
A "less is more" attitude is critical for solving problems with:
- Fewer layers
- Fewer statements  
- Less generalization
- Less complexity
- Less effort

This makes everything easier for both your team and the hardware executing the transformations.

### Type is Life
> "Integrity means that every allocation, every read of memory and every write of memory is accurate, consistent and efficient. The type system is critical to making sure we have this micro level of integrity." - William Kennedy

If data drives everything, then the types representing that data are critical. Types provide the compiler with the ability to ensure data integrity and dictate semantic rules.

### Data with Capability
> "Methods are valid when it is practical or reasonable for a piece of data to have a capability." - William Kennedy

Methods give data capability. The focus should be on the data because it drives:
- The algorithms you write
- The encapsulations you put in place
- The performance you can achieve

### Polymorphism Through Data
> "Polymorphism means that you write a certain program and it behaves differently depending on the data that it operates on." - Tom Kurtz (inventor of BASIC)

A function can behave differently based on the data it operates on. This decouples functions from concrete data types and is key to architecting adaptable systems.

### Prototype First Approach
> "Unless the developer has a really good idea of what the software is going to be used for, there's a very high probability that the software will turn out badly." - Brian Kernighan

Focus first on understanding concrete data and algorithms. Write concrete implementations that could be deployed in production. Once working, refactor to decouple implementation from concrete data by giving data capability.

## Semantic Guidelines

### Core Rules
You must decide which semantic (value or pointer) to use for a particular data type at the time the type is declared. APIs must respect the semantic chosen for the type.

**Basic Guidelines:**
- At type declaration time, to decide the semantic
- Functions and methods must respect the semantic choice
- To avoid method receivers with different semantics than the type
- To avoid functions that accept/return data with different semantics
- To avoid changing semantics for a given type

**Exception:** Unmarshaling always requires pointer semantics.

### Built-In Types: Value Semantics

Go's built-in types (numeric, textual, boolean) should use value semantics. Not to use pointers unless you have a very good reason.

```go
// From strings package - all use value semantics
func Replace(s, old, new string, n int) string
func LastIndex(s, sep string) int
func ContainsRune(s string, r rune) bool
```

### Reference Types: Value Semantics

Reference types (slices, maps, interfaces, functions, channels) should use value semantics because they're designed to stay on the stack and minimize heap pressure.

```go
// From net package
type IP []byte
type IPMask []byte

// Value semantic API design
func (ip IP) Mask(mask IPMask) IP {
    // Creates and returns a new IP value
    n := len(ip)
    if n != len(mask) {
        return nil
    }
    out := make(IP, n)
    for i := 0; i < n; i++ {
        out[i] = ip[i] & mask[i]
    }
    return out
}

// append uses value semantics
var data []string
data = append(data, "string")  // Returns new slice value
```

### User-Defined Types: The Decision Point

This is where you must make the most decisions. The factory function for a type tells you which semantic was chosen.

#### Example: Time Type (Value Semantics)

```go
type Time struct {
    sec int64
    nsec int32
    loc *Location
}

// Factory function shows value semantics
func Now() Time {
    sec, nsec := now()
    return Time{sec + unixToInternal, nsec, Local}
}

// Method respects value semantics
func (t Time) Add(d Duration) Time {
    t.sec += int64(d / 1e9)
    nsec := t.nsec + int32(d%1e9)
    if nsec >= 1e9 {
        t.sec++
        nsec -= 1e9
    } else if nsec < 0 {
        t.sec--
        nsec += 1e9
    }
    t.nsec = nsec
    return t
}

// Function accepts value semantics
func div(t Time, d Duration) (qmod2 int, r Duration) {
    // Implementation...
}
```

Only unmarshal-related functions use pointer semantics:

```go
func (t *Time) UnmarshalBinary(data []byte) error
func (t *Time) GobDecode(data []byte) error
func (t *Time) UnmarshalJSON(data []byte) error
func (t *Time) UnmarshalText(data []byte) error
```

#### Example: File Type (Pointer Semantics)

```go
// Factory function shows pointer semantics
func Open(name string) (file *File, err error) {
    return OpenFile(name, O_RDONLY, 0)
}

// Methods respect pointer semantics
func (f *File) Chdir() error {
    if f == nil {
        return ErrInvalid
    }
    if e := syscall.Fchdir(f.fd); e != nil {
        return &PathError{"chdir", f.name, e}
    }
    return nil
}

// Functions accept pointer semantics
func epipecheck(file *File, e error) {
    if e == syscall.EPIPE {
        if atomic.AddInt32(&file.nepipe, 1) >= 10 {
            sigpipe()
        }
    } else {
        atomic.StoreInt32(&file.nepipe, 0)
    }
}
```

## Decision Framework

### When to Use Value Semantics
- Built-in types (int, string, bool, etc.)
- Reference types (slice, map, channel, interface, function)
- Small structs that can be reasonably copied
- When copying is correct and reasonable
- When you want to reduce GC pressure
- When isolation is beneficial

### When to Use Pointer Semantics
- Large structs where copying is expensive
- When changes need to be shared across functions
- When you're not 100% sure copying is reasonable
- Resource handles (files, network connections)
- When unmarshaling data
- When state must be shared

### The Golden Rule
> If you are not 100% sure it is correct or reasonable to make copies, then use pointer semantics.

## Consistency and Maintainability

### Code Review Focus
Consistent use of value/pointer semantics should be a focus in code reviews because it:
- Keeps code consistent and predictable
- Allows everyone to maintain a clear mental model
- Becomes more important as code base and team grow

### Language-Wide Impact
The choice between pointer and value semantics extends beyond receivers and function parameters. It appears throughout the language:
- How `for range` works
- Interface mechanics
- Function values
- Slice operations

## Key Takeaways

1. **To Decide at Declaration**: To choose semantics when you declare the type
2. **To Be Consistent**: APIs must respect the chosen semantic
3. **To Default to Value**: To use value semantics unless you have a good reason
4. **Data Drives Design**: To focus on data and its transformations
5. **Prototype First**: To start with concrete implementations
6. **Less is More**: Simplicity aids maintainability
7. **Mental Models Matter**: Consistency helps maintain mental models

## In Summary

- **Data-oriented design** starts with understanding the data and its transformations
- **Value semantics** reduce GC pressure but require copying
- **Pointer semantics** are efficient but can increase GC pressure
- **Consistency** is critical for maintainability and mental models
- **Built-in and reference types** should use value semantics
- **User-defined types** require deliberate semantic choices
- **Factory functions** reveal the intended semantic
- **Unmarshaling** is the primary exception to semantic rules

---

*This summary is based on Bill Kennedy's ["Design Philosophy On Data And Semantics"](https://www.ardanlabs.com/blog/2017/06/design-philosophy-on-data-and-semantics.html) from Ardan Labs, the final post in a four-part series covering Go's mechanics and design philosophy.*
