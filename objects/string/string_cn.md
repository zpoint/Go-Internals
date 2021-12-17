# string![image title](http://www.zpoint.xyz:8080/count/tag.svg?url=github%2Fgo-Internals%2F/runtime/string_cn)

## 目录

[相关位置文件](#相关位置文件)

[内存构造](#内存构造)

[编码](#编码)

[更多资料](#更多资料)

## 相关位置文件

* src/runtime/string.go

## 内存构造

 `string` 的内存构造比较简单

`str` 指向了 byte 数组的头部

`len` 存储了 `string` 的长度

![./layout](./layout.png)

## 编码

```go
package main

import "strconv"

func main() {
	var s string = "我是"
	for pos, val := range s {
		println(pos, val, strconv.FormatInt(int64(val), 16))
	}
	println(len(s), &s)
}

```

上述代码的运行结果如下

```shell
0 25105 6211
3 26159 662f
6 0xc000046768
```

> 对于字符串来说, range 会按照 UTF-8 格式进行解码并把每个unicode字符返回给你

`0x6211` 是 `我` 的 unicode 编码

`0x662f` 是 `是` 的 unicode 编码

`我是` 的 `utf-8` 编码为 `b'\xe6\x88\x91\xe6\x98\xaf'`

```go
package main

import (
	"strconv"
)

func main() {
	var s string = "我是"

	for pos, val := range []rune(s) {
		println(pos, strconv.FormatInt(int64(val), 16))
	}
	for pos, val := range []byte(s) {
		println(pos, strconv.FormatInt(int64(val), 16))
	}
}
```

上述代码的运行结果如下

```shell
0 6211
1 662f
0 e6
1 88
2 91
3 e6
4 98
5 af
```

从上述结果我们可以发现, `rune` 会遍历 `string` 中的每个 `unicode` 字符

 `byte` 则会遍历  `string` 中的每个字节

并且 `string` 是以 `utf-8` 格式编码存储的

下图表示了 `unicode` 和 `utf8` 的区别(`unicode` 是一个字符映射标准, 表明了每个字符对应的码是多少, `utf8` 是一种编码方式, 它基于 `unicode` 进行压缩存储节省空间, 知道编解码方法后就可以存储和还原)

如果我们把 `utf-8`  格式中的标记位去除, 把红色部分的 bit 拼起来, 就还原除了 `unicode` 字符

对于不了解 `utf-8` 的同学, 当我们扫描第一个字节的时候, 在遇到第一个`0`之前, 统计有多少个连续的`1`, 这样就能知道存储当前这一个

`nicode` 字符用了多少个字节, 在当前的例子是 3, 那么就一共是 3 个字节, 除开开始的 `1` 和 `0`, 剩下的 `bit` 仍然存储了我们需要的信息

除开第一个字节的话, 剩下的字节都以 `10` 开头, 所以后面的两个字节, 排除前两个 `bit`, 也存了我们需要的信息

把需要的信息拼接起来,  就是我们的 unicode 字符

重复上述过程即可完成整个字符串的扫描

![./unicode](./unicode.png)

## 更多资料

[Strings In Go's Runtime](https://boakye.yiadom.org/go/strings/)

