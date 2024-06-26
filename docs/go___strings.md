- Go 语言中，字符串是只读的
-
- ## 拼接字符串 `strings.Builder`
- 每次修改操作都会创建一个新的字符串。
- 如果需要拼接多次，应使用 `strings.Builder`，最小化内存拷贝次数。
- ```go
  var str strings.Builder
  for i := 0; i < 1000; i++ {
      str.WriteString("a")
  }
  fmt.Println(str.String())
  ```
-
- 格式化打印
	- %-4d "42  " right aligned/ 4characters/ padding with space
	- %4d  "  42" left aligned/ 4characters/ padding with space