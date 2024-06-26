- roundrobin，表示简单的轮询，这个不多说，这个是负载均衡基本都具备的
- static-rr，表示根据权重，建议关注
- leastconn，表示最少连接者先处理，建议关注
- source，表示根据请求源IP，这个跟Nginx的IP_hash机制类似，我们用其作为解决session问题的一种方法，建议关注
- ri，表示根据请求的URI
- rl_param，表示根据请求的URL参数’balance url_param’ requires an URL parameter name
  hdr(name)，表示根据HTTP请求头来锁定每一次HTTP请求
- rdp-cookie(name)，表示根据据cookie(name)来锁定并哈希每一次TCP请求