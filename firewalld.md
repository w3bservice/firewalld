## firewalld使用

### 配置文件

firewalld 的配置储存在 /usr/lib/firewalld/ 和 /etc/firewalld/ 里的各种 XML 文件里，这样保持了这些文件被编辑、写入、备份的极大的灵活性，使之可作为其他安装的备份等等。

### 1、运行、停止、禁用firewalld

启动：# systemctl start  firewalld
查看状态：# systemctl status firewalld 或者 firewall-cmd --state
停止：# systemctl disable firewalld
禁用：# systemctl stop firewalld

## firewalld zone 说明

	drop（丢弃）
	任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。

	block（限制）
	任何接收的网络连接都被 IPv4 的 icmp-host-prohibited 信息和 IPv6 的 icmp6-adm-prohibited 信息所拒绝。

	public（公共）
	在公共区域内使用，不能相信网络内的其他计算机不会对您的计算机造成危害，只能接收经过选取的连接。

	external（外部）
	特别是为路由器启用了伪装功能的外部网。您不能信任来自网络的其他计算，不能相信它们不会对您的计算机造成危害，只能接收经过选择的连接。

	dmz（非军事区）
	用于您的非军事区内的电脑，此区域内可公开访问，可以有限地进入您的内部网络，仅仅接收经过选择的连接。

	work（工作）
	用于工作区。您可以基本相信网络内的其他电脑不会危害您的电脑。仅仅接收经过选择的连接。

	home（家庭）
	用于家庭网络。您可以基本信任网络内的其他计算机不会危害您的计算机。仅仅接收经过选择的连接。

	internal（内部）
	用于内部网络。您可以基本上信任网络内的其他计算机不会威胁您的计算机。仅仅接受经过选择的连接。

	trusted（信任）
	可接受所有的网络连接。
	指定其中一个区域为默认区域是可行的。当接口连接加入了 NetworkManager，它们就被分配为默认区域。安装时，firewalld 里的默认区域被设定为公共区域。

### 2、配置firewalld

查看版本：$ firewall-cmd --version

查看帮助：$ firewall-cmd --help

查看设置：

	显示状态：$ firewall-cmd --state
		[root@localhost ~]# firewall-cmd --state
		running

	查看区域信息: $ firewall-cmd --get-active-zones
		[root@localhost ~]# firewall-cmd --get-active-zones
			public
	  	interfaces: ens33

	查看指定接口所属区域：$ firewall-cmd --get-zone-of-interface=eth0

拒绝所有包：# firewall-cmd --panic-on

取消拒绝状态：# firewall-cmd --panic-off

查看是否拒绝：$ firewall-cmd --query-panic

更新防火墙规则：# firewall-cmd --reload

\# firewall-cmd --complete-reload

    两者的区别就是第一个无需断开连接，就是firewalld特性之一动态添加规则，第二个需要断开连接，类似重启服务

将接口添加到区域，默认接口都在public

	# firewall-cmd --zone=public --add-interface=eth0

永久生效再加上 --permanent 然后reload防火墙

设置默认接口区域

	# firewall-cmd --set-default-zone=public
立即生效无需重启

打开端口（貌似这个才最常用）
查看所有打开的端口：

	# firewall-cmd --zone=dmz --list-ports

加入一个端口到区域：

	# firewall-cmd --zone=dmz --add-port=8080/tcp

若要永久生效方法同上

打开一个服务，类似于将端口可视化，服务需要在配置文件中添加，/etc/firewalld 目录下有services文件夹

	# firewall-cmd --zone=work --add-service=smtp

移除服务

	# firewall-cmd --zone=work --remove-service=smtp

还有端口转发功能、自定义复杂规则功能、lockdown\


###  红帽官方文档

中文文档地址

https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html



CentOS7防火墙firewalld简单配置和使用
    网上找了好多文章，关于CentOS7的防火墙配置和使用，都没有比较理想的说明firewalld的用法，还有一些网上摒弃centos7 firewalld防火墙，使用旧版本的iptables的替代的做法，这里笔者非常不赞同其再使用iptables。
    CentOS7使用的是Linux Kernel 3.10.0的内核版本，新版的Kernel内核已经有了防火墙netfilter，并且firewalld的使用效能更高，稳定性更好。
        CentOS7配置防火墙的两种方法：
    一、使用xml配置文件的方式配置；
方法一
cp /usr/lib/firewalld/services/http.xml /etc/firewalld/services/
firewall-cmd --reload
    二、使用命令的方式配置；
方法二
##Add
firewall-cmd --permanent --zone=public --add-port=80/tcp

##Remove
firewall-cmd --permanent --zone=public --remove-port=80/tcp

##Reload
firewall-cmd --reload

命令含义：

    --zone #作用域

    --add-port=80/tcp  #添加端口，格式为：端口/通讯协议

   --permanent   #永久生效，没有此参数重启后失效

    其中，方法二的配置方式是间接修改/etc/firewalld/zones/public.xml文件，
    方案一也需要在public.xml里面新增<service name="http"/>，否则http的防火墙规则不会生效，
    而且两种配置方式都需要重新载入防火墙。
   附：
查看防火墙状态

systemctl status firewalld.service

启动防火墙

systemctl start firewalld.service

关闭防火墙

systemctl stop firewalld.service

重新启动防火墙

systemctl restart firewalld.service


## nat

自动开启内核转发

firewall-cmd --permanent --zone=public --add-masquerade
firewall-cmd --permanent --zone=public --add-rich-rule='rule family=ipv4 source address=10.8.0.0/24 masquerade'

## 端口转发


## 绑定mac

## rich rule

Example 1

       Enable new IPv4 and IPv6 connections for protocol 'ah'
       rule protocol value="ah" accept
Example 2

       Allow new IPv4 and IPv6 connections for service ftp and log 1 per minute using audit
       rule service name="ftp" log limit value="1/m" audit accept
Example 3

       Allow new IPv4 connections from address 192.168.0.0/24 for service tftp and log 1 per minutes using syslog
       rule family="ipv4" source address="192.168.0.0/24" service name="tftp" log prefix="tftp" level="info" limit value="1/m" accept
       firewall-cmd --add-rich-rule='rule family="ipv4" source ipset="zabbix_server" service name="tftp" log prefix="tftp" level="info" accept'

------------------------------

所有网卡绑定public 为默认区域

单网卡绑定多ip

将所有网络接口都绑定到public


创建ipset
白名单
	跳板机
	私网ip段
	snmp_server
	zabbix_server

rule family="ipv4" source NOT ipset="white_addr" service name="ssh" log level="warning" reject  #除白名单以外的都不可以通信

黑名单
	手动临时添加

端口list
	3306  mysql    系统自带服务   默认允许私网通信
	22    ssh      系统自带服务		默认允许跳板机通信
	80		http		 系统自带服务   默认允许所有主机
	443		https		 系统自带服务   默认允许所有主机
	10050 zabbix-agent   自定义服务  默认与zabbix_server通信
	9000  php-fpm				 自定义服务  默认私网通信
	161   snmp					 系统自带服务  默认与snmpserver通信
	1311  dell open manage  自定义服务   默认允许跳板机通信


	rule family="ipv4" source NOT ipset="white_addr" service name="ssh" log level="warning" reject

如有其它服务创建其它服务到相应zone

临时开启端口

	删除，添加端口 协议：port

开启nat配置

白名单lock开启禁止无程序修改firewalld

富规则添加
firewall-cmd --add-rich-rule='rule family="ipv4" source ipset="zabbix_server" service name="tftp" log prefix="tftp" level="info" accept'


-----------------------------------

	sshd:123.57.248.15:allow
	sshd:121.22.28.50:allow
	sshd:all:den

	*filter
	:INPUT DROP
	:FORWARD ACCEPT
	:OUTPUT ACCEPT

	-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	-A INPUT -i lo -j ACCEPT
	-A INPUT -p icmp -j ACCEPT

	-A INPUT -p tcp -m tcp --dport 80:85 -j ACCEPT
	-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
	-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
	-A INPUT -s 10.0.0.0/24 -p tcp -m tcp --dport 3306 -j ACCEPT
	-A INPUT -s 10.0.0.0/24 -p tcp -m tcp --dport 9000 -j ACCEPT

	-A INPUT -s 123.56.92.88  -p tcp -m tcp --dport 1311 -j ACCEPT
	-A INPUT -s 121.22.8.82 -p tcp -m tcp --dport 1311 -j ACCEPT
	-A INPUT -s 10.0.0.106 -p tcp -m tcp --dport 10050 -j ACCEPT
	-A INPUT -s 202.63.165.106 -p tcp -m tcp --dport 10050 -j ACCEPT
	-A INPUT -s 123.57.156.2 -p tcp -m tcp --dport 22 -j ACCEPT
	-A INPUT -s 123.56.92.88 -p tcp -m tcp --dport 22 -j ACCEPT
	-A INPUT -s 123.57.248.15 -p tcp -m tcp --dport 22 -j ACCEPT
	-A INPUT -s 121.22.28.50 -p tcp -m tcp --dport 22 -j ACCEPT
	-A INPUT -s 123.59.50.68 -p udp --dport 161 -j ACCEPT
	-A INPUT -s 103.218.243.80 -p udp --dport 161 -j ACCEPT
	COMMIT
