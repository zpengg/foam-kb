- 默认 REQUIRED
-
-
-
- db.Transaction()
	- db.Transaction()方法接受一个函数作为参数，这个函数也接受一个*gorm.DB类型的参数，表示当前的事务对象。这个函数需要返回一个error类型的值，表示事务是否成功或失败。
	- db.Transaction()方法在内部会调用db.Begin()方法来开启一个新的事务，并将事务对象作为参数传递给函数。
	- 函数在内部可以使用事务对象来执行一些数据库操作，比如增删改查等。如果所有的操作都成功，则返回nil表示事务提交；如果有任何操作失败，则返回错误表示事务回滚。
	- db.Transaction()方法在接收到函数的返回值后，会根据返回值的类型来决定是调用tx.Commit()方法来提交事务，还是调用tx.Rollback()方法来回滚事务。
	- db.Transaction()方法最终也会返回一个error类型的值，表示整个事务模板的执行结果
- ```go
  // Transaction 方法源码
  func (db *DB) Transaction(fc func(tx *DB) error, opts ...*sql.TxOptions) (err error) {
  	panicked := true // 定义一个标志变量，表示是否发生了panic
  
  	if committer, ok := db.Statement.ConnPool.(TxCommitter); ok && committer != nil {
  		// 如果当前的连接池是一个事务提交器，并且不为空，那么说明已经存在一个事务
  		// 这时候就需要创建一个嵌套事务
  		if !db.DisableNestedTransaction {
  			// 如果没有禁用嵌套事务，那么就创建一个保存点，用于在后续回滚到某个状态
  			// 保存点的名字是根据回调函数的内存地址生成的，保证唯一性
  			err = db.SavePoint(fmt.Sprintf("sp%p", fc)).Error
  			if err != nil {
  				return // 如果创建保存点失败，那么直接返回错误
  			}
  			defer func() {
  				// 使用延迟函数，在函数结束之前执行以下操作
  				// 如果发生了panic，或者有错误返回，那么就回滚到保存点处
  				if panicked || err != nil {
  					db.RollbackTo(fmt.Sprintf("sp%p", fc))
  				}
  			}()
  		}
  		// 调用回调函数，并传入一个新的*gorm.DB实例作为参数
  		// 这个实例是基于当前实例复制的，但是不会复用之前的Statement
  		err = fc(db.Session(&Session{NewDB: db.clone == 1}))
  	} else {
  		// 如果当前的连接池不是一个事务提交器，或者为空，那么说明不存在一个事务
  		// 这时候就需要开启一个新的事务
  		tx := db.Begin(opts...) // 调用Begin方法，并传入可选的事务选项，开启一个新的*gorm.DB实例作为事务句柄
  		if tx.Error != nil {
  			return tx.Error // 如果开启事务失败，那么直接返回错误
  		}
  
  		defer func() {
  			// 使用延迟函数，在函数结束之前执行以下操作
  			// 如果发生了panic，或者有错误返回，那么就回滚整个事务
  			if panicked || err != nil {
  				tx.Rollback()
  			}
  		}()
  		 // 调用回调函数，并传入事务句柄作为参数，如果没有错误返回，那么就提交整个事务
  		if err = fc(tx); err == nil {
  			panicked = false // 将标志变量设置为false，表示没有发生panic
  			return tx.Commit().Error // 返回提交事务的结果
  		}
  	}
  
  	panicked = false // 将标志变量设置为false，表示没有发生panic
  	return // 返回错误结果
  }
  
  
  ```