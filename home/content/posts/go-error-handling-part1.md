---
title: "Go Mechanics: Error Handling Fundamentals - Part I"
date: 2026-02-02
tags: ["Go", "Error Handling", "Interface", "Best Practices"]
---

In Go, error handling uses a simple interface-based approach. Here's a concise summary of Go's error handling fundamentals from William Kennedy's Ardan Labs article.

## The Error Interface

```go
type error interface {
    Error() string
}
```

### Key Characteristics
- **Single Method**: Only requires `Error()` returning string
- **Universal**: Used throughout standard library
- **Flexible**: Any type implementing `Error()` can be an error

### Standard Library Implementation
- **`errorString` struct**: Unexported type with pointer receiver
- **Pointer semantics**: Ensures unique error values
- **Encapsulation**: Access only via `Error()` method

## Creating Error Values

### `errors.New()` for Simple Messages
```go
var ErrInvalidParam = errors.New("mypackage: invalid parameter")
```

### `fmt.Errorf()` for Formatted Messages
```go
var ErrInvalidParam = fmt.Errorf("invalid parameter [%s]", param)
```

Both create `errorString` pointers under the hood.

## Error Handling Pattern

```go
resp, err := c.Get(url)
if err != nil {
    log.Println(err)
    return
}
```

### Key Principles
- **Check Immediately**: Always check errors after function calls
- **Return Early**: Use early returns to avoid nested ifs
- **Propagate Up**: Let errors bubble up to appropriate handlers

## Standard Library Error Variables

Many packages export predefined error variables:

```go
// bufio package
var (
    ErrInvalidUnreadByte = errors.New("bufio: invalid use of UnreadByte")
    ErrBufferFull        = errors.New("bufio: buffer full")
)

// io package
var (
    EOF              = errors.New("EOF")
    ErrUnexpectedEOF = errors.New("unexpected EOF")
)
```

## Comparing Error Values

### Use Package Variables (Recommended)
```go
switch err {
case bufio.ErrNegativeCount:
    // Handle specific error
case bufio.ErrBufferFull:
    // Handle specific error
}
```

### Avoid String Comparison
```go
// ANTI-PATTERN: Fragile and slow!
switch err.Error() {
case "bufio: negative count":  // Breaks if message changes
```

## Design Philosophy: Pointer Receivers

### Why Pointer Receivers?
- **Uniqueness**: Each call creates unique pointer value
- **Identity**: Error identity based on pointer, not content
- **API Stability**: Prevents accidental equality matches

### Named Type Problems
Named types would allow different instances with same message to compare equal, creating unwanted behavior.

## Best Practices

### 1. Create Package Error Variables
```go
var (
    ErrInvalidInput = errors.New("mypackage: invalid input")
    ErrTimeout      = errors.New("mypackage: operation timeout")
)
```

### 2. Use Descriptive Messages
```go
// Good: Includes package prefix
ErrInvalidInput = errors.New("mypackage: invalid input parameter")

// Bad: Generic message
ErrInvalidInput = errors.New("invalid input")
```

### 3. Follow Naming Conventions
- **Prefix**: Use package name as prefix
- **Err Prefix**: Start error variables with `Err`
- **Export**: Export error variables as part of API

## Key Takeaways

1. **Interface-Based Design**: Simple, elegant interface for maximum flexibility
2. **Pointer Semantics**: Using pointer receivers ensures error uniqueness
3. **Package Variables**: Export predefined error variables as part of API
4. **Early Returns**: Handle errors immediately and return early
5. **Descriptive Messages**: Include package context in error messages
6. **Avoid String Comparison**: Use error variables, not message strings
7. **Performance**: Pointer-based comparison is fast and efficient

---

*This summary is based on William Kennedy's ["Error Handling In Go, Part I"](https://www.ardanlabs.com/blog/2014/10/error-handling-in-go-part-i.html) from Ardan Labs.*
