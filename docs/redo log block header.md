- LOG_BLOCK_HDR_NO
	- 每个redo log block都会分配一个唯一的编号。
- LOG_BLOCK_HDR_DATA_LEN
	- 代表该block已经使用了多少字节，从12开始，因为header部分是被固定占用的，512代表block被写满了。
- LOG_BLOCK_FIRST_REC_GROUP
	- 一个MTR会生成多条redo log，有时甚至会跨越多个block，
	- 该属性代表block内第一个MTR第一条redo log的起始地址。
- LOG_BLOCK_CHECKPOINT_NO
	- 记录[[check point]]序号，**日志空间中的每条日志对应一个LSN**
- | **属性** | **长度** | **说明** |
  | ---- | ---- | ---- |
  | LOG_BLOCK_HDR_NO | 4字节 | block唯一编号 |
  | LOG_BLOCK_HDR_DATA_LEN | 2字节 | block已经使用了多少字节，512代表全部写满 |
  | LOG_BLOCK_FIRST_REC_GROUP | 2字节 | MTR第一条日志的起始地址 |
  | LOG_BLOCK_CHECKPOINT_NO | 4字节 | checkpoint序号 |
-
-