# `GMP`模型

**G**oroutine + **M**achine + **P**rocessor

1. G（Goroutine）
   - Goroutine是Go语言中用于并发执行的轻量级线程，每个Goroutine都有自己的栈和上下文信息。
   - Goroutine相对于操作系统的线程更加轻量级，可以在同一时间内运行成千上万的Goroutine。
   - G需要绑定到M上才能运行。

2. P（Processor）
   - P 是处理Goroutine的调度器的上下文，每个P包含一个本地运行队列（Local Run Queue），用于存储需要运行的Goroutine。
   - P的数量由GOMAXPROCS设置决定，它决定了并行执行的最大线程数。
   - P不仅管理Goroutine，还负责与M协作，将Goroutine分配给M执行。
3. M（Machine）
   - M 代表操作系统的线程，负责执行Goroutine。一个M一次只能执行一个Goroutine。
   - M是实际执行代码的工作单元，M与P绑定后才能执行Goroutine。
   - M可以通过调度器从全局运行队列中拉取新的Goroutine，也可以与其他M协作完成工作。
   - M的工作很简单，就是在g0（一个特殊的goroutine，负责调度）和普通的G之间反复横跳：执行g0时，它在找任务；执行普通G时，它在处理任务。



Go的容器有两种队列：

1. P的本地队列（LRQ - Local Run Queue）：这是每个P私有的G队列。
2. 全局队列（GRQ - Global Run Queue）：这是一个全局共享的G队列。



G的存放与获取逻辑：

- 放G（put g）：当你在一个goroutine里通过 go func(){...} 创建一个新的goroutine时，它会优先被放到当前P的LRQ里。如果LRQ满了，没办法，只能加个全局锁，把它扔到GRQ里去。这遵循的是"就近原则"。

- 取G（get g）：当M上的g0开始找活干时，它会遵循一个"负载均衡"的策略，按以下顺序来寻找G：

  1. 先从当前P的LRQ里找（无锁，速度最快）。
  2. 如果LRQ没有，就去全局GRQ里看看（需要加锁）。
  3. 如果GRQ也没有，就去网络轮询器（netpoll）里找找有没有因为IO操作而就绪的G。
  4. 如果还是没有，就只能去"偷"了，从别的P的LRQ里偷一半过来（work-stealing，无锁）。

  **!!!** 为了防止GRQ里的G被饿死，调度器规定，每进行61次调度循环，就必须强制下一次去GRP取。

​	阻塞与调度：当M执行的Goroutine阻塞（例如I/O操作）时，M会释放当前的P并等待P重新分配任务，从而避免资源浪费。

