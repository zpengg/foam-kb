- `getg()` 判断在系统栈上还是在用户栈上
  title:: getg()
  	 如果执行在系统或信号栈时，getg() 会返回当前 m 的 [[g0]] 或 [[gsignal]]
  	因此可以通过 gp.m.curg == gp 来判断所在栈
-