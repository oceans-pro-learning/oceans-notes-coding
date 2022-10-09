

# 一、前置安装
- VMware Fusion
- centos7
	- http://isoredirect.centos.org/centos/7/isos/x86_64/
	- 选择 CentOS-7-x86_64-Minimal-2009.iso
	- 将下载链接复制到迅雷进行下载
- item2或者royal tsx

# 二、配置网络和SSH

方便之后在mac上远程登录以方便复制粘贴命令，需要先配置网络和SSH。

![[Pasted image 20221009113446.png]]

```sh
ip addr # 发现找不到ip地址
```

![[Pasted image 20221009135736.png]]

```sh
cd /etc/sysconfig/netword-scripts
vi ifcfg-ens33
```

![[Pasted image 20221009135858.png]]

最后一行onboot=no改为yes

```sh
service network restart
ip addr
```

结果如下

![[Pasted image 20221009140136.png]]

ip地址即为192.168.185.2（注意是前面的这个地址）

```sh
ssh root@192.168.185.2
```


# 三、基础软件

```sh
ssh root@192.168.185.2 # 管理员登录
yum -y install wget # 下载
yum -y install net-tools # netstat之类的
yum -y install vim # 代替vi
```

ftp工具安装：[https://linuxhint.com/setup_proftpd_ftp_server_centos7/](https://linuxhint.com/setup_proftpd_ftp_server_centos7/)

# 四、新建用户
新建用户

```sh
useradd work  
passwd work
```

![[Pasted image 20221009141133.png]]


# 五、QA

## 如何解决Mac苹果上运行VMware Fusion虚拟机提示“未找到文件”

其实解决方法很简单，只要在删除VMware里=之前安装的已经无效的虚拟机，重启即可

1. 屏幕右上角菜单栏点击VMware小图标，在弹出的菜单中点击【虚拟机资源库】
2. 在【虚拟机资源库】列表中，右键删除这个虚拟机
3. 关闭重启VMware就可以了

## 如何解决 WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!

如果出现下面的警告

```text
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

在自己的mac电脑上

```sh
code ~/.ssh/known_hosts
```

删除 192.168.185.2 对应的那一行，重新执行ssh命令即可。