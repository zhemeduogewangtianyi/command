

```
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine


yum install docker

systemctl enable docker

systemctl start docker
```







```
DockerClient使用

开启docker远程连接服务
docker默认只提供本地unix，sock文件的连接方式，让docker能够监听tcp端口还需要进行一些配置。

vim /lib/systemd/system/docker.service 
#配置普通模式，-H参数指定docker应用程序监听方式，或者采用下面的安全连接方式2选1
ExecStart=/usr/bin/dockerd -H unix://var/run/docker.sock -H tcp://0.0.0.0:2375
#配置安全连接，注意这里的/home/user/certs/ 根据自己情况替换为实际的的密钥存放路径 
ExecStart=/usr/bin/dockerd -D --tlsverify=true --tlscert=/home/user/certs/server-cert.pem --tlskey=/home/user/certs/server-key.pem --tlscacert=/home/user/certs/ca.pem -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
#重载配置，重启服务
systemctl daemon-reload 
systemctl restart docker
#查看端口监听
netstat -nlp |grep 2375
```



```
vim /etc/docker/daemon.json
```



docker 源

```
{
"registry-mirrors":[
    "https://ustc-edu-cn.mirror.aliyuncs.com",
    "http://hub-mirror.c.163.com",
    "https://registry.aliyuncs.com"
    ]
}
```



重新加载配 + 重启docker

```


systemctl daemon-reload && systemctl restart docker.service
```



挂载 nginx 页面

```
docker run -d -p 80:80 -v /root/games_page/games:/usr/share/nginx/html 605c77e624dd
```



2375防止攻击

```
1. 创建一个目录用于存储生成的证书和秘钥
mkdir /docker-ca && cd /docker-ca

2. 使用openssl创建CA证书私钥，期间需要输入两次密码，生成文件为ca-key.pem
openssl genrsa -aes256 -out ca-key.pem 4096
Enter pass phrase for ca-key.pem: Your password

（hylink2014）

3. 根据私钥创建CA证书，期间需要输入上一步设置的私钥密码，然后依次输入国家是 CN，省例如是Guangdong、市Guangzhou、组织名称、组织单位、姓名或服务器名、邮件地址，都可以随意填写，生成文件为ca.pem（注意证书有效期）
openssl req -new -x509 -days 3650 -key ca-key.pem -sha256 -out ca.pem

4. 创建服务端私钥，生成文件为server-key.pem
openssl genrsa -out server-key.pem 4096

5. 创建服务端证书签名请求文件，用于CA证书给服务端证书签名。IP需要换成自己服务器的IP地址，或者域名都可以。生成文件server.csr
openssl req -subj "/CN=Your server IP" -sha256 -new -key server-key.pem -out server.csr

openssl req -subj "/CN=35.1.176.8" -sha256 -new -key server-key.pem -out server.csr

6. 配置白名单，用多个用逗号隔开，例如： IP:192.168.0.1,IP:0.0.0.0，这里需要注意，虽然0.0.0.0可以匹配任意，但是仍然需要配置你的服务器IP，如果省略会造成错误
echo subjectAltName = IP:Your server IP,IP:0.0.0.0 >> extfile.cnf

echo subjectAltName = IP:1.117.221.38,IP:0.0.0.0 >> extfile.cnf

7. 将Docker守护程序密钥的扩展使用属性设置为仅用于服务器身份验证
echo extendedKeyUsage = serverAuth >> extfile.cnf

8. 创建CA证书签名好的服务端证书，期间需要输入CA证书私钥密码，生成文件为server-cert.pem
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \-CAcreateserial -out server-cert.pem -extfile extfile.cnf

9. 创建客户端私钥，生成文件为key.pem
openssl genrsa -out key.pem 4096

10. 创建客户端证书签名请求文件，用于CA证书给客户证书签名，生成文件client.csr
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

11. 要使密钥适合客户端身份验证，请创建扩展配置文件
echo extendedKeyUsage = clientAuth >> extfile.cnf

12. 创建CA证书签名好的客户端证书，期间需要输入CA证书私钥密码，生成文件为cert.pem
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \-CAcreateserial -out cert.pem -extfile extfile.cnf

13. 删除不需要的文件，两个证书签名请求
rm -v client.csr server.csr

14. 修改证书为只读权限保证证书安全
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem
15. 归集服务器证书
cp server-*.pem /etc/docker/ && cp ca.pem /etc/docker/
最终生成文件如下，有了它们我们就可以进行基于TLS的安全访问了

ca.pem CA证书
ca-key.pem CA证书私钥
server-cert.pem 服务端证书
server-key.pem 服务端证书私钥
cert.pem 客户端证书
key.pem 客户端证书私钥

scp root@1.117.221.38:/root/ssl/cert.pem /Users/wty/project/anyone/dao/src/main/resources/ssl
scp root@1.117.221.38:/root/ssl/key.pem /Users/wty/project/anyone/dao/src/main/resources/ssl
scp root@1.117.221.38:/root/ssl/ca.pem /Users/wty/project/anyone/dao/src/main/resources/ssl
scp root@1.117.221.38:/root/ssl/ca-key.pem /Users/wty/project/anyone/dao/src/main/resources/ssl

16. 配置Docker支持TLS
修改docker.service文件
vi /usr/lib/systemd/system/docker.service

实际位置：vi /etc/systemd/system/docker.service

修改以ExecStart开头的配置，开启TLS认证，并配置好CA证书、服务端证书和服务端私钥
ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=/etc/docker/ca.pem --tlscert=/etc/docker/server-cert.pem --tlskey=/etc/docker/server-key.pem -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
重新加载daemon
systemctl daemon-reload && systemctl restart docker
重启docker
service docker restart

17. 在客户端测试连接
保存相关客户端的pem文件到本地

IDEA->Preferences->Bulild, Execution,Deployment->Docker->TCP socket
到此，Docker对外开放2375端口再也不怕被攻击了！
```





git 死慢

```
Linux服务器hosts修改
查看：
cat /etc/hosts

修改：
vim /etc/hosts

按 i 切换至编辑模式。
将上面查询到的合适ip加到hosts文件即可
20.205.243.166 http://github.com
185.199.111.133 http://raw.githubusercontent.com
185.199.111.133 http://raw.githubusercontent.com
输入完成后，按 Esc，输入 :wq，保存文件并返回。
```

