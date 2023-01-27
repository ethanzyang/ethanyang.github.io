---
title: "swap 跟 fstab"
date: 2018-12-10 12:01:00
slug: "swap-and-fstab"
tags: ["swap", "fstab", "linux"]
categories: [Dev]
draft: false
---

## 1 swap
如果系统的物理内存用光了，则会用到swap。系统就会跑得很慢，但仍能运行;如果Swap空间用光了，那么系统就会发生错误。通常会出现“application is out of memory”的错误，严重时会造成服务进程的死锁。所以要高度重视。

#### 1.1 swap空间大小:
通常情况下，Swap空间应大于或等于物理内存的大小，最小不应小于64M，通常Swap空间的大小应是物理内存的2-2.5倍。但根据不同的应用，应有不同的配置：如果是小的桌面系统，则只需要较小的Swap空间，而大的服务器系统则视情况不同需要不同大小的Swap空间。特别是数据库服务器和Web服务器，随着访问量的增加，对Swap空间的要求也会增加，具体配置参见各服务器产品的说明。
#### 1.2 
Swap分区的数量对性能也有很大的影响。因为Swap交换的操作是磁盘IO的操作，如果有多个Swap交换区，Swap空间的分配会以轮流的方式操作于 所有的Swap，这样会大大均衡IO的负载，加快Swap交换的速度。如果只有一个交换区，所有的交换操作会使交换区变得很忙，使系统大多数时间处于等待 状态，效率很低。用性能监视工具就会发现，此时的CPU并不很忙，而系统却慢。这说明，瓶颈在IO上，依靠提高CPU的速度是解决不了问题的。
#### 1.3 添加 swap 空间
```shell
复制代码
# 查看当前内存
free -g

# 查看交换分区使用情况
swapon -s

# 创建一个分区添加交换文件，创建交换空间，然后启动新增的交换空间(1G大小)
dd if=/dev/zero of=/opt/swap bs=1024 count=1024000   
dd if=/dev/zero of=/opt/swap bs=1024 count=2048000
/sbin/mkswap /opt/swap
/sbin/swapon /opt/swap
## 报错 不安全的权限 0644，建议使用 0600
chmod 0600 /opt/swap
# 再次尝试
# 报错 swapon 失败: 设备或资源忙
/sbin/swapoff /opt/swap
/sbin/swapon /opt/swap
# 再次查看内存情况
free -g

修改/etc/fstab,使新加的2G交换空间在系统重新启动后自动生效
echo "/opt/swap swap swap defaults 0 0" >>/etc/fstab
```

#### 1.4 释放 swap 空间
物理内存接近饱和时，系统会自动将不常用的内存文件转储到SWAP中，但SWAP使用率达30%的时候对系统性能可能有一定影响。
```shell
sync                         # 先执行下同步
swapoff -a                   # 关闭swap分区
swapon -a                    # 开启swap分区
swapoff -a && swapon -a      # 刷新swap空间，即将SWAP里的数据转储回内存，并清空SWAP里的数据。刷新原理就是把swap关闭后再重启。
```
#### 1.5 常用命令
```shell
free -m  #查看内存和虚拟内存
```
> *注意 docker 容器中使用时候需要容器具有 SYS_ADMIN 权限*

## 2 /etc/fstab
在linux中常常用mount命令把硬盘分区或者光盘挂载到文件系统中。/etc/fstab就是在开机引导的时候自动挂载到linux的文件系统。
#### 2.1 属性详解
` /dev/device   mountpoint   type   rules   0   order `

/dev/device|mountpoint|type|rules|order|0
:--:|:--:|:--:|:--:|:--:|:--:
需要挂载的设备|挂载点|文件系统类形|挂载时的规则|dump(系统备份工具)|启动时fsck检查的顺序
**`rules`参数取值**
* auto 开机自动挂载 
* default 按照大多数永久文件系统的缺省值设置挂载定义
* noauto 开机不自动挂载
* nouser 只有超级用户可以挂载
* ro 按只读权限挂载
* rw 按可读可写权限挂载
* user 任何用户都可以挂载