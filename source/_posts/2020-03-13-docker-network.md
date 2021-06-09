---
title: docker-network
date: 2020-03-13 10:12:37
tags: [docker]
---

docker-network学习
<!--more-->

# 引言 
阶段性的记录一下docker学习,docker-network里面还有很多问题很搞明白,亟待解决.
> 以下操作都是基于`docker desktop for windows`的`linux container`模式下

# 介绍
1. `docker` 官方有5种网络模式,`none,bridge,macvlan,host,overlay`.  

    1.1 `none` 模式,参数 `--network=none`,无网络模式,这种模式一般很少用,官方说法是和自定义网络驱动时使用.  

    1.2 `birdge` 模式,参数 `--network ${bridge_name}`.该模式是默认模式,在安装`docker`会创建一个默认的`docker0`的linux网桥,如果启动容器时不指定网络,就会默认连接到`docker0`网桥   

    > `docker desktop for windows`看不见`docker0`,这是因为,`docker desktop for windows`实际上是把Docker装在Hyper-v虚拟机上,打开Hyper-v虚拟机管理,可以看到`DockerDesktopVM`这个虚拟机,Docker实际是运行在这个里面,而`docker0`就在这个里面,但是你如果连接这个虚拟机,发现连接不上.  

    1.3 `host` 模式,参数`--network=host`,该模式下,容器和宿主机共用一个网络,在指定`--privileged=true` 时候,如果你不小心修改了网络参数,那么就会造成不必要的麻烦,所以一般不推荐这中做法,上面说到`docker desktop for windows` 无法查看虚拟机里面的`docker0`网桥,此时如果你以`--network=host`启动一个容器,此时你就可以执行`ifconfig`看到`docker0`网桥.    

    > 另一个查看方式`docker run -it --rm --privileged --pid=host justincormack/nsenter1`,启动之后,执行 `ifconfig`,指定 `--pid=host` 参数就是说,让当前启动的容器可以看到宿主机上所有的进程.  
 
    1.4 `overlay` 模式, 参数 `network=container:${conatiner_id}`,该模式下,容器1和容器2共用一个网络.  

    1.5 `macvlan` 指定网络的mac地址.

# 实战`bridge`模式  
1. `docker network ls `查看网络,  
![docker-network-ls.png](https://i.loli.net/2020/08/04/otAxfl6vescaw5j.png)  
可以看到上面讲到的3种类型的网络,至于`overlay`呢,是需要依赖别的容器,所以取决与别的容器的网络模式,主要学习一下桥接模式的内容.  
2. `bridge` 网桥模式的原理, 当创建一个容器的时候,`Docker` 会创建两个网络模块,一个是在容器里面叫`eth0`, 另一个则在宿主机里面,名字为`vethxxxx`,`Docker` 这个网络模块桥街道容器里面的`eth0`.  
    2.1 首先我们以`bridge`启动一个`ubuntu` 容器,执行: `docker run  -ti --rm ubuntu:14.04 bash`, 启动一个一次性容器并进入  
    2.2  在容器中执行`ifconfig`, 看到如下图所示:   
    ![Docker-ifconfig.png](https://i.loli.net/2020/08/04/VYBJ4Imwhkv6Xcj.png)    
    `lo`: loopback 回环网络接口,也就是执行`localhost`或者`127.0.0.1`时候会走这个网络接口  
    `eth0`: 所有来自外部的流量都会通过这个网络接口  

    2.2 新开一个终端窗口,执行`docker run -it --rm --privileged --pid=host justincormack/nsenter1` , 进入容器之后,执行`ifconfig` 可以看到如图所示:  
    ![docker-host-ifconfig.png](https://i.loli.net/2020/08/04/uMSpOI8qr3cR7VH.png)
    其中一个`vethxxx`是桥接到上面`ubuntu` 容器的网络接口,另一个是当前这个容器的网络接口(因为当前网络没有指定网络模式,所以以默认桥接模式启动,所以也会给当前容器创建一个网络接口).  

3. 自定义网桥,之前使用`docker-compose` 创建了一个`compose`, 里面包含了一个`gateway`和一个`zookeeper`, 里面还定义了一个`network` ,让两个容器能够以容器名称相互访问, 了解了`docker network`之后,其实里面`network` 节点就是创建了一个`bridge` 网桥, 此时容器就可以通过名称相互访问, 官方名称叫 ` automatic service discovery` 服务自动发现(瞎鸡巴翻译的).
    3.1 执行`docker network create -d bridge my-network`创建一个`bridge` 类型网络, 执行`docker inspect my-network` 看一下里面都有什么,  如图:  
    ![docker-network-inspect-my-network.png](https://i.loli.net/2020/08/04/5F4cn2ah6VtJs9H.png)  
    可以看到,子网掩码是`172,19.0.0/16`,网关是`172,19.0.1`,可以使用`--subnet=192.168.0.0/16` 来指定子网掩码,另外还可以使用`--gateway=192.168.0.1` 指定网关, `--ip-range=192.168.2.0/25`指定ip范围  

    3.2 最好是指定子网掩码,免得网络冲突.如果不指定网关,会从地址范围内自动选择一个出来作为网关,目前测试结论默认是xxx.xxx.xxx.1.  

4. 使用`docker run --network=my-network` 来启动一个容器,执行ifconfig:  
![docker-run-network-my-network.png](https://i.loli.net/2020/08/04/uUfES7oBhPpYskj.png) 
可以看到此时ip地址为我们创建的桥接网络借口的ip地址范围  

5. 一个容器可以同时连接两个网络,使用`docker network connect ${networkid|networkname} ${containerid|containername}`, 如图:  
 ![docker-network-connect.png](https://i.loli.net/2020/08/04/9aZx65RWMFXbq1Y.png)    
会有两个网络接口一个`eth0`,另一个`eth1`, 这样该容器就可以同时访问两个网路  

6. `docker network disconnect ${networkid|networkname} ${containerid|containername}` 取消容器和网络的链接  

7. `--icc`高级参数, 如果网络该参数为禁止,则两个容器之间是无法访问的.  


# 后记  

网络这块真的是比较重要,也比较难的一块,还可以创建`overlay`类型的网络,有待研究.官方说法生产环境最好不要使用默认的`docker0`网桥,所以使用`docker-compose`管理容器还是一个比较好的方式.或者努力学习k8s吧.

> 积土成山,风雨兴焉;积水成,渊蛟龙生