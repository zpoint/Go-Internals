# setup ![image title](http://www.zpoint.xyz:8080/count/tag.svg?url=github%2Fgo-Internals%2Fsetup)

# contents

[why](#why)

[how](#how)

* [install other go version from source](#Install-other-go-version-from-source)
* [add a print function call to runtime/map.go](#add-a-print-function-call-to-runtime/map.go)

[read more](#read-more)

# why

In order to debug the [go](https://github.com/golang/go) programming language, The first step is to add a print function call to any of the runtime package and build from the source code, compile a helloworld example to observe the output

There exist other alternative [delve](https://github.com/go-delve/delve) as go debugger

I need them both

# how

Let's begin with ```src/runtime/map.go```

The following steps are executed on Mac OS, The idea is the same but commands are platform dependent, For other platform, please search for the proper command

## Install other go version from source

```shell
# based on the current master branch which is 1.15
cd go
git reset --hard d317ba5d4489c1ef53d3077afbff30eb72d7d3b0
```

We have already installed a go

```bash
% brew install go
...
% go version
go version go1.15.2 darwin/amd64
```

And we need to build a new one from source

```bash
% git clone https://github.com/golang/go.git
% cd go
% git reset --hard d317ba5d4489c1ef53d3077afbff30eb72d7d3b0
% cd src
% vim all.bash
```

Comment the 13th line, and you won't run the test case which may take about 5 minutes each time you build from source, Because I will change the source code and rebuild frequently, I comment this line

```bash
./make.bash "$@" --no-banner
# bash run.bash --no-rebuild
```

In ```make.bash```, replace the ```./cmd/dist/dist bootstrap $buildall $vflag $GO_DISTFLAGS "$@"``` to ```./cmd/dist/dist bootstrap -a -v```, If you don't replace it, the first build will also success

```bash
% vim make.bash
# the origin line
# ./cmd/dist/dist bootstrap $buildall $vflag $GO_DISTFLAGS "$@"
# the replaced line, -a means build all, -v will output any errors occur
./cmd/dist/dist bootstrap -a -v
# If you only change one file and don't want to build all, you can build a single directory
# ./cmd/dist/dist install  -v "runtime"
```

And build from the source

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

Add the specific version to `~/.zshenv`(or `~/.bash_profile` depends on your shell)

```bash
% vim ~/.zshenv
alias go_dev=/Users/zpoint/Desktop/go/bin/go
% source ~/.zshenv
% go_dev version
go version devel +d317ba5d44 Sat Oct 10 00:55:54 2020 +0000 darwin/amd64
% which go_dev
go_dev: aliased to /Users/zpoint/Desktop/go/bin/go
% go_dev env GOROOT
/Users/zpoint/Desktop/go
```

## add a print function call to runtime/map.go

If we edit the file ```src/runtime/map.go``` to import a ```fmt``` package and add a ```fmt.Println``` function call in ```makemap``` function

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

rebuild the runtime package

```bash
% vim make.bash
# ./cmd/dist/dist bootstrap -a -v
./cmd/dist/dist install  -v "runtime"
src % ./all.bash
```

And try to compile and run an example

```go
% cat my_dict.go 
package main

import "fmt"

func main() {
        d := make(map[int]string, 10)
        fmt.Println("d", d)
}

% go_dev run my_dict.go 
package command-line-arguments
        imports fmt
        imports errors
        imports internal/reflectlite
        imports runtime
        imports fmt: import cycle not allowed
```

We find that the `fmt` package import `runtime` and `runtime` import `fmt`(we manually add it) which cause import cycle

So we're not able to use `fmt` inside the `runtime` package, Luckily, There  exist a file `runtime/print.go` which has some low level print function can be used directly inside the  `runtime` package

Or we can use the built-in `print` and `println` function 

Again we edit ```src/runtime/map.go```

```go
func makemap(t *maptype, hint int, h *hmap) *hmap {
	printstring("t *maptype: ")
	printpointer(unsafe.Pointer(t))
	printstring("\thint： ")
	printint(int64(hint))
	printstring("\thmap： ")
	printpointer(unsafe.Pointer(h))
	printstring("\n")
	println("maptype", t)
	// ...
}
```

rebuild `runtime` package and run our example

```bash
src % ./all.bash
...
example % zpoint@zpoints-MacBook-Pro example % go_dev run my_dict.go
t *maptype: 0x10b5840   hint： 10       hmap： 0x0
maptype 0x10b5840
d map[]
```

And it works

# read more

[Go: Installing Multiple Go Versions from Source](https://medium.com/@vCabbage/go-installing-multiple-go-versions-from-source-db5573067c)

[is-there-anything-in-zsh-like-bash-profile](https://stackoverflow.com/questions/23090390/is-there-anything-in-zsh-like-bash-profile)

[difference-between-fmt-println-and-println-in-go](https://stackoverflow.com/questions/14680255/difference-between-fmt-println-and-println-in-go)