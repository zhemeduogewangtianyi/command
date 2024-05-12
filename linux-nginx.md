一、安装依赖

安装nginx之前，需要安装  gcc-c++ 编译器。nginx依赖的 pcre 和 zlib 包。

1、gcc-c++ 编译器

yum install gcc-c++

yum install -y openssl openssl-devel
2、pcre 包

yum install -y pcre pcre-devel
3、zlib 包

yum install -y zlib zlib-devel
如果上面的都没有安装过个，可以选择下面的一键安装命令。

yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
二、安装Nginx

nginx版本：Index of /download/

1、下载安装文件，解压并进入目录。

//下载文件
wget http://nginx.org/download/nginx-1.18.0.tar.gz
//解压文件
tar -zxvf nginx-1.18.0.tar.gz
//进入安装目录
cd /usr/local/nginx-1.18.0
2、配置文件

./configure
这里需要注意一下，如果不使用其他指令，默认的安装位置为 /usr/local/nginx，如果需要安装在其他地方使用下面的命令

./configure --prefix=/home/www/server/nginx
./configure --prefix=/root/server
这里我是安装在/home/www/server/nginx

./configure --prefix=/root/server --with-http_ssl_module
安装 open ssl

./configure --prefix=/root/server --with-compat --add-module=/root/ngx_healthcheck_module

3、编译并安装

make && make install
4、检查配置文件

进入到上面的安装路径（/home/www/server/nginx）下的sbin目录下，执行测试命令

 ./nginx -t
或者可以直接使用 /home/www/server/nginx/sbin/nginx -t   红色部分为nginx的安装路径。

5、启动/停止nginx

/** 启动 **/
//如果在nginx的目录里面就执行下面的命令
./nginx

//如果不在nginx的安装目录里面就执行下面的命令
/home/www/server/nginx/sbin/nginx

/** 停止 **/
/home/www/server/nginx/sbin/nginx -s stop

/** 重启 **/
/home/www/server/nginx/sbin/nginx -s reload
这时候输入服务器的ip，出现下面的页面就代表安装成功，



 三、配置systemctl

使用上面的方法启停nginx，稍微有点不方便，不是要进入安装目录输入命令，就是要打出长长的一串，那么就可以配置一下systemctl

systemd是Linux系统最新的初始化系统(init),作用是提高系统的启动速度，尽可能启动较少的进程，尽可能更多进程并发启动。

systemd对应的进程管理命令是 systemctl

1、创建nginx.service文件

vim usr/lib/systemd/system/nginx.service 
2、写入内容

使用vim命令之后，按 i 键到插入模式，然后把下面内容复制上去，注意下面的 ExecStart 、ExecStop、ExecReload 后面的内容为上面执行nginx启动、停止、重启的命令。

[Unit]                                                                                                                
Description=nginx - high performance web server                                                                       
After=network.target remote-fs.target nss-lookup.target                                                               

[Service]                                                                                                             
Type=forking      
ExecStart=/home/www/server/nginx/sbin/nginx          
ExecStop=/home/www/server/nginx/sbin/nginx/sbin/nginx -s stop                                                                                                  
ExecReload=/home/www/server/nginx/sbin/nginx/sbin/nginx -s reload        
PrivateTmp=true                                                                                                       

[Install]
WantedBy=multi-user.target
3、重载systemctl命令

systemctl daemon-reload
 4、设置nginx开机启动

如果有需要让nginx开机启动，可运行下面的命令

//开启 nginx开机启动
systemctl enable nginx.service 

//关闭 nginx开机启动
systemctl disable nginx.service 
下面就可以在任意位置使用systemctl启动、停止nginx

//开启nginx
systemctl start nginx

//关闭nginx
systemctl stop nginx

//重启nginx
systemctl restart nginx

//查看状态
systemctl status nginx





为了在 Nginx 1.18.0 中实现健康检查和多 IP 集群功能，您可以按照以下步骤操作：

下载并解压 Nginx 1.18.0 源代码：
wget https://nginx.org/download/nginx-1.18.0.tar.gz
tar -zxvf nginx-1.18.0.tar.gz

下载并解压 ngx_http_healthcheck_module 源代码：
git clone https://github.com/zhouchangxun/ngx_healthcheck_module.git
进入 Nginx 源代码目录，并将 ngx_http_healthcheck_module 添加为 Nginx 的模块：

cd nginx-1.18.0
./configure --with-stream --add-module=../ngx_healthcheck_module
make
sudo make install
配置 Nginx：
在您的 Nginx 配置文件中添加健康检查和多 IP 集群的配置。以下是一个示例配置：

http {
    upstream backend {
        server 192.168.1.1:8080;
        server 192.168.1.2:8080;
        server 192.168.1.3:8080;
        check interval=3000 rise=2 fall=5 timeout=1000;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
        }
    }
}
在这个配置中，我们定义了一个名为 backend 的 upstream，其中包含了多个服务器的 IP 和端口，并配置了健康检查选项。然后，在 server 块中，我们将请求代理到 backend 上。

测试并启动 Nginx：
确保您的配置文件没有语法错误，并通过以下命令测试：

nginx -t
如果没有错误，则启动 Nginx：

nginx
现在，您的 Nginx 已经配置了健康检查和多 IP 集群功能。您可以根据需要对配置进行调整和优化。



######

解决方法：
这里介绍修改配置文件nginx.conf两种方法：
1）在server段里插入如下正则：
listen    80;
server_name www.yuyangblog.net;
if ($host != 'www.yuyangblog.net'){
  return 403;
}



if ($host != 'ireading.uk'){
  return 403;
}


2)添加一个server
新加的server（注意是新增，并不是在原有的server基础上修改）
server {
 listen 80 default;
 server_name _;
 return 403;
}
原来server里面插入：
listen    80;

server_name www.yuyangblog.net;

proxy_max_temp_file_size 0;

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;

    server {
        listen       80;
        server_name  123123 123123;
        proxy_max_temp_file_size 0;

        set $flag 0;	
        if ($host = '1.com'){
            set $flag 1;
        }
        if ($host = 'www.1.com') {
            set $flag 1;
        }
        if ($host = '2.uk'){
            set $flag 2;
        }
        if ($host = 'www.2.uk') {
           set $flag 2;
        }
        if ($host = '3.com'){
            set $flag 3;
        }
        if ($host = '3.com') {
            set $flag 3;
        }
        if ($flag = 0) {
            return 403;
        }

        location / {
						if ($flag = 1) {
							proxy_pass http://127.0.0.1:8087;
							break;
	   				 }
	   		 		if ($flag = 2) {
                proxy_pass http://127.0.0.1:8088;
                break;
            }  
            if ($flag = 3) {
                proxy_pass http://127.0.0.1:8089;
                break;
            }

            root   html;
            index  index.html index.htm;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

       
    }

}
```

