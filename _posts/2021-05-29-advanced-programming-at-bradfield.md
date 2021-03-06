---
layout: article
title: Advanced Programming at Bradfield
date: 2021-05-29 09:49:44
categories: programming
tags:
- programming
---

I feel bad that I haven't been able to keep up with blogging. But with work, the baby, workouts, Bradfield, and a couple of side projects, it's been pretty tough to find the time.

Anyways, in the interest of my one reader (hi BT), I'll try to briefly summarize my most recent Bradfield module and an interesting takeaway.

## Advanced Programming

So what is "Advanced Programming"? Well, it's been sort of a survey of the following topics:

- garbage collection (mark and sweep)
- foreign function interfaces (calling C code)
- data structures (interface and slice representation)
- concurrency, locking (mutexes, atomic instructions, goroutines)

with the Go Programming Language as the reference implementation we've studied to learn about these areas. I'm a big fan of Go, in contrast to a lot of people I work with,
so it's been fun to dig into the source code and learn how things work. Some of the feedback from other students in the class was that the emphasis on Go, especially for people
that don't work with it frequently, was just a little bit off-putting. I get that. We use it at work so I have slightly more motivation _but_ I also thought it was interesting
that you could carry a lot of the concepts to other areas (e.g. how Go represents strings vs how C does it, etc.).

## Biggest Takeaway

So there's some recency bias in this but I found the final week on calling C code pretty interesting. I ended up writing a [Go wrapper around RocksDB](https://github.com/facebook/rocksdb). It was a little
tricky to get right because of the tedium involved when switching from Go->C->Go. There were problems like, "Well I should convert this value to a Golang type so the runtime
can track it and the garbage collector can eventually reap it", and "Well, should I store these values in the struct so I can call 'free' later?". So there are some nitty-gritty
details there. Anyways...the takeaway was on performance. Check out this sample code:

```go
package main

// ignore considerations of overflow here :)

/* int square(int x) {
  return x * x;
}
*/
import "C"

func sumFirstNSquares(n int) int {
  total := 0
  for i := 0; i < n; i++ {
    total += int(C.square(C.int(i)))
  }
  return total
}

func goSumFirstNSquares(n int) int {
  total := 0
  for i := 0; i < n; i++ {
    total += i * i
  }
  return total
}

// in square_test.go
package main

import "testing"

func BenchmarkSumNSquare(b *testing.B) {
  for n := 0; n < b.N; n++ {
    sumFirstNSquares(10000)
  }
}

func BenchmarkGoSumNSquare(b *testing.B) {
  for n := 0; n < b.N; n++ {
    goSumFirstNSquares(10000)
  }
}

```

On my machine here are the results:
```
╰─$ go test -bench=.
goos: darwin
goarch: amd64
pkg: github.com/arlandism/benchmark-cgo
BenchmarkSumNSquare-12            1989    549716 ns/op
BenchmarkGoSumNSquare-12        294370      4051 ns/op
PASS
ok    github.com/arlandism/benchmark-cgo2.572s
```

Native Go code has a crazy performance advantage in this scenario. The reasons why are beyond the scope of this post but the main takeaway is that there's overhead in calling C code that can have serious performance impacts. In this scenario, it's better to move the loop to C code to minimize the number of calls you have to make to CGo.


```go
package main

/* int square(int x) {
  return x * x;
}

  int sumFirstNSquares(int n) {
    int total = 0;
    for (int i = 0; i < n; i++) {
      total += square(i);
    }
    return total;
  }
*/

func betterSumFirstNSquares(n int) int {
  return int(C.sumFirstNSquares(C.int(n)))
}

// square_test
func BenchmarkBetterSumNSquare(b *testing.B) {
  for n := 0; n < b.N; n++ {
    betterSumFirstNSquares(10000)
  }
}
```

The results here are much better:

```
╰─$ go test -bench=.
goos: darwin
goarch: amd64
pkg: github.com/arlandism/benchmark-cgo
BenchmarkSumNSquare-12                2174    618537 ns/op
BenchmarkGoSumNSquare-12            295021      4050 ns/op
BenchmarkBetterSumNSquare-12      21644619        55.0 ns/op
PASS
ok    github.com/arlandism/benchmark-cgo4.107s
```

## Addendum

By the way, if you're curious about the rocksdb wrapper, I'll share it here.
It does make some assumptions about where rocksdb is installed on your system
so it won't run without some fiddling, but here it is:

```go
package main

/*
#cgo CFLAGS: -I${SRCDIR}/rocksdb/include
#cgo LDFLAGS: ./librocksdb.6.20.dylib
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "rocksdb/c.h"
*/
import "C"
import (
  "fmt"
  "unsafe"
)

type DB struct {
  options *C.rocksdb_options_t
  conn    *C.rocksdb_t
  path    *C.char
  err     *C.char
}

func NewDB(dbPath string) *DB {
  options := C.rocksdb_options_create()
  path := C.CString("./tmp")
  err := C.CString("")
  conn := C.rocksdb_open(options, path, &err)
  if C.GoString(err) != "" {
    panic(fmt.Sprintf("couldn't open database: %s", C.GoString(err)))
  }
  return &DB{
    options: options,
    conn:    conn,
    path:    path,
    err:     err,
  }
}

func (db *DB) TeardownDB() {
  C.free(unsafe.Pointer(db.options))
  C.free(unsafe.Pointer(db.conn))
  C.free(unsafe.Pointer(db.path))
  C.free(unsafe.Pointer(db.err))
}

func (db *DB) Get(key string) string {
  readOptions := C.rocksdb_readoptions_create()
  cKey := C.CString(key)
  retLen := C.size_t(0)
  val := C.rocksdb_get(db.conn, readOptions, cKey, C.ulong(len(key)), &retLen, &db.err)
  if C.GoString(db.err) != "" {
    panic(fmt.Sprintf("problem retrieving key: %s", C.GoString(db.err)))
  }
  goVal := C.GoString(val) // switch back to Go-land so GC can track
  defer func() {
    C.free(unsafe.Pointer(cKey))
    C.free(unsafe.Pointer(readOptions))
    C.free(unsafe.Pointer(val))
  }()
  return goVal
}

func (db *DB) Put(key string, value string) {
  cKey := C.CString(key)
  cVal := C.CString(value)
  writeOptions := C.rocksdb_writeoptions_create()
  defer func() {
    C.free(unsafe.Pointer(cKey))
    C.free(unsafe.Pointer(cVal))
    C.free(unsafe.Pointer(writeOptions))
  }()
  C.rocksdb_put(db.conn, writeOptions, cKey, C.ulong(len(key)), cVal, C.ulong(len(value))+1, &db.err)
  if C.GoString(db.err) != "" {
    panic(fmt.Sprintf("problem writing key/value: %s", C.GoString(db.err)))
  }
}

func main() {
  db := NewDB("./tmp/database")
  db.Put("name", "arlandis")
  fmt.Printf("Retrieved name: %s\n", db.Get("name"))
  defer db.TeardownDB()
}

```

### Footnotes
1. [Code for this post](https://github.com/arlandism/toy-cgo-benchmarks)
2. [Cgo function call source](https://golang.org/src/runtime/cgocall.go)
3. [Dave Cheney on CGo](https://dave.cheney.net/2016/01/18/cgo-is-not-go)
4. [Bradfield's CSI description](https://bradfieldcs.com/csi/) - includes a description of Advanced Programming
