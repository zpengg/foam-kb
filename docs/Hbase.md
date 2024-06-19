link:: [HBase：一个可扩展的、高性能分布式数据库](http://www.uml.org.cn/bigdata/2018051551.asp)

-
- HBase是一个高可靠、高性能、面向列、可伸缩的分布式数据库，是谷歌BigTable的开源实现，主要用来存储非结构化和半结构化的松散数据。
- HBase的目标是处理非常庞大的表，可以通过水平扩展的方式，利用廉价计算机集群处理由超过10亿行数据和数百万列元素组成的数据表。
-
- HBase可以理解为一个**分布式的KV存储引擎**
-
- 水平扩容
	- HBase把一起访问的数据也存储在一起，实现了高扩展。主要的设计思想是按照主键(row key)分片数据。针对数据做水平分片（horizontal shard）， 根据主键的范围(row key range)分片。 不同主键范围（row key range）下的数据被分配到不同的服务器中。每台服务器的数据是全体数据的子集。HBase实际上是BigTable存储系统的一个开源实现，Bigtable是由Google开发的分布式存储系统，用来管理可扩展的，大容量的结构化数据。
-
- [[数据库事务隔离级别]]
	- HBase 不支持跨行事务，目前只支持单行级别的 read uncommitted 和 read committed 隔离级别。
-
- [[CAP]]
	- HBase 是一个**强一致性数据库**，不是“最终一致性”数据库。 CP
	  collapsed:: true
		- 每个值只出现在一个 [[Region]]
		- 同一时间一个 Region 只分配给一个 RS
		- 行内的 mutation 操作都是原子的
	- HBase 降低可用性提高了一致性。
	  collapsed:: true
		- 当某台 RS fail 的时候，它管理的 Region failover 到其他 RS 时，
		  需要根据 WAL（Write-Ahead Logging）来 redo (redolog，有一种日志文件叫做重做日志文件)，
		  这时候进行 redo 的 Region 应该是不可用的，所以 HBase 降低了可用性，提高了一致性。
-
- 事务 [[ACID]]
	- 传统的关系型数据库一般都提供了跨越所有数据的 ACID 特性；
	- 为了性能考虑，Hbase 只提供了**基于单行的ACID**。
-
-
- [[Hbase 数据模型]]
- [[Hbase/部署结构]]
- [[Hbase/存储引擎]]
- [[Hbase/数据一致性]]
- [[Hbase/使用建议]]
- [[Hbase/资源隔离]]
-