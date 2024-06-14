- [go-chassis 运行时做了什么？ - 掘金](https://juejin.cn/post/6900457796018372616/)
-
-
- ## chassis init
- chassis init Steps #card
	- 配置初始化
		- 配置路径
			- require
				- global config 对应 conf_path/chassis.yaml
				- microservice config 对应 conf_path/microservice.yaml
			- optional
				- ```
				  // pkg/util/fileutil/fileutil.go
				  
				  const (
				  	//ChassisConfDir is constant of type string
				  	ChassisConfDir = "CHASSIS_CONF_DIR"
				  	//ChassisHome is constant of type string
				  	ChassisHome = "CHASSIS_HOME"
				  	//SchemaDirectory is constant of type string
				  	SchemaDirectory = "schema"
				  )
				  
				  const (
				  	//Global is a constant of type string
				  	Global = "chassis.yaml"
				  	//LoadBalancing is constant of type string
				  	LoadBalancing = "load_balancing.yaml"
				  	//RateLimiting is constant of type string
				  	RateLimiting = "rate_limiting.yaml"
				  	//Definition is constant of type string
				  	Definition = "microservice.yaml"
				  	//Hystric is constant of type string
				  	Hystric = "circuit_breaker.yaml"
				  	//PaasLager is constant of type string
				  	PaasLager = "lager.yaml"
				  	//TLS is constant of type string
				  	TLS = "tls.yaml"
				  	//Monitoring is constant of type string
				  	Monitoring = "monitoring.yaml"
				  	//Auth is constant of type string
				  	Auth = "auth.yaml"
				  	//Tracing is constant of type string
				  	Tracing = "tracing.yaml"
				  	//Router is constant of type string
				  	Router = "router.yaml"
				  )
				  ```
		- [[archaius]]
		-
	- 插件初始化
	- 初始化handler chain
	- 初始化 server
	- 其它
- 配置路径 ${ChassisConfDir}  > ${ChassisHome}/conf
-
-
- ## chassis run
- chassis run steps #card
  card-last-interval:: 4
  card-repeats:: 1
  card-ease-factor:: 2.6
  card-next-schedule:: 2024-06-18T16:45:24.634Z
  card-last-reviewed:: 2024-06-14T16:45:24.634Z
  card-last-score:: 5
  collapsed:: true
	- 服务注册
	- 信号监听
-
-