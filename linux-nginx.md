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

