- 随着核数的增加， runtime.epollwait 占用时间增加。
- 大段的Proc暂停
	- 原因：增加GOMAXPROC时，网络I/O阻塞导致进程等待的问题。
- 测试程序
	- ```go
	  func main() {
	  
	      var ncpu int
	  
	      flag.IntVar(&ncpu, "ncpu", 64, "cpu num")
	  
	      flag.Parse()
	  
	      runtime.GOMAXPROCS(ncpu)
	  
	      go func() {
	  
	          if err := http.ListenAndServe(":6060", nil); err != nil {
	  
	              log.Println(err)
	  
	          }
	  
	          os.Exit(0)
	  
	      }()
	  
	      listen, err := net.Listen("tcp", ":13500")
	  
	      if err != nil {
	  
	          log.Println("listen error: ", err)
	  
	          return
	  
	      }
	  
	      for {
	  
	          conn, err := listen.Accept()
	  
	          if err != nil {
	  
	              log.Println("accept error: ", err)
	  
	              break
	  
	          }
	  
	          go HandleConn(conn)
	  
	      }
	  
	  }
	  
	  func HandleConn(conn net.Conn) {
	  
	      defer conn.Close()
	  
	      packet := make([]byte, 1024)
	  
	      for {
	  
	          _, err := conn.Read(packet)
	  
	          if err != nil {
	  
	              log.Println("read socket error: ", err)
	  
	              return
	  
	          }
	  
	          _, _ = conn.Write([]byte("+OK\r\n"))
	  
	      }
	  
	  }
	  ```
-
- StackTrace
	- runtime.mcall：在没有I/O事件时触发来停止goroutine的运行。
	- runtime.park_m：修改当前运行的goroutine状态，表明goroutine进入等待。M需要重新调度goroutine运行。
	- runtime.findrunnable：寻找可运行的goroutine。
	- runtime.netpoll：通过epoll获取可以运行的goroutine列表。
	- epollwait 使用的是**非阻塞轮询**的方式，当空闲的CPU较少时，轮询epollwait的时间也会增加，从而延时也会增加。
-
- 方向
	- 使用端口复用，尝试用4个16核的与1个64核进行对比。
	- 替换代码中的go net设计，减少goroutine个数，从而减少调度带来的影响，可以使用类似nginx的Reactor网络模型。
	- 修改系统参数，允许更快的触发I/O事件。在压测时，cpu的使用率只能到85%。而I/O成为性能瓶颈，TCP报文重传。
-
- 变量
	- 读写比 （影响不大）
	- CPU 核数
	- Ops 数
-
- 指标
	- P95 lat
	- Ops count
-
- 工具
	- memtier_benchmark
-
- 最佳 core = 4