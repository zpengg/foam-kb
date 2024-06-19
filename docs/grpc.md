- [Core concepts, architecture and lifecycle | gRPC](https://grpc.io/docs/what-is-grpc/core-concepts/)
- ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0990d1d71544a5281797b26b806e877~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
-
- [[RPC]]
-
- 服务定义
	- gRPC使用的是 [[Protocol Buffers]]作为接口定义语言 (默认)
- 实现
	- channel on [[http2]]
	- 长链接
	- ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc303677bd0d4981b2a167d3e27873e9~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
-
- grpc 与 HTTP2 的关系 #card
	- | gRPC | 关系 | HTTP/2 |
	  | ---- | ---- | ---- |
	  | Bi-directional stream | Mapped to | Stream |
	  | Call Header[Initial-Metadata] | Sent as | Headers + HPACK |
	  | Payload Messages(Serialized byte stream) | Fragmented into | Frame |
	  | Status[Status-Metadata(Trailing-Metadata)] | Sent as | Trailing headers |
-
- ## rpc-life-cycle
	- 单项 RPC，即客户端发送一个请求给服务端，从服务端获取一个应答，就像一次普通的函数调用。
	  
	  ```
	  rpc SayHello(HelloRequest) returns (HelloResponse){
	  }
	  ```
	- 服务端流式 RPC，即客户端发送一个请求给服务端，可获取一个数据流用来读取一系列消息。客户端从返回的数据流里一直读取直到没有更多消息为止。
	  
	  ```
	  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
	  }
	  ```
	- 客户端流式 RPC，即客户端用提供的一个数据流写入并发送一系列消息给服务端。一旦客户端完成消息写入，就等待服务端读取这些消息并返回应答。
	  
	  ```
	  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
	  }
	  ```
	- 双向流式 RPC，即两边都可以分别通过一个读写数据流来发送一系列消息。这两个数据流操作是相互独立的，所以客户端和服务端能按其希望的任意顺序读写，例如：服务端可以在写应答前等待所有的客户端消息，或者它可以先读一个消息再写一个消息，或者是读写相结合的其他方式。每个数据流里消息的顺序会被保持。
	  
	  ```
	  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
	  }
	  ```
-
- gRPC Channel
	- 提供可以访问gRPC Server的connection， 这个connection就是由主机和端口号组成的。
	- 当client创建Stub对象的时候，connection就被创建。
	- Client可以修改参数来修改gRPC的默认行为，例如：是否针对消息做压缩处理。
	- 每个Channel都是**有状态的**，这些状态包括：Connected以及idle.