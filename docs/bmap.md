- [一个系列彻底搞懂map三:go语言map剖析 | 辜飞俊的博客](https://gufeijun.com/post/map/3/)
- ```go
  type bmap struct {
  	tophash [bucketCnt]uint8
  }
  
  // 编译期间会动态变成
  type bmap struct {
      topbits  [8]uint8
      keys     [8]keytype
      values   [8]valuetype
      pad      uintptr
      overflow uintptr        //只保留下一个同义词桶的地址，而不引用计数。
    //overflow *bmap			//overflow就是链表节点的next指针，指向下一个同义词桶
  
  }
  ```
-
- 特点
	- bmap overflow 连续存
		- ![](https://fastly.jsdelivr.net/gh/gufeijun/comments@master/go_map-overflow_buckets.png)
	- key value 分开连续存， 可以节约padding
	- tophash
	- [[overflow]]
		- inline& uintptr .
			- key和value类型的size都小于128B且不是指针(称为inline)的情况下，减少指针引用对象的次数
			- 溢出桶记录在hmap `mapextra`对象的`overflow`成员
		- 普通 bmap 引用
-
- 设计优点
	- 降低了gc回收负担。
		- 每8个键值才需要一个next指针，减少内存消耗的同时，也减少了bucket对象的数，
-
- 结构图
	- ![bmap struct](https://golang.design/go-questions/map/assets/1.png)