- [go-chassis 运行时做了什么？ - 掘金](https://juejin.cn/post/6900457796018372616/)
-
-
- ## chassis init
- chassis init Steps #card
	- 配置初始化
	  collapsed:: true
		- 配置路径
			- require
				- global config 对应 conf_path/chassis.yaml
				- microservice config 对应 conf_path/microservice.yaml
			- optional 一些标准插件
				- ```go
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
	- 插件初始化
	  collapsed:: true
		- 隐式加载
			- 使用各自的init 方法自动执行的
			- main 包 import
		-
	- 初始化handler chain
		- ```go 
		  // Handler interface for handlers
		  type Handler interface {
		      // handle invocation transportation,and tr response
		      Handle(*Chain, *invocation.Invocation, invocation.ResponseCallBack)
		      Name() string
		  }
		  
		  ```
		- 类型
			- default
			- provider
			- consumer
		- chain
			- ```go
			  type Chain struct {
			      ServiceType string
			      Name string
			      Handlers []Handler
			  }
			  
			  // GetChain is to get chain
			  func GetChain(serviceType string, name string) (*Chain, error) {
			      if name == "" {
			          name = common.DefaultChainName
			      }
			      origin, ok := ChainMap[serviceType+name]
			      if !ok {
			          return nil, fmt.Errorf("get chain [%s] failed", serviceType+name)
			      }
			      return origin, nil
			  }
			  
			  // 
			  chainMap := chaninMap[strint]*Chain{
			      "Provider+rest":  &Chain{
			            ServiceType: "Provider",
			            Name: "rest",
			            Handlers: []Handler{jwt},},
			      "Provider+default": &Chain{
			            ServiceType: "Provider",
			            Name: "default",
			            Handlers: []Handler{tracing-provider}},,
			  }
			  
			  ```
	- 初始化 server
		- 根据schema 找到服务，将对应的handle func 使用 handler chain 封装
			- http -> invocation -> router
			- invocation
				- ```go
				   inv := &invocation.Invocation{
				          MicroServiceName:   runtime.ServiceName,
				          SourceMicroService: common.GetXCSEContext(common.HeaderSourceName, req.Request),
				          Args:               req,
				          Reply:              resp,
				          Protocol:           common.ProtocolRest,
				          SchemaID:           schema,
				          OperationID:        operation,
				          URLPathFormat:      req.Request.URL.Path,
				          Metadata: map[string]interface{}{
				              common.RestMethod: req.Request.Method,
				          },
				      }
				  
				  ```
			- WrapHandlerChain()
				- 取出 Route 中 ResourceFunc （即real handler func）
				- 将 HttpRequest 转换成 chassis Invocation，
				- 将Invocation 再添加回 request 中, 添加到 handler chain 中
				- 返回一个闭包函数。
		- 启动服务，将服务注册到服务中心
		- 监听退出信号
			- ```go
			  func waitingSignal() {
			      //Graceful shutdown
			      c := make(chan os.Signal) // 创建一个os.Signal channel
			      // 注册要接收的信号
			      signal.Notify(c, syscall.SIGINT, syscall.SIGHUP, syscall.SIGTERM, syscall.SIGQUIT, syscall.SIGILL, syscall.SIGTRAP, syscall.SIGABRT)
			      select {
			      case s := <-c:
			          openlogging.Info("got os signal " + s.String())
			      case err := <-server.ErrRuntime:
			          openlogging.Info("got server error " + err.Error())
			      }
			  
			      // 判断服务是否有注册
			      if !config.GetRegistratorDisable() {
			          registry.HBService.Stop()// 停掉心跳服务
			          openlogging.Info("unregister servers ...")
			          // 从server center 中退出
			          if err := server.UnRegistrySelfInstances(); err != nil {
			              openlogging.GetLogger().Warnf("servers failed to unregister: %s", err)
			          }
			      }
			  
			      for name, s := range server.GetServers() {
			          // 遍历服务，调用服务的 stop 方法
			          openlogging.Info("stopping server " + name + "...")
			          err := s.Stop()
			          if err != nil {
			              openlogging.GetLogger().Warnf("servers failed to stop: %s", err)
			          }
			          openlogging.Info(name + " server stop success")
			      }
			  
			      openlogging.Info("go chassis server gracefully shutdown")
			  }
			  
			  ```
			- > go http Server 从1.8 之后支持优雅退出。
			- 处理步骤
				- 判断是否注册到服务中心，
				- 如果注册，停掉心跳发送，退出注册，
				- 然后调用 server.Shutdown() 来优雅退出。
			- 监听信号` signal.Notify(c, syscall.SIGINT, syscall.SIGHUP, syscall.SIGTERM, syscall.SIGQUIT, syscall.SIGILL, syscall.SIGTRAP, syscall.SIGABRT)`
			- Why no kill? #todo
		-
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
	- 服务注册
	- 信号监听
		- [[go 优雅启停]]
-
-