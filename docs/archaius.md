- 配置来源
	- ```go
	     err = archaius.Init( // 初始化配置
	             archaius.WithCommandLineSource(),
	             archaius.WithMemorySource(),
	             archaius.WithENVSource(),
	             archaius.WithRequiredFiles(requiredFiles),
	             archaius.WithOptionalFiles(optionalFiles))
	  ```
	-