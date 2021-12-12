# Slice

## contents

[related file](#related-file)

[memory layout](#memory-layout)

[makeslice](#makeslice)

[growslice](#growslice)

## related file

* src/runtime/slice.go

## memory layout

The layout of slice is quiet simple

`array` is the pointer to the beginning of the data

`len` is the size of element currently in `slice`

`cap` is the capacity of the `slice`

![./layout](./layout.png)





```go
package main

func main() {
	a := make([]int, 3)
	println(a)
}
```

If you try to compile the above code `go tool compile -S -N -l main.go`

```shell
"".main STEXT size=123 args=0x0 locals=0x50 funcid=0x0
        0x0000 00000 (main.go:3)        TEXT    "".main(SB), ABIInternal, $80-0
        0x0000 00000 (main.go:3)        CMPQ    SP, 16(R14)
        0x0004 00004 (main.go:3)        PCDATA  $0, $-2
        0x0004 00004 (main.go:3)        JLS     116
        0x0006 00006 (main.go:3)        PCDATA  $0, $-1
        0x0006 00006 (main.go:3)        SUBQ    $80, SP
        0x000a 00010 (main.go:3)        MOVQ    BP, 72(SP)
        0x000f 00015 (main.go:3)        LEAQ    72(SP), BP
        0x0014 00020 (main.go:3)        FUNCDATA        $0, gclocals·69c1753bd5f81501d95132d08af04464(SB)
        0x0014 00020 (main.go:3)        FUNCDATA        $1, gclocals·713abd6cdf5e052e4dcd3eb297c82601(SB)
        0x0014 00020 (main.go:4)        MOVQ    $0, ""..autotmp_1+24(SP)
        0x001d 00029 (main.go:4)        LEAQ    ""..autotmp_1+32(SP), AX
        0x0022 00034 (main.go:4)        MOVUPS  X15, (AX)
        0x0026 00038 (main.go:4)        LEAQ    ""..autotmp_1+24(SP), AX
        0x002b 00043 (main.go:4)        TESTB   AL, (AX)
        0x002d 00045 (main.go:4)        JMP     47
        0x002f 00047 (main.go:4)        MOVQ    AX, "".a+48(SP)
        0x0034 00052 (main.go:4)        MOVQ    $3, "".a+56(SP)
        0x003d 00061 (main.go:4)        MOVQ    $3, "".a+64(SP)
.......
```

We can find that go compiler use the current function stack to store slice `a` instead of mallocing from heap

If we modify it to a larger slice

```go
package main

func main() {
	a := make([]int, 30000)
	println(a)
}
```

Now the `runtime.makeslice` is called and the slice is malloced from the runtime heap

```shell
"".main STEXT size=118 args=0x0 locals=0x38 funcid=0x0
        0x0000 00000 (main.go:3)        TEXT    "".main(SB), ABIInternal, $56-0
        0x0000 00000 (main.go:3)        CMPQ    SP, 16(R14)
        0x0004 00004 (main.go:3)        PCDATA  $0, $-2
        0x0004 00004 (main.go:3)        JLS     111
        0x0006 00006 (main.go:3)        PCDATA  $0, $-1
        0x0006 00006 (main.go:3)        SUBQ    $56, SP
        0x000a 00010 (main.go:3)        MOVQ    BP, 48(SP)
        0x000f 00015 (main.go:3)        LEAQ    48(SP), BP
        0x0014 00020 (main.go:3)        FUNCDATA        $0, gclocals·69c1753bd5f81501d95132d08af04464(SB)
        0x0014 00020 (main.go:3)        FUNCDATA        $1, gclocals·713abd6cdf5e052e4dcd3eb297c82601(SB)
        0x0014 00020 (main.go:4)        LEAQ    type.int(SB), AX
        0x001b 00027 (main.go:4)        MOVL    $30000, BX
        0x0020 00032 (main.go:4)        MOVQ    BX, CX
        0x0023 00035 (main.go:4)        PCDATA  $1, $0
        0x0023 00035 (main.go:4)        CALL    runtime.makeslice(SB)
        0x0028 00040 (main.go:4)        MOVQ    AX, "".a+24(SP)
        0x002d 00045 (main.go:4)        MOVQ    $30000, "".a+32(SP)
        0x0036 00054 (main.go:4)        MOVQ    $30000, "".a+40(SP)

```

## makeslice

The `makeslice` calculate how many bytes needed according to the `type` and `cap`  of the current `slice`, and do something if overflow occured

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
  mem, overflow := math.MulUintptr(et.size, uintptr(cap))
  // omit ...
	return mallocgc(mem, et, true)
}
```

## growslice

If you call `append` and compile the file, you will find that it calls down to `growslice` in `src/runtime/slice.go`

```go
a = append(a, 3)
```

The grow pattern can be described as

![./grow_pattern](./grow_pattern.png)

When `cap` is less than 1024, `cap` is double each time, the grow speed is the yellow line

When `cap` is greater than or equal 1024, `cap` grows `1/4` each time, the grow speed is the blue line

