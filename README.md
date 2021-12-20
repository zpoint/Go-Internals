# Go-Internals![image title](http://www.zpoint.xyz:8080/count/tag.svg?url=github%2Fgo-Internals)
* [简体中文](https://github.com/zpoint/Go-Internals/blob/master/README_CN.md)
* **Watch** this repo if you need to be notified when there's update

This repository is my notes/blog for [go](https://github.com/golang/go) source code

```shell script
# based on the current master branch which is 1.17
cd go
git reset --hard 891547e2d4bc2a23973e2c9f972ce69b2b48478e
```

# Table of Contents

* [Debug](#Debug)
* [Objects](#Objects)
* [Runtime](#Runtime)

# Debug

- [x] [setup(build other go from source  code and add a print call to runtime package)](https://github.com/zpoint/Go-Internals/blob/master/debug/setup/setup.md)

# Objects

- [x] [map](https://github.com/zpoint/Go-Internals/blob/master/objects/map/map.md)
- [x] [channel](https://github.com/zpoint/Go-Internals/blob/master/objects/chan/chan.md)
- [x] [slice](https://github.com/zpoint/Go-Internals/blob/master/objects/slice/slice.md)
- [x] [string](https://github.com/zpoint/Go-Internals/blob/master/objects/string/string.md)

# Runtime

- [x] goroutine
	- [x] [overview and schedule](https://github.com/zpoint/Go-Internals/blob/master/runtime/goroutine/goroutine.md)
- [x] [gc](https://github.com/zpoint/Go-Internals/blob/master/runtime/gc/gc.md)
- [x] [memory management](https://github.com/zpoint/Go-Internals/blob/master/runtime/memory_management/memory_management.md)

# Learning material

[\<\<Go语言设计与实现\>\>](https://item.jd.com/13521160.html)
