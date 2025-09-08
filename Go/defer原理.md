# defer原理

## defer的底层结构

进行defer函数调用的时候会生成一个_defer结构，一个函数中可能有多次defer调用，所以会生成多个这样的_defer结构，这些_defer结构链式存储构成一个_defer链表，当前goroutine的_defer指向这个链表的头节点，

```go
type _defer struct {
   started bool   // 标志位，标识defer函数是否已经开始执行,默认为false
   heap    bool   // 标记位，标志当前defer结构是否是分配在堆上
   openDefer bool  // 标记位，标识当前defer是否以开放编码的方式实现
   sp        uintptr // 调用方的sp寄存器指针，即栈指针
   pc        uintptr // 调用方的程序计数器指针
   fn        func()  // defer注册的延迟执行的函数
   _panic    *_panic // 标识是否panic时触发，非panic触发时，为nil
   link      *_defer // defer链表
   fd   unsafe.Pointer // defer调用的相关参数
   varp uintptr        // value of varp for the stack frame
   framepc uintptr
}
```

