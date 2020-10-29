#### golang的context的主要用途在于在多个goroutine之间传递数据，管理多个goroutine的生命周期。实际的应用场景有比如，在http服务中，每个请求就对应一个goroutine，而请求之中可能又会调用别的api，而产生更多的goroutine，用context来管理这些goroutine就能比较方便在这些goroutine中传递数据和管理。


#### 实现Context，只需要实现Done, Err, Value, Deadline

有四种Context:

EmptyCtx  作为根context

ValueCtx  传递上下文数据

CancelCtx  父协程可取消子协程

TimerCtx  是一种特殊的CancelCtx，不是主动cancel 而是计时器到了之后cancel
