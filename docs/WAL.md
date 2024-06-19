- [HBase 最全存储原理（二）_请简述hbase数据库中region服务器store和hlog的工作原理-CSDN博客](https://blog.csdn.net/ytp552200ytp/article/details/115478801)
- [Multiversion Concurrency Control in HBase | XGugeng](https://www.xgugeng.com/post/multiversion-concurrency-control-in-hbase/)
-
- WAL 使用 Hadoop 的序列化文件，将日志存储为 key - value 格式的数据集 ,  其中 value 部分为用户的 put、delete 等操作的数据，key 为 HLogKey，它由以下内容组成
- Region : 数据归属的 Region
  TableName : 数据归属的表
  SequenceNumber : 序列号
  WriteTime : 日志写入时间
-
- HLogSyncer
	- 定时刷写 ： 根据设定的时间间隔去刷写
	- 内存阈值 ： 当内存达到阈值时，刷写到文件系统
-
- HLogRoller
	- 文件拆分策略，类似 log 文件
	- 在特定的时间滚动日志，形成新的文件，防止单个文件过大
	  根据 HLog 的 SequenceNumber 对比已经持久化的序列号，删除旧的、不需要的日志
-
-
- 写写同步
	- 锁机制， [[ACID]]
- 写读同步
	- MVCC， writenumber
- ![](https://www.xgugeng.com/images/hbase-possible-order-with-locks.png)
- ![](https://www.xgugeng.com/images/hbase-write-steps-with-mvcc.png)
-
-
- 为何先写 WAL, 再写 Memstore？ #card
	- HBase提供了一个[[MVCC]]机制，来保障写数据阶段,老版本数据的**数据可见性**。
	- 写内存，避免多Region情形下带来的过多的分散**IO操作**。
	- 如果先写 WAL 失败的话，MemStore 存的数据会被回滚， 新的write num没被写入。
-
- 为什么提交WriteNumber时，会出现调整后的ReadNumber小于本次写操作所分配的WriteNumber呢？ #card
	- 类似滑动窗口
	- 这是因为并发写入时，多个线程的写入速度是随机的，可能存在WriteNumber比较大（假设值为x）
	  的写入操作比WriteNumber较小的（假设值为y）写入操作先结束了，但此时并不能将ReadNumber的值调整为x，
	- 因为此时还存在WriteNumber比x小的写入操作正在进行中，
	- ReadNumber为x即表示MemStore中所有WriteNumber小于或等于x的数据都可以被读取了，
	- 但实际上还有值没有被写入完成，可能会出现数据不一致的情况，
	- 所以如果写队列中WriteNumber比较大的写入操作如果较快的结束了，
	- 则需要进行相应的等待，直到写队列中它前面的那些写入操作完成为止。
-
- hbase 需要行锁吗？#card
	- 在之前的版本中，行级别的任何并发写入/更新都是互斥的，由一个行锁控制。但在2.0版本中，这一点行为发生了变化，多个线程可以同时更新一行数据，这里的考虑点为：
	- 如果多个线程写入同一行的不同列族，是不需要互斥的
	- 多个线程写同一行的相同列族，也不需要互斥，即使是写相同的列，也完全可以通过HBase的MVCC机制来控制数据的一致性
	- 当然，CAS操作(如checkAndPut)或increment操作，依然需要独占的行锁