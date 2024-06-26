- build constraints
-
- ```go
  //go:build fuzz
  // +build fuzz
  
  go test -tags fuzz ./...
  
  ```
-
- go:build 引入于Go 1.17
	- 支持bool 表达式 `//go:build linux && amd64`
- + build 是以前的写法