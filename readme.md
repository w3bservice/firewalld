### 防火墙默认区域为drop

默认规则，系统没有开启的端口或服务，都不能访问。系统只能与主动与外部主机通信。

### 常用系统服务

    端口list
    	3306  mysql    
    	22    ssh      
    	80		http		 
    	443		https		 
    	10050 zabbix-agent   
    	9000  php-fpm				 
    	1311  dell open manage  


### 服务、端口

默认开启80、443端口，所有主机都可访问该端口

### ipsets 说明

    blacklist
      临时添加
    pravite_addr
      私网地址段  类型为hash：net
          192.168.2.0/24
          10.0.0.0/8
    white_addr
      跳板机、管理主机
        121.22.28.50
        123.57.248.15
    zabbix-server
      zabbix服务器地址
        202.63.165.106

    注意ipset类型有一下类型：
    hash:ip
    只能保存ip地址或ip/mask，不能是网段
    hash:net
    只能保持为网段形式例如192.168.2.0/24
    hash:mac
    只能保持mac地址

### 系统富规则设置策略

    富规则说明
      php-fpm开放私网段
      rule family="ipv4" source ipset="pravite_addr" service name="php-fpm" log level="warning" accept

      ssh只针对白名单开放
      rule family="ipv4" source ipset="white_addr" service name="ssh" log level="warning" accept

      dhcp只针对私网开放
      rule family="ipv4" source ipset="pravite_addr" service name="dhcp" log level="warning" accept

      zabbix-agent服务仅与zabbix-server通信
      rule family="ipv4" source ipset="zabbix_server" service name="zabbix-agent" log level="warning" accept

      dell管理主机管理服务仅与白名单通信
      rule family="ipv4" source ipset="white_addr" service name="DellOpenManage" log level="warning" accept

      mysql服务仅与私网地址段通信
      rule family="ipv4" source ipset="pravite_addr" service name="mysql" log level="warning" accept

    需要自定义规则

      rule family="ipv4" source address="IPADDRESS" service name="SERVICE" log level="warning" accept|jecect
      大写换成响应内容，最后动作指定拒绝还是允许


## 系统下载的配置文件及目录说明

    ipset目录下为自定义的ipset

    service目录为自定的服务
        dell主机管理
        php-fpm
        zabbix服务

    zones
        区域配置信息
            开放的端口和服务
            配置的富规则

    firewalld.conf
        修改了默认区域
        开启白名单控制，只能是白名单的软件和user才能修改防火墙

    lockdown-whitelist.xml
        防火墙锁定白名单配置文件，将firewall-cmd添加进去。

## firewalld配置文件目录结构

	/etc/firewalld/
	├── firewalld.conf
	├── lockdown-whitelist.xml
	├── icmptypes
	├── ipsets
	│   ├── blacklist.xml
	│   ├── pravite_addr.xml
	│   ├── white_addr.xml
	│   ├── zabbix_server.xml
	├── services
	│   ├── DellOpenManage.xml
	│   ├── php-fpm.xml
	│   ├── zabbix-agent.xml
	└── zones
	    ├── drop.xml
