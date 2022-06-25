git config --system --unset credential.helper
git config --global credential.helper store

如果你是Redhat、Centos版本linux

 yum install git
1
如果你是ubuntu、Debian版本linux

apt-get install git
1
配置git
git config --global user.name "你的github用户名"
git config --global user.email "你的github邮箱"
#查看配置是否生效
git config --list
1
2
3
4
然后

ssh-keygen -t rsa -C "你的github邮箱"
ls -a #查看.ssh隐藏文件夹
cd .ssh
ls #可以看到  id_rsa.pub  文件
cat id_rsa.pub #复制输出的内容
1
2
3
4
5
目的是生成一个ssh key，中间默认确定即可

然后 打开GitHub网站—>>>点自己头像—>>>打开设置—>>>SSH and GPG keys—>>>点new SSH key，把刚才复制的内容粘贴进 Key 中，最后添加即可
