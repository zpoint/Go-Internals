# map

## contents

[related file](#related-file)

[memory layout](#memory-layout)

[introduce](#introduce)

[buckets](#buckets)

* [tophash](#tophash)
* [key and element](#key-and-element)

## related file

* src/runtime/map.go
* src/runtime/map_fast32.go
* src/runtime/map_fast64.go
* src/runtime/map_faststr.go
* src/runtime/type.go

## memory layout

There exists various different version of map implementation

```shell
example % cat my_dict.go
```

```go
package main

import "fmt"

func main() {
        m1 := make(map[int]string)
        for i := 0; i < 2; i++ {
                m1[i] = "aaa"
        }
        m1[300] = "aaa"
        m1[400] = "bbb"
        m2 := make(map[int]string)
        m2[300] = "ccc"
        m3 := make(map[int]int)
        m3[300] = 500
}
```

m1, m2 shares the same `maptype`, while m3 is of different `maptype`, the `maptype` is not attached to any of the instance, compiler already knows the type of instance when you declare it and `maptype` is stored as hint in the runtime system

![maptype](./maptype.png)

This is the layout of `hmap`

![hmap](./hmap.png)

## introduce

```go
func main() {
	m1 := make(map[int]string)
	m1[300] = "aaa"
}
```

`B` stores log2 of buckets num, you can get the total bucket count by `1 << B`, The initial value of `B` is 0, means there's only 1 bucket

When you access an element, the hash function attached to the `maptype` will be called, the hash result ` & bucketMask` will locate a bucket

```go
hash := t.hasher(noescape(unsafe.Pointer(&key)), uintptr(h.hash0))
bucket := hash & bucketMask(h.B)
```

![assign](./assign.png)

This is our example

![assign1](./assign1.png)

We can find that `count` stores how many elements currently stored inside the `map` so that `len(m1)` can be calculated in `O(1)` 

`flags` is used for indicating the state of current map

```go
iterator     = 1 // there may be an iterator using buckets
oldIterator  = 2 // there may be an iterator using oldbuckets
hashWriting  = 4 // a goroutine is writing to the map
sameSizeGrow = 8 // the current map growth is to a new map of the same size
```

`B` is bucket count in log2 format

`noverflow # TO DO`

`buckets` is a pointer which points to the head of current buckets

`nevacuate # TO DO`

`extra # TO DO`

## buckets

buckets is the actual memory container that stores the **key** and **value**, the **hash** function take the **key** and a **hash seed** and locate a bucket, and the **hash** part is done

The next step is linear search, traverse the bucket and find an entry which is currently avaliable, take it

If the current bucket is full, the linear search algorithm will cross the bucket boundary to the next bucket, and repeat the searching process

### tophash

### key and element

