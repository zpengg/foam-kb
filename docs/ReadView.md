- 当前读
	- **锁定读【Locking Read】，读取的是记录数据的最新版本，并且需要先获取对应记录的锁**
	- ```sql
	  SELECT * FROM student LOCK IN SHARE MODE;  # 共享锁
	  SELECT * FROM student FOR UPDATE; # 排他锁
	  INSERT INTO student values ...  # 排他锁
	  DELETE FROM student WHERE ...  # 排他锁
	  UPDATE student SET ...  # 排他锁
	  ```
- 快照读
	- **普通读，读取的是记录数据的可见版本，不加锁，不加锁的普通select语句都是快照读**
	- **快照读的执行方式是生成 [[ReadView]]，直接利用 [[MVCC]] 机制来进行读取**
	- ```sql
	  select * from table;
	  ```
-
- What's ReadView
	- data struct
		- m_ids 未提交的事务，不允许读
		- min_trx_id 就是快要执行完的事务 id
		- max_trx_id 就是下一个新的事务的 id
		- creator_trx_id 就是创建这个 ReadView 是哪个事务
-
- When？
  id:: 667427a6-5d55-4251-88df-e7d9e4869a2a
	- `READ COMMITTD`
		- 在`每一次进行普通SELECT操作前都会生成一个ReadView`
		- 读 undolog 顺序找到第一条
			- 已提交 trx_id < min_trx_id， 代表已提交
			- 自己事务更改的 trx_id = creator_trx_id
			- ![image.png](https://cdn.nlark.com/yuque/0/2022/png/22070997/1653554265834-8933dc8c-a62d-4cb0-9bf3-7bbd1e045946.png#clientId=u21246102-96ae-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=566&id=u81e9f2eb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=707&originWidth=1810&originalType=binary&ratio=1&rotation=0&showTitle=false&size=212731&status=done&style=none&taskId=u0e9c7aca-cbd4-445d-a65e-caf39f81510&title=&width=1448)
	- `REPEATABLE READ`
		- 只在`第一次进行普通SELECT操作前生成一个ReadView`，之后的查询操作都`重复使用这个ReadView`就好了。
		- 但是当多次快照读中间存在当前读（特指当前事务），ReadView会重新生成，导致产生幻读
			- ![image.png](https://cdn.nlark.com/yuque/0/2022/png/22070997/1653556503844-0dfc46df-5abe-465d-8f14-89bdf4166025.png#clientId=u21246102-96ae-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=502&id=u6f1fa510&margin=%5Bobject%20Object%5D&name=image.png&originHeight=627&originWidth=1717&originalType=binary&ratio=1&rotation=0&showTitle=false&size=169599&status=done&style=none&taskId=u4b5a39bb-3123-4e6b-9856-6edd90a0aad&title=&width=1373.6){:height 237, :width 626}
			- ![image.png](https://cdn.nlark.com/yuque/0/2022/png/22070997/1653556832381-8a03a62a-4403-40fd-81fa-051f10085255.png#clientId=u21246102-96ae-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=509&id=u2cc91172&margin=%5Bobject%20Object%5D&name=image.png&originHeight=636&originWidth=1496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=102269&status=done&style=none&taskId=ud937a1a0-442e-402c-a5eb-50cc5ecf454&title=&width=1196.8)