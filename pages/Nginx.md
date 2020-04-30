六种负载均衡方式：（作用也是避免请求过了集中到一台机器）

> 1. 轮询
> 2. weight权重方式
> 3. ip_hash根据ip分配方式，可以让同一个用户访问同一个资源。
> 4. least_conn最少链接方式
> 5. fair响应时间方式，请求向响应时间较短的机器发。
> 6. url_hash依据URL分配方式，根据url请求到同一个服务器，避免缓存资源多次请求。

**配置相关**

1. event标签里面，有**worker_processer**（表示连接处理请求的线程个数，一般和cpu核数保持一致），**worker_connections**（每一个worker最大连接数，一般和ulimit -a中的openfile参数[这个参数表示操作系统最大打开文件数量，最好改大些，我见过netty实战中直接把它改成100万了]除以worker_processer数）。
2. **keepalive_timeout、send timeout **请求和响应的超时时间。
3. **default_type  application/octet-stream;**将一些路径下的例如文件通知到浏览器是文件流。
4. **client_max_body_size 10m;**上传文件大小限制。
5. 转发配置


	server{
		  listen 80;
		  charset utf-8;
		  server_name www.ican.com;
		  access_log logs/host.access.log;
		  location /{
			  proxy_pass http://127.0.0.1:7001;
		  }
	}

