- 和[[页]]类似
- 相较于一个完整的页，redo log占用的空间极小
-
- ![redo log block结构](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70aa57f306854a21b4e6847b19e1aa7b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
- | **属性** | **长度** | **说明** |
  | ---- | ---- | ---- |
  | log block header | 12字节 | block头信息 |
  | log block body | 496字节 | 存放redo log |
  | log block trailer | 4字节 | 存放block checkSum |
- [[redo log block header]]
-
- body
	- 部分用来存储实际的redo log，
-
-
-