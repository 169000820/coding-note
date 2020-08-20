## docker 拉取 nginx镜像 部署 前端vue项目

## 1. docker 操作

``` shell
#docker命令参数
# 1. run命令参数
-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；

-d: 后台运行容器，并返回容器ID；

-i: 以交互模式运行容器，通常与 -t 同时使用；

-P: 随机端口映射，容器内部端口随机映射到主机的高端口

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

--name="nginx-lb": 为容器指定一个名称；

--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

-h "mars": 指定容器的hostname；

-e username="ritchie": 设置环境变量；

--env-file=[]: 从指定文件读入环境变量；

--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

-m :设置容器使用内存最大值；

--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

--link=[]: 添加链接到另一个容器；
--privileged: docker 特权参数, 加上可获得 root 权限
--expose=[]: 开放一个端口或一组端口；

--volume , -v:	绑定一个卷 linux路径:nginx容器路径形式
--privileged 可以启动docker的特权模式，这种模式允许我们以其宿主机具有（几乎）所有能力来运行容器，包括一些内核特性和设备访问。这是让我们可以在dokcer中运行docker的必要参数让docker运行在--privileged特权模式会有一些安全风险。这种模式下运行容器对docker宿主机拥有root访问权限。

#安装docker
yum install docker
#运行docker
systemctl start docker
#拉取最新nginx镜像
docker pull nginx:latest
#启动nginx容器
docker run --name nginx -d -p 80:80 --net host  -v /docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /docker/nginx/log:/var/log/nginx 
-v /docker/nginx/html:/usr/share/nginx/html -v /docker/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf --privileged nginx:latest # 这样主要的文件就映射到了宿主机
```

## 2. 配置 nginx.conf 文件

``` shell
#user root
worker_processes 1;  #每秒请求数量相关, 值越大能处理的请求就越多
events {
	worker_connections 1024; 连接超时时间
}
stream {
	#设置代理80端口,可直接用宿主机ip+11001端口即可访问
	server {
		listen: 11001;
		proxy_connect_timeout 1s;
		proxy_timeout 3s;
		proxy_pass 127.0.0.1:80
	}
}
http {
	include       /etc/nginx/mime.types;
	default_type  application/octet-stream;
	sendfile             on;
	keepalive_timeout    65;
	include   /etc/nginx/conf.d/*.conf;
}
```

## 3. 配置 default.conf 文件

```shell
server {
	#监听 80端口
	listen              80;
	# 监听地址
	server_name   localhost;
	
	location / {
		#前提建打包好的 dist 文件夹放在 /docker/nginx/html/ 文件目录下
		root   /usr/share/nginx/html/dist;
		index  index.html index.htm;
	}
	
	error_page    500 502 503 504 /50x.html;
	
	location = /50x.html {
		root /usr/share/nginx/html;
	}
}
```

## 4. 重启 nginx 容器

```shell
docker restart nginx
```

**<span style="color: red">【重点】</span>**: <kbd>**需关闭防火墙**</kbd>(~~长得帅随意~~)

```shell
#查看防火墙状态
systemctl status firewalld
#关闭防火墙
systemctl stop firewalld
```

然后即可通过服务器 ip+80 端口访问部署好的项目