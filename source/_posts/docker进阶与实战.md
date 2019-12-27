---
title: docker进阶与实战
date: 2019-12-27 23:56:54
tags: book docker
categories: Docker
---


###  docker 简介

#### docker 架构图




#### 功能与组件

1. docker 客户端
2. docker daemon
3. docker 容器
4. docker 镜像
5. registry

> docker 客户端

    docker 是一个典型的C/S 架构的应用程序


> docker daemon

    docker daemon 也可以被理解为Docker Server .另外人们常用 Docker Engine 直接描述 驱动整个docker功能的核心引擎
 
> docker 容器

    docker 通过libcontainer 实现对容器生命周期的管理,信息的设置和查询,以及监控和通信等功能.而容器也是对镜像的完美诠释,容器以镜像为基础,同时又为镜像提供了一个标准的和隔离的执行环境
    
    
 > docker 镜像
 
    如果说容器提供了一个完整的,隔离的运行环境,那么镜像则是这个运行环境的静态体现,是一个还没有运行起来的运行环境. 相对于传统虚拟化的ISO镜像,Docker镜像要轻量很多,它是一个可定制的rootfs.
    
 
 > Registry

    Registry 是一个存放镜像的仓库,它通常被部署在互联网服务器或者云端
    
    Docker 公司的叫Docker Hub
    
    
    
 #### docker 安装
 
* ubuntu

    sudo apt-get install docker.io
    
 
 #### Doker 与 虚拟机间的联系
    容器与虚拟机是互补.虚拟机是用来进行硬件资源划分的完美解决方案,它利用硬件虚拟化技术,会同时通过一个hypervisor 层来实现对资源的彻底隔离;而容器则是操作系统级别的虚拟化,利用的是内核的Cgroup和Namespace特性,此功能完全通过软件来实现,仅仅是进程本身就可以与其他进程隔离开,不需要任何辅助
    Docker 容器与主机共享操作系统内核,不同的容器之间可以共享部分系统资源
    
    
    
    
### 关于容器技术

    容器技术,又称为容器虚拟化,之所以欢迎是因为它已经集成到Linux内核中
    
 
 
 #### 容器技术的特性
 
 * Namespace 
 
         Namespace又称为命名空间,它主要做访问隔离.其原理是针对一类资源进行抽象,并将其封装在一起提供给一个容器使用
         
 * Cgroup

        Cgroup 是control group 的简称,又称为控制组,它主要是做资源控制.其原理是将一组进程放在一个控制组里,通过给这个控制组分配指定的可用资源,达到控制一组进程的可用资源的目的
        
        
 #### 容器的组成
 
    Container = cgroup + namespace + rootfs + 容器引擎
    
 1. Cgroup 资源控制
 2. Namespace 访问控制
 3. roottfs 文件系统隔离
 4. 容器引擎 生命周期控制


  ##### Cgroup
  
* Cgroup 是control group 的简写 用于限制和隔离一组进程对系统资源的使用,也就是做资源Qos,这些资源主要包括CPU,内存,block I/O 和网络宽带

 > Cgroup 中实现的子系统及其作用


1. devices: 设备权限控制
2. cpuset 分配指定CPU 和 内存节点
3. cpu 控制CPU占用率
4. cpuacct 统计CPU 使用情况
5. memory 限制内存的使用上限
6. freezer 冻结Cgroup中的进程
7. net_cls 配合tc限制网络宽带
8. net_prio 设置进程的网络流量优先级
9. huge_tlb 设置进程的网络流量优先级
10. perf_event 允许HugeTLB工具基于Cgroup 分组做性能监控



    
    
  
