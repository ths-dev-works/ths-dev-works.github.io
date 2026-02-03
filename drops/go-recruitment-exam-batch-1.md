---
title: "Go Recruitment Exam - Batch 1"
date: 2026-02-02T22:00:00Z
tags: ["go", "programming", "interview", "exam", "ardanlabs"]
categories: ["Ultimate Go Series"]
description: "Full Go Recruitment Exam - Batch 1 of 4. 100 multiple-choice questions following Ardan Labs style."
---

Welcome to the first batch of our comprehensive Go Recruitment Exam. This exam consists of 100 multiple-choice questions designed to test your knowledge of Go programming fundamentals, advanced concepts, and best practices following the Ardan Labs methodology.

## Instructions

- Read each question carefully
- Choose the best answer from the provided options
- Answers are provided at the end of this post
- Time limit: 90 minutes recommended

---

## Questions 1-25: Go Fundamentals

**1. What is the zero value for a slice in Go?**
A) nil
B) [] (empty slice)
C) []{} (slice with zero elements)
D) panic

**2. Which of the following is NOT a valid way to declare a variable in Go?**
A) var x int = 10
B) x := 10
C) var x = 10
D) int x = 10

**3. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var s []int
    fmt.Println(len(s), cap(s))
}
```
A) 0 0
B) 0 nil
C) nil nil
D) panic

**4. Which statement about Go's garbage collection is true?**
A) Go uses reference counting
B) Go uses a concurrent mark-and-sweep garbage collector
C) Go requires manual memory management
D) Go does not have garbage collection

**5. What is the purpose of the `init()` function in Go?**
A) To initialize package-level variables
B) To serve as the main entry point
C) To handle errors
D) To clean up resources

**6. Which of the following is a valid Go keyword?**
A) class
B) inherit
C) defer
D) virtual

**7. What does the `make()` function do?**
A) Creates new values of maps, slices, and channels
B) Allocates memory for any type
C) Initializes variables
D) Compiles the program

**8. Which statement about Go interfaces is correct?**
A) Interfaces are explicitly implemented
B) Interfaces are implicitly satisfied
C) Interfaces can only contain methods
D) Interfaces cannot be nested

**9. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    x := []int{1, 2, 3}
    y := x[1:2]
    y[0] = 99
    fmt.Println(x)
}
```
A) [1 2 3]
B) [1 99 3]
C) [99 2 3]
D) panic

**10. Which of the following is NOT a built-in type in Go?**
A) int
B) string
C) map
D) list

**11. What is the zero value for a map in Go?**
A) nil
B) empty map
C) map{} 
D) panic

**12. Which statement about Go's `range` clause is correct?**
A) It always returns index and value
B) It can return only index or only value
C) It works on all types
D) It modifies the original collection

**13. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int, 1)
    ch <- 1
    fmt.Println(<-ch)
}
```
A) 1
B) deadlock
C) panic
D) compilation error

**14. Which of the following is a valid Go package name?**
A) 1package
B) my-package
C) my_package
D) my package

**15. What does the `select` statement do in Go?**
A) Chooses between multiple channel operations
B) Selects elements from a slice
C) Handles multiple conditions
D) Filters data

**16. Which statement about Go structs is correct?**
A) Structs can have methods
B) Structs support inheritance
C) Structs are reference types
D) Structs cannot be compared

**17. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var i interface{} = "hello"
    switch v := i.(type) {
    case string:
        fmt.Println(v)
    default:
        fmt.Println("unknown")
    }
}
```
A) hello
B) unknown
C) panic
D) compilation error

**18. Which of the following is NOT a valid Go control flow statement?**
A) if
B) switch
C) for
D) while

**19. What is the purpose of the `defer` statement?**
A) To execute a function call after the surrounding function returns
B) To delay execution indefinitely
C) To handle errors
D) To optimize performance

**20. Which statement about Go's error handling is correct?**
A) Go uses exceptions for error handling
B) Go uses error values returned from functions
C) Go ignores errors by default
D) Go requires try-catch blocks

**21. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    x := 5
    fmt.Println(&x)
}
```
A) 5
B) memory address
C) compilation error
D) panic

**22. Which of the following is not a valid Go constant declaration?**
A) const x = 10
B) const x int = 10
C) const x := 10
D) All of the above

**23. What is the zero value for a pointer in Go?**
A) nil
B) 0
C) empty
D) panic

**24. Which statement about Go's `append()` function is correct?**
A) It always creates a new slice
B) It may modify the original slice if capacity allows
C) It only works on slices of the same type
D) It returns a boolean indicating success

**25. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    m := make(map[string]int)
    m["a"] = 1
    fmt.Println(m["b"])
}
```
A) 0
B) nil
C) panic
D) key not found error

---

## Questions 26-50: Intermediate Go Concepts

**26. Which statement about Go's goroutines is correct?**
A) Goroutines are heavier than threads
B) Goroutines are managed by the Go runtime
C) Goroutines require explicit creation of threads
D) Goroutines cannot communicate with each other

**27. What is the purpose of the `sync.Mutex` type?**
A) To synchronize access to shared resources
B) To create goroutines
C) To handle channels
D) To manage memory

**28. Which of the following is a valid way to close a channel?**
A) close(ch)
B) ch.close()
C) ch <- nil
D) Channels cannot be closed

**29. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int)
    go func() {
        ch <- 1
    }()
    fmt.Println(<-ch)
}
```
A) 1
B) deadlock
C) panic
D) compilation error

**30. Which statement about Go's `context` package is correct?**
A) It's used for handling HTTP requests only
B) It provides cancellation and timeout capabilities
C) It replaces error handling
D) It's only available in Go 1.18+

**31. What is the output of this code?**
```go
package main
import "fmt"
func add(a, b int) int {
    return a + b
}
func main() {
    f := add
    fmt.Println(f(2, 3))
}
```
A) 5
B) compilation error
C) panic
D) 0

**32. Which of the following is NOT a valid Go method receiver?**
A) func (s *Struct) Method()
B) func (s Struct) Method()
C) func (Struct) Method()
D) func (s map[string]int) Method()

**33. What is the purpose of the `interface{}` type in Go?**
A) To represent any type
B) To define empty interfaces
C) To replace all other types
D) To improve performance

**34. Which statement about Go's `panic()` and `recover()` is correct?**
A) panic() always terminates the program
B) recover() can only be called in a deferred function
C) panic() should be used for normal error handling
D) recover() can catch any type of error

**35. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = 10
    if i, ok := x.(int); ok {
        fmt.Println(i * 2)
    }
}
```
A) 20
B) 10
C) panic
D) compilation error

**36. Which of the following is a valid Go tag?**
A) `json:"name"`
B) `json:name`
C) `json='name'`
D) `json(name)`

**37. What is the purpose of the `sync.WaitGroup` type?**
A) To wait for a collection of goroutines to finish
B) To synchronize access to shared resources
C) To create goroutines
D) To handle channels

**38. Which statement about Go's `reflect` package is correct?**
A) It's used for runtime type inspection
B) It's faster than static typing
C) It's recommended for regular operations
D) It can modify private fields

**39. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    s := []int{1, 2, 3, 4, 5}
    s = append(s[:2], s[3:]...)
    fmt.Println(s)
}
```
A) [1 2 4 5]
B) [1 2 3 4 5]
C) [1 2 3 5]
D) panic

**40. Which of the following is NOT a valid Go build tag?**
A) // +build linux
B) // +build !windows
C) // +build linux,amd64
D) // +build "linux"

**41. What is the purpose of the `runtime.GOMAXPROCS()` function?**
A) To set the maximum number of CPUs
B) To limit memory usage
C) To control goroutine scheduling
D) To optimize garbage collection

**42. Which statement about Go's `time` package is correct?**
A) It only works with UTC time
B) It provides duration and time types
C) It cannot handle time zones
D) It's deprecated in Go 1.20

**43. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int, 2)
    ch <- 1
    ch <- 2
    close(ch)
    for v := range ch {
        fmt.Println(v)
    }
}
```
A) 1 2
B) 1 2 panic
C) deadlock
D) compilation error

**44. Which of the following is a valid Go embedding syntax?**
A) type A struct { B }
B) type A struct { *B }
C) type A struct { B; C }
D) All of the above

**45. What is the purpose of the `io.Reader` interface?**
A) To read data from a source
B) To write data to a destination
C) To handle both read and write operations
D) To manage file operations

**46. Which statement about Go's `sort` package is correct?**
A) It only sorts slices
B) It can sort any collection that implements sort.Interface
C) It uses bubble sort by default
D) It cannot sort custom types

**47. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    x := []string{"a", "b", "c"}
    for i, v := range x {
        if i == 1 {
            x = append(x, "d")
        }
        fmt.Println(i, v)
    }
}
```
A) 0 a 1 b 2 c
B) 0 a 1 b 2 c 3 d
C) 0 a 1 b 2 c
D) infinite loop

**48. Which of the following is NOT a valid Go escape sequence?**
A) \n
B) \t
C) \v
D) \d

**49. What is the purpose of the `unsafe` package in Go?**
A) To bypass type safety
B) To improve performance
C) To handle memory management
D) To provide security features

**50. Which statement about Go's `testing` package is correct?**
A) It only supports unit tests
B) It provides benchmarking capabilities
C) It cannot test concurrent code
D) It requires external frameworks

---

## Questions 51-75: Advanced Go Concepts

**51. Which statement about Go's type assertion is correct?**
A) It always succeeds
B) It can panic if the type is wrong
C) It's only used with interfaces
D) It's a compile-time operation

**52. What is the purpose of the `sync.Pool` type?**
A) To reuse objects to reduce garbage collection pressure
B) To synchronize access to shared resources
C) To manage memory pools
D) To create object pools

**53. Which of the following is a valid Go generic type constraint?**
A) any
B) comparable
C) interface{ int | string }
D) All of the above

**54. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var f func() int
    fmt.Println(f == nil)
}
```
A) true
B) false
C) panic
D) compilation error

**55. Which statement about Go's `cgo` is correct?**
A) It allows calling C code from Go
B) It's automatically enabled
C) It's faster than pure Go
D) It's recommended for all projects

**56. What is the purpose of the `context.WithCancel()` function?**
A) To create a cancelable context
B) To cancel all operations
C) To handle timeouts
D) To manage deadlines

**57. Which of the following is NOT a valid Go channel direction?**
A) chan<- int
B) <-chan int
C) chan int
D) int<-chan

**58. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int)
    select {
    case <-ch:
        fmt.Println("received")
    default:
        fmt.Println("default")
    }
}
```
A) received
B) default
C) deadlock
D) panic

**59. Which statement about Go's `runtime` package is correct?**
A) It provides low-level runtime information
B) It's only for debugging
C) It's deprecated
D) It's not thread-safe

**60. What is the purpose of the `encoding/json` package?**
A) To encode and decode JSON data
B) To validate JSON
C) To parse XML
D) To handle HTTP requests

**61. Which of the following is a valid Go struct literal?**
A) Point{x: 1, y: 2}
B) Point{1, 2}
C) Point{x: 1}
D) All of the above

**62. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    m := map[string]int{"a": 1, "b": 2}
    for k, v := range m {
        fmt.Println(k, v)
    }
}
```
A) a 1 b 2 (order may vary)
B) a 1 b 2 (always in this order)
C) compilation error
D) panic

**63. Which statement about Go's `flag` package is correct?**
A) It's used for command-line flag parsing
B) It's only available in main packages
C) It cannot handle boolean flags
D) It's deprecated

**64. What is the purpose of the `io.Closer` interface?**
A) To close resources
B) To read data
C) To write data
D) To handle errors

**65. Which of the following is NOT a valid Go slice operation?**
A) s[1:3]
B) s[:3]
C) s[1:]
D) s[::]

**66. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x [3]int
    fmt.Println(len(x), cap(x))
}
```
A) 3 3
B) 3 0
C) 0 3
D) compilation error

**67. Which statement about Go's `log` package is correct?**
A) It provides structured logging
B) It only writes to stderr
C) It cannot be customized
D) It's thread-safe by default

**68. What is the purpose of the `math/big` package?**
A) To handle arbitrary-precision arithmetic
B) To provide mathematical functions
C) To optimize calculations
D) To handle floating-point operations

**69. Which of the following is a valid Go type switch?**
A) switch v := i.(type) { ... }
B) switch i.(type) { ... }
C) switch type i { ... }
D) All of the above

**70. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    f := func(x int) int { return x * 2 }
    fmt.Println(f(f(3)))
}
```
A) 12
B) 6
C) 3
D) compilation error

**71. Which statement about Go's `net/http` package is correct?**
A) It provides HTTP client and server implementations
B) It only supports HTTP/1.1
C) It cannot handle HTTPS
D) It's deprecated

**72. What is the purpose of the `strings.Builder` type?**
A) To efficiently build strings
B) To parse strings
C) To format strings
D) To validate strings

**73. Which of the following is NOT a valid Go rune literal?**
A) 'a'
B) '\n'
C) '\u0061'
D) "a"

**74. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    x := make([]int, 0, 5)
    x = append(x, 1, 2, 3)
    fmt.Println(len(x), cap(x))
}
```
A) 3 5
B) 3 3
C) 0 5
D) compilation error

**75. Which statement about Go's `database/sql` package is correct?**
A) It provides a generic database interface
B) It only works with PostgreSQL
C) It cannot handle transactions
D) It's deprecated

---

## Questions 76-100: Expert Level & Best Practices

**76. Which statement about Go's memory model is correct?**
A) Go uses a happens-before relationship
B) Go guarantees sequential consistency
C) Go doesn't have a memory model
D) Go uses locks for all operations

**77. What is the purpose of the `sync/atomic` package?**
A) To provide atomic operations
B) To synchronize goroutines
C) To handle memory barriers
D) To manage atomic types

**78. Which of the following is a valid Go build constraint?**
A) //go:build linux
B) // +build linux
C) //build:linux
D) All of the above

**79. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = []int{1, 2, 3}
    switch v := x.(type) {
    case []int:
        fmt.Println(len(v))
    default:
        fmt.Println("not a slice")
    }
}
```
A) 3
B) not a slice
C) panic
D) compilation error

**80. Which statement about Go's `trace` tool is correct?**
A) It traces execution events
B) It's only available in debug mode
C) It cannot trace goroutines
D) It's deprecated

**81. What is the purpose of the `pprof` tool?**
A) To profile Go programs
B) To debug memory leaks
C) To optimize performance
D) All of the above

**82. Which of the following is NOT a valid Go module command?**
A) go mod init
B) go mod get
C) go mod update
D) go mod tidy

**83. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int, 1)
    ch <- 1
    v, ok := <-ch
    fmt.Println(v, ok)
    v, ok = <-ch
    fmt.Println(v, ok)
}
```
A) 1 true 0 false
B) 1 true 1 true
C) 1 true panic
D) Deadlock

**84. Which statement about Go's `vet` tool is correct?**
A) It finds suspicious constructs
B) It's a compiler
C) It's only for security
D) It's deprecated

**85. What is the purpose of the `gofmt` tool?**
A) To format Go source code
B) To compile Go programs
C) To test Go programs
D) To lint Go programs

**86. Which of the following is a valid Go module version?**
A) v1.0.0
B) v1.0.0-pre
C) v1.0.0+meta
D) All of the above

**87. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x []int = nil
    if x == nil {
        fmt.Println("nil")
    } else {
        fmt.Println("not nil")
    }
}
```
A) nil
B) not nil
C) compilation error
D) panic

**88. Which statement about Go's `race` detector is correct?**
A) It detects race conditions
B) It's enabled by default
C) It slows down compilation
D) It's only available on Linux

**89. What is the purpose of the `embed` package?**
A) To embed files in the binary
B) To embed HTML templates
C) To embed static assets
D) All of the above

**90. Which of the following is NOT a valid Go pointer operation?**
A) &x
B) *p
C) p++
D) p == nil

**91. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    m := make(map[int]string)
    m[1] = "a"
    delete(m, 2)
    fmt.Println(len(m))
}
```
A) 1
B) 0
C) panic
D) compilation error

**92. Which statement about Go's `cover` tool is correct?**
A) It measures test coverage
B) It's only for unit tests
C) It cannot measure branch coverage
D) It's deprecated

**93. What is the purpose of the `go:generate` directive?**
A) To generate code
B) To compile code
C) To test code
D) To format code

**94. Which of the following is a valid Go workspace mode?**
A) go work init
B) go work use
C) go work sync
D) All of the above

**95. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = func() int { return 42 }
    if f, ok := x.(func() int); ok {
        fmt.Println(f())
    }
}
```
A) 42
B) compilation error
C) panic
D) 0

**96. Which statement about Go's `linkname` directive is correct?**
A) It links symbols across packages
B) It's recommended for regular use
C) It's only available in C
D) It's deprecated

**97. What is the purpose of the `go.mod` file?**
A) To define module dependencies
B) To store build configuration
C) To manage versions
D) All of the above

**98. Which of the following is NOT a valid Go compiler flag?**
A) -gcflags
B) -ldflags
C) -tags
D) -debug

**99. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan struct{})
    go func() {
        close(ch)
    }()
    _, ok := <-ch
    fmt.Println(ok)
}
```
A) false
B) true
C) panic
D) deadlock

**100. Which statement about Go's `go.mod` and `go.sum` files is correct?**
A) go.mod stores module requirements, go.sum stores checksums
B) go.sum stores module requirements, go.mod stores checksums
C) Both files store the same information
D) Only go.mod is required

---

## Answers

**1-25:**
1. A) nil
2. D) int x = 10
3. A) 0 0
4. B) Go uses a concurrent mark-and-sweep garbage collector
5. A) To initialize package-level variables
6. C) defer
7. A) Creates new values of maps, slices, and channels
8. B) Interfaces are implicitly satisfied
9. B) [1 99 3]
10. D) list
11. A) nil
12. B) It can return only index or only value
13. A) 1
14. C) my_package
15. A) Chooses between multiple channel operations
16. A) Structs can have methods
17. A) hello
18. D) while
19. A) To execute a function call after the surrounding function returns
20. B) Go uses error values returned from functions
21. B) memory address
22. C) const x := 10
23. A) nil
24. B) It may modify the original slice if capacity allows
25. A) 0

**26-50:**
26. B) Goroutines are managed by the Go runtime
27. A) To synchronize access to shared resources
28. A) close(ch)
29. A) 1
30. B) It provides cancellation and timeout capabilities
31. A) 5
32. D) func (s map[string]int) Method()
33. A) To represent any type
34. B) recover() can only be called in a deferred function
35. A) 20
36. A) `json:"name"`
37. A) To wait for a collection of goroutines to finish
38. A) It's used for runtime type inspection
39. A) [1 2 4 5]
40. D) // +build "linux"
41. A) To set the maximum number of CPUs
42. B) It provides duration and time types
43. A) 1 2
44. D) All of the above
45. A) To read data from a source
46. B) It can sort any collection that implements sort.Interface
47. C) 0 a 1 b 2 c
48. D) \d
49. A) To bypass type safety
50. B) It provides benchmarking capabilities

**51-75:**
51. B) It can panic if the type is wrong
52. A) To reuse objects to reduce garbage collection pressure
53. D) All of the above
54. A) true
55. A) It allows calling C code from Go
56. A) To create a cancelable context
57. D) int<-chan
58. B) default
59. A) It provides low-level runtime information
60. A) To encode and decode JSON data
61. D) All of the above
62. A) a 1 b 2 (order may vary)
63. A) It's used for command-line flag parsing
64. A) To close resources
65. D) s[::]
66. A) 3 3
67. D) It's thread-safe by default
68. A) To handle arbitrary-precision arithmetic
69. A) switch v := i.(type) { ... }
70. A) 12
71. A) It provides HTTP client and server implementations
72. A) To efficiently build strings
73. D) "a"
74. A) 3 5
75. A) It provides a generic database interface

**76-100:**
76. A) Go uses a happens-before relationship
77. A) To provide atomic operations
78. D) All of the above
79. A) 3
80. A) It traces execution events
81. D) All of the above
82. C) go mod update
83. D) Deadlock
84. A) It finds suspicious constructs
85. A) To format Go source code
86. D) All of the above
87. A) nil
88. A) It detects race conditions
89. D) All of the above
90. C) p++
91. A) 1
92. A) It measures test coverage
93. A) To generate code
94. D) All of the above
95. A) 42
96. A) It links symbols across packages
97. D) All of the above
98. D) -debug
99. A) false
100. A) go.mod stores module requirements, go.sum stores checksums

---

**Score Interpretation:**
- 90-100: Expert Level
- 80-89: Advanced Level
- 70-79: Intermediate Level
- 60-69: Junior Level
- Below 60: Needs Improvement

Good luck with your Go recruitment process!
