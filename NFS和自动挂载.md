NAS：网络附加`存储`		NFS	CIFS

SAN：存储区域`网络`		IP-SAN	FC-SAN

网线连接：IP-SAN

光纤连接：FC-SAN

文件系统级共享：	NFS	Samba(CIFS)

块设备共享：	ISCSI

## NFS

文件系统级共享	**Linux to Linux**

包：nfs-utils	rpc-bind

服务：nfs-server

### 服务器端

1. 修改配置文件 /etc/exports，在文件中添加需要共享的文件目录。

   ```shell
   [root@RHCE ~]# vim /etc/exports
   /share  192.168.1.0/24(rw)
   # 共享的目录（绝对路径 谁可以访问{IP，网段，主机}
   ```

2. 启动服务

   ```shell
   [root@RHCE ~]# exportfs -r
   # -r 让配置文件生效
   [root@RHCE ~]# exportfs -v
   /share	192.168.1.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,root_squash,no_all_squash)
   # -v 查看共享的详细信息
   [root@RHCE ~]# systemctl enable --now nfs-server
   ```

3. 设置防火墙

   ```shell
   [root@RHCE ~]# systemctl stop firewalld.service 
   [root@RHCE ~]# setenforce 0
   ```

### 客户端

1. 查看是否有共享

   ```shell
   [root@CentOS ~]# showmount -e 192.168.1.108
   Export list for 192.168.1.108:
   /share 192.168.1.0/24
   # -e 指定 IP 地址 查看有无共享
   ```

2. 挂载使用

   ```shell
   [root@CentOS ~]# mount -t nfs 192.168.1.108:/share /data/
   # -t 指定文件系统为 nfs  IP 地址，指定共享目录
   [root@CentOS ~]# df -h
   Filesystem            Size  Used Avail Use% Mounted on
   192.168.1.108:/share   20G  4.2G   16G  21% /data
   ```

3. 永久挂载

   ```shell 
   [root@CentOS data]# vim /etc/fstab 
   192.168.1.108:/share /data nfs defaults  0 0
   [root@CentOS data]# mount -a
   [root@CentOS data]# df -h
   ```

### 权限映射

| Client           | Server                |
| ---------------- | --------------------- |
| root ( UID = 0 ) | nobody( UID = 65534 ) |
| 普通用户         | 相同 UID              |

### autofs服务配置

> ?	为什么使用自动挂载
>
> !	可以优化 nfs  ，根据需要自动挂载 nfs 共享，不需要时自动卸载
>
> > 使用传统 NFS 
> >
> > 1. 消耗网络资源
> > 2. 启动的时候，费时间费资源

#### 需求

服务端：/share/user1

客户端：使用两级目录	/data/user1

这样就可以把 /etc/fstab 中的挂载删除

#### 流程

1. 安装软件包

2. 启动服务

3. 配置主配置文件 /etc/auto.master

4. 配置文件

   可直接编写空白的，也可编写模板文件 /etc/auto.misc，配置文件的文件名一定要对应主配置文件中定义的名称

5. 重启服务