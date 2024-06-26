- ```go
  // src/runtime/hashmap.go/mapassign
  // 触发扩容时机
  if !h.growing() && (overLoadFactor(int64(h.count), h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
  	hashGrow(t, h)
  }
  // cond1. 大于桶数量 且 装载因子超过 6.5 
  func overLoadFactor(count int64, B uint8) bool {
  	return count >= bucketCnt && float32(count) >= loadFactor*float32((uint64(1)<<B))
  }
  // cond2. overflow buckets 太多
  func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
  	if B < 16 {
  		return noverflow >= uint16(1)<<B
  	}
  	return noverflow >= 1<<15
  }
  ```
- overLoadFactor
	- [[loadFactor]]装载因子超过阈值，源码里定义的阈值是 6.5。
- tooManyOverflowBucketsoverflow 的 bucket 数量过多：
	- 当 B 小于 15，也就是 bucket 总数 2^B 小于 2^15 时，如果 overflow 的 bucket 数量超过 2^B；
	- 当 B >= 15，也就是 bucket 总数 2^B 大于等于 2^15，如果 overflow 的 bucket 数量超过 2^15。
-
- 判断是否在扩容 growing()
	- ```go
	  func (h *hmap) growing() bool {
	  	return h.oldbuckets != nil
	  }
	  ```
-
- [[map 扩容/hashGrow]]