- ```go
  // runtime/slice.go
  type slice struct {
  	array unsafe.Pointer // 元素指针
  	len   int // 长度 
  	cap   int // 容量
  }
  ```
-
- slice
- 结构体，是值类型
-
- [Go Slice Tricks Cheat Sheet](https://ueokande.github.io/go-slice-tricks/)
- [[slice 扩容机制]]
	-
- 相等
	- go 语言中可以使用反射 `reflect.DeepEqual(a, b)` 判断 a、b 两个切片是否相等
	- 更好的方法是 遍历
-