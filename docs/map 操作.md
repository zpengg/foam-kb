- 创建
	- 计算哈希占用的内存是否溢出或者超出能分配的最大值；
	- 调用 [`runtime.fastrand`](https://draveness.me/golang/tree/runtime.fastrand) 获取一个随机的哈希种子；
	- 根据传入的 `hint` 计算出需要的最小需要的桶的数量；
	- 使用 [`runtime.makeBucketArray`](https://draveness.me/golang/tree/runtime.makeBucketArray) 创建用于保存桶的数组
-
- 访问
	- ```go
	  v     := hash[key] // => v     := *mapaccess1(maptype, hash, &key)
	  v, ok := hash[key] // => v, ok := mapaccess2(maptype, hash, &key)
	  
	  
	  // 实现
	  func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	  	alg := t.key.alg
	  	hash := alg.hash(key, uintptr(h.hash0))
	  	m := bucketMask(h.B)
	  	b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
	  	top := tophash(hash)
	  bucketloop:
	  	for ; b != nil; b = b.overflow(t) {
	  		for i := uintptr(0); i < bucketCnt; i++ {
	  			if b.tophash[i] != top {
	  				if b.tophash[i] == emptyRest {
	  					break bucketloop
	  				}
	  				continue
	  			}
	  			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
	  			if alg.equal(key, k) {
	  				v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
	  				return v
	  			}
	  		}
	  	}
	  	return unsafe.Pointer(&zeroVal[0])
	  }
	  ```
	- [[comma func 实现]]
-
-
- 扩容
	- 等量扩容
	- 翻倍
	- [[map 扩容条件]]
	- [[map 扩容/hashGrow]]
	- [[map 扩容/evacuate]]
-
-