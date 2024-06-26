- 例子
	- 以下`application()`函数就是符合WSGI标准的一个HTTP处理函数
	- ``` python
	  def application(environ, start_response):
	      start_response('200 OK', [('Content-Type', 'text/html')])
	      return [b'<h1>Hello, web!</h1>']
	  ```
-
- WSGI 含义：
	- environ：一个包含所有HTTP请求信息的`dict`对象；
	- start_response：一个发送HTTP响应的函数
- WSGI server
	- uwsgi
	- [[gunicorn]]