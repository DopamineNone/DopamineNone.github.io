---
layout: passenge
title: Golang并发
date: 2024-07-29 17:52:00
categories: Golang
tags: 
    - Golang
    - Grammar
    - Concurrent
---

> 并行与并发的区别：
>
> - 并发是指一个处理器同时处理多个任务。
> - 并行是指多个处理器或者是多核的处理器同时处理多个不同的任务。

Go 的并发模型基于 **协程** 和 **通道**（channels）。

## Goroutine

Goroutine就是Go并发中的协程，是一种更轻量的用户级线程，由Go在运行时管理。特点如下：

- 轻量：系统线程栈空间通常$\ge$1MB，Goroutine 的栈空间初始大小只有 2KB，可以动态扩容
- 高效：Goroutine 的调度器采用 M:N 模型，可以将 M 个 Goroutine 映射到 N 个 OS 线程上，实现高效调度
- 高并发：可创建数十万协程
- 方便：在Golang中，只要在函数调用前加上关键字`go`就可以启动异步Goroutine

这里给出一个简单的Goroutine例子：

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int) {
	fmt.Printf("Worker %d started\n", id)
	time.Sleep(2 * time.Second)
	fmt.Printf("Worker %d finished\n", id)
}

func main() {
	for i := range 5 {
		go worker(i) // 启动worker协程
	}
	time.Sleep(5 * time.Second) // 阻塞主线程，等待所有worker完成
    fmt.Println("Done.")
}

// 输出：
/*
Worker 1 started
Worker 0 started
Worker 4 started
Worker 3 started
Worker 2 started
Worker 2 finished
Worker 4 finished
Worker 3 finished
Worker 0 finished
Worker 1 finished
Done.
*/
```

这里补充一个**Go1.22**之前的版本存在的一个问题。在**Go1.22**之前，对于 `for` 循环中的范围表达式（`for range`），循环变量的初始化是在循环开始时仅执行一次的。这意味着每次循环迭代时，都会使用同一个变量，而不是为每次迭代创建一个新的变量副本。

通常情况下不会出现问题，除了`for range`循环变量和Goroutine结合使用时：

```go
func main() {
    alpha := []string{"a", "b"}
    for _, v := range alpha {
        go func() {
            fmt.Printf("%s", v)
        }()
    }
}
```

直观上觉得程序会输出`"ab"`，但实际上往往会输出`"bb"`。这是因为goroutine是异步的，当第一个循环结束时，程序可能还未打印出`v`，结果进入第二个循环后，`v`被修改为了`"b"`，两个goroutine就都打印了最后的`v`，即结果为`"bb"`

解决办法：

1. 函数传参
2. 创建新局部变量

```go
func main() {
    alpha := []string{"a", "b"}
    // 方法1
    for _, v := range alpha {
        go func(x string) {
            fmt.Printf("%s", x)
        }(v)
    }
    // 方法2
    for _, v := range alpha {
        newV := v
        go func() {
            fmt.Println("%s", newV)
        }()
    }
}
```

在Go1.22中，`for range`循环变量改成为每次迭代创建一个新的变量副本，故不存在上述问题了。

## Channel

`channel`是Go中的一种复杂数据类型，可看作特殊的队列，具有先进先出的特点，用于同步协程间通信。一般用于协程间的数据通信。

### 声明格式

一般的通道声明格式如下：

```go
// var ch chan [type]
var ch chan int
ch = make(chan int)
// 或者
ch := make(chan int)
// 当然你还可以指定缓冲区大小
ch := make(chan int, 10)
```

一个通道只能传输一种类型的数据；所有的类型的数据都可以用于通道，包括空接口。

使用`make()`对通道进行声明时，如果不指定缓冲区大小，则返回**无缓冲通道**，否则返回**带缓冲通道**。

通道中还有**只读通道**和**只写通道**。当然，声明一个只读或只写的通道没有意义，所以这两种通道一般用于构建函数参数：

```go
func Reader(ch <-chan int) {} // 只读通道
func Writer(ch chan<- int) {} // 只写通道
```

这样就可以保证`Reader`中ch通道是只读的，`Writer`中`ch`通道是只写的。下文会给出更详细的例子。

### 发送数据

```go
// 向通道写入10
ch <- 10 
```

值得注意的是，当向一个无缓冲通道（或者有缓冲但会写入数据量超过缓冲区的通道）写入数据时，必须保证有一个读协程随时准备从通道数据，否则会出现死锁报错：`fatal error: all goroutines are asleep - deadlock!`

```go
// Bad!
func main() {
    ch := make(chan int)
    ch <- 5 // Dead lock!
    fmt.Println(<-ch)
}

// Good!
func main() {
    ch := make(chan int)
    go func() {
        fmt.Println(<-ch)
    }()
    ch <- 5
}
```

### 接收数据

```go
// 读操作的第二个返回值如果是false，则管道关闭且为空
ret, ok := <-ch
// 当然你也可以省略ok
ret := <-ch
```

注意的是，当尝试向空通道进行读操作时，会引发**通道阻塞**，直到通道中有新值写入，这样新值就会被读出，并结束阻塞。

### 关闭channel

通道是可以关闭的。一旦关闭，就无法向该channel写入数据。**注意的是**，空通道关闭后，仍可以多次从通道读出零值。

```go
ch := make(chan int)
close(ch)
fmt.Println(<-ch) // 0
fmt.Println(<-ch) // 0
fmt.Println("Channel closed")
```

### 遍历channel

想要遍历`channel`前，必须先关闭`channel`。关闭`channel`后就不能向`channel`写入数据。

```go
// 关闭通道
close(ch)

// Bad!
ch <- 10 // panic: send on closed channel
```

遍历方法一般有两种，如下：

```go
// 方案A，能自动检测管道是否关闭
for val := range ch {
    fmt.Println(val)
}

// 方案B
for {
    val, ok := <- ch
    if !ok {
        break
    } 
    fmt.Println(val)
}

// Bad!
for i := 0; i < len(ch); i++ {
    // len(ch)会变化！
    fmt.Println(<-ch)
}
```



### Select语句

Go中的`select`语句是专门处理通道操作的语句，一般与`for`语句配合使用，也被频繁用在Go的并发编程中：

```go
for {
	select {
    case <-ch1:
        // 当 ch1 准备好接收时执行这里的代码
    case ch2 <- "value":
        // 当 ch2 准备好发送时执行这里的代码
    default:
        // 如果没有通道准备就绪，则执行这里的代码
    }
}
```

select语句特点如下：

1. **非阻塞**:
   - 如果没有任何 case 的通道准备就绪，`select` 语句将选择执行 `default` 子句（如果有的话）。
   - 如果没有 `default` 子句并且所有 case 的通道都不准备就绪，则 `select` 语句将阻塞，直到其中一个通道准备就绪。
2. **随机选择**:
   - 如果有多个 case 的通道都准备就绪，`select` 语句将随机选择一个 case 来执行。
   - 这种随机选择有助于避免死锁和其他竞态条件。
3. **case 表达式**:
   - `select` 语句的每个 case 必须是一个通道操作，例如发送或接收。

## WaitGroup

除了`channel`，Go中还提供了一些重要的工具来协调goroutine之间的同步。`sync`包下的`WaitGroup`是其中之一。

`WaitGroup`用于确保一些goroutine完成其任务后程序再执行其他内容。这里我们可以回顾一下全文的第一个代码示例：

```go
func main() {
	for i := range 5 {
		go worker(i) // 启动worker协程
	}
	time.Sleep(5 * time.Second) // 阻塞主线程，等待所有worker完成
    fmt.Println("Done.")
}
```

这里的`time.Sleep(5 * time.Second)`是用于阻塞主线程的进行，来确保goroutine执行完再打印最后的`"Done."`。显然我们无法预测所有程序的goroutine执行所需的大概时间，所以这里使用`WaitGroup`可以更有效地同步协程与主线程：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("Worker %d started\n", id)
	time.Sleep(2 * time.Second)
	fmt.Printf("Worker %d finished\n", id)
}

func main() {
	var wg sync.WaitGroup
	for i := range 5 {
		wg.Add(1)
		go worker(i, &wg)
	}
	wg.Wait()
	fmt.Println("All workers finished")
}
```

代码变换体现在`worker`函数和`main`函数中。`WaitGroup`本质上用一个**计数器**来实现协程的同步。接下来是对`WaitGroup`的三个方法的解释。

### WaitGroup.Add

用于向计数器添加一个计数值，表示当前任务列表中新值了多少任务（参数为负数时表示减少）。一般在你想协调的goroutine**任务执行前**调用`wg.Add(n)`。

最好不要使`Add`和`Wait`并发调用，否则有可能会达不到同步协程的效果：

```go
func main() {
    var wg sync.WaitGroup
    for i := range 2 {
        go func() {
            wg.Add(1)
            doSomething()
            wg.Done()
        }
    }
    wg.Wait()
    doMain()
}
```

该例子中，我们期望goroutine都执行完后执行`doMain()`，但实际上很有可能在goroutine开始执行前（执行`wg.Add(1)`前）就跳过了`wg.Wait()`，导致提前执行`doMain()`。

### WaitGroup.Done

用于让计数器减一，表示当前列表中有一个任务完成了。事实上，`wg.Done()`的底层实现就是`wg.Add(-1)`。一定要在每个goroutine完成任务后执行`wg.Done()`，否则会造成死锁。

当然执行了多余的`wg.Done()`也会导致死锁。

### WaitGroup.Wait

执行函数时，检测当前计数器的值是否为0，不是则阻塞当前进程/协程。

> 在需要将`WaitGroup`变量传入协程函数时，要使用指针引入，而不是值引入：
>
> ```go
> func worker(id int, wg sync.WaitGroup) { // 值引入，Bad!
>     defer wg.Done() // 看似wg.Done()执行了，实际上和main中的wg没关系。
>     doSomething()
> }
> 
> func main() {
> 	var wg sync.WaitGroup
> 	for i := range 5 {
> 		wg.Add(1)
> 		go worker(i, &wg)
> 	}
>     wg.Wait() // wg没Done()过，死锁。
> 	fmt.Println("All workers finished")
> }
> ```

## Mutex

在进行并发操作时，对于临界区的操作需要通过加锁来实现并发安全。Golang标准包`sync`提供了两种锁：

- 互斥锁（Mutex）
- 读写锁（RWMutex）

### Mutex

`Mutex`就两个方法：`Lock()`和`Unlock()`，对临界区操作前`Lock()`，如果有其他goroutine获取的锁，当前goroutine阻塞，否则当前goroutine获得锁（信号量机制），操作结束后`Unlock()`。

这里给一个场景：

```go
func main() {
	var wg sync.WaitGroup
	cnt := 0
	for i := 0; i < 100000; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			cnt++
		}()
	}
	wg.Wait()
	fmt.Println(cnt) // 97856

}
```

由于条件竞争，`cnt`并没有累加到100000。通过上锁就能解决问题。

```go
func main() {
	var mu sync.Mutex
	var wg sync.WaitGroup
	cnt := 0
	for i := 0; i < 100000; i++ {
		wg.Add(1)
		go func() {
			mu.Lock()
			defer mu.Unlock()
			defer wg.Done()
			cnt++
		}()
	}
	wg.Wait()
	fmt.Println(cnt) // 100000
}
```

### RWMutex

`RWMutex`相比互斥锁，它允许多个读操作同时进行，但在写操作进行时，会阻塞所有的读操作和写操作。这样可以提高并发性能。

`RWMutex`提供了四个方法：`RLock()`，`RUnlock()`，`Lock()`，`Unlock()`，前两者用于读操作，后两者用于写操作。使用方法与`Mutex`类似。

## Atomic

`sync/atomic`包提供了Golang中的一些**原子变量**和**原子操作**。原子操作，即不会被分割的操作，作用上与互斥锁相似，但底层由CPU指令实现，不涉及加锁解锁，故性能高于互斥锁。原子变量则是用于执行原子操作的特殊变量。

`sync/atomic`中原子操作一共有五类：

- 读取（Load）
- 写入（Store）
- 交换（Swap）
- 比较并交换 （CompareAndSwap）
- 增减（Add）

`sync/atomic`中原子变量类型有以下几种：

- bool 布尔值
- (u)int32 32位整型
- (u)int64 64位整型
- pointer 不可参与指针运算的指针
- uinptr  无法持有对象的指针
- value 空接口

所有的原子变量都实现了前四类的原子操作方法（Load, Store, Swap, CompareAndSwap）,能参与加减运算的类型还实现了第五类原子操作方法（Add）

### 原子操作函数

原子操作函数中第一个参数往往是数据地址，如

```go
func main() {
    var (
        wg sync.WaitGroup
        count int64
    )
	for _ = range 100000 {
		wg.Add(1)
		go func() {
			defer wg.Done()
            // atomic.AddInt64(addr *int64, delta int64)
			atomic.AddInt64(&count, 1)
		}()
	}
	wg.Wait()
	fmt.Println(count) // 100000
}
```

原子操作函数的操作对象限制在`(u)int32`,`(u)int64`,`uintptr`,`Pointer`上。

### 原子操作方法

官方文档更推荐使用原子变量，通过调用其方法来进行原子操作。这样比直接调用原子操作函数更加直观和不容易出错，支持的操作对象类型的更多。

```go
func main() {
    var (
        wg sync.WaitGroup
        count atomic.Int64
    )
	for _ = range 100000 {
		wg.Add(1)
		go func() {
			defer wg.Done()
            count.Add(1)
		}()
	}
	wg.Wait()
    fmt.Println(count.Load()) // 100000
}
```

## Context

Golang中`context`可用来定义goroutine的上下文，用优雅的方式传递取消信号和设置超时。

### 创建根节点Context

有两种方法创建空context：

```go
// 方法一：
ctx := context.Background()

// 方法二：
ctx := context.TODO()
```

两种方法能返回一个没有 deadline、没有取消函数的 `context.Context` 对象，只有语义的区别，即：

- **`context.Background()`**:
  - 表示一个顶层或根上下文。
  - 适用于程序启动时或作为顶级上下文来开始处理一个请求。
  - 一般用于实际的生产代码中。
- **`context.TODO()`**:
  - 表示一个待办事项上下文。
  - 主要用于代码尚未完成时作为占位符。
  - 不推荐在生产代码中使用。

### 创建派生节点

派生节点由根节点派生而来，用形如`WithXXX`格式的函数进行创建。

#### WithValue

创建一个带键值对的节点，同时保留父节点的数据。

```go
// WithValue(parent Context, key, val any) Context
ctx := context.WithValue(context.Background(), "root", "123456")
son := context.WithValue(ctx, "son", "234567")

fmt.Println(ctx.Value("root").(string)) // 123456
fmt.Println(son.Value("son").(string)) // 234567
```

#### WithCancel

创建一个派生节点和终止该节点执行的`cancel()`函数。

```go
func main() {
	var (
		wg sync.WaitGroup
		ch = make(chan int)
	)
    // WithCancel(ctx Context) (Context, CancelFunc)
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(2)
	// 生产者协程: 不断生成数据并放入通道直到消费者取消读取
	go func() {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("生产者协程退出")
				wg.Done()
				return
			default:
				ch <- 1
				fmt.Println("生产者协程生产了一个数据")
			}
		}
	}()
	// 消费者协程：消费10个数据后就取消任务
	go func() {
		defer wg.Done()
		for i := 0; i < 10; i++ {
			fmt.Println(<-ch)
			fmt.Println("消费者协程消费了一个数据")
		}
		cancel()
	}()
	wg.Wait()
	fmt.Println("Done")
}
```

这里`ctx`就是消费者协程和生产者协程通信的桥梁。通过调用`cancel`函数，关闭与`ctx`关联的`done channel`，这样`case <-ctx.Done()`就不再阻塞，可以执行关闭生产者的相关代码。父节点被取消后还会将取消消息传递给**所有**派生的子节点。

#### WithDeadline

在`WithCancel`的基础上，设置一个超时时间。被创建的子`context`会在指定的时间点自动关闭 `Done` 通道。

```go
deadline, err := time.Parse("2006-01-02 15:04:05", "2024-12-31	23:59:59")\
if err!= nil {
    fmt.Println(err)
    return
}
// WithDeadline(ctx Context, d time.Time) (Context, CancelFunc)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel()
// 2024-12-31 23:59:59 才会结束程序
for {
    select {
    case <-ctx.Done():
        fmt.Println("Context deadline exceeded")
        return
    }
}
```

#### WithTimeout

与`WithDeadline`类似，只不过接受一个持续时间而不是一个绝对时间。事实上`WithTimeout(1*time.Second)` 等同于 `WithDeadline(time.Now().Add(1*time.Second))`

这里用生产者-消费者模型，来展示channel的基本使用：

```go
package main

import (
	"fmt"
	"time"
)

// Producer ，只写
func Producer(id int, ch chan<- int) {
	fmt.Println("Producer", id, "started")
	for i := range 5 {
		fmt.Println("Producer", id, "sent", i)
		ch <- i
		time.Sleep(time.Second)
	}
	fmt.Println("Producer", id, "finished")
}

// Consumer ，只读
func Consumer(id int, ch <-chan int) {
	fmt.Println("Consumer", id, "started")
	for {
		res, ok := <-ch
		if !ok {
			break
		}
		fmt.Println("Consumer", id, "received", res)
		time.Sleep(time.Second)
	}
	fmt.Println("Consumer", id, "finished")
}

func main() {
	ch := make(chan int)

	// 启动两个生产者和三个消费者
	for i := 0; i < 2; i++ {
		Producer(i, ch)
	}
    close(ch)
	for i := 0; i < 3; i++ {
		Consumer(i, ch)
	}
}
```

## Runtime

`runtime`是Golang的核心组件之一，负责管理程序执行过程中的各种底层细节。这里只介绍一些和并发有关的接口。

- `runtime.GOMAXPROCS(n)`: 设置最多可以并发运行的 CPU 数量。
- `runtime.Goexit()`: 使当前 Goroutine 退出。
- `runtime.Gosched()`: 让出当前 Goroutine 的 CPU 时间片，允许其他 Goroutine 运行。

这些只是 `runtime` 包提供的众多功能中的一部分，对于更深入的了解和使用，请查阅官方文档和相关教程。
