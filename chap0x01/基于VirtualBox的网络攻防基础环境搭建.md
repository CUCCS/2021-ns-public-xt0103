# 基于VirtualBox的网络攻防基础环境搭建

## 一、实验目的

- 掌握VirtualBox虚拟机的安装与使用；

- 掌握VirtualBox的虚拟网络类型和按需配置；

- 掌握VirtualBox的虚拟硬盘多重加载；

## 二、实验环境

- VirtualBox虚拟机

- 攻击者主机（Attacker）：Kali Rolling 2109.2

- 网关（Gateway, GW）：Debian Buster

- 靶机（Victim）：From Sqli to shell / xp-sp3 / Kali

  


## 三、实验要求

- 虚拟硬盘配置成多重加载；

  ![多重加载](img/多重加载.jpg)



- 搭建满足如下拓扑图所示的虚拟机网络拓扑![示例网络拓扑](img/示例网络拓扑.jpg)



- 完成以下网络连通性测试；
  - [x] 靶机可以直接访问攻击者主机
  - [x] 攻击者主机无法直接访问靶机
  - [x] 网关可以直接访问攻击者主机和靶机
  - [x] 靶机的所有对外上下行流量必须经过网关
  - [x] 所有节点均可以访问互联网

## 四、实验过程

### 虚拟硬盘配置为多重加载

（以kali为例）

- 在虚拟介质管理中释放虚拟硬盘后，将类型改为多重加载。![释放](img/释放.jpg)



- 经过刚才释放，虚拟硬盘已经移除，我们需要重新添加。![多重加载kali](img/多重加载kali.jpg)



- 如上图所示，虚拟硬盘成功改为多重加载类型，其他的虚拟硬盘如上所示依次更改

  ![多重加载Debian](img/多重加载Debian.jpg)

  ![多重加载xp](img/多重加载xp.jpg)



### 准备实验所需虚拟机



攻击者主机：Kali-Attacker

网关：Gateway

网络一中靶机：Victim-XP-1, Victim-Kali-1

网络二中靶机：Victim-XP-2, Victim-Debian-2

![虚拟机准备](img/虚拟机准备.jpg)



### 配置虚拟机

#### 网关(Gateway)

- 网卡一：NAT网络

  为了使网关和攻击者主机在同一个局域网内，所以用NAT网络。![网关-网卡一](img/网关-网卡一.jpg)



- 网卡二：Host-only网络

  ![网关-网卡二](img/网关-网卡二.jpg)

- 网卡三：内部网络intnet1

  ![网关-网卡三](img/网关-网卡三.jpg)

- 网卡四：内部网络intnet2

  ![网关-网卡四](img/网关-网卡四.jpg)

  打开`/etc/network/interfaces`配置网卡

  ![配置网卡](img/配置网卡.jpg)

  查看各网卡地址：

  ![各网卡地址](img/各网卡地址.jpg)

#### 靶机Victim-XP-1

配置网卡一连接intnet1：

![xp1-网卡1](img/xp1-网卡1.jpg)

同时把XP系统控制芯片改为PCnet-Fast，否则在系统看不见网卡。



启动系统后，查看网络连接属性：![xp1网络连接属性](img/xp1网络连接属性.jpg)

默认网关地址为网卡设置的第三块网卡地址相同，连接正确。



检测xp1到网关的连通性：

`ping 172.16.111.1`可以连通。

![xp1连通性检测1](img/xp1连通性检测1.jpg)



#### 靶机Victim-Kali-1

配置网卡一连接intnet1：

![kali1-网卡1](img/kali1-网卡1.jpg)



测试kali 1与xp1的连通性:

`ping 172.16.111.102`可以连通

![kali1连通性检测](img/kali1连通性检测.jpg)



反向检测xp1到kali1的连通性：

`ping 172.16.111.136`可以连通

![xp1到kali1连通性](img/xp1到kali1连通性.jpg)

#### 靶机Victim-XP-2

配置网卡一连接intnet2：

![xp2-网卡1](img/xp2-网卡1.jpg)



查看网络连接属性：

![xp2网络连接属性](img/xp2网络连接属性.jpg)



#### 靶机Victim-Debian-2

配置网卡一连接intnet2：

![debian2-网卡1](img/debian2-网卡1.jpg)

同时把XP系统控制芯片改为PCnet-Fast，否则在系统看不见网卡。

#### 攻击者主机Kali-Attacker

配置网卡一：

![attacker-网卡一](img/attacker-网卡一.jpg)



### 检测连通性



1、靶机可以直接访问攻击者主机

在靶机xp2访问攻击者主机（10.0.2.15），可以访问：

![靶机访问攻击者](img/靶机访问攻击者.jpg)



2、攻击者主机无法直接访问靶机

攻击者访问靶机xp2（172.16.222.147），不可以访问：

![攻击者访问靶机](img/攻击者访问靶机.jpg)



3、网关可以直接访问攻击者主机和靶机

可以访问攻击者主机（10.0.2.15）：

![网关访问攻击者](img/网关访问攻击者.jpg)



可以访问靶机xp2（172.16.222.147）：

![网关访问xp2](img/网关访问xp2.jpg)



可以访问靶机xp1（172.16.111.102）：

![网关访问xp1](img/网关访问xp1.jpg)



可以访问kali1（172.16.111.136）：

![网关访问kali1](img/网关访问kali1.jpg)



可以访问debian2（172.16.222.120）：

![网关访问debian2](img/网关访问debian2.jpg)



4、靶机的所有对外上下行流量必须经过网关。

以xp2为例：

执行`tcpdump -i enp0s10 -n -w 20210909.1.pcap`进行抓包。

![抓包](img/抓包.jpg)



然后在xp2上进行网络访问，产生流量。



抓包停止后，将20210909.1.pcap文件通过执行`cp 20210909.1.pcap /root`保存到/root目录下。

在宿主机打开新的终端，ssh连接远程主机将pcap文件保存到本地桌面。

![宿主机保存pcap文件](img/宿主机保存pcap文件.jpg)



打开20210909.1.pcap文件对数据包进行分析

![pcap](img/pcap.jpg)

可以看到，当xp2（172.16.222.147）访问百度（220.181.38.150）时候，会通过网关（172.16.222.1）进行发送数据包，可知靶机的所有对外上下行流量必须经过网关。



5、所有节点均可访问互联网

xp1可以访问互联网：

![xp1访问互联网](img/xp1访问互联网.jpg)



kali1可以访问互联网：

![kali1访问互联网](img/kali1访问互联网.jpg)



xp2可以访问互联网：

![xp2访问互联网](img/xp2访问互联网.jpg)



debian2可以访问互联网：

![debian2访问互联网](img/debian2访问互联网.jpg)



攻击者可以访问互联网：

![攻击者访问互联网](img/攻击者访问互联网.jpg)



## 五、问题及解决方法

debian-gw ssh服务器拒绝了密码，不支持root用户连接，编辑/etc/ssh/sshd_config文件。将PermitRootLogin改为yes

![改ssh文件](img/改ssh文件.jpg)

![改ssh文件后](img/改ssh文件后.jpg)

然后重启ssh ：

```
systemctl restart sshd
```

## 六、参考资料

[Virtual Box多重加载](https://blog.csdn.net/jeanphorn/article/details/45056251)

[ssh服务拒绝密码](https://blog.csdn.net/weixin_30852451/article/details/102365633?utm_source=app&app_version=4.15.0&code=app_1562916241&uLinkId=usr1mkqgl919blen)

[实验详解视频](https://www.bilibili.com/video/BV16t4y1i7rz?p=12&spm_id_from=pageDriver)



