- 使用前4个 [[redo log block]] 来存储log file的头信息和checkpoint相关的信息。也就是说，redo log file是从第2048个字节开始写入redo log block的。
-
- 第一个block存储log file头信息，格式如下：
	- | **属性** | **长度** | **说明** |
	  | ---- | ---- | ---- |
	  | LOG_HEADER_FORMAT | 4字节 | redo log版本 |
	  | LOG_HEADER_PAD1 | 4字节 | 填充字节 |
	  | LOG_HEADER_START_LSN | 8字节 | 2048字节处对应的lsn值 |
	  | LOG_HEADER_CREATOR | 32字节 | log file创建者 |
	  | LOG_BLOCK_CHECKSUM | 4字节 | 校验和 |
- 第二和四个block分别存储checkpoint1和checkpoint2
	- | **属性** | **长度** | **说明** |
	  | ---- | ---- | ---- |
	  | LOG_CHECKPOINT_NO | 8字节 | checkpoint编号 |
	  | LOG_CHECKPOINT_LSN | 8字节 | 最后一次checkpoint对应的lsn值 |
	  | LOG_CHECKPOINT_OFFSET | 8字节 | 最后一次checkpoint对应的lsn值在文件里的偏移量 |
	  | LOG_CHECKPOINT_LOG_BUF_SIZE | 8字节 | 执行checkpoint时log buffer的大小 |
	  | LOG_BLOCK_CHECKSUM | 4字节 | 校验和 |
- LSN, log sequence number
	- redo log写入的日志总量
	-
-