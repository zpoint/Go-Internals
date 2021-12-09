# Slice

## contents

[related file](#related-file)

[memory layout](#memory-layout)



## related file

* src/runtime/slice.go

## memory layout

The layout of slice is quiet simple

`array` is the pointer to the beginning of the data

`len` is the size of element currently in `slice`

`cap` is the capacity of the `slice`

![./layout](./layout.png)
