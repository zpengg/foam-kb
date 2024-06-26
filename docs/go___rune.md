- [[Unicode]]
- [[go/string]] 的底层表示是 byte (8 bit) 序列，而非 rune (32 bit) 序列
- ```go
  fmt.Println(len("Go语言")) // 8
  fmt.Println(len([]rune("Go语言"))) // 4
  ```