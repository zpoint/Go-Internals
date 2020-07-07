# setup

# 目录

[为什么](#为什么)

[怎么做](#怎么做)

* [从源代码编译安装另一个版本的 go](#从源代码编译安装另一个版本的-go)
* [在 runtime/map.go 中调用 print 函数](#在-runtime/map.go-中调用-print-函数)

[更多资料](#更多资料)

# 为什么

我们需要能对 [go](https://github.com/golang/go) 的源码进行调试, 修改并重建, 第一步是在任意一个内建的运行时包里增加一个 print 调用, 重新编译, 之后用编译好的编译器去运行一个 helloworld 脚本观察输出

还有另一个工具  [delve](https://github.com/go-delve/delve) 可配套作为 go 的 debugger 也可以使用

我需要多种方式的支持

# 怎么做

我们从 ```src/runtime/map.go``` 开始

下列的命令和步骤都是在 Mac OS 下运行的, 不同平台的具体命令不同, 但步骤和思路是相同的, 其他平台替换成对应的命令即可

## 从源代码编译安装另一个版本的 go

比如我先前已经安装过了一个版本的 go

```bash
% brew install go
...
% go version
go version go1.14.4 darwin/amd64
```

我们需要从源代码编译安装一个新的 go

```bash
% git clone https://github.com/golang/go.git
% cd go
% git reset --hard dd150176c3cc49da68c8179f740eadc79404d351
% cd src
% vim all.bash
```

因为我需要频繁的修改源代码, 并且目的是观察输出而不是开发功能模块, 所以我把第 13 行注释掉了, 这样每次构建时, 不需要重新跑一遍测试脚本(大概耗时5分钟)

```bash
. ./make.bash "$@" --no-banner
# bash run.bash --no-rebuild
```

在 ```make.bash``` 中, 把 ```./cmd/dist/dist bootstrap $buildall $vflag $GO_DISTFLAGS "$@"```  替换成 ```./cmd/dist/dist bootstrap -a -v```, 不替换的话第一次构建也是能成功的, 默认下 `-v` 标记没有打开, 你是看不到进度和错误提示的

```bash
% vim make.bash
# 原本的那行
# ./cmd/dist/dist bootstrap $buildall $vflag $GO_DISTFLAGS "$@"
# 替换之后的行, -a 表示所有模块, -v 会输出任何错误提示
./cmd/dist/dist bootstrap -a -v
# 如果你只更改了某一个包下的文件, 并不需要重新构建整个项目, 你可以只指定某一个目录
# ./cmd/dist/dist install  -v "runtime"
```

执行编译脚本

```bash
src % ./all.bash
...
Installed Go for darwin/amd64 in /Users/zpoint/Desktop/go
Installed commands in /Users/zpoint/Desktop/go/bin
---
Installed Go for darwin/amd64 in /Users/zpoint/Desktop/go
Installed commands in /Users/zpoint/Desktop/go/bin
*** You need to add /Users/zpoint/Desktop/go/bin to your PATH
```

并把当前编译好的二进制文件目录起一个别名加到 `~/.zshenv` 中(因 shell 脚本程序而异, 有的是 `~/.bash_profile`)

```bash
% vim ~/.zshenv
alias go_dev=/Users/zpoint/Desktop/go/bin/go
% source ~/.zshenv
% go_dev version
go version devel +dd150176c3 Fri Jul 3 03:31:29 2020 +0000 darwin/amd64
% which go_dev
go_dev: aliased to /Users/zpoint/Desktop/go/bin/go
% go_dev env GOROOT
/Users/zpoint/Desktop/go
```

## 在 runtime/map.go 中调用 print 函数

如果我们修改 ```src/runtime/map.go``` 并且导入 ```fmt```, 之后在 ```makemap``` 中调用 ```fmt.Println``` 方法

```go
import (
	// ...
   "fmt"
)
// ...
func makemap(t *maptype, hint int, h *hmap) *hmap {
	fmt.Println("in t *maptype", t, "hint int", hint, "h *hmap", h)
  // ...
}
```

重建 runtime 包

```bash
% vim make.bash
# ./cmd/dist/dist bootstrap -a -v
./cmd/dist/dist install  -v "runtime"
src % ./all.bash
```

并且尝试编译运行一个示例

```go
% cat my_dict.go 
package main

import "fmt"

func main() {
        d := make(map[int]string, 10)
        fmt.Println("d", d)
}

% go_dev run my_dict.go 
warning: GOPATH set to GOROOT (/Users/zpoint/Desktop/go) has no effect
package command-line-arguments
        imports fmt
        imports errors
        imports internal/reflectlite
        imports runtime
        imports fmt: import cycle not allowed
```

我们发现 `fmt` 导入了 `runtime` 并且 `runtime` 导入了 `fmt`(我们刚刚进行的修改), 导致了引用循环

所以我们没法在 `runtime` 中使用 `fmt` 包, 幸运的是, 我在相同的包下发现了 `runtime/print.go`  文件, 里面有比较多的低级的 print 函数可在`runtime` 下直接使用

我们再次编辑 ```src/runtime/map.go```

```go
// ...
func makemap(t *maptype, hint int, h *hmap) *hmap {
	printstring("t *maptype: ")
	printpointer(unsafe.Pointer(t))
	printstring("\thint： ")
	printint(int64(hint))
	printstring("\thmap： ")
	printpointer(unsafe.Pointer(h))
	printstring("\n")
  // ...
}
```

重建 runtime 包并编译运行我们的示例脚本

```bash
src % ./all.bash
...
example % go_dev run my_dict.go
t *maptype: 0x10b5400   hint： 10       hmap： 0x0
d map[]
```

成功了

# 更多资料

[Go: Installing Multiple Go Versions from Source](https://medium.com/@vCabbage/go-installing-multiple-go-versions-from-source-db5573067c)

[is-there-anything-in-zsh-like-bash-profile](https://stackoverflow.com/questions/23090390/is-there-anything-in-zsh-like-bash-profile)

