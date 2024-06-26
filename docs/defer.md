-
- 后进先出(Last In First Out，LIFO)的原则，
	- 最后声明的 defer 语句，最先得到执行。
- return 语句之后执行，但在函数退出之前
	- defer 可以修改返回值。
- ```go
  defer func() {
  		fmt.Println("defer1")
  	}()
  ```