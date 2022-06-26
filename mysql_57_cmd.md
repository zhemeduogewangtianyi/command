# centos7 安装 Mysql 5.7.28，详细完整教程

发布于2021-10-09 15:17:36阅读 1.8K0

### 1. 下载 [MySQL](https://cloud.tencent.com/product/cdb?from=10680) yum包

```javascript
wget http://repo.mysql.com/mysql57-community-release-el7-10.noarch.rpm
```

复制

###  2.安装MySQL源

```javascript
rpm -Uvh mysql57-community-release-el7-10.noarch.rpm
```

复制

### 3.安装MySQL服务端,需要等待一些时间

rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

```javascript
yum install -y mysql-community-server
```

复制

### 4.启动MySQL

```javascript
systemctl start mysqld.service
```

复制

### 5.检查是否启动成功

```javascript
systemctl status mysqld.service
```

复制

### 6.获取临时密码，MySQL5.7为root用户随机生成了一个密码

```javascript
grep 'temporary password' /var/log/mysqld.log 
```

复制

![img](https://ask.qcloudimg.com/http-save/yehe-1159019/fd9d5d73d1a67a153f727dd860420f03.png?imageView2/2/w/1620)

### 7.通过临时密码登录MySQL，进行修改密码操作

```javascript
mysql -uroot -p
```

复制

使用临时密码登录后，不能进行其他的操作，否则会报错，这时候我们进行修改密码操作

### 8.因为MySQL的密码规则需要很复杂，我们一般自己设置的不会设置成这样，所以我们全局修改一下

```javascript
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;
```

复制

这时候我们就可以自己设置想要的密码了

```javascript
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Aqc_paas123';
```

复制

### 9.授权其他机器远程登录

```javascript
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
 
FLUSH PRIVILEGES;
```

复制

### 10.开启开机自启动

先退出mysql命令行，然后输入以下命令

```javascript
systemctl enable mysqld
systemctl daemon-reload
```

复制

### 11.设置MySQL的字符集为UTF-8，令其支持中文

```javascript
vim /etc/my.cnf
```

复制

改成如下,然后保存

```javascript
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
 
[mysql]
default-character-set=utf8
 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
default-storage-engine=INNODB
character_set_server=utf8
 
symbolic-links=0
 
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

复制

### 12.重启一下MySQL,令配置生效

```javascript
service mysqld restart
```

复制

### 13.防火墙开放3306端口

```javascript
firewall-cmd --state
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

复制

### 14.卸载MySQL仓库

一开始的时候我们安装的yum，每次yum操作都会更新一次，耗费时间，我们把他卸载掉

```javascript
rpm -qa | grep mysql
```

复制

![img](https://ask.qcloudimg.com/http-save/yehe-1159019/43d2cecaf465780dd891cf6dbda1bd8d.png?imageView2/2/w/1620)

```javascript
yum -y remove mysql57-community-release-el7-10.noarch
```

复制

### 15.[数据库](https://cloud.tencent.com/solution/database?from=10680)的操作

（1）查看mysql是否启动：service mysqld status

启动mysql：service mysqld start

停止mysql：service mysqld stop

重启mysql：service mysqld restart

（2）查看临时密码：grep password /var/log/mysqld.log



rpm -qa |  grep mysql

yum remove mysql mysql-server mysql-libs mysql-server

rpm -qa | grep mysql

rpm -ev mysql80-community-release-el7-1.noarch
rpm -ev mysql-community-client-plugins-8.0.25-1.el7.x86_64
rpm -ev mysql-community-common-8.0.25-1.el7.x86_64

rpm -ev mysql-community-common-5.7.38-1.el7.x86_64
rpm -ev mysql57-community-release-el7-10.noarch



######## 初始化密码

vi /etc/my.cnf

在最后一行回车，输入 skip-grant-tables

重启mysql：service mysqld restart



####连接数

mysql> set global max_connections = 1000;
mysql> show processlist;
mysql> show status;
修改完成后进行查看，mysql的最大连接数
mysql> show variables like '%max_connections%';