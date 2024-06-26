- ```go
  1// 预先声明的函数 panic 的实现
  func gopanic(e interface{}) {
  	gp := getg()
  
  	// 判断在系统栈上还是在用户栈上
  	// 如果执行在系统或信号栈时，getg() 会返回当前 m 的 g0 或 gsignal
  	// 因此可以通过 gp.m.curg == gp 来判断所在栈
  	// 系统栈上的 panic 无法恢复
  	if gp.m.curg != gp {
  		print("panic: ") // 打印
  		printany(e)      // 打印
  		print("\n")      // 继续打印，下同
  		throw("panic on system stack")
  	}
  
  	// 如果正在进行 malloc 时发生 panic 也无法恢复
  	if gp.m.mallocing != 0 {
  		print("panic: ")
  		printany(e)
  		print("\n")
  		throw("panic during malloc")
  	}
  
  	// 在禁止抢占时发生 panic 也无法恢复
  	if gp.m.preemptoff != "" {
  		print("panic: ")
  		printany(e)
  		print("\n")
  		print("preempt off reason: ")
  		print(gp.m.preemptoff)
  		print("\n")
  		throw("panic during preemptoff")
  	}
  
  	// 在 g 锁在 m 上时发生 panic 也无法恢复
  	if gp.m.locks != 0 {
  		print("panic: ")
  		printany(e)
  		print("\n")
  		throw("panic holding locks")
  	}
  	...
  }
  ```
-