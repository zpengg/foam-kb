# thread running
Threads_running 表明 MySQL 工作的强度。threads_running 代表同时执行 DB 任务的线程数，
- ## 经验值
  | Threads_running | MySQL |
  | ---- | ---- | ---- |
  | 0 - 10 | Normal：几乎所有硬件都没问题 |
  | 10 - 30 | Busy：大多数硬件通常都可以，因为服务器多核 |
  | 30 - 50 | High：很少有工作负载需要运行这么多线程。它可以短期爆发（<5min），但如果持续时间较长，则响应时间很可能很慢 |
  | 50 - 100 | Overloaded：某些硬件可以处理此问题，但是不能期望在此范围内成功运行。对于我们的本地部署硬件而言，此范围内的瞬时突发（<5s）通常是可以的。 |
  | > 100 | Failing：在极少数情况下，MySQL可以运行大于100个线程，但在此范围内可能会失败 |
  
  
  建议值 < 50
  日常值 master< 10, slave <2
-
- ## db 观测
  ![db 毛刺](assets/image-5.png)
  ![流量大的库](assets/image-1.png)
  ![流量更大的库](assets/image-3.png)
  ![流量](assets/image-2.png)
  ![流量 2](assets/image-4.png)
-
- ## 实时的观测机制
  基于 MySQL 的 information_schema 的 PROCESSLIST 数据表，从 DB 层一直往 URL 上推导。
  PROCESSLIST 是记录了 MySQL 实时的进程处理情况，等同于 Linux 的 top。
  
  优点：不紧急的现场可以看一下
  缺点：未止损，放任可能产生其他问题
-
- ## 离线分析
- ### 实战
  熔断降级，不再产生新的
  dba kill 慢查询清理存量
  重启 pod 清理重试队列
- ### 事后离线分析
  binlog-analyze