# gc

# contents

[related file](#related-file)

[read more](#read-more)

# related file

* src/runtime/mgc.go
* src/runtime/malloc.go
* src/runtime/mgcmark.go
* src/runtime/mbitmap.go

```bash
go tool compile -S -N example_new.go > file.s
go run -gcflags '-N -l' example_new.go
```

```go
// example_new.go
package main

func i() *int {
	r := new(int)
	return r
}

func main() {
	i()
}

```

If we inspect the `file.s`, we can learn that `new(int)` will call `runtime.newobject` by comipler, follow the function definition in `runtime.newobject`, we can find that a resource named `span` will be selected and the memory space required by the type will be allocated from the span, and the span wil mark the object inside it's structure in a bitmap, so that gc can track which object is allocated by inspecting the span

# read more

[garbage-collection-in-go-part1](https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html)

[garbage-collection-in-go-part2](https://www.ardanlabs.com/blog/2019/05/garbage-collection-in-go-part2-gctraces.html)

[garbage-collection-in-go-part3](https://www.ardanlabs.com/blog/2019/07/garbage-collection-in-go-part3-gcpacing.html)

[easy-to-read-golang-assembly-output](https://stackoverflow.com/questions/23789951/easy-to-read-golang-assembly-output)

[go-how-does-the-garbage-collector-mark-the-memory](https://medium.com/a-journey-with-go/go-how-does-the-garbage-collector-mark-the-memory-72cfc12c6976)

