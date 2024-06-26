-
- ```go
  // src/runtime/hashmap.go
  func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer
  func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)
  ```
-
- 编译器实现，将两种语法对应到底层两个不同的函数。
- 在编译的[类型检查](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/)期间，`hash[key]` 以及类似的操作都会被转换成哈希的 `OINDEXMAP` 操作，[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)阶段会在 [`cmd/compile/internal/gc.walkexpr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walkexpr) 函数中将这些 `OINDEXMAP` 操作转换成如下的代码：
- 根据 key 的不同类型，编译器还会将查找、插入、删除的函数用更具体的函数替换，以优化效率：
-