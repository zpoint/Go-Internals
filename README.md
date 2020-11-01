# Go-Internals![image title](http://www.zpoint.xyz:8080/count/tag.svg?url=github%2Fgo-Internals)
* [简体中文](https://github.com/zpoint/Go-Internals/blob/1.15/README_CN.md)
* **Watch** this repo if you need to be notified when there's update

This repository is my notes/blog for [go](https://github.com/golang/go) source code

```shell script
# based on the current master branch which is 1.15
cd go
git reset --hard d317ba5d4489c1ef53d3077afbff30eb72d7d3b0
```

# Table of Contents

* [Debug](#Debug)
* [Objects](#Objects)
* [Runtime](#Runtime)

# Debug

- [x] [setup(build other go from source  code and add a print call to runtime package)](https://github.com/zpoint/Go-Internals/blob/1.14/debug/setup/setup.md)

# Objects

- [x] [map](https://github.com/zpoint/Go-Internals/blob/1.15/objects/map/map.md)
- [x] [channel](https://github.com/zpoint/Go-Internals/blob/1.15/objects/chan/chan.md)

# Runtime

- [x] goroutine
	- [x] [overview and schedule](https://github.com/zpoint/Go-Internals/blob/1.15/runtime/goroutine/goroutine.md)
- [ ] [gc](https://github.com/zpoint/Go-Internals/blob/1.15/runtime/gc/gc.md)