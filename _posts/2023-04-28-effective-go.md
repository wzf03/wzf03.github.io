---
title: "Notes of Effective Go"
date: 2023-04-28 16:49:50 +08:00
categories: [Golang]
tags: [programming, golang]
image: assets/img/covers/golang.png
---

Reference: [Effective Go](https://go.dev/doc/effective_go)

## Introduction

- Don't simply translate program written in other language
  to go, the conventions are quite different.
- The guide is out of date but continues to be useful.

## Formatting

- use `go fmt` to automatically format the code. (operates at package level)
- some format details
  - use tabs for indentation
  - has no line length limit
  - needs fewer parentheses and operator precedence hierarchy is shorter and clearer

## Commentary

- `/**/` block comments : mostly as package comments and disable large swaths
  of code
- `//` line comments : the norm
- doc comments: [Go Doc Comments](https://go.dev/doc/comment)

## Names

- the visibility of a name outside a package is determined by whether its first
  character is upper case.

### Package names

- lower case, single-word
- don't worry about collisions a priori, can choose a different name when imported
- package name is the base name of its source directory
- package name means a lot, so use name like `once.Do` instead of `once.DoOrWaitUntilDone`

### Getters

- getter method: `obj.Owner()` instead of `obj.GetOwner()`
- setter method: `obg.SetOwner()`

### Interface names

- one-method interfaces are named by the method name plus an `-er` suffix or
  similar modification to construct an agent noun, like `Reader`, `Formatter`

### MixedCaps

- use `MixedCaps` or `mixedCaps` rather than underscores

## Semicolons

- the lexer uses a simple rule to insert semicolons automatically
- if the last token before a newline is an identifier, a basic literal or one
  of `break continue fallthrough return ++ --`
- a semicolon can also be omitted immediately before a closing brace
- cannot put the opening brace of a control structure on the next line

## Control structures

- `if` and `switch` accept an optional initialization statement like that of `for`
- `break` and `continue` statements take an optional label to identify what to
  break or continue

  - the label can be set like `goto` label in c

  ```go
  Loop:
    for n := 0; n < 10; n++ {
      switch n {
        case 2:
          break Loop
      }
    }
  ```

### If

- when an `if` statement doesn't flow into the next statement, omit the
  unnecessary `else`

### Redeclaration and reassignment

- a variable v can be redeclaration in a `:=` declaration
  - the declaration is in the same scope as the exist
  - the corresponding value is assignable to v
  - at least one other variable that is created by the declaration

### For

- for strings, the `range` will parse the UTF-8 and break out individual Unicode
  code points.
  - `rune` represent a single Unicode code point.
- `++` and `--` are statements not expressions.

### Switch

- expressions need not be constants or even integers, just like a
  `if-else-if-else` chain
- cases can be presented in comma-separated lists

## Functions

### Multiple return values

- multiple return values can be used to simulate a reference parameter.
- `x, i = nextInt(b, i)`

### Named result parameters

- when named, they are initialized to the zero values for their types when
  the function begins
- they can make code shorter and clearer

### Defer

- schedules a function call
- guarantees that you will never forget to release resources
- much clearer
- deferred functions are executed in LIFO order
- arguments to deferred functions are evaluated when the defer executes

## Data

### Allocation with new

- `new(T)` allocates memory and zeros it, then returns a pointer of type `*T*`
- `zero-value-is-useful` property: some type with zero value can be used
  immediately like `bytes.Buffer`, `sync.Mutex`

### Constructors and composite literals

- taking the address of a composite literal allocates a fresh instance
  each time it is evaluated
  - `return &File{fd, name, nil, 0}` is ok
- Composite literals can also be created for arrays, slices, and maps

### Allocation with make

- creates slices, maps, and channels only
- return an **initialized**(not zeroed) value of type T (not _T_)

### Arrays

- arrays are values. Assigning one to another copies all the elements.
- pass an array to a function, it will receive a copy of the array, not a pointer
- the size of an array is part of its type

### Slices

- hold references to an underlying array
  - assign one slice to another, both refer to the same array
- use slice as the arguments for function can tell the destination and limit
  length at the same time
- `len` and `cap` are legal when applied to `nil` value, and return `0`

### Two-dimensional slices

- define an `array-of-arrays` or `slice-of-slices`
- two way to allocate a 2D slice

  - allocate each slice independently

    - use `for` loop to allocate

    ```go
     picture := make([][]uint8, YSize)
     for i := range picture {
         picture[i] = make([]uint8, XSize)
     }
    ```

  - allocate a single array and point the individual slices into it

    - allocate a big slice than use `for` loop to assign

    ```go
     picture := make([][]uint8, YSize)
     pixels := make([]uint8, XSize*YSize)
     for i :=  range picture {
         picture[i], pixels = pixels[:XSize], pixels[XSize:]
     }
    ```

### Maps

- the key can be of any type for which the equality operator is defined
  - `integers`, `floating point`, `complex`, `strings`, `pointers`,
    `interfaces`, `struct`, `arrays`
- hold references to an underlying data structure
- fetch a map value with key that is no present in the map will return the zero
  value for the type of the entries in the map
- the second return value is usually named `ok`, it will be set to `true`
  if the key is present, `false` as the opposite
- use `delete` built-in function to delete a entry
  - safe even if the key is already absent

### Printing

- `Println` insert a blank between arguments and append a newline to the output
- `Print` add blanks only if the operand on neither side is a string
- `fmt.Fprint` and friends take as a first argument any object that implements
  the `io.Writer` interface, like `os.Stdout` and `os.Stderr`
- you can use the catchall format `%v` for default conversion, the result is
  the same with `Print` and `Println`
- use `%+v` and `%#v` for more details when print `struct`
- use `%q` to print quoted string for type `string` and `[]byte`
  - `%#q` will use backquotes instead if possible
  - also can be applied to integers and runes, producing a single-quoted rune constant
- use `%x` to generate a long hexadecimal string for strings, byte array/slice, integers
  - `% x` will put space between the bytes
- use `%T` to print the type of a value
- use `%f` to format a float-point number
- implement `String() string` to custom output format for custom type
  - the receiver for String must be of value type if you need to print values
    of type T as well as pointers to T
  - don't construct a `String` method by calling `Sprintf` in a way that will
    recur into your `String` method indefinitely

### Append

- use `...` to append a slice to slice

  ```go
   x := []int{1,2,3}
   y := []int{4,5,6}
   x = append(x, y...)
  ```

## Initializtion

### Constants

- they are created at compile time, even when defined as locals in functions

- can only be numbers, characters (runes), strings or booleans

- the expressions that define them must be constant expression

- use `iota` enumerator to create enumerated constants

  - and expressions can be implicitly repeated

  ```go
    type ByteSize float64

    const (
        _           = iota // ignore first value by assigning to blank identifier
        KB ByteSize = 1 << (10 * iota)
        MB
        GB
        TB
        PB
        EB
        ZB
        YB
    )
  ```

### The init function

- `init` is called after all the variable declarations in the package have
  evaluated their initializers, and those are evaluated only after all the
  imported packages have been initialized

## Methods

### Pointers vs. Values

- value methods can be invoked on pointers and values, but pointer methods can
  only be invoked on pointers
- when the value is addressable, the language takes care of the common case of
  invoking a pointer method on a value by inserting the address operator automatically

## Interfaces and other types

### Conversion

It's an idiom in Go programs to convert the type of an expression to access a
different set of methods.

### Interface conversions and type assertions

Type swithches

```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string: // concrete value
    return str
case Stringer: // interface
    return str.String()
}
```

Type assertions `value.(typeName)`

```go
str := value.(string) // crash when the value does not contain a string
```

```go
str, ok := value.(string) // use `ok` to test is conversion is successful
// Attention: If the type assertion fails, str will still exist and be of type string
// But it will have the zero value
```

### Generality

Export the interface only when a type exits only to implement an interface and
will never have exported methods beyond that interface.

In such cases, the constructor should return an interface value rather than
the implementing type.

### Interfaces and methods

- The receiver of a methods can be a function, but it should be warpped in a
  type like `http.HandlerFunc`

## The blank identifier

### The blank identifier in multiple assignment

It can be used to ignore some value in mutiple assignment

### Unused imports and variables

To slience the unused variable error

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

### Import for side effect

To import the package only for its side effects, rename the package to the
blank identifier

```go
import _ "net/http/pprof"
```

### Interface checks

To check if a type implement a interface correctly.

By convention, such declarations are only used when there are no static
conversions already present in the code, which is a rare event.

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

## Embedding

- We can embed multiple interfaces to form a new one.

  ```go
    // ReadWriter is the interface that combines the Reader and Writer interfaces.
    type ReadWriter interface {
        Reader
        Writer
    }
  ```

- We can embed several structs by using their pointer to form a new one but
  does not give them field names.

  ```go
    // ReadWriter stores pointers to a Reader and a Writer.
    // It implements io.ReadWriter.
    type ReadWriter struct {
        *Reader  // *bufio.Reader
        *Writer  // *bufio.Writer
    }
  ```

- Attention: If you give them field name, you will need to provide forwarding method.

- When we embed a type, the methods of that type become methods of the outer
  type, but when type are invoked the receiver of the method is the inner type.

- You can mix named value and unnamed value in a struct.

  ```go
    type Job struct {
        Command string
        *log.Logger
    }

    func (job *Job) Printf(format string, args ...interface{}) {
        // access the *log.Logger
        job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
    }
  ```

- A field or method X hides ant other item X in a more deeply nested part
  of the type.

- If the same name appears at the same nesting level, it is ususally an error.

  - If the deplicate name is never mentioned in the program outside the type
    definition, it is OK.

## Concurrency

### Share by communicating

- Never actively shared by separate threads of execution.
- Only one goroutine has access to the value at any given time.

**Do not communicate by sharing memory; instead, share memory by communicating.**

### Goroutine

In Go, function literals are closures, and the variables referred to by the
function survive as long as they are active.

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

### Channels

- If an optional integer parameter is provide, `make` sets the buffer size
  for the channel.

  ```go
    ci := make(chan int)            // unbuffered channel of integers
    cj := make(chan int, 0)         // unbuffered channel of integers
    cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
  ```

- It can be used to block a goroutine util there is data to receive.

- Different to limit throughput

  ```go
    var sem = make(chan int, MaxOutstanding)

    func handle(r *Request) {
        sem <- 1    // Wait for active queue to drain.
        process(r)  // May take a long time.
        <-sem       // Done; enable next request to run.
    }

    func Serve(queue chan *Request) {
        for {
            req := <-queue
            go handle(req)  // Don't wait for handle to finish.
        }
    }
  ```

  ```go
    func Serve(queue chan *Request) {
        for req := range queue {
            sem <- 1
            go func(req *Request) {
                process(req)
                <-sem
            }(req)
        }
    }
  ```

  ```go
    func Serve(queue chan *Request) {
        for req := range queue {
            req := req // Create new instance of req for the goroutine.
            sem <- 1
            go func() {
                process(req)
                <-sem
            }()
        }
    }
  ```

  ```go
    func handle(queue chan *Request) {
        for r := range queue {
            process(r)
        }
    }

    func Serve(clientRequests chan *Request, quit chan bool) {
        // Start handlers
        for i := 0; i < MaxOutstanding; i++ {
            go handle(clientRequests)
        }
        <-quit  // Wait to be told to exit.
    }

  ```

### Channels of channels

- Channel is a first-class value that can be allocated and passed around lik
  e any other.
- So channels can be transport by channels. For example when wrap in a
  structure with other field.

### Parallelization

- `runtime.NumCPU()` returns the number of hardware CPU cores in the machine
- `runtime.GOMAXPROCS(0)` returns the user-specified number of cores or NumCPU
  by default. It can be set by similarly named shell environment.

### A leaky buffer

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}

func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

## Errors

- Having a prefix naming the operation or package that generated the error.
- Use type assertion to examine for exact type of errors and more information.

### Panic

- The built-in function `panic` that in effect creates a run-time error that
  will stop the program
- The function takes a single argument of arbitary type, often a string, to be
  printed as the program dies.

### Recover

- When panic is called, it immediately stops execution of the current function
  and begins unwinding the stack of goroutine, running any **deferred**
  functions along the way.
- If that unwinding reaches the top of the goroutine's stack, the program dies.
- A call to `recover` stops the unwinding and returns the argument passed to
  panic. And `recover` is only useful inside deferred functions.
- Can be used to turn internal panic calls into error values.
- `re-panic` is also allowed and both the original and new failures will be
  presented in the crash report.
