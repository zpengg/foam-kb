- ```go
  package main
  func main() {
      defer func() {
      	recover()
      }()
      panic(nil)
  }
  ```
-
- [[go/panic]]
- [[go/recover]]
- 他们的实现跟调度器和 [[go/defer]] 关键字也紧密相关
-