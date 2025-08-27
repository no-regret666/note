# Context

一个用于`goroutine`之间传递信息的并发安全的包，context可以翻译为上下文，用于上层与下层goroutine的取消控制以及数据共享，也是go语言中goroutine之间通信的一种方式。



## context的底层实现

### context接口

```go
type Context interface {
   Deadline() (deadline time.Time, ok bool) 
   Done() <-chan struct{}
   Err() error
   Value(key interface{}) interface{}
}
```

- Deadline：返回context被取消的时间，即截止时间
- Done：返回一个Channel，当Context被取消或者到达截止时间，这个Channel就会被关闭，表示context结束，多次调用该方法返回的channel是同一个
- Err：返回context结束的原因
- Value：获取键对应的值，类似于map的get方法



### canceler接口

```go
type canceler interface {
   cancel(removeFromParent bool, err error)  //创建cancel接口实例的goroutine调用cancel方法通知被创建的goroutine退出
   Done() <-chan struct{}  // 返回一个channel，后续被创建的goroutine通过监听这个channel的信号来完成退出
}
```



## context实现

### emptyCtx

不具备任何功能，一般作为根context来派生出有实际用处的context。



### cancelCtx

```go
type cancelCtx struct {
   Context                         // 组合了一个Context ，所以cancelCtx 一定是context接口的一个实现
   mu       sync.Mutex             // 互斥锁，用于保护以下三个字段
  // value是一个chan struct{}类型，原子操作做锁优化
   done     atomic.Value          
   // key是一个取消接口的实现，map其实存储的是当前canceler接口的子节点，当前context被取消时，会遍历子节点发送取消信号
   children map[canceler]struct{}  
   err      error                  // context被取消的原因
}
```

用法：

```go
ctx,cancel := context.WithCancel(context.Background())
```

WithCancel源码：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
   if parent == nil { // 传入的父context不能为空，否则报panic
      panic("cannot create context from nil parent")
   }
   c := newCancelCtx(parent)   // 这里就会创建一个cancelCtx
   propagateCancel(parent, &c)   // 这里主要是关联父context ctx和子congtxt c的逻辑
   return &c, func() { c.cancel(true, Canceled) }  // 具体的取消函数cancel的实现
}
```

propagateCancel函数集中情形：

1. 父context的通信管道done为空或者已经被取消，就不用关联了，直接取消当前子context即可''
2. 父context可以被取消，但是还未被取消，并且父context可以提取出标准的cancelCtx结构，则创建父context的children map，将当前子context加入到这个map中
3. 父context可以被取消，但是还未被取消，父context不能提取出标准的cancelCtx结构，新起一个goroutine监控父子context的通信管道有没有取消信号



### timerCtx

在cancelCtx的基础上，还可以设置一个截止时间deadline，在deadline到来时，自动取消context。

```go
type timerCtx struct {
   cancelCtx
   timer *time.Timer // Under cancelCtx.mu.
   deadline time.Time
}
```

用法：

```go
 ctx, cancel := context.WithDeadline(context.Background(),time.Now().Add(4*time.Second))  // 截止时间当前时间4s后
 ctx, cancel := context.WithTimeout(context.Background(), 4*time.Second)   // 超时时间为4s后
```

父context未取消的情况下，在创建timerCtx的时候有两种情况：

1. 设置的截止时间晚于父context的截止时间，则不会创建timerCtx，会直接创建一个可取消的context，因为父context的截止时间更早，会先被取消，父context被取消的时候会级联取消这个子context
2. 设置的截止时间早于父context的截止时间，会创建一个正常的timerCtx



### valueCtx

用于数据共享。作用类似于一个map，不过数据的存储和读取是在两个context，用于goroutine之间的数据传递。

valueCtx实现了String()方法和Value方法

```go
func (c *valueCtx) Value(key interface{}) interface{} {
   if c.key == key {
      return c.val
   }
   return c.Context.Value(key)
}
```

向上递归的查找key所对应的value，如果找到则直接返回 value，否则查找该context的父context，一直顺着 context 向上，最终找到根节点（一般是 emptyCtx），直接返回一个 nil。



## Context有什么用

1. 用于并发控制，控制协程的优雅退出
2. 上下文的信息传递