- panic 这个关键词本质上只是一个 `runtime.gopanic` 调用
- ```nasm
  TEXT main.main(SB) /Users/changkun/dev/go-under-the-hood/demo/7-lang/panic/main.go
    (...)
    main.go:7		0x104e05b		0f57c0			XORPS X0, X0				
    main.go:7		0x104e05e		0f110424		MOVUPS X0, 0(SP)			
    main.go:7		0x104e062		e8d935fdff		CALL runtime.gopanic(SB)		
    main.go:7		0x104e067		0f0b			UD2					
    (...)
  ```
-
-
-
- [[gopanic()]] 实现
	- [[getg()]] 方法获取 g， 判断无法恢复的情况
		- 系统栈上的 panic 无法恢复
		- 用户栈上特定时机不行
			- mallocing
			- preemptoff 禁止抢占时
			- g 锁在 m 上时
-
- _panic 保存了一个活跃的 panic
	- _panic 的值必须位于栈上
		- ```go
		  // _panic 保存了一个活跃的 panic
		  //
		  // 这个标记了 go:notinheap 因为 _panic 的值必须位于栈上
		  //
		  // argp 和 link 字段为栈指针，但在栈增长时不需要特殊处理：因为他们是指针类型且
		  // _panic 值只位于栈上，正常的栈指针调整会处理他们。
		  //
		  //go:notinheap
		  type _panic struct {
		  	argp      unsafe.Pointer // panic 期间 defer 调用参数的指针; 无法移动 - liblink 已知
		  	arg       interface{}    // panic 的参数
		  	link      *_panic        // link 链接到更早的 panic
		  	recovered bool           // 表明 panic 是否结束
		  	aborted   bool           // 表明 panic 是否忽略
		  }
		  ```
	- panic 保存了对应的消息，并指向了保存在 goroutine 链表中先前的 panic 链表
		- ```go
		  	var p _panic
		  	p.arg = e
		  	p.link = gp._panic
		  	gp._panic = (*_panic)(noescape(unsafe.Pointer(&p)))
		  
		  	atomic.Xadd(&runningPanicDefers, 1)
		  ```
	- 首先，当 panic 发生时，如果错误是可恢复的错误
	- 那么 会逐一遍历该 goroutine 对应 [[defer]] 链表中的 defer 函数链表，
	- 直到 defer 遍历完毕、 或者再次进入调度循环（recover 的 mcall 调用） 后才会停止。
-
- [[panic & recover]]