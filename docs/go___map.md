- Go 语言采用的是**哈希查找表**，并且使用[[链表解决哈希冲突]]。
-
- 内存模型
	- hmap
	- ![hashmap bmap](https://golang.design/go-questions/map/assets/0.png)
	- ```go
	  // A header for a Go map.
	  type hmap struct {
	      // 元素个数，调用 len(map) 时，直接返回此值
	  	count     int
	  	flags     uint8
	  	// buckets 的对数 log_2
	  	B         uint8
	  	// overflow 的 bucket 近似数
	  	noverflow uint16
	  	// 计算 key 的哈希的时候会传入哈希函数
	  	hash0     uint32
	      // 指向 buckets 数组，大小为 2^B
	      // 如果元素个数为0，就为 nil
	  	buckets    unsafe.Pointer
	  	// 等量扩容的时候，buckets 长度和 oldbuckets 相等
	  	// 双倍扩容的时候，buckets 长度会是 oldbuckets 的两倍
	  	oldbuckets unsafe.Pointer
	  	// 指示扩容进度，小于此地址的 buckets 迁移完成
	  	nevacuate  uintptr
	  	extra *mapextra // optional fields
	  }
	  ```
		- [[hash func]]
		- 桶数量 二进制表示， B 个 bit 位, buckets 的对数。
			- 如果 B = 5，那么桶的数量，也就是 buckets 数组的长度是 2^5 = 32。
	- buckets  会指向一个 [[bmap]] 桶，桶里面会最多装 8 个 key
	- flags
		- 表示map现在处于的状态，可取iterator、oldIterator、hashWriting、sameSizeGrow。
	- oldbuckets表示旧桶，扩容用
		- go语言的map，在旧桶迁移到新桶[[扩容]]的过程中，并不是一蹴而就的，而是在操作中逐次进行，因此有必要保存旧桶。
	- [[overflow]]
-
- [[map 操作]]
-
- [[go map和java map比较]]
  id:: 66791b8f-91e5-44cd-904b-53e8c1add6f8
	-
-