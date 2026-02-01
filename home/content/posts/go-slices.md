---
title: "Go Mechanics: Understanding Slices and Internals"
date: 2026-02-01
tags: ["Go", "Internals", "Memory", "Data Structures"]
---

In Go, slices are one of the most powerful and frequently used data structures. They provide a flexible and efficient way to work with sequences of data. Here's a comprehensive summary of slice mechanics based on the official Go blog and Ardan Labs insights.

## Arrays: The Foundation

Before understanding slices, we must understand arrays. Arrays in Go are fixed-size collections with the size being part of the type.

### Array Properties
- **Fixed Size**: `[4]int` and `[5]int` to be different, incompatible types
- **Value Type**: Arrays to be values, not pointers to first element
- **Memory Layout**: Elements laid out sequentially in memory
- **Copy Semantics**: Assignment to create a full copy of the array

```go
// Array declarations
var a [4]int
a[0] = 1
i := a[0] // i == 1

// Array literals
b := [2]string{"Penn", "Teller"}
c := [...]string{"Penn", "Teller"} // Compiler counts elements
```

## Slices: The Abstraction

Slices build on arrays to provide flexibility and power. A slice is a **descriptor** of an array segment.

### Slice Type Specification
```go
[]T // Where T is the element type
```

### Slice Creation Methods

#### 1. Slice Literals
```go
letters := []string{"a", "b", "c", "d"}
```

#### 2. Using make()
```go
// make([]T, length, capacity)
s := make([]byte, 5, 5) // len=5, cap=5
s := make([]byte, 5)    // len=5, cap=5 (capacity defaults to length)
```

#### 3. Slicing Existing Arrays/Slices
```go
// From array
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // slice referencing array storage

// From slice
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
slice := b[1:4] // []byte{'o', 'l', 'a'}, shares storage
```

## Slice Internals: The Three-Part Structure

A slice consists of three components:

1. **Pointer**: Reference to the underlying array
2. **Length**: Number of elements in the slice
3. **Capacity**: Maximum number of elements in the underlying array

### Memory Visualization

```
┌─────────────────────────────────────────────────────────┐
│                    Slice Header                          │
├─────────────────┬─────────────────┬─────────────────────┤
│   Pointer       │     Length      │     Capacity        │
│   (0x10430000)  │       5         │         8           │
└─────────────────┴─────────────────┴─────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────┐
│                  Underlying Array                       │
├─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┤
│ [0] │ [1] │ [2] │ [3] │ [4] │ [5] │ [6] │ [7] │ ... │
│ 10  │ 20  │ 30  │ 40  │ 50  │  0  │  0  │  0  │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
      ↑─────────────────↑─────────────────────────────────┐
      Accessible Elements    Reserved Capacity            │
      (Length = 5)           (Capacity = 8)               │
```

## Length vs. Capacity

### Length (`len()`)
- Number of elements currently accessible
- Always ≤ capacity
- Used for bounds checking

### Capacity (`cap()`)
- Total available space in underlying array
- Determines when reallocation is needed
- Can be larger than length

```go
s := make([]int, 3, 8) // len=3, cap=8
fmt.Printf("len=%d, cap=%d\n", len(s), cap(s))
// Output: len=3, cap=8
```

## Slice Operations and Behavior

### Slicing Operations
```go
// Original slice
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}

// Various slicing operations
b[:2]  // []byte{'g', 'o'}
b[2:]  // []byte{'l', 'a', 'n', 'g'}
b[:]   // []byte{'g', 'o', 'l', 'a', 'n', 'g'} (full copy of slice header)
b[1:4] // []byte{'o', 'l', 'a'}
```

### Shared Storage Warning
Slicing does **not** copy data - it creates a new slice header pointing to the same array:

```go
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:] // e == []byte{'a', 'd'}
e[1] = 'm' // e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'} - original is modified!
```

## Growing Slices: Append and Reallocation

### The Append Function
Go's built-in `append` function handles slice growth automatically:

```go
func append(s []T, x ...T) []T

// Basic usage
a := make([]int, 1) // []int{0}
a = append(a, 1, 2, 3) // []int{0, 1, 2, 3}

// Append another slice
b := []string{"John", "Paul"}
c := []string{"George", "Ringo"}
b = append(b, c...) // []string{"John", "Paul", "George", "Ringo"}
```

### Growth Strategy
When capacity is insufficient, `append`:
1. Allocates a new, larger array
2. Copies existing elements to the new array
3. Adds new elements
4. Returns a new slice header

### Manual Growth Control
For precise control over growth:

```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    
    if n > cap(slice) {
        // Reallocate: double what's needed for future growth
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```

## The Copy Function

The built-in `copy` function efficiently copies data between slices:

```go
func copy(dst, src []T) int

// Usage
t := make([]byte, len(s), (cap(s)+1)*2)
copied := copy(t, s) // returns number of elements copied
```

### Copy Properties
- Copies up to the minimum of len(dst) and len(src)
- Handles overlapping slices correctly
- More efficient than manual element-by-element copying

## Common Patterns and Gotchas

### 1. Nil Slice vs Empty Slice
```go
var s []byte     // nil slice, len=0, cap=0
t := []byte{}    // empty slice, len=0, cap=0
u := make([]byte, 0) // empty slice, len=0, cap=0

// All behave similarly with append
s = append(s, 1) // works fine
```

### 2. Slice Truncation
```go
s := []int{1, 2, 3, 4, 5}
s = s[:3] // truncate to first 3 elements
// Underlying array still contains all 5 elements
```

### 3. Multi-dimensional Slices
```go
// Slice of slices
board := [][]int{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
}
```

## Performance Considerations

### Memory Efficiency
- **Slices share storage**: Multiple slices can reference the same array
- **Copy-on-grow**: Only when capacity is exceeded
- **Header copying**: Slice headers are small (3 words)

### Cache Performance
- **Contiguous memory**: Better cache locality than linked lists
- **Predictable access**: Sequential access patterns are fast

### Allocation Patterns
```go
// Good: Pre-allocate known capacity
s := make([]int, 0, 1000) // Avoid multiple reallocations

// Less efficient: Let append grow automatically
var s []int
for i := 0; i < 1000; i++ {
    s = append(s, i) // May cause multiple reallocations
}
```

## Advanced Slice Techniques

### 1. Filtering with Append
```go
func Filter(s []int, fn func(int) bool) []int {
    var p []int // nil slice
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

### 2. In-place Modification
```go
// Remove elements while preserving order
func Remove(slice []int, i int) []int {
    return append(slice[:i], slice[i+1:]...)
}

// Remove elements without preserving order (faster)
func RemoveUnordered(slice []int, i int) []int {
    slice[i] = slice[len(slice)-1]
    return slice[:len(slice)-1]
}
```

### 3. Stack Implementation
```go
type Stack []int

func (s *Stack) Push(v int) {
    *s = append(*s, v)
}

func (s *Stack) Pop() (int, bool) {
    if len(*s) == 0 {
        return 0, false
    }
    index := len(*s) - 1
    element := (*s)[index]
    *s = (*s)[:index]
    return element, true
}
```

## Key Takeaways

1. **Slices are headers**: Pointer + length + capacity, not the data itself
2. **Shared storage**: Multiple slices can reference the same underlying array
3. **Growth triggers reallocation**: When length exceeds capacity
4. **Append is your friend**: Handles most growth scenarios automatically
5. **Copy for explicit transfers**: Use when you need precise control
6. **Nil slices are useful**: Start with nil and let append handle allocation
7. **Capacity planning**: Pre-allocate when you know the size for better performance

## In Summary

- **Slices provide a window** into an underlying array through a three-part header
- **Length vs. Capacity**: Length is what you can access, capacity is what's available
- **Shared storage means** modifications through one slice affect all slices sharing that array
- **Append handles growth** by allocating new arrays when needed
- **Copy provides efficient** element transfer between slices
- **Performance comes from** contiguous memory and minimal copying
- **Pre-allocation prevents** multiple reallocations in growth scenarios

---

*This summary is based on the official Go blog post ["Go Slices: usage and internals"](https://go.dev/blog/slices-intro) and William Kennedy's ["Understanding Slices in Go Programming"](https://www.ardanlabs.com/blog/2013/08/understanding-slices-in-go-programming.html) from Ardan Labs.*
