# chan

## 目录

[相关位置文件](#相关位置文件)

[内存构造](#内存构造)

[创建](#创建)

[send](#send)

[recv](#recv)

[select](#select)

[示例](#示例)

* [sendq 和 recvq](#sendq-和-recvq)
* [sendx 和 recvx](#sendx-和-recvx)



## 相关位置文件

* src/runtime/chan.go

## 内存构造

![hchan](./hchan.png)

`qcount` 表示当前队列中存储了多少个元素(当前数量)

`dataqsiz` 表示环形队列的的大小(最大数量)

`buf` 指向环形队列所在内存的起始位置

`elemsize` 是每个元素占用的大小(单位为字节)

`closed` 表示当前的通道是否已关闭

稍后会解释 `sendx`  和  `recvx` 

`recvq` 是一个 goroutine 接收者队列, 链表存储

`sendq` 是一个 goroutine 发送者队列, 链表存储

`lock` 是用作保护 `hchan` 数据结构的锁

## 创建

管道的创建最终都会调用到 `src/runtime/chan.go` 中的 `makechan` 函数

对于不同的元素大小, 会使用不同的内存分配策略

```go
	switch {
    case mem == 0:
      // 队列或者元素大小为 0
      c = (*hchan)(mallocgc(hchanSize, nil, true))
      // 在这个位置进行竞争同步
      c.buf = c.raceaddr()
    case elem.ptrdata == 0:
      // 元素结构不包含指针
      // 通过一次分配获得 hchan 和 buf 所需的空间
      c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
      c.buf = add(unsafe.Pointer(c), hchanSize)
    default:
      // 元素结构包含指针
      c = new(hchan)
      c.buf = mallocgc(mem, elem, true)
	}
```

# send

```go
//  c <- x 编译后的代码执行入口
//go:nosplit
func chansend1(c *hchan, elem unsafe.Pointer) {
	chansend(c, elem, true, getcallerpc())
}
```

`chansend` 会锁住 `lock` 这把锁, 如果`recvq` 不为空的话, 则尝试从里面弹出一个 goroutine, 并把元素发送给这个弹出的 goroutine 处理. 如果 `recvq` 为空, 则尝试把这个元素存储到环形队列中, 如果环形队列满了的话, 则把当前协程加入 `sendq` 并阻塞当前的 goroutine(有一个 block 参数判断需不需要阻塞, 当前值为 `true`)

![chansend](./chansend.png)

## recv

```go
// <- c 编译后的代码执行入口
//go:nosplit
func chanrecv1(c *hchan, elem unsafe.Pointer) {
	chanrecv(c, elem, true)
}
```

`chanrecv` 的流程和 `chansend` 的流程类似

首先, 锁住 `lock` 这把锁, 如果 `sendq` 不为空的话, 则尝试从里面弹出一个 goroutine 并获取该协程中存储的对应的元素, 如果 `sendq` 为空, 则尝试从环形队列中获取一个元素, 如果队列已经为空了, 则把当前协程加入`recvq` 并阻塞当前的 goroutine(有一个 block 参数判断需不需要阻塞, 当前值为 `true`)

![chanrecv](./chanrecv.png)

## select

select 被编译器处理后转换成以下两个函数

```go
// 编译器实现
//
//	select {
//	case v = <-c:
//		... foo
//	default:
//		... bar
//	}
//
// 编译成
//
//	if selectnbrecv(&v, c) {
//		... foo
//	} else {
//		... bar
//	}
//
func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
	return chansend(c, elem, false, getcallerpc())
}

// 编译器实现
//
//	select {
//	case v, ok = <-c:
//		... foo
//	default:
//		... bar
//	}
//
// 编译成
//
//	if c != nil && selectnbrecv2(&v, &ok, c) {
//		... foo
//	} else {
//		... bar
//	}
//
func selectnbrecv(elem unsafe.Pointer, c *hchan) (selected bool) {
	selected, _ = chanrecv(c, elem, false)
	return
}
```

select 的发送和接收和 [send](#send) 还有 [recv](#recv) 调用的是相同的函数, 唯一的区别是 `block` 参数由 `true` 变成了 `false`

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	// ...
  	if !block {
			unlock(&c.lock)
			return false, false
	}
  // ...
}

func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
  // ...
  	if !block {
			unlock(&c.lock)
			return false
	}
  // ...
}
```

`chanrecv` 和 `chansend` 都会检查 `block` 参数来确认是否需要阻塞当前的 goroutine

## 示例

### sendq 和 recvq

```go
package main

var c chan int = make(chan int, 5)

func main() {
	for i := 0; i < 5; i++ {
		c <- i
	}
}

```

![example](./example.png)

这是执行完 `example` 后的状态, `dataqsiz` 为 5 表示当前队列里有 5 个元素, `buf` 指向存储对象的内存块的起始位置, `elemsize` 为 8, 也是我的机器上 golang 的 `int` 的大小

`closed`  表示当前的通道是否已被关闭

如果我们再发送一次

```go
c <- 6
```

![example2](./example2.png)

当前的 goroutine 会被添加到 `sendq` 的队列中, 并阻塞, 发送的元素的指针也会保存到当前的 goroutine 中, 存储在 `sudog` 结构中

如果我们再添加多一个发送方

```go
package main

import "time"

var c chan int = make(chan int, 5)

func send() {
	time.Sleep(2 * time.Second)
	c <- 7
}

func main() {
	go send()
	for i := 0; i < 5; i++ {
		c <- i
	}
	c <- 6
}
```

![example3](./example3.png)

The `recvq` has the same structure as `sendq`, while it stores all the goroutines blocking in receiving from the channel

### sendx 和 recvx

If we run the following new example

```go
package main

var c chan int = make(chan int, 5)

func main() {
	for i := 0; i < 5; i++ {
		c <- i
	}
	// break point 1
	<- c
	<- c
	// break point 2
	c <- 6
	c <- 7
	// break point 3
}

```

In `break point 1`, the situation is the same as the initial state in [example](#example)

![breakpoint1](./breakpoint1.png)

In `break point 2`, the `recvx` is moved forward by 2, while `sendx` remains the same, `qcount` becomes 3

![breakpoint2](./breakpoint2.png)

In `break point 3`， the `recvx` also moved forawrd 2, `qcount` becomes 5， the channel becomes full

![breakpoint3](./breakpoint3.png)

