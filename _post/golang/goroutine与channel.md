

## goroutine

goroutine是一种轻量级的线程，可以叫他协程，协程是一个编程语言层面的概念，目前操作系统并没有这个概念，goroutine需要与 channel配合使用

### 创建goroutine

```go
package main
 
 import (
 	"fmt"
	"time"
	"math/rand"
 )

func routine(name string, delay time.Duration) {

    t0 := time.Now()
    fmt.Println(name, " start at ", t0)
    time.Sleep(delay)

    t1 := time.Now()
    fmt.Println(name, " end at ", t1)
    fmt.Println(name, " lasted ", t1.Sub(t0))
}
 
func main() {
    //生成随机种子
    rand.Seed(time.Now().Unix())
    var name string
    for i:=0; i<3; i++{
        name = fmt.Sprintf("go_%02d", i) //生成ID
        //生成随机等待时间，从0-4秒
        go routine(name, time.Duration(rand.Intn(5)) * time.Second)
    }

    //让主进程停住，不然主进程退了，goroutine也就退了
    var input string
    fmt.Scanln(&input)
    fmt.Println("done")
}
```

### 多个goroutine的同步

```go
var wg sync.WaitGroup

func hello(i int) {
	defer wg.Done() // goroutine结束就登记-1
	fmt.Println("Hello Goroutine!", i)
}
func main() {

for i := 0; i < 10; i++ {
	wg.Add(1) // 启动一个goroutine就登记+1
	go hello(i)
}
wg.Wait() // 等待所有登记的goroutine都结束
}
```

### goroutine的并发安全性

下面一个经常出现在教科书里卖票的例子，启了5个goroutine来卖票，卖票的函数sell_tickets很简单，就是随机的sleep一下，然后对全局变量total_tickets作减一操作

```go
package main
import "fmt"
import "time"
import "math/rand"
import "runtime"

var total_tickets int32 = 10;

func sell_tickets(i int){
    for{
        if total_tickets > 0 { //如果有票就卖
            time.Sleep( time.Duration(rand.Intn(5)) * time.Millisecond)
            total_tickets-- //卖一张票
            fmt.Println("id:", i, "  ticket:", total_tickets)
        }else{
            break
        }
    }
}

func main() {
    // runtime.GOMAXPROCS(4) //go1.5版本前默认启动一核，之后的版本跟电脑的核数一致
    rand.Seed(time.Now().Unix()) //生成随机种子

    for i := 0; i < 5; i++ { //并发5个goroutine来卖票
         go sell_tickets(i)
    }
    //等待线程执行完
    var input string
    fmt.Scanln(&input)
    fmt.Println(total_tickets, "done") //退出时打印还有多少票
}
```

上面案例会存在并发问题，打印的结果可能是：

id: 0   ticket: 9

id: 0   ticket: 8

id: 4   ticket: 7

id: 1   ticket: 6

id: 3   ticket: 5

id: 0   ticket: 4

id: 3   ticket: 3

id: 2   ticket: 2

id: 0   ticket: 1

id: 3   ticket: 0

id: 1   ticket: -1

id: 4   ticket: -2

id: 2   ticket: -3

id: 0   ticket: -4
-4 done

为了解决这个问题引入sync，将上面的代码修改为：

```go
package main
import "fmt"
import "time"
import "math/rand"
import "sync"
import "runtime"

var total_tickets int32 = 10
var mutex = &sync.mutex{}  //可简写成：var mutex sync.Mutex

func sell_tickets(i int){
    for total_tickets>0 {
        mutex.Lock()
        if total_tickets > 0 {
            time.Sleep( time.Duration(rand.Intn(5)) * time.Millisecond)
            total_tickets--
            fmt.Println(i, total_tickets)
        }
        mutex.Unlock()
    }
}

func main() {
    // runtime.GOMAXPROCS(4) //go1.5版本前默认启动一核，之后的版本跟电脑的核数一致
    rand.Seed(time.Now().Unix()) //生成随机种子

    for i := 0; i < 5; i++ { //并发5个goroutine来卖票
         go sell_tickets(i)
    }
    //等待线程执行完
    var input string
    fmt.Scanln(&input)
    fmt.Println(total_tickets, "done") //退出时打印还有多少票
}
```

打印的结果可想而知

单纯地将函数并发执行是没有意义的。函数与函数间需要交换数据才能体现并发执行函数的意义。

虽然可以使用共享内存进行数据交换，但是共享内存在不同的goroutine中容易发生竞态问题。为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题，因此go引入一直新的万物模具：Channel

> Go语言的并发模型是CSP（Communicating Sequential Processes），提倡通过通信共享内存而不是通过共享内存而实现通信

## Channel

一种万物模具


### 长相
```go
var 变量 chan 元素类型

var ch1 chan int   // 声明一个传递整型的通道
var ch2 chan bool  // 声明一个传递布尔型的通道
var ch3 chan []int // 声明一个传递int切片的通道
var ch4 chan interface{} // 任意类型的通道
var ch5 chan struct_1 // struct类型的通道
```

### 创建channel
```go
make(chan 元素类型, [缓冲大小])
```

### channel操作类型

- 发送
- 接收
- 关闭

```go
发送
将一个值发送到通道中
ch <- 10 // 把10发送到ch中

接收
从一个通道中接收值
x := <- ch // 从ch中接收值并赋值给变量x
<-ch       // 从ch中接收值，忽略结果

关闭
我们通过调用内置的close函数来关闭通道。
close(ch)
```


### 产生阻塞的条件
![发生阻塞的条件](https://s2.ax1x.com/2019/10/06/ucoT91.png "发生阻塞的条件")


### channel发生死锁的条件

- 主线程阻塞
- 主线程阻塞之前没有开启goroutine或者(阻塞前开启了goroutine但是所有的goroutine执行完主线程仍然没有找到自己的“伴侣”)


### 一个简单的Channel

```go
package main
import "fmt"

func main() {
    messages := make(chan string)
	// 开启一个goroutine发送数据
    go func() { messages <- "ping" }()
	// 接收收据
    msg := <-messages
    fmt.Println(msg)
}
```

### 具有缓冲的Channel

```go
package main
import "fmt"
func main() {
	// 定一个具有两个缓冲的Channel
    messages := make(chan string, 2)
	// 之所以这样不会阻塞，因为有两个缓冲
    messages <- "buffered"
    messages <- "channel"

    fmt.Println(<-messages)
    fmt.Println(<-messages)
}
```

### Channel状态同步

```go
package main
import "fmt"
import "time"

func worker(done chan bool) {
    fmt.Print("working...")
    time.Sleep(time.Second)
    fmt.Println("done")
	// 处理完成后，向通道发送数据
    done <- true
}

func main() {
	// 创建一个通道(无论是否有缓冲)
    done := make(chan bool)
	// 开启一个go协程处理某件事情
    go worker(done)
	// <-done保证worker干完活了
    <-done
}
```

### Channel方向
```go
// pings为发送通道
func ping(pings chan<- string, msg string) {
	pings <- msg
}

// pings为接受通道，pongs为发送通道
func pong(pings <-chan string, pongs chan<- string) {
	msg := <-pings
	pongs <- msg
}

func main() {
	pings := make(chan string, 1)
	pongs := make(chan string, 1)
	ping(pings, "passed message")
	pong(pings, pongs)
	fmt.Println(<-pongs)
}
```

### Channel选择器

```go
func main() {
    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        time.Sleep(time.Second * 100)
        c1 <- "one"
    }()

    go func() {
        time.Sleep(time.Second * 200)
        c2 <- "two"
    }()

    for i := 0; i < 2; i++ {
        select
			case msg1 := <-c1:
				fmt.Println("received", msg1)
			case msg2 := <-c2:
				fmt.Println("received", msg2)
        }
    }
}
每一次select 必须有结果，如果没有结果，则select 会等待，事实上select 就是将通道发送或者接收统一管理起来
```

### 超时处理
```go
func main() {
    c1 := make(chan string, 1)
    go func() {
		// 两秒后发送数据到通道
        time.Sleep(time.Second * 2)
        c1 <- "result 1"
    }()

    select {
		// 两个case 哪个先满足就走哪里(无论是发送还是接收)，如果两个都满足，则随机选择
		case res := <-c1:
			fmt.Println(res)
		// 由于是两秒，因此正常情况下下面case先满足
		case <-time.After(time.Second * 1):
			fmt.Println("timeout 1")
    }

    c2 := make(chan string, 1)
    go func() {
        time.Sleep(time.Second * 2)
        c2 <- "result 2"
    }()
    select {
		// 两个case 哪个先满足就走哪里(无论是发送还是接收)，如果两个都满足，则随机选择
		// 由于是两秒，因此正常情况下下面case先满非阻塞通道操作足
		case res := <-c2:
			fmt.Println(res)
		case <-time.After(time.Second * 3):
			fmt.Println("timeout 2")
    }
}
```

### 非阻塞Channel操作

```go
func main() {
    messages := make(chan string)
    signals := make(chan bool)
	// 这个select 中的default先被满足
    select {
		case msg := <-messages:
			fmt.Println("received message", msg)
		default:
			fmt.Println("no message received")
    }

    msg := "hi"
	// 这个select 中的default先被满足
    select {
		// 发送数据，虽然msg是有值的，但是仍然不能被执行，因为没有接收者
		case messages <- msg:
			fmt.Println("sent message", msg)
		default:
			fmt.Println("no message sent")
    }
	// 毫无疑问，下面还是default先执行
    select {
		case msg := <-messages:
			fmt.Println("received message", msg)
		case sig := <-signals:
			fmt.Println("received signal", sig)
		default:
			fmt.Println("no activity")
    }
}
```

### Channel的关闭

```go
func main() {
    jobs := make(chan int, 5)
    done := make(chan bool)

    go func() {
        for {
            j, more := <-jobs
            if more {
                fmt.Println("received job", j)
			// close后会走到else
            } else {
                fmt.Println("received all jobs")
                done <- true
                return
            }
        }
    }()

    for j := 1; j <= 3; j++ {
        jobs <- j
        fmt.Println("sent job", j)
    }
	// 关闭 一个通道意味着不能再向这个通道发送值了。这个特性可以用来给这个通道的接收方传达工作已经完成的信息
    close(jobs)
    fmt.Println("sent all jobs")
    <-done
}
```
