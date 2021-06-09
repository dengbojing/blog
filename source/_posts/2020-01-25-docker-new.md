---
title: docker入门学习
date: 2020-01-25 21:12:50
tags: docker
---
docker入门
<!--more-->
# 引言
  作于2020年春节大年初一晚,此时疫情真的是猛于虎,为了避免给我媳妇她们医院带来不必要的麻烦,老实在家呆着学学新知识,提升自我.  
> # Talk is cheap, show me the code
# 介绍  
  > Docker 使用 Google 公司推出的 Go 语言 进行开发实现,基于 Linux 内核的
   cgroup,namespace,以及 AUFS 类的 Union FS 等技术,对进程进行封装隔离,属于操作
   系统层面的虚拟化技术  

   个人理解就是一种虚拟化技术,类似之前接触过的lvm(linux virtual machine),但是有很大的不同,lvm是有一套完整的系统,虚拟出一套硬件系统和操作系统,然后在该系统之上又有很多的应用;而容器技术里面的应用直接是运行在容器宿主的内核之上,也没有虚拟除一套硬件,一套操作系统;所以容器很轻便小巧.  
   ps: 从以上描述可以看出,容器技术更适用于持续集成和devOps,有点 `java--Complie Once,Run Anywhere` 的意思,只需要打包一次,在任何地方,任何系统之上都能使用,不存在之前 `各种环境` (开发环境,测试环境,灰度环境,正式环境)导致的[这段代码在我机器上好好的啊,怎么可能有问题]
# 安装
  1. 注册docker hub 账号  

  2. 在[docker](https://docs.docker.com/)官方文档上找到对应操作系统的![docker.com.png](https://i.loli.net/2020/08/04/Gg3EerLRHzWOyb4.png)  
  ps: 文件还是有点大,建议使用迅雷等p2p工具下载.  

  3. 本人使用`windows专业版系统`,选择`docker desktop for windows`按照官网提示下载,完成之后不着急安装,此时需要先启动`hyper-v`,`docker desktop`是依赖`hyper-v`的,开启`hype-v`之后需要重启电脑,重启之后安装`docker desktop`,安装过程省略,下一步下一步即可.  
  ps: 如果你是`windows 家庭版`,请参考[这篇文章](https://www.jianshu.com/p/1329954aa329/)  

  4. 启动`docker desktop`,在系统托盘图标`右键->setting`,找到`resources->file sharing`,选择你要共享的盘符,因为`docker desktop`默认是使用`hyper-v`虚拟机,而`hyper-v`虚拟机默认的镜像地址都是在`C盘`,也就是系统盘,这会导致`c盘`不够用,可以直接在`resources->advanced`下面找到`disak image location`修改镜像位置,本人是没有修改,这里本人是开启了共享盘符,把之后镜像使用的存储空间映射到本地其他磁盘上,这样解决了大部分存储空间的问题,镜像存储就不管了    

# 入门  
  1. 打开`power shell`,这里最好是不要使用其他命令终端,本人之前学习时候使用cmd,使用cmd是有坑的,后面会讲到,输入`docker login`,按照提示输入之前在`docker hub`上注册的用户名,密码,第一次需要输入,后续可以不用,也可使用图像化界面登录,右键`docker desktop`系统托盘图标->login.  

  2. 登录之后我们就可以是用`docker search`命令来搜索我们感兴趣的仓库了,比如我们安装一个mysql吧,`docker search mysql`,可以看到有很多的`mysql`镜像,还有`star数`和`official`  
  ps: 根据是否是官方提供,可将镜像资源分为两类.一种是类似 `mysql` 这样的镜像,被称为基础镜像或根镜像.这些基础镜像由 Docker 公司创建、验证、支持、提供.这样的镜像往往使用单个单词作为名字.还有一种类型,比如 `bitnami/mysql` 镜像,它是由 Docker 的用户创建并维护的,往往带有用户名称前缀.  

  3. 使用`docker pull`拉去镜像,具体命令如下  

      > docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]   


      具体选项可以通过`docker pull --help` 来查看,  

      - docker registry地址: 默认是docker hub,一般不需要指定
      - 仓库名: 前文提到镜像名称分为两段,`<username>/<soft name>`如果不指定默认是library,即docker官方的镜像.
      - tag: 标签,如果不指定默认为latest,当然可以指定需要的版本,查询tag目前本人没有找到相应的命令,只能去[docker hub](https://hub.docker.com/)搜索镜像然后查看tag    

      综上所述,此时只需使用`docker pull mysql`就可以了  

   4. 可以是用`docker image ls`或者`docker images` 查看本地镜像![docker-images.png](https://i.loli.net/2020/08/04/dIoT2zwKU4BSQV6.png)  
   此时可以看到镜像的大小和在`docker hub`大小是不一样的,这是因为,在`docker hub`显示的是压缩大小,`docker hub`作为一个中央镜像仓库,关心的是流量大小,而不是体积大小,所以会尽可能的压缩镜像体积;而且在这里,显示的大小可能和实际占用磁盘空间大小还不一样,这是因为,镜像是有很多文件层组成,而文件层之间又存在继承,复用的关系,如果不同的镜像使用相同的基础层,那么只需要保存一份该基础层就可以了,不同镜像的其他层可以直接引用该基础层,可以使用`docker system df -v`来查看具体的占用情况,同时可以使用`docker system prune`来清理磁盘空间.  
   `docker image ls` 支持通配符,如 `docker images my*` 可以将所有my开头的镜像都列出  
   `docker image ls -q` 可以只显示镜像的 `ID` 
   此外还支持 `-f(--filter)` 过滤模式, 比如: `docker images -f since=mysql` 会列出在 `mysql`之后的所有镜像,将 `since` 换成 `before` 可以列出之前的镜像  

   5. 启动容器,每一个容器都是一个镜像实例,这个就像`java里面的oop思想`一样,一个镜像就是一个类,一个类的对象对应一个容器,java 里面使用`new`关键字实例化对象,这里我们使用`docker run`来启动一个容器,具体命令参数:  

      > docker run [OPTIONS] IMAGE [COMMAND] [ARG...]   

      此处常用的 `options` 有 `-p` 指定端口号, `-P` 指定端口号映射为本机随机端口号; `-e` 指定容器内部的环境变量;`-d` 指定为后台启动;`-i` 保持stdin标准的输入流打开状态,即使没有链接; `-t` 分配一个伪终端, `-v` 指定文件映射,即讲本地的一个目录或者文件映射到容器内部;具体启动命令:  

      > docker run -d -p 3305:3306 --privileged=true -e MYSQL_ROOT_PASSWORD=xxxx --name   mysqltest -v /f/mysql/data:/var/lib/mysql -v /f/mysql/conf/my.cnf:/etc/mysql/my.cnf -v //f/mysql/mysql-files:/var/lib mysql-files/ mysql:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci  

      解释一下,此时我们就把容器内部的3306端口映射为宿主机的3305端口,并且设置了mysql的配置,数据存储位置,并且进行了一系列的mysql参数设置,指定了 `msyql root` 的密码为xxx,--privileged是让容器内部的用户获取宿主机特殊权限,此处一定要把文件映射在你已经共享的盘符,不然会失败  

   6. 查看容器状态, `docker container ls -a` 或 `docker ps -a` 可以看到容器的状态,映射端口,名字等;
   使用 `docker stop <container name\id>` 停止容器, 使用 `docker start <container name\id>` 再次启动容器,注意第一次启动容器使用 `docker  run` 命令,该命令会创建并启动一个容器, 之后并不需要在run,只需要执行start就可以了,此外 `docker run` 执行时,如果指定镜像不存在,该命令会自动拉去默认仓库所匹配的镜像  
  
   
   7. 删除镜像和容器  
   删除镜像使用 `docker image rm` 或 `docker rmi` 加上镜像的 `ID` 来删除镜像  
   删除容器使用 `docker container rm` 或 `docker rm` 加上容器 `ID` 删除容器,可以添加 `-f` 强制删除一个正在运行的容器;  
   此外如果只是想清理未被容器使用的镜像可以使用 `docker image prune`  
   同理,可以使用  `docker container prune` 清理未运行的容器  
   如果需要删除的容器或者镜像太多,手动一个个输入就显得有点智障了,此时可以使用 `docker image ls -q` 来配合批量删除,比如:   
   `docker rmi $(docker image ls -q -f since=mysql)`  
   此时就会将mysql之前所有的镜像都删除  
   ps: 此处有之前说的 `cmd` 的坑,使用 `cmd` 会报错,改用 `power shell` 就不会
   

# 后记   

基础入门就这些,后续根据自身所学内容,会陆陆续续的记录更多的东西,比如如何创建一个docker容器啊,导入导出镜像啊,私有仓库的搭建等等.  

> 不积跬步，无以至千里；不积小流，无以成江海

--------------------------------------------------------------------------------------------
