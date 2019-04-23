## 配置注意

firewalld的所有配置命令，如不加--permanent 都是临时生效。要永久生效需要加上该选项。
直接修改配置文件需要重载防火墙。
  firewall-cmd --reload   不断开连接，重读配置文件
  firewall-cmd --complete-reload  断开连接，类似服务重启。有时候需要，但很少使用。
使用临时配置防火墙，也可以通过命令将临时配置保存下来，为永久配置。

## 接口配置

所有的接口都要属于一个区域。不能同时属于多个区域。

drop（丢弃）
任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。

设置默认接口区域

	firewall-cmd --set-default-zone=public

将接口添加到区域

  firewall-cmd --zone=public --add-interface=eth0

## 打开所有人可以访问的端口或服务

都在区域配置文件中配置，一般在这里开启公共服务，例如http、https...
或是临时开启一些端口。还可以设置超时时间，超过这个时间就关闭。

## 端口

  查看打开的端口

    firewall-cmd --zone=dmz --list-ports
    所有人都可以访问该端口，如需要指定ip访问。不在这里打开。在富规则中设置

  添加打开端口

    firewall-cmd --zone=dmz --add-port=8080/tcp
     --add-port=portid[-portid]/protocol [--timeout=timeval]

    所有人都可以访问该端口，如需要指定ip访问。不在这里打开。在富规则中设置

  删除端口

  firewall-cmd --zone=work --remove-port=smtp

### 服务

端口list
	3306  mysql    系统自带服务   默认允许私网通信
	22    ssh      系统自带服务		默认允许跳板机通信
	80		http		 系统自带服务   默认允许所有主机
	443		https		 系统自带服务   默认允许所有主机
	10050 zabbix-agent   自定义服务  默认与zabbix_server通信
	9000  php-fpm				 自定义服务  默认私网通信
	1311  dell open manage  自定义服务   默认允许跳板机通信

ipsets

blacklist
  临时添加
pravite_addr
  私网地址段
white_addr
  跳板机
zabbix-server
  zabbix服务器地址




  查看打开的服务

    firewall-cmd --zone=drop --list-service
    所有人都可以访问该端口，如需要指定ip访问。不在这里打开。在富规则中设置

  添加打开服务

    firewall-cmd --zone=drop --add-service=service
    所有人都可以访问该端口，如需要指定ip访问。不在这里打开。在富规则中设置

  删除服务

    firewall-cmd --zone=work --remove-service=smtp

## 端口转发

  [--permanent] [--zone=zone] --list-forward-ports
  [--permanent] [--zone=zone] --add-forward-port=port=portid[-portid]:proto=protocol[:toport=portid[-portid]][:toaddr=address[/mask]] [--timeout=timeval]
  [--permanent] [--zone=zone] --remove-forward-port=port=portid[-portid]:proto=protocol[:toport=portid[-portid]][:toaddr=address[/mask]]
  [--permanent] [--zone=zone] --query-forward-port=port=portid[-portid]:proto=protocol[:toport=portid[-portid]][:toaddr=address[/mask]]



## nat

  自动开启内核转发
  firewall-cmd --permanent --zone=public --add-masquerade
  firewall-cmd --permanent --zone=public --add-rich-rule='rule family=ipv4 source address=10.8.0.0/24 masquerade'

## 配置富规则

一般用来配置需要有限制的服务或端口，例如ssh...
富规则也是存在于区域配置文件之中

General rule structure

    rule
      [source]
      [destination]
      service|port|protocol|icmp-block|masquerade|forward-port|source-port
      [log]
      [audit]
      [accept|reject|drop|mark]

Rule structure for source black or white listing

    rule
      source
      [log]
      [audit]
      accept|reject|drop|mark

limit value="rate/duration"      duration："s", "m", "h", "d". "s"
"2/d", which means at maximum two matches per day.


Example

    Example 1
        Enable new IPv4 and IPv6 connections for protocol 'ah'

            rule protocol value="ah" accept

    Example 2
        Allow new IPv4 and IPv6 connections for service ftp and log 1 per minute using audit

            rule service name="ftp" log limit value="1/m" audit accept

    Example 3
        Allow new IPv4 connections from address 192.168.0.0/24 for service tftp and log 1 per minutes using syslog

            rule family="ipv4" source address="192.168.0.0/24" service name="tftp" log prefix="tftp" level="info" limit value="1/m" accept

    Example 4
        New IPv6 connections from 1:2:3:4:6:: to service radius are all rejected and logged at a rate of 3 per minute. New IPv6 connections from other sources are accepted.

            rule family="ipv6" source address="1:2:3:4:6::" service name="radius" log prefix="dns" level="info" limit value="3/m" reject
            rule family="ipv6" service name="radius" accept

    Example 5
        Forward IPv6 port/packets receiving from 1:2:3:4:6:: on port 4011 with protocol tcp to 1::2:3:4:7 on port 4012

            rule family="ipv6" source address="1:2:3:4:6::" forward-port to-addr="1::2:3:4:7" to-port="4012" protocol="tcp" port="4011"

    Example 6
        White-list source address to allow all connections from 192.168.2.2

            rule family="ipv4" source address="192.168.2.2" accept

    Example 7
        Black-list source address to reject all connections from 192.168.2.3

            rule family="ipv4" source address="192.168.2.3" reject type="icmp-admin-prohibited"

    Example 8
        Black-list source address to drop all connections from 192.168.2.4

            rule family="ipv4" source address="192.168.2.4" drop


### 规则管理

还可以直接修改配置文件，但是需要重载防火墙
列出配置
[--permanent] [--zone=zone] --list-rich-rules
添加
[--permanent] [--zone=zone] --add-rich-rule='rule' [--timeout=timeval]
删除
[--permanent] [--zone=zone] --remove-rich-rule='rule'
查询
[--permanent] [--zone=zone] --query-rich-rule='rule'

firewall-cmd --add-rich-rule='rule family="ipv4" source ipset="zabbix_server" service name="tftp" log prefix="tftp" level="info" accept'

## 查看日志

--get-log-denied
