# goroutine

## contents

[related file](#related-file)

[overview](#overview)

[memory layout](#memory-layout)

[read more](#read-more)

## related file

* src/runtime/runtime2.go
* src/runtime/proc.go
* src/plugin/plugin_dlopen.go

## overview

If you're coufused about **What MPG means in go scheduling and how it works**, please read [scheduling-in-go-part1](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html) ~ [scheduling-in-go-part3](https://www.ardanlabs.com/blog/2018/12/scheduling-in-go-part3.html) first in [read more](#read-more)

```shell
# GOSSAFUNC=main GOOS=linux GOARCH=amd64 go build -gcflags "-S" simple.go
# GOSSAFUNC=main go_dev build -gcflags "-S" num_cpu.go
# GODEBUG=schedtrace=DURATION,gctrace=1 go_dev run num_cpu.go
GODEBUG=schedtrace=DURATION go_dev run num_cpu.go
find . -name '*.go' -exec grep -nHr 'inittask' {} \;
```





 ## read more

[scheduling-in-go-part1](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html)

[scheduling-in-go-part2](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)

[scheduling-in-go-part3](https://www.ardanlabs.com/blog/2018/12/scheduling-in-go-part3.html)

[how-a-go-program-compiles-down-to-machine-code](https://getstream.io/blog/how-a-go-program-compiles-down-to-machine-code/)

[https://sitano.github.io/2016/04/28/golang-private/](How to call private functions (bind to hidden symbols) in GoLang)

