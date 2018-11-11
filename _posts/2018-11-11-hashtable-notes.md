---
layout: article
title: Hash Table Notes
date: 2018-11-06 21:00:00
categories: programming
tags:
- programming
- learning
---

Below is a random collection of notes from reading the CLRS chapter on hash tables.

This is an overview of how hash tables work while glossing over implementation details. Real-life hash tables barely resemble this since they need to account for far more than comprehension.

## Data Structure

A hash table is essentially a kind of multi-dimensional list. Lookups are performed by using a hash function to generate an index for accessing the top-level list and then searching through the corresponding secondary list for the matching value. The layout may look something like:

```go
type HashTableItem struct {
  key   string
  value string
}

type HashTableChain struct {
  items []HashTableItem
}

type HashTable struct {
  size   int
  chains [size]HashTableChain
}
```

## Hash Functions

An example of a hash function could be something as simple as:

```go
package main

import "fmt"

// Sum the ascii values of the string and then mod it to generate an array index
func hash(key string, hashLen int) int {
  sum := 0
  for char := range key {
    sum += int(char)
  }
  return sum % hashLen
}

func main() {
  keys := []string{"grey", "apple", "dance", "community", "bottle", "cup", "free", "bird", "country", "elephant"}
  var lowtechhashtable [10][]string
  for i := 0; i < 10; i++ {
    lowtechhashtable[i] = make([]string, 5)
  }

  for _, k := range keys {
    hashKey := hash(k, 10)
    chain := append(lowtechhashtable[hashKey], k)
    lowtechhashtable[hashKey] = chain
    fmt.Printf("hash key: %d - ", hashKey)
    fmt.Println(lowtechhashtable)
  }
}

// hash key: 6 - [[    ] [    ] [    ] [    ] [    ] [    ] [     grey] [    ] [    ] [    ]]
// hash key: 0 - [[     apple] [    ] [    ] [    ] [    ] [    ] [     grey] [    ] [    ] [    ]]
// hash key: 0 - [[     apple dance] [    ] [    ] [    ] [    ] [    ] [     grey] [    ] [    ] [    ]]
// hash key: 6 - [[     apple dance] [    ] [    ] [    ] [    ] [    ] [     grey community] [    ] [    ] [    ]]
// hash key: 5 - [[     apple dance] [    ] [    ] [    ] [    ] [     bottle] [     grey community] [    ] [    ] [    ]]
// hash key: 3 - [[     apple dance] [    ] [    ] [     cup] [    ] [     bottle] [     grey community] [    ] [    ] [    ]]
// hash key: 6 - [[     apple dance] [    ] [    ] [     cup] [    ] [     bottle] [     grey community free] [    ] [    ] [    ]]
// hash key: 6 - [[     apple dance] [    ] [    ] [     cup] [    ] [     bottle] [     grey community free bird] [    ] [    ] [    ]]
// hash key: 1 - [[     apple dance] [     country] [    ] [     cup] [    ] [     bottle] [     grey community free bird] [    ] [    ] [    ]]
// hash key: 8 - [[     apple dance] [     country] [    ] [     cup] [    ] [     bottle] [     grey community free bird] [    ] [     elephant] [    ]]
```

The chosen hash function should be really good at randomizing the chosen index for an item to maximize utilization. If this is the case, then the average expected access time is O(1). In the worst case, every item could hash to the same index so looking for an item degrades to searching through all of the items (i.e. O(n)).
