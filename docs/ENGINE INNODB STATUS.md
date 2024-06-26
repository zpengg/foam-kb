- 命令`SHOW ENGINE INNODB STATUS`可以查看InnoDB引擎的状态信息，
- 这里面就包含各种[[LSN]] 的值。
- ``` sql
- Log sequence number 7853861756693 # redo log写入的日志总量
  Log flushed up to   7853861756453  # 写入磁盘的redo log日志总量
  Pages flushed up to 7853579810002 # Buffer Pool里最早被修改页面 oldest_modification
  Last checkpoint at  7853579810002 # checkpoint_lsn
- ```
- ((LSN))
- [[check point]]
-