---
title: "Go Recruitment Exam - Batch 2"
date: 2026-02-02T22:30:00Z
tags: ["go", "programming", "interview", "exam", "ardanlabs"]
categories: ["Ultimate Go Series"]
description: "Full Go Recruitment Exam - Batch 2 of 4. 100 multiple-choice questions following Ardan Labs style."
---

Welcome to the second batch of our comprehensive Go Recruitment Exam. This exam consists of 100 multiple-choice questions focusing on intermediate to advanced Go concepts, concurrency patterns, and practical applications following the Ardan Labs methodology.

## Instructions

- Read each question carefully
- Choose the best answer from the provided options
- Answers are provided at the end of this post
- Time limit: 90 minutes recommended

---

## Questions 1-25: Concurrency & Parallelism

**1. Which statement about Go's goroutine scheduling is correct?**
A) Goroutines are scheduled by the OS
B) Goroutines are scheduled by the Go runtime using M:N scheduling
C) Each goroutine runs on its own OS thread
D) Goroutines cannot be preempted

**2. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int)
    go func() {
        ch <- 1
        ch <- 2
    }()
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```
A) 1 2
B) deadlock
C) panic
D) compilation error

**3. Which of the following is a valid channel operation?**
A) ch <- value
B) <-ch
C) close(ch)
D) len(ch)
E) All of above

**4. What is the purpose of the `sync.Once` type?**
A) To ensure a function is executed exactly once
B) To synchronize multiple goroutines
C) To create singletons
D) To handle initialization

**5. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int, 2)
    ch <- 1
    fmt.Println(len(ch), cap(ch))
}
```
A) 1 2
B) 0 2
C) 1 0
D) compilation error

**6. Which statement about Go's mutex is correct?**
A) Mutex is reentrant
B) Mutex is not reentrant
C) Mutex can be locked recursively
D) Mutex is automatically unlocked

**7. What is the purpose of the `select` statement with default case?**
A) To provide non-blocking communication
B) To handle errors
C) To optimize performance
D) To create timeouts

**8. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var wg sync.WaitGroup
    wg.Add(2)
    go func() {
        defer wg.Done()
        fmt.Print("1")
    }()
    go func() {
        defer wg.Done()
        fmt.Print("2")
    }()
    wg.Wait()
}
```
A) 12 (order may vary)
B) 21 (order may vary)
C) compilation error
D) deadlock

**9. Which of the following is a valid way to create a buffered channel?**
A) make(chan int, 10)
B) make(chan int, buffer=10)
C) chan int{10}
D) make(chan int)[10]

**10. What is the purpose of the `context.WithTimeout()` function?**
A) To create a context with timeout
B) To set a deadline
C) To cancel operations
D) To handle errors

**11. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int)
    select {
    case <-ch:
        fmt.Println("received")
    case ch <- 1:
        fmt.Println("sent")
    default:
        fmt.Println("default")
    }
}
```
A) received
B) sent
C) default
D) deadlock

**12. Which statement about Go's race detector is correct?**
A) It detects all race conditions
B) It may miss some race conditions
C) It slows down the program significantly
D) It's enabled by default

**13. What is the purpose of the `sync.RWMutex` type?**
A) To allow multiple readers or one writer
B) To synchronize all operations
C) To handle read-write operations
D) To optimize performance

**14. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    done := make(chan bool)
    go func() {
        fmt.Print("hello")
        done <- true
    }()
    <-done
    fmt.Print("world")
}
```
A) helloworld
B) hello (deadlock)
C) world (deadlock)
D) compilation error

**15. Which of the following is NOT a valid goroutine creation?**
A) go func() { }()
B) go myFunction()
C) go myFunction(arg)
D) go := myFunction()

**16. What is the purpose of the `runtime.Gosched()` function?**
A) To yield the processor
B) To schedule goroutines
C) To optimize performance
D) To handle garbage collection

**17. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int)
    go func() {
        for i := 0; i < 3; i++ {
            ch <- i
        }
        close(ch)
    }()
    for v := range ch {
        fmt.Print(v)
    }
}
```
A) 012
B) 0123
C) deadlock
D) compilation error

**18. Which statement about Go's channels is correct?**
A) Channels are always buffered
B) Channels are always unbuffered
C) Channels can be buffered or unbuffered
D) Channels cannot be closed

**19. What is the purpose of the `sync.Cond` type?**
A) To notify goroutines of conditions
B) To synchronize access
C) To handle timeouts
D) To manage resources

**20. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go func() { ch1 <- 1 }()
    go func() { ch2 <- 2 }()
    select {
    case v1 := <-ch1:
        fmt.Println(v1)
    case v2 := <-ch2:
        fmt.Println(v2)
    }
}
```
A) 1 or 2 (non-deterministic)
B) 1
C) 2
D) deadlock

**21. Which of the following is a valid channel direction?**
A) chan<- int
B) <-chan int
C) chan int
D) All of the above

**22. What is the purpose of the `context.WithCancel()` function?**
A) To create a cancelable context
B) To cancel all operations
C) To handle timeouts
D) To manage deadlines

**23. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    ch := make(chan int, 1)
    ch <- 1
    close(ch)
    v, ok := <-ch
    fmt.Println(v, ok)
}
```
A) 1 true
B) 1 false
C) 0 false
D) panic

**24. Which statement about Go's worker pool pattern is correct?**
A) It uses a fixed number of goroutines
B) It creates unlimited goroutines
C) It's not recommended in Go
D) It requires special libraries

**25. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    done := make(chan struct{})
    go func() {
        <-done
        fmt.Print("1")
    }()
    go func() {
        <-done
        fmt.Print("2")
    }()
    close(done)
}
```
A) 12 or 21 (order may vary)
B) deadlock
C) compilation error
D) panic

---

## Questions 26-50: Memory Management & Performance

**26. Which statement about Go's garbage collection is correct?**
A) It uses stop-the-world collection
B) It uses concurrent collection
C) It requires manual triggering
D) It's disabled by default

**27. What is the purpose of the `runtime.GC()` function?**
A) To force garbage collection
B) To check memory usage
C) To optimize performance
D) To handle memory leaks

**28. Which of the following is NOT a valid way to reduce allocations?**
A) Use sync.Pool
B) Pre-allocate slices
C) Use pointers everywhere
D) Use value types

**29. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x *int
    fmt.Println(x == nil)
}
```
A) true
B) false
C) panic
D) compilation error

**30. Which statement about Go's escape analysis is correct?**
A) It determines if variables escape to heap
B) It's only for optimization
C) It's disabled by default
D) It's a runtime process

**31. What is the purpose of the `sync.Map` type?**
A) To provide a concurrent-safe map
B) To improve performance
C) To handle large maps
D) To replace regular maps

**32. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    s := make([]int, 3, 5)
    s = append(s, 1, 2)
    fmt.Println(len(s), cap(s))
}
```
A) 5 5
B) 5 10
C) 5 3
D) compilation error

**33. Which of the following is a valid optimization technique?**
A) Avoid unnecessary allocations
B) Use interfaces everywhere
C) Create many goroutines
D) Use reflection frequently

**34. What is the purpose of the `runtime.ReadMemStats()` function?**
A) To read memory statistics
B) To optimize memory
C) To handle memory leaks
D) To check garbage collection

**35. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    m := make(map[string]int)
    for i := 0; i < 1000; i++ {
        m[fmt.Sprintf("key%d", i)] = i
    }
    fmt.Println(len(m))
}
```
A) 1000
B) 0
C) panic
D) compilation error

**36. Which statement about Go's stack growth is correct?**
A) Stacks grow automatically
B) Stacks have fixed size
C) Stacks grow only on heap
D) Stacks cannot grow

**37. What is the purpose of the `strings.Builder` type?**
A) To efficiently build strings
B) To parse strings
C) To format strings
D) To validate strings

**38. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x []int
    x = append(x, 1, 2, 3)
    fmt.Println(len(x), cap(x))
}
```
A) 3 3
B) 3 4
C) 3 0
D) compilation error

**39. Which of the following is NOT a valid memory optimization?**
A) Use value receivers
B) Use pointer receivers
C) Pre-allocate slices
D) Use sync.Pool

**40. What is the purpose of the `runtime.SetFinalizer()` function?**
A) To set finalizers for objects
B) To handle garbage collection
C) To optimize memory
D) To manage resources

**41. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    s := []int{1, 2, 3}
    t := s
    t[0] = 99
    fmt.Println(s[0])
}
```
A) 99
B) 1
C) panic
D) compilation error

**42. Which statement about Go's memory alignment is correct?**
A) Go handles alignment automatically
B) Alignment must be manual
C) Alignment affects performance
D) Alignment is not important

**43. What is the purpose of the `bytes.Buffer` type?**
A) To efficiently build byte slices
B) To handle I/O operations
C) To manage memory
D) To optimize performance

**44. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var p *int
    if p == nil {
        fmt.Println("nil")
    }
}
```
A) nil
B) panic
C) compilation error
D) nothing

**45. Which of the following is a valid way to profile memory?**
A) go test -memprofile
B) go tool pprof
C) runtime/pprof
D) All of the above

**46. What is the purpose of the `runtime.GOMAXPROCS()` function?**
A) To set maximum number of CPUs
B) To limit memory usage
C) To control goroutines
D) To optimize performance

**47. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    m := map[string]int{"a": 1}
    delete(m, "b")
    fmt.Println(len(m))
}
```
A) 1
B) 0
C) panic
D) compilation error

**48. Which statement about Go's allocation patterns is correct?**
A) Stack allocation is faster
B) Heap allocation is faster
C) Both are equally fast
D) Allocation speed doesn't matter

**49. What is the purpose of the `sync.Pool` type?**
A) To reuse objects
B) To synchronize access
C) To manage memory
D) To handle concurrency

**50. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    s := make([]int, 0, 10)
    for i := 0; i < 15; i++ {
        s = append(s, i)
    }
    fmt.Println(len(s), cap(s))
}
```
A) 15 20
B) 15 15
C) 15 10
D) compilation error

---

## Questions 51-75: Advanced Go Features

**51. Which statement about Go generics is correct?**
A) Generics are available since Go 1.18
B) Generics use inheritance
C) Generics are only for interfaces
D) Generics are not recommended

**52. What is the purpose of the `any` keyword in Go?**
A) To represent any type
B) To replace interface{}
C) To improve type safety
D) All of the above

**53. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = []string{"a", "b"}
    switch v := x.(type) {
    case []string:
        fmt.Println(len(v))
    default:
        fmt.Println("unknown")
    }
}
```
A) 2
B) unknown
C) panic
D) compilation error

**54. Which of the following is a valid generic function?**
A) func[T any](x T) T { return x }
B) func[T](x T) T { return x }
C) func generic[T any](x T) T { return x }
D) All of the above

**55. What is the purpose of the `comparable` constraint?**
A) To allow comparison operations
B) To handle equality
C) To support ordering
D) All of the above

**56. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    f := func(x int) func() int {
        return func() int { return x * 2 }
    }
    g := f(5)
    fmt.Println(g())
}
```
A) 10
B) 5
C) compilation error
D) panic

**57. Which statement about Go's type parameters is correct?**
A) They are specified in square brackets
B) They are specified in parentheses
C) They are specified in angle brackets
D) They are specified in curly brackets

**58. What is the purpose of the `~` token in type constraints?**
A) To represent underlying types
B) To handle interfaces
C) To manage generics
D) To optimize performance

**59. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = map[string]int{"a": 1}
    if m, ok := x.(map[string]int); ok {
        fmt.Println(m["a"])
    }
}
```
A) 1
B) 0
C) panic
D) compilation error

**60. Which of the following is a valid type constraint?**
A) interface{ int | string }
B) interface{ ~int }
C) interface{ comparable }
D) All of the above

**61. What is the purpose of the `reflect` package?**
A) Runtime type inspection
B) Performance optimization
C) Memory management
D) Error handling

**62. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = 42
    t := reflect.TypeOf(x)
    fmt.Println(t.Kind())
}
```
A) int
B) interface
C) compilation error
D) panic

**63. Which statement about Go's `unsafe` package is correct?**
A) It bypasses type safety
B) It's recommended for regular use
C) It's safer than regular Go
D) It's deprecated

**64. What is the purpose of the `cgo` tool?**
A) To interface with C code
B) To compile C programs
C) To optimize performance
D) To handle memory

**65. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = []int{1, 2, 3}
    v := x.([]int)
    fmt.Println(v[1])
}
```
A) 2
B) panic
C) compilation error
D) 0

**66. Which statement about Go's build tags is correct?**
A) They control conditional compilation
B) They are only for testing
C) They are deprecated
D) They are not recommended

**67. What is the purpose of the `go:generate` directive?**
A) To generate code
B) To compile code
C) To test code
D) To format code

**68. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = func(y int) int { return y * 2 }
    if f, ok := x.(func(int) int); ok {
        fmt.Println(f(5))
    }
}
```
A) 10
B) compilation error
C) panic
D) 0

**69. Which of the following is a valid Go directive?**
A) //go:noinline
B) //go:norace
C) //go:nosplit
D) All of the above

**70. What is the purpose of the `embed` package?**
A) To embed files in binaries
B) To handle templates
C) To manage static assets
D) All of the above

**71. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = struct{ name string }{"test"}
    if s, ok := x.(struct{ name string }); ok {
        fmt.Println(s.name)
    }
}
```
A) test
B) compilation error
C) panic
D) empty string

**72. Which statement about Go's assembly is correct?**
A) It's written in Plan 9 assembly
B) It's written in x86 assembly
C) It's not supported
D) It's deprecated

**73. What is the purpose of the `linkname` directive?**
A) To link symbols across packages
B) To optimize performance
C) To handle memory
D) To manage imports

**74. What is the output of this code?**
```go
package main
import "fmt"
func main() {
    var x interface{} = []interface{}{1, "a", true}
    if s, ok := x.([]interface{}); ok {
        fmt.Println(len(s))
    }
}
```
A) 3
B) compilation error
C) panic
D) 0

**75. Which statement about Go's plugin package is correct?**
A) It loads Go plugins at runtime
B) It's only available on Linux
C) It's deprecated
D) It's not recommended

---

## Questions 76-100: Testing & Best Practices

**76. Which statement about Go's testing package is correct?**
A) Tests must be in files ending with _test.go
B) Tests can be in any file
C) Tests must be in main package
D) Tests are not recommended

**77. What is the purpose of the `testing.T` type?**
A) To report test failures
B) To run tests
C) To manage test state
D) To handle errors

**78. Which of the following is a valid test function?**
A) func TestXxx(t *testing.T)
B) func testXxx(t *testing.T)
C) func Test(t *testing.T)
D) func Xxx(t *testing.T)

**79. What is the output of this test?**
```go
func TestAdd(t *testing.T) {
    result := 2 + 2
    if result != 4 {
        t.Errorf("Expected 4, got %d", result)
    }
}
```
A) PASS
B) FAIL
C) compilation error
D) panic

**80. Which statement about Go's benchmarks is correct?**
A) They use the testing.B type
B) They use the testing.T type
C) They are not recommended
D) They are deprecated

**81. What is the purpose of the `go test` command?**
A) To run tests
B) To compile tests
C) To generate tests
D) To format tests

**82. Which of the following is a valid benchmark function?**
A) func BenchmarkXxx(b *testing.B)
B) func benchmarkXxx(b *testing.B)
C) func Benchmark(b *testing.B)
D) func Xxx(b *testing.B)

**83. What is the purpose of the `testing.M` type?**
A) To manage test main functions
B) To run benchmarks
C) To handle examples
D) To manage subtests

**84. Which statement about Go's table-driven tests is correct?**
A) They use slices of test cases
B) They are not recommended
C) They are only for unit tests
D) They are deprecated

**85. What is the output of this code?**
```go
func ExampleAdd() {
    fmt.Println(2 + 2)
    // Output: 4
}
```
A) PASS
B) FAIL
C) compilation error
D) panic

**86. Which of the following is a valid example function?**
A) func ExampleXxx()
B) func exampleXxx()
C) func Example()
D) func Xxx()

**87. What is the purpose of the `go vet` tool?**
A) To find suspicious constructs
B) To compile code
C) To test code
D) To format code

**88. Which statement about Go's race detector is correct?**
A) It's enabled with -race flag
B) It's enabled by default
C) It's not recommended
D) It's deprecated

**89. What is the purpose of the `go cover` tool?**
A) To measure test coverage
B) To generate tests
C) To compile code
D) To format code

**90. Which of the following is a valid coverage command?**
A) go test -cover
B) go test -coverage
C) go test -cov
D) go test -c

**91. What is the purpose of the `go mod` command?**
A) To manage modules
B) To compile modules
C) To test modules
D) To format modules

**92. Which statement about Go's modules is correct?**
A) They replace GOPATH
B) They work with GOPATH
C) They are optional
D) They are deprecated

**93. What is the output of this code?**
```go
func TestSubtest(t *testing.T) {
    t.Run("sub1", func(t *testing.T) {
        t.Log("running sub1")
    })
}
```
A) PASS with subtest
B) FAIL
C) compilation error
D) panic

**94. Which of the following is a valid module command?**
A) go mod init
B) go mod get
C) go mod tidy
D) All of the above

**95. What is the purpose of the `go.sum` file?**
A) To store checksums
B) To store versions
C) To store dependencies
D) To store metadata

**96. Which statement about Go's dependency management is correct?**
A) It uses semantic versioning
B) It uses semantic versioning with minimal version selection
C) It doesn't use versioning
D) It uses custom versioning

**97. What is the purpose of the `go work` command?**
A) To manage workspaces
B) To compile workspaces
C) To test workspaces
D) To format workspaces

**98. Which of the following is a valid workspace command?**
A) go work init
B) go work use
C) go work sync
D) All of the above

**99. What is the output of this code?**
```go
func TestParallel(t *testing.T) {
    t.Run("parallel", func(t *testing.T) {
        t.Parallel()
        t.Log("parallel test")
    })
}
```
A) PASS with parallel test
B) FAIL
C) compilation error
D) panic

**100. Which statement about Go's best practices is correct?**
A) Keep functions small and focused
B) Use global variables frequently
C) Avoid interfaces
D) Use reflection for regular operations

---

## Answers

**1-25:**
1. B) Goroutines are scheduled by the Go runtime using M:N scheduling
2. A) 1 2
3. E) All of above
4. A) To ensure a function is executed exactly once
5. A) 1 2
6. B) Mutex is not reentrant
7. A) To provide non-blocking communication
8. A) 12 (order may vary)
9. A) make(chan int, 10)
10. A) To create a context with timeout
11. C) default
12. B) It may miss some race conditions
13. A) To allow multiple readers or one writer
14. A) helloworld
15. D) go := myFunction()
16. A) To yield the processor
17. A) 012
18. C) Channels can be buffered or unbuffered
19. A) To notify goroutines of conditions
20. A) 1 or 2 (non-deterministic)
21. D) All of the above
22. A) To create a cancelable context
23. A) 1 true
24. A) It uses a fixed number of goroutines
25. A) 12 or 21 (order may vary)

**26-50:**
26. B) It uses concurrent collection
27. A) To force garbage collection
28. C) Use pointers everywhere
29. A) true
30. A) It determines if variables escape to heap
31. A) To provide a concurrent-safe map
32. A) 5 5
33. A) Avoid unnecessary allocations
34. A) To read memory statistics
35. A) 1000
36. A) Stacks grow automatically
37. A) To efficiently build strings
38. B) 3 4
39. B) Use pointer receivers
40. A) To set finalizers for objects
41. A) 99
42. A) Go handles alignment automatically
43. A) To efficiently build byte slices
44. A) nil
45. D) All of the above
46. A) To set maximum number of CPUs
47. A) 1
48. A) Stack allocation is faster
49. A) To reuse objects
50. A) 15 20

**51-75:**
51. A) Generics are available since Go 1.18
52. D) All of the above
53. A) 2
54. D) All of the above
55. D) All of the above
56. A) 10
57. A) They are specified in square brackets
58. A) To represent underlying types
59. A) 1
60. D) All of the above
61. A) Runtime type inspection
62. A) Int
63. A) It bypasses type safety
64. A) To interface with C code
65. A) 2
66. A) They control conditional compilation
67. A) To generate code
68. A) 10
69. D) All of the above
70. D) All of the above
71. A) test
72. A) It's written in Plan 9 assembly
73. A) To link symbols across packages
74. A) 3
75. A) It loads Go plugins at runtime

**76-100:**
76. A) Tests must be in files ending with _test.go
77. A) To report test failures
78. A) func TestXxx(t *testing.T)
79. A) PASS
80. A) They use the testing.B type
81. A) To run tests
82. A) func BenchmarkXxx(b *testing.B)
83. A) To manage test main functions
84. A) They use slices of test cases
85. A) PASS
86. A) func ExampleXxx()
87. A) To find suspicious constructs
88. A) It's enabled with -race flag
89. A) To measure test coverage
90. A) go test -cover
91. A) To manage modules
92. A) They replace GOPATH
93. A) PASS with subtest
94. D) All of the above
95. A) To store checksums
96. B) It uses semantic versioning with minimal version selection
97. A) To manage workspaces
98. D) All of the above
99. A) PASS with parallel test
100. A) Keep functions small and focused

---

**Score Interpretation:**
- 90-100: Expert Level
- 80-89: Advanced Level
- 70-79: Intermediate Level
- 60-69: Junior Level
- Below 60: Needs Improvement

This batch focuses on concurrency, performance, and advanced Go features. Master these topics to become a proficient Go developer!
