---
title: "Go Mechanics: Context Package Semantics"
date: 2026-02-02
tags: ["Go", "Context", "Concurrency", "Timeouts", "Cancellation"]
description: "Understanding the essential semantics of Go's context package for proper cancellation, timeouts, and request-scoped data management."
---

In Go, the context package is essential for managing request-scoped data, cancellation signals, and deadlines across API boundaries and goroutines. Since Go has no built-in keywords for terminating goroutines, the context package provides the critical mechanism for managing service health and operation.

This post summarizes William Kennedy's core insights on context package semantics from Ardan Labs, focusing on the established rules and patterns for proper usage.

## Introduction: The Context Problem

Go provides the `go` keyword to create goroutines but offers no direct support for terminating them. In real-world services, the ability to timeout and terminate goroutines is critical for maintaining service health. No request should run forever - managing latency is every programmer's responsibility.

The context package, introduced by Sameer Ajmani in 2014, solves this fundamental problem by providing:
- **Deadline support**: Automatic cancellation at specific times
- **Cancellation signals**: Manual cancellation propagation
- **Request-scoped values**: Data that travels across API boundaries

## The Context Interface

The Context type is an interface with four key methods:

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

**Key conventions**:
- Use variable name `ctx` for all Context values
- Since Context is an interface, use value semantics (no pointers)
- Every function accepting a Context gets its own copy

## Semantic Rules for Context Usage

### Rule 1: Incoming requests should create a Context

Create Context as early as possible in request processing. This forces API design to accept Context as the first parameter.

#### HTTP Request Context Creation
```go
// Go 1.7+ - Context is already in the request
h := func(w http.ResponseWriter, r *http.Request, params map[string]string) {
    ctx, span := trace.StartSpan(r.Context(), "internal.platform.web")
    defer span.End()
    // ... handler logic
}

// Pre-Go 1.7 - Create empty Context manually
ctx := context.Background()
ctx, span := trace.StartSpan(ctx, "internal.platform.web")
defer span.End()
```

**Background Context**:
- `context.Background()` returns a non-nil, empty Context
- Never canceled, no values, no deadline
- Used by main function, initialization, tests, and as top-level Context for incoming requests

### Rule 2: Outgoing calls should accept a Context

Higher-level calls should tell lower-level calls how long they're willing to wait. This is essential for proper timeout propagation.

#### HTTP Client with Context
```go
func main() {
    // Create request
    req, err := http.NewRequest("GET", "https://api.example.com", nil)
    if err != nil {
        log.Println("ERROR:", err)
        return
    }

    // Create context with timeout
    ctx, cancel := context.WithTimeout(req.Context(), 50*time.Millisecond)
    defer cancel()

    // Bind context to request
    req = req.WithContext(ctx)

    // Make call - Do method respects the timeout
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        log.Println("ERROR:", err)
        return
    }
    defer resp.Body.Close()

    // Process response
    io.Copy(os.Stdout, resp.Body)
}
```

**Key Points**:
- Higher-level function tells lower-level function how long to wait
- `http.Do()` respects the timeout in the Context
- Context enables proper timeout propagation across API boundaries

### Rule 3: Don't store Contexts in structs; pass explicitly

Functions performing I/O should accept Context as their first parameter and respect timeout/deadline configurations.

#### Correct Pattern
```go
// Database function accepting Context
func List(ctx context.Context, db *sqlx.DB) ([]User, error) {
    ctx, span := trace.StartSpan(ctx, "internal.user.List")
    defer span.End()

    users := []User{}
    const q = `SELECT * FROM users`

    if err := db.SelectContext(ctx, &users, q); err != nil {
        return nil, errors.Wrap(err, "selecting users")
    }

    return users, nil
}

// Handler calling database function
func (u *User) List(ctx context.Context, w http.ResponseWriter, r *http.Request, params map[string]string) error {
    ctx, span := trace.StartSpan(ctx, "handlers.User.List")
    defer span.End()

    users, err := user.List(ctx, u.db)  // Context propagated
    if err != nil {
        return err
    }

    return web.Respond(ctx, w, users, http.StatusOK)
}
```

**Why not store in structs?**
- Context is request-scoped, not struct-scoped
- Explicit passing makes dependencies clear
- Avoids hidden dependencies and lifetime issues

### Rule 4: Propagate Context through call chains

Context and any changes made during request processing must be propagated and respected throughout the call chain.

#### Propagation Pattern
```go
// Handler level
func (u *User) List(ctx context.Context, w http.ResponseWriter, r *http.Request, params map[string]string) error {
    ctx, span := trace.StartSpan(ctx, "handlers.User.List")
    defer span.End()

    users, err := user.List(ctx, u.db)  // Propagate Context
    if err != nil {
        return err
    }

    return web.Respond(ctx, w, users, http.StatusOK)  // Propagate Context
}

// Business logic level
func List(ctx context.Context, db *sqlx.DB) ([]User, error) {
    ctx, span := trace.StartSpan(ctx, "internal.user.List")
    defer span.End()

    users := []User{}
    const q = `SELECT * FROM users`

    if err := db.SelectContext(ctx, &users, q); err != nil {  // Propagate Context
        return nil, errors.Wrap(err, "selecting users")
    }

    return users, nil
}
```

**Important**: Don't create new top-level Context values in middle functions - this loses existing Context information from higher-level calls.

### Rule 5: Replace Context using With functions

Context uses value semantics - any change creates a new Context value. Use the appropriate With function for modifications.

#### Context Creation Patterns
```go
func main() {
    // Create timeout context
    duration := 150 * time.Millisecond
    ctx, cancel := context.WithTimeout(context.Background(), duration)
    defer cancel()  // CRITICAL: Always call cancel

    // Create channel for work result
    ch := make(chan data, 1)

    // Start goroutine work
    go func() {
        time.Sleep(50 * time.Millisecond)  // Simulate work
        ch <- data{"123"}
    }()

    // Wait for work with timeout
    select {
    case d := <-ch:
        fmt.Println("work complete", d)
    case <-ctx.Done():
        fmt.Println("work cancelled")
    }
}
```

**Critical Rules**:
- Always call the cancel function (use `defer`)
- Use appropriate With function for your need:
  - `WithCancel`: Manual cancellation
  - `WithDeadline`: Cancellation at specific time
  - `WithTimeout`: Cancellation after duration
  - `WithValue`: Store request-scoped values

### Rule 6: Cancellation propagates to derived Contexts

When a parent Context is canceled, all Contexts derived from it are also canceled. This enables efficient cleanup of entire request trees.

#### Cancellation Propagation
```go
func main() {
    // Create cancellable Context
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    var wg sync.WaitGroup
    wg.Add(10)

    // Create 10 goroutines with derived Contexts
    for i := 0; i < 10; i++ {
        go func(id int) {
            defer wg.Done()

            // Derive Context with unique value
            ctx := context.WithValue(ctx, key, id)

            // Wait for cancellation
            <-ctx.Done()
            fmt.Println("Cancelled:", id)
        }(i)
    }

    // Cancel all derived Contexts with one call
    cancel()
    wg.Wait()
}
```

**Key Benefits**:
- One cancel call cancels all derived Contexts
- Context is safe for simultaneous use by multiple goroutines
- Efficient cleanup of entire request trees

### Rule 7: Never pass nil Context; use TODO when unsure

Always pass a valid Context. If unsure which Context to use, use `context.TODO()` instead of `nil`.

#### TODO Usage
```go
// When you know you need a Context but aren't sure where it comes from
func processData(data []byte) error {
    ctx := context.TODO()  // Temporary until you figure out the source
    return database.Save(ctx, data)
}

// Later, when you know the Context source
func processData(ctx context.Context, data []byte) error {
    return database.Save(ctx, data)
}
```

**Background vs TODO**:
- `Background`: For top-level Context (main, init, tests)
- `TODO`: When you need a Context temporarily during development

### Rule 8: Use Context values only for request-scoped data

This is the most critical semantic rule. Don't use Context to pass required function parameters.

#### What NOT to do - Required Data in Context
```go
// ANTI-PATTERN: Database connection in Context
func List(ctx context.Context) ([]User, error) {
    db := ctx.Value("database").(*sqlx.DB)  // Hidden dependency
    
    users := []User{}
    const q = `SELECT * FROM users`
    
    if err := db.SelectContext(ctx, &users, q); err != nil {
        return nil, err
    }
    
    return users, nil
}
```

#### What TO do - Explicit Dependencies
```go
// GOOD PATTERN: Database connection as parameter
func List(ctx context.Context, db *sqlx.DB) ([]User, error) {
    users := []User{}
    const q = `SELECT * FROM users`
    
    if err := db.SelectContext(ctx, &users, q); err != nil {
        return nil, err
    }
    
    return users, nil
}

// When signature can't be changed (like HTTP handlers)
func (u *User) List(ctx context.Context, w http.ResponseWriter, r *http.Request, params map[string]string) error {
    users, err := user.List(ctx, u.db)  // db from receiver
    if err != nil {
        return err
    }
    return web.Respond(ctx, w, users, http.StatusOK)
}

type User struct {
    db *sqlx.DB  // Explicit dependency
    authenticator *auth.Authenticator
}
```

#### Data Passing Priority
1. **Function parameters** - Clearest, no hidden dependencies
2. **Receiver fields** - When signature can't be changed
3. **Context values** - Only for request-scoped data

#### Safe Context Values
Request-scoped data that's safe for Context:
- **Trace IDs**: For distributed tracing
- **Request start time**: For performance monitoring
- **Authentication tokens**: When they transit API boundaries
- **Correlation IDs**: For request tracking

```go
// GOOD: Request-scoped tracing data
type Values struct {
    TraceID   string
    StartTime time.Time
    StatusCode int
}

// Store in Context for middleware and logging
v := Values{
    TraceID:   span.SpanContext().TraceID.String(),
    StartTime: time.Now(),
}
ctx = context.WithValue(ctx, KeyValues, &v)
```

## Context Value Safety

When using Context values, always check for integrity:

```go
// Safe Context value retrieval
func logRequest(ctx context.Context, r *http.Request) {
    v, ok := ctx.Value(KeyValues).(*Values)
    if !ok {
        // Major integrity issue - shutdown service
        return web.NewShutdownError("web value missing from context")
    }
    
    log.Printf("%s : %s %s -> %s (%s)",
        v.TraceID,
        r.Method, r.URL.Path,
        r.RemoteAddr,
        time.Since(v.StartTime),
    )
}
```

**Consequences of misuse**:
- Need integrity checking and shutdown mechanisms
- Testing and debugging become harder
- Reduced code clarity and readability

## Key Takeaways

1. **Create Context early** in request processing
2. **Accept Context explicitly** as first parameter
3. **Don't store Context in structs** - pass explicitly
4. **Propagate Context** through entire call chain
5. **Use With functions** to create derived Contexts
6. **Always call cancel functions** with defer
7. **Cancellation propagates** to all derived Contexts
8. **Never pass nil Context** - use TODO when unsure
9. **Use Context values only for request-scoped data**, not required parameters
10. **Prioritize explicit dependencies** over hidden Context dependencies

## In Summary

- **Context manages request lifecycle** through deadlines, cancellation, and values
- **Value semantics ensure** changes don't affect existing Context references
- **Propagation patterns enable** proper timeout and cancellation behavior
- **Explicit dependencies** provide better clarity than hidden Context values
- **Request-scoped data** like trace IDs are appropriate for Context values
- **Proper usage patterns** are essential for reliable, maintainable Go services

---

*Based on William Kennedy's ["Context Package Semantics In Go"](https://www.ardanlabs.com/blog/2019/09/context-package-semantics-in-go.html) from Ardan Labs.*
