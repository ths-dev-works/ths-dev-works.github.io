---
title: "Go Mechanics: Custom Error Types - Part II"
date: 2026-02-02
tags: ["Go", "Error Handling", "Custom Types", "Best Practices"]
---

In Go, custom error types provide additional context when simple error messages aren't sufficient. Here's a concise summary of custom error types from William Kennedy's Ardan Labs article.

## When to Use Custom Error Types

### Use Simple Errors When
- Error message provides enough context
- No additional state is needed
- Caller doesn't need to make decisions based on error type

### Use Custom Error Types When
- Caller needs extra context for informed decisions
- Error requires associated state
- You need to wrap original errors with context

## The net Package Example: OpError

### Custom Error Type Declaration
```go
type OpError struct {
    Op   string // Operation that caused error ("read", "write")
    Net  string // Network type ("tcp", "udp6")
    Addr Addr   // Network address where error occurred
    Err  error  // The actual error that occurred
}
```

### Error Interface Implementation
```go
func (e *OpError) Error() string {
    if e == nil {
        return "<nil>"
    }
    s := e.Op
    if e.Net != "" {
        s += " " + e.Net
    }
    if e.Addr != nil {
        s += " " + e.Addr.String()
    }
    s += ": " + e.Err.Error()
    return s
}
```

### Usage Pattern
```go
func Listen(net, laddr string) (Listener, error) {
    la, err := resolveAddr("listen", net, laddr, noDeadline)
    if err != nil {
        return nil, &OpError{
            Op:   "listen", 
            Net:  net, 
            Addr: nil, 
            Err:  err
        }
    }
    // ... rest of implementation
}
```

## The json Package Example: Multiple Error Types

### UnmarshalTypeError
```go
type UnmarshalTypeError struct {
    Value string       // Description of JSON value
    Type  reflect.Type // Go type it couldn't be assigned to
}

func (e *UnmarshalTypeError) Error() string {
    return "json: cannot unmarshal " + e.Value + 
           " into Go value of type " + e.Type.String()
}
```

### InvalidUnmarshalError
```go
type InvalidUnmarshalError struct {
    Type reflect.Type
}

func (e *InvalidUnmarshalError) Error() string {
    if e.Type == nil {
        return "json: Unmarshal(nil)"
    }
    if e.Type.Kind() != reflect.Ptr {
        return "json: Unmarshal(non-pointer " + e.Type.String() + ")"
    }
    return "json: Unmarshal(nil " + e.Type.String() + ")"
}
```

## Identifying Concrete Error Types

### Type Switch Pattern
```go
err := json.Unmarshal(data, &v)
if err != nil {
    switch e := err.(type) {
    case *json.UnmarshalTypeError:
        log.Printf("Type Error: Value[%s] Type[%v]", e.Value, e.Type)
        // Handle type mismatch
    case *json.InvalidUnmarshalError:
        log.Printf("Invalid Unmarshal: Type[%v]", e.Type)
        // Handle invalid argument
    default:
        log.Println(err)
        // Handle other errors
    }
    return
}
```

### Key Benefits of Type Switch
- **Type Identification**: Determine concrete error type
- **State Access**: Access error-specific fields
- **Targeted Handling**: Handle different errors appropriately

## Design Patterns

### 1. Wrapper Pattern (net package)
- Wrap original error with context
- Add operation, network, address information
- Preserve original error in `Err` field

### 2. Context Pattern (json package)
- Type name provides error context
- Include relevant state in struct fields
- Generate descriptive error messages

### 3. Naming Convention
- Postfix custom error types with "Error"
- Examples: `OpError`, `UnmarshalTypeError`, `InvalidUnmarshalError`

## Best Practices

### 1. Start Simple
```go
// Use this when possible
return errors.New("mypackage: operation failed")
```

### 2. Add Context When Needed
```go
// Use this when context matters
return &MyError{
    Operation: "save",
    Context:   userData,
    Err:       originalErr,
}
```

### 3. Implement Error() Method
```go
func (e *MyError) Error() string {
    return fmt.Sprintf("mypackage: %s operation failed: %v", 
        e.Operation, e.Err)
}
```

### 4. Export Error Types
- Export custom error types as part of your API
- Allow callers to use type switches
- Document when each error type occurs

## Decision Framework

### Questions to Ask
1. **Does the caller need to make decisions based on error type?**
   - Yes → Consider custom error type
   - No → Use simple error

2. **Do you need to provide additional state?**
   - Yes → Custom error type with fields
   - No → Simple error message

3. **Are you wrapping another error with context?**
   - Yes → Wrapper pattern like OpError
   - No → Context pattern like json errors

## Key Takeaways

1. **Start Simple**: Use `errors.New()` and `fmt.Errorf()` first
2. **Add Context**: Create custom types when callers need more information
3. **Follow Patterns**: Use wrapper or context patterns from standard library
4. **Type Switches**: Enable callers to identify and handle specific errors
5. **Naming Convention**: End custom error types with "Error"
6. **Export Types**: Make error types part of your public API
7. **Preserve Original**: Wrap original errors, don't replace them

---

*This summary is based on William Kennedy's ["Error Handling In Go, Part II"](https://www.ardanlabs.com/blog/2014/11/error-handling-in-go-part-ii.html) from Ardan Labs.*
