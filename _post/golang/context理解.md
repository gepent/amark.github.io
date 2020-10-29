#### golang的context的主要用途在于在多个goroutine之间传递数据，管理多个goroutine的生命周期。实际的应用场景有比如，在http服务中，每个请求就对应一个goroutine，而请求之中可能又会调用别的api，而产生更多的goroutine，用context来管理这些goroutine就能比较方便在这些goroutine中传递数据和管理。


#### 实现Context，只需要实现Done, Err, Value, Deadline

有四种Context:

EmptyCtx  作为根context

ValueCtx  传递上下文数据

CancelCtx  父协程可取消子协程

TimerCtx  是一种特殊的CancelCtx，不是主动cancel 而是计时器到了之后cancel



有时候一个功能可能涉及多个goroutine的调用，当该功能中途需要取消时，需要通知其它goroutine，这个时候Context就可以派上用场了，Context主要用来处理退出通知以及上下文数据传递问题。

Context之间被设计为父子关系，要创建Context，首先要创建根节点，通常是context.Background()，该Context不能被取消，没有值，也没有过期时间。

有了根节点后，接下来就是创建子节点，子节点还可以继续创建子节点，有四种方式创建子节点：WithCancel/WithDeadline/WithTimeout/WithValue，前三种方法都会返回Context和CancelFunc，父节点通过调用CancelFunc通知子节点取消，子节点通过Context提供的Done方法检测是否被取消。WithValue将一对key-value关联到Context，通过Context提供的Value方法获取指定key对应的value。

WithDeadline/WithTimeout创建的Context在超时后会自动调用CancelFunc，如果调用没有超时，需要主动调用CancelFunc以避免不必要的资源浪费，所以通常在在调用WithDeadline/WithTimeout之后会使用defer cancel()确保资源尽快释放。

```golang
package main

import (
    "context"
    "fmt"
    "time"
)


func say(ctx context.Context){
    for{
        select {
        case <-ctx.Done():
            return
        default:
            fmt.Println(ctx.Value("test"))
            time.Sleep(time.Second)
        }
    }

}


func main(){
    //5秒后调用cancelFunc取消say
    ctx,cancelFunc:=context.WithCancel(context.Background())
    ctx=context.WithValue(ctx,"test","hello")
    go say(ctx)
    time.Sleep(time.Second*5)
    cancelFunc()



    //2秒后调用cancelFunc取消say
    ctx,_=context.WithTimeout(context.Background(),time.Second*2)
    ctx=context.WithValue(ctx,"test","world")
    go say(ctx)

    time.Sleep(time.Second*5)
}
```
