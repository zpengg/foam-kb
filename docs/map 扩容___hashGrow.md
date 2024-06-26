- [[draws/2024-06-24-15-38-37.excalidraw]]
- `hashGrow()` 函数实际上并没有真正地“搬迁”，它只是分配好了新的 buckets，并将老的 buckets 挂到了 oldbuckets 字段上。
- ```go
  func hashGrow(t *maptype, h *hmap) {
  	// B+1 相当于是原来 2 倍的空间
  	bigger := uint8(1)
  
  	// 对应条件 2
  	if !overLoadFactor(int64(h.count), h.B) {
  		// 进行等量的内存扩容，所以 B 不变
  		bigger = 0
  		h.flags |= sameSizeGrow
  	}
  	// 将老 buckets 挂到 buckets 上
  	oldbuckets := h.buckets
  	// 申请新的 buckets 空间
  	newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger)
  
  	flags := h.flags &^ (iterator | oldIterator)
  	if h.flags&iterator != 0 {
  		flags |= oldIterator
  	}
  	// 提交 grow 的动作
  	h.B += bigger
  	h.flags = flags
  	h.oldbuckets = oldbuckets
  	h.buckets = newbuckets
  	// 搬迁进度为 0
  	h.nevacuate = 0
  	// overflow buckets 数为 0
  	h.noverflow = 0
  
  	// ……
  }
  ```
-
- ```
  func (h *hmap) growing() bool {
  	return h.oldbuckets != nil
  }
  ```