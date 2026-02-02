---
title: "Go Mechanics: Methods, Interfaces and Embedded Types"
date: 2026-02-02
tags: ["Go", "Methods", "Interfaces", "Embedded Types", "Composition"]
---

In Go, methods, interfaces, and embedded types create a powerful system for code organization using composition rather than inheritance. Here's a concise summary of these fundamental mechanics from William Kennedy's Ardan Labs article.

## Methods in Go

### What is a Method?
A method is a function with a **receiver** - a value or pointer of a named type. All methods for a type belong to its **method set**.

```go
type User struct {
    Name  string
    Email string
}

// Value receiver
func (u User) Notify() error { ... }

// Pointer receiver  
func (u *User) UpdateEmail(email string) { ... }
```

### Value vs Pointer Receivers

#### Value Receiver
- Operates on a **copy** of the receiver
- Cannot modify the original value
- Can be called with both values and pointers

#### Pointer Receiver
- Operates on the **original** value
- Can modify the original value
- Can be called with both values and pointers (Go handles dereferencing)

## Interfaces in Go

### Interface Declaration
```go
type Notifier interface {
    Notify() error
}
```

### Key Characteristics
- **Implicit Implementation**: Types implement interfaces automatically by satisfying method sets
- **Small Interfaces**: Most standard library interfaces have 1-2 methods
- **Naming Convention**: Single-method interfaces use `-er` suffix (Reader, Writer)

### Interface Compliance Rules

#### Critical Rule
> The method set of type T **does not** include methods with receiver type *T

**Consequences:**
- If method has **pointer receiver**, only **pointers** satisfy the interface
- If method has **value receiver**, both **values and pointers** satisfy the interface

```go
// Pointer receiver - only pointers work
func (u *User) Notify() error { ... }

user := User{"bill", "bill@email.com"}
SendNotification(user)  // ERROR: User doesn't implement Notifier

pUser := &User{"jill", "jill@email.com"}  
SendNotification(pUser) // OK: *User implements Notifier
```

## Embedded Types (Composition)

### What is Type Embedding?
Embedding allows anonymous fields in structs, creating composition:

```go
type Admin struct {
    User  // Embedded type
    Level string
}
```

### Method Promotion
When a type is embedded, its methods are **promoted** to the outer type:

```go
admin := &Admin{
    User: User{"john", "john@email.com"},
    Level: "super",
}

// Both work - method is promoted
admin.Notify()        // Calls User.Notify()
admin.User.Notify()   // Explicit call to embedded type
```

### Method Promotion Rules

#### Rule 1: Value Receiver Promotion
If S contains anonymous field T, both S and *S include promoted methods with receiver T.

#### Rule 2: Pointer Receiver Promotion  
Only *S includes promoted methods with receiver *T.

#### Rule 3: Pointer Embedding
If S contains anonymous field *T, both S and *S include methods with receiver T or *T.

### Derived Rule
> If S contains anonymous field T, the method set of S **does not** include promoted methods with receiver *T.

This is consistent with interface compliance rules.

## Interface Implementation with Embedded Types

### Multiple Implementations
When both outer and embedded types implement the same interface:

```go
// User implements Notifier
func (u *User) Notify() error { ... }

// Admin also implements Notifier
func (a *Admin) Notify() error { ... }
```

### Resolution Rules
1. **Outer Type Priority**: If outer type implements interface, it's used
2. **Inner Type Fallback**: If outer type doesn't implement, promoted methods are used
3. **No Conflicts**: Both implementations coexist uniquely

```go
admin := &Admin{...}
SendNotification(admin)  // Calls Admin.Notify()
admin.Notify()           // Calls Admin.Notify()
admin.User.Notify()      // Calls User.Notify()
```

## Answering Key Questions

### Q1: Compiler Error?
**No**. Embedded types use unique names for inner/outer implementations, allowing both to coexist.

### Q2: Implementation Selection?
**Priority-based selection**: Outer type implementation takes precedence, inner type available via explicit access.

## Best Practices

### 1. Interface Design
- Keep interfaces small and focused
- Discover interfaces from usage, don't design upfront

### 2. Receiver Selection
- Use value receivers for immutable operations
- Use pointer receivers for modifications or large structs

### 3. Composition Patterns
- Use embedding for code reuse, not inheritance hierarchies
- Prefer composition over inheritance

## Key Takeaways

1. **Implicit Implementation**: Types implement interfaces automatically
2. **Method Set Rules**: Value receivers work with both values/pointers; pointer receivers only with pointers
3. **Composition over Inheritance**: Use embedding for code reuse
4. **Small Interfaces**: Keep interfaces focused and minimal
5. **Method Promotion**: Governed by specific rules based on receiver types
6. **Priority Resolution**: Outer type implementations take precedence
7. **No Conflicts**: Multiple implementations can coexist uniquely

---

*This summary is based on William Kennedy's ["Methods, Interfaces and Embedded Types in Go"](https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html) from Ardan Labs.*
