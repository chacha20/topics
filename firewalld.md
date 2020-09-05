firewalld简易小教程

学习完整内容可以去官网查看：
https://www.firewalld.org

# 1. 基础操作
## 安装
```
# CentOS安装：
yum install firewalld -y
```
```
# Ubuntu安装
apt install firewalld -y
```
```
# 设置firewalld服务为开机自启动。
# 注意执行完并不会立即启动firewalld服务，除非重启系统或者手动启动服务
systemctl enable firewalld
```
```
# 手动启动firewalld服务
systemctl start firewalld
```
```
# 重新加载服务
firewall-cmd --reload
```
## 区域
firewalld有区域的概念，内置的区域有：drop/block/public/external/dmz/work/home/internal/trusted。
详细描述在这里看：https://firewalld.org/documentation/zone/predefined-zones.html
对于CentOS/Ubuntu来说，装好了firewalld后，它的默认设置是将现有网卡加入public区域中，如果没有多网卡多区域的需求的话，可以将代码中--zone=public省略，后面的代码均为省略了该参数的代码，如果要对其它区域操作，而不是对默认区域操作，必须加上--zone参数。
```
# 查看public区域的参数清单（默认隐藏了--zone=public参数）
firewall-cmd --list-all
```
## 端口操作
```
# 临时开放80的tcp端口，重启服务或系统后会失效，用于临时测试。
fireawll-cmd --add-port=80/tcp
```
```
永久开放80的tcp端口
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```
```
# 临时关闭80端口，重启服务或系统后失效
firewall-cmd --remove-port=80/tcp
```
```
# 永久关闭80端口
firewall-cmd --permanent --remove-port=80/tcp
firewall-cmd --reload
```
```
# 永久开放端口段10000-65535内的端口，同时允许tcp和udp
firewall-cmd --permanent --add-port=10000-65535/tcp
firewall-cmd --permanent --add-port=10000-65535/udp
firewall-cmd --reload
```
```
# 永久开放多个端口80，443，8080，8443的tcp流量
firewall-cmd --permanent --add-port=80/tcp --add-port=443/tcp --add-port=8080/tcp --add-port=8443/tcp
firewall-cmd --reload
# 分多行写的用法如下，用"\"符号放行尾可多行显示，便于阅读
firewall-cmd --permanent\
       --add-port=80/tcp\
       --add-port=443/tcp\
       --add-port=8080/tcp\
       --add-port=8443/tcp
firewall-cmd --reload
```

## 服务操作
```
# 列出firewalld内置服务清单。
# 用户可自行添加新服务，添加方法参考官网https://www.firewalld.org
firewall-cmd --get-services
```
```
# 永久开放http服务，等效于开放80端口
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```
```
# 永久开放https服务，等效于开放443端口
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```
```
# 批量永久开通http、https、ftp服务(ftp服务默认只开放21端口，如果设置了pasv模式ftp，需要开放一个端口范围并设置在ftp服务器方可使用)
firewall-cmd --permanent --add-service=http --add-service=https --add-service=ftp
firewall-cmd --reload
```
## 转发操作
```
# 启用转发功能
# firewalld默认是关闭转发的 ，启用该功能需要检查两个设置：
# 1. /etc/sysctl.conf文件中已设置了如下参数：
#        net.ipv4.ip_forward=1
# 2. firewalld中是否开启转发，开启方式如下：
firewall-cmd --permanent --add-masquerade
firewall-cmd --reload
```
```
# 添加一项转发，转发本地443端口tcp流量至另一ip地址223.5.5.5指定端口8443
firewall-cmd --permanent --add-forward-port=port=443:toport=8443:toaddr=223.5.5.5:proto=tcp

```
```
# 添加一项转发，转发本地443端口tcp流量至本地端口8443
firewall-cmd --permanent --add-forward-port=port=443:toport=8443:proto=tcp
firewall-cmd --reload
```
```
# 添加一组转发，转发1001端口的tcp流量与udp流量至另一ip地址223.5.5.5指定端口53265
firewall-cmd --permanent --add-forward-port=port=1001:toport=53265:toaddr=223.5.5.5:proto=tcp
firewall-cmd --permanent --add-forward-port=port=1001:toport=53265:toaddr=223.5.5.5:proto=udp
firewall-cmd --reload

```
# 2. 高级操作
高级操作主要讲一下自己用得最普遍的ipset与富规则（rich-rule)的操作内容
```
# 新建一个ipset，类型为hash:net，名称为ALLOWED，后面用此做白名单
firewall-cmd --permanent --new-ipset=ALLOWED --type=hash:net
firewall-cmd --reload

# 新建一个ipset，类型为hash:net，名称为BLOCKED，后面用此做黑名单
firewall-cmd --permanent --new-ipset=BLOCKED --type=hash:net
firewall-cmd --reload
```
```
# 删除一个ipset
firewall-cmd --permanent --delete-ipset=ALLOWED
firewall-cmd --reload
```
```
# 设置22端口只允许IPSET为ALLOWED的地址访问
# 该操作需要设置富规则（rich-rule）要义如下：
# 指定source段使用ipset=ALLOWED
# 指定port段的两个参数：port和protocol，注意代码中两个port不是写错的，第一个表示port配置段，第二个表示port参数
# accept表示接收，如果是拒绝访问的话可以写为drop，共有这几种：accept|reject|drop|mark
firewall-cmd --permanent --add-rich-rule="rule family=ipv3 source ipset=ALLOWED port port=22 protocol=tcp accept"
firewall-cmd --reload
```
```
#向IPSET为ALLOWED的地址库中添加四川省地址库
cd /tmp/
wget https://github.com/metowolf/iplist/blob/master/data/cncity/500000.txt
firewall-cmd --permanent --ipset=ALLOWED --add-entries-from-file=500000.txt
rm -rf 500000.txt
firewall-cmd --reload
```
```
#向IPSET为ALLOWED的地址库中添加指定ip
firewall-cmd --permanent --ipset=ALLOWED --add-entry=233.5.5.5
firewall-cmd --permanent --ipset=ALLOWED --add-entry=233.6.6.6
firewall-cmd --reload
```
```
#在ipset为ALLOWED的地址库中删除1个ip
firewall-cmd --permanent --ipset=ALLOWED --remove-entry=233.5.5.5
firewall-cmd --reload
```
```
# 在ipset为ALLOWED的地址库中删除一个ip段（CIDR)
firewall-cmd --permanent --ipset=ALLOWED --remove-entry=177.23.20.0/24
firewall-cmd --reload
```
