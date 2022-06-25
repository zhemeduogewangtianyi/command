# 前置条件

1. **确保libaio软件已在linux安装**

```
# 查看libaio是否安装
[root@localhost ~]# rpm -q libaio
#  出现这个即代表安装
libaio-0.3.109-13.el7.x86_64
# 没有则执行下面命令
yum -y install libaio
复制代码
```

1. **检测当前是否已经有mysql**

```
# 检查是否安装mysql
rpm -qa | grep mysql
# 出现这个代表有安装
mysql-libs.xxx

# 删除原有安装,有多个执行多次即可
rpm -e --nodeps mysql-libs.xxx

# 删除好以后再次执行,为空则代表已删除
rpm -qa | grep mysql
复制代码
```

1. **删除相关文件夹**

```
# 查找mysql
whereis mysql

# 删除所搜到的所有文件
rm -rf /usr/local/mysqlxxxxxx /usr/var/mysqlxxxxxx
复制代码
```

1. **检查mysql用户组和用户是否存在，如果没有，则创建**

```
[root@localhost /]# cat /etc/group | grep mysql
[root@localhost /]# cat /etc/passwd |grep mysql
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd -r -g mysql mysql
[root@localhost /]# 
复制代码
```

# 开始安装

## 1. 使用wget下载MYSQL-5.7

```
# 可以使用该网址进行下载,官网在国内下载较慢
wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
复制代码
```

## 2.解压

```
# 解压压缩包到 /usr/local
tar zxvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz -C /usr/local/

复制代码
```

## 3.创建软连接

```
# /usr/local
ln -s mysql-5.7.24-linux-glibc2.12-x86_64 mysql
复制代码
```

## 4. 修改目录权限

```
# /usr/local/mysql
chown -R mysql:mysql ./
复制代码
```

## 5.初始化mysql数据库（建立默认的库和表）

```
# /usr/local/mysql
./bin/mysqld --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --initialize

# 将生成的密码保存起来
dvhAYKl!k0! :密码每个人都不一样
复制代码
```

## 6.修改配置文件

```
# /usr/local/mysql
vim /etc/my.cnf

[mysqld]
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
explicit_defaults_for_timestamp=true
symbolic-links=0

[mysqld_safe]
log-error=/usr/local/mysql/data/mysql.log
pid-file=/usr/local/mysql/data
复制代码
```

## 7.运行MYSQL

```
# /usr/local/mysql
./support-files/mysql.server start
复制代码
```

## 8.将mysqd服务添加到系统服务中

```
# /usr/local/mysql
[root@localhost mysql]# vim /etc/my.cnf
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld
[root@localhost mysql]# ls -l /etc/init.d/mysqld
-rwxr-xr-x. 1 root root 10576 Apr  2 17:58 /etc/init.d/mysqld
[root@localhost mysql]# service  mysqld restart
Shutting down MySQL.. SUCCESS!
Starting MySQL. SUCCESS!
[root@localhost mysql]#
复制代码
```

## 9.修改root密码

```
# /usr/local/mysql
./bin/mysqladmin -u root -p'dvhAYKl!k0!C' password '123456'   
dvhAYKl!k0!C :为上面生成的密码

update user set authentication_string=password('123456') where user='root';

复制代码
```

## 10.将mysql命令添加到系统命令执行路径中，便于使用

```
[root@localhost mysql]# ln -s /usr/local/mysql/bin/* /usr/local/bin/
[root@localhost mysql]# cd
[root@localhost ~]# mysql -u root -p123456
```



## 11.设置允许外部访问

```
CREATE USER 'carrot'@'localhost' IDENTIFIED BY '123456';

alter user user() identified by '123456a';

GRANT ALL PRIVILEGES ON *.* TO 'carrot'@'%' IDENTIFIED BY '123456';

flush privileges; 
清除缓存，重新加载权限
```





systemctl status firewalld  



systemctl stop firewalld  （暂时关闭防火墙）