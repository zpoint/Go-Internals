# Go-Internals![image title](http://www.zpoint.xyz:8080/count/tag.svg?url=github%2Fgo-Internals-CN)
* [English](https://github.com/zpoint/Go-Internals)
* 如果你需要接收更新通知, 点击右上角的 **Watch**, 当有文章更新时会在 issue 发布相关标题和链接

这个仓库是本人分析 [go](https://github.com/golang/go) 源码的时候做的记录/博客

```shell script
# based on the current master branch which is 1.14
cd go
git reset --hard dd150176c3cc49da68c8179f740eadc79404d351
```

本人是 [go](https://github.com/golang/go) 新手, 本人在写下列内容时会尽力保持和 [CPython-Internals](https://github.com/zpoint/CPython-Internals) 还有 [Redis-Internals](https://github.com/zpoint/Redis-Internals) 一致的风格



# 目录

* [Debug](#Debug)
* [Objects](#Objects)
* [Runtime](#Runtime)

# Debug

- [x] [setup(编译安装额外版本/runtime包中增加print函数调用)](https://github.com/zpoint/Go-Internals/blob/1.14/debug/setup/setup_cn.md)

# Objects

- [x] [map](https://github.com/zpoint/Go-Internals/blob/1.14/objects/map/map_cn.md)
- [ ] [channel](https://github.com/zpoint/Go-Internals/blob/1.14/objects/channel/channel_cn.md)

# Runtime

[goroutine](https://github.com/zpoint/Go-Internals/blob/1.14/runtime/goroutine/goroutine_cn.md)

[gc](https://github.com/zpoint/Go-Internals/blob/1.14/runtime/gc/gc_cn.md)

