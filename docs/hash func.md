- ![mapacess](https://golang.design/go-questions/map/assets/2.png){:height 1001, :width 780}
- hash 算法
	- 会检测 cpu 是否支持 aes，如果支持，则使用 aes hash，否则使用 memhash。
	  id:: 6677db1b-9ae9-4c94-a319-764b5e4cc68f
	- hash 算法在编译期确定
-
- hash 低 B 位
	- 决定桶
- tophash 计算桶中 slot
	- 高8位
-
-
- 解决 hash 冲突的方法#card
	- [[拉链法]]
	- [[开放寻址法]]
- 为什么有key 还需要单独存 tophash?#card
	- 碰撞概率低，比较hash已经可以排除大部分了
	- 虽然逻辑上仅占用1B，64位机器下因为[[内存对齐]]会扩充为8B
	-
- typeAlg
	- 包含hash/equal 两个方法
	- ```go
	  // src/runtime/alg.go
	  type typeAlg struct {
	  	// (ptr to object, seed) -> hash
	  	hash func(unsafe.Pointer, uintptr) uintptr
	  	// (ptr to object A, ptr to object B) -> ==?
	  	equal func(unsafe.Pointer, unsafe.Pointer) bool
	  }
	  ```