- [HBase技术原理 | 曹世宏的博客](https://cshihong.github.io/2018/05/17/HBase%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86/)
-
- ![HBase系统架构](https://cshihong.github.io/2018/05/17/HBase%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86/HBase%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84.png)
- HBase的实现包括三个主要的功能组件：
	- 库函数：链接到每个客户端
	- 一个Master主服务器
	- 许多个Region服务器
- 主服务器Master负责管理和维护HBase表的分区信息，维护Region服务器列表，分配Region，负载均衡
- Region服务器负责存储和维护分配给自己的Region，处理来自客户端的读写请求
	- 客户端并不是直接从Master主服务器上读取数据，而是在获得Region的存储位置信息后，直接从Region服务器上读取数据
	- 客户端并不依赖Master，而是通过Zooeeper来获得Region位置信息，大多数客户端甚至从来不和Master通信，这种设计方式使得Master负载很小
-
- 寻址
	- Zookeeper文件
		- 记录以-ROOt-表的位置信息。
	- -ROOT-表
		- 记录了.META.表的Region位置信息。-ROOT-表只能一个Region。通过-ROOT-表，就可以访问.META。表中的数据。
		- MAX 128MB
	- .META.表
		- 记录了用户数据表的Region位置（Region和Region服务器 关系），.META.表可以有多个Region，保存了HBase中所有用户数据表的Region位置信息。
		- 记录在内存中
		-
- ![HBase的三层结构](https://cshihong.github.io/2018/05/17/HBase%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86/HBase%E7%9A%84%E4%B8%89%E5%B1%82%E7%BB%93%E6%9E%84.png)
-
- ## Zookeeper为HBase提供：
- 分布式锁的服务：
  
  多个HMaster进程都会尝试着去Zookeeper中写入一个对应的节点，该结点只能被一个HMaster进程创建成功，创建成功的HMaster进程就是Active。
- 事件监听机制：
  
  主HMaster进程宕掉之后，备HMaster在监听对应的Zookeeper节点。主HMaster进程宕掉之后，该节点会被删除，其他的备HMaste就可以收到响应的消息。
- 微型数据库角色：
  
  Zookeeper中存放了Region Server的地址，此时，可以将它理解成一个微型数据库。