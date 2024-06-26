- ```go
  type _type struct {
  	size       uintptr
  	ptrdata    uintptr // size of memory prefix holding all pointers
  	hash       uint32
  	tflag      tflag
      align      uint8
  	fieldalign uint8
  	kind       uint8
  	alg        *typeAlg
  	gcdata    *byte
  	str       nameOff
  	ptrToThis typeOff
  }
  ```
-
- typeAlg 用于 hash 函数
	- ```go
	  // src/runtime/alg.go
	  type typeAlg struct {
	  	// (ptr to object, seed) -> hash
	  	hash func(unsafe.Pointer, uintptr) uintptr
	  	// (ptr to object A, ptr to object B) -> ==?
	  	equal func(unsafe.Pointer, unsafe.Pointer) bool
	  }
	  ```
-