---
title: dockerfile构建自己的应用
date: 2020-02-01 11:45:08
tags: 
  - docker 
  - springboot
  - gradle
---
使用docker构建自己的应用
<!--more-->
# 引言
  自从docker入门以后,一发不可收拾,越学习越感觉有趣,本文记录一下在学习dockerfile构建自己应用遇到的坑以及学习心得  
> # Talk is cheap, show me the code  

# 环境
  使用gradle+jdk11编译及打包springboot项目,然后使用docker制作镜像  

# 项目
  springboot,作为现在最流行的微服务基础框架,我相信大家已经非常非常熟悉了,即使没有使用过,肯定听说过.一般比较通用的创建方式是使用springboot官网提供的[创建工具](https://start.spring.io/)进行创建,如果你使用intellij idea那么也可以在创建的时候使用spring initializr,这个和使用官方提供的创建工具是一回事  

# 命令  

  1. FROM  
    该指令是dockerfile的起始命令,是必须的,而且必须是第一个,作用是以一个镜像为基础,在该镜像上进行定制.  
        > `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]  `  
        > `FROM [--platform=<platform>] <image> [AS <name>]`  
        > `FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`  
    
  2. ARG  
    该指令是声明一个变量: 
      ```
      ARG <name>[=<default value>]
      ```
      如果想覆盖默认值,可以在执行 `docker build` 命令时候指定 `--build-arg <name>=<value>`


      ps:在FROM之前声明的ARG在构建阶段之外，因此，FROM之后的任何指令都不能使用它。要使用在第一个FROM之前声明的ARG的默认值，请使用ARG指令，且在构建阶段内部不带值  
        ```
          ARG VERSION=latest  
          FROM busybox:$VERSION  
          ARG VERSION  
          RUN echo $VERSION > image_version  
        ```

  3. LABEL  
    该指令添加 `metadata` 到镜像之中,格式为键值对,如: 
       > `LABEL maintainer="dengbojing@qq.com"   `

        ps: 这里正好用 `maintainer` 字段来说明一下,官方已经将`MAINTAINER` 这个命令废弃,改用 `LABEL` 代替   

  4. RUN  
    该指令有两种格式  

        - shell格式, `RUN <command>` command将会在shell中执行,对于linux系统shell为/bin/bash, 对于windows系统shell为 cmd /S /D   

        - exec格式, `RUN ["executable", "param1", "param2"]` , 注意该指令不会进行shell处理,比如 `RUN ["echo", "$home"]` 是不会对 `$home` 处理的,你需要自己指定shell,`RUN ["sh", "-c", "echo $home"]`.   

        该命令执行一次会产生一层layer,所以应该尽量合并 `RUN` 后面 `command` 比如:  
        > RUN && apt-get update \
            && apt-get install -y $buildDeps  
  5. CMD 
    该指令主要作用是为容器提供一个默认的执行命令,三种格式:  
    
        - exec格式, `CMD ["executable","param1","param2"]` ,该格式是官方推荐首选格式,同样该格式也不会进行shell处理.  

        - 参数格式: `CMD ["param1","param2"]`, 该格式需要指定 `ENTRYPOINT` ,作为 `ENTRYPOINT` 的参数  

        - shell格式, `CMD command param1 param2`   

        ps: 该指令在文件中只有一个,如果有多个那么只有最后一个 `CMD` 才会起作用,如果在`docker run` 后面指定了其他命令或者参数会覆盖 `CMD` 后面的命令或者参数
  6. ENTRYPOINT  
    该指令主要作用是为容器提供一个每次都执行的命令,该命令有两种格式:  
      > exec格式: `ENTRYPOINT ["executable", "param1", "param2"]` ,官方推荐
      > shell格式: ` ENTRYPOINT command param1 param2`  

      ps: 同 `CMD` 指令如果有多个 `ENTRYPOINT` 也只有最有一个起作用,如果想覆盖默认的`ENTRYPOINT` 可以使用: `docker run --entrypoint`;不同点在于,该指令可以直接在 `docker run` 后面跟参数,而 `CMD ` 指令不可以.  

  7. COPY  
    顾名思义,该指令主要作用就是--复制,两种格式:  
        > `COPY [--chown=<user>:<group>] <src>... <dest>`  
        > `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]`  

        ps: 1. 该指令的 `--chown` 只有linux才有,windows和linux权限管理不一样;另外该指令还支持通配符  
            2. 该指令只会cp源目标下文件.  
            3. 如果目标目录没有/, 则会将目标地址当成一个文件  
            4. 如果目标目录不存在, 则会创建目标目录的所有层级的目录   


  8. EXPOSE  
    该指令暴露一个容器内部端口到外部,格式为:  

        > `EXPOSE <port> [<port>/<protocol>...]`  
        ps: 该指令并非真正暴露一个端口供外部使用,只是一种说明,说明容器内部哪些端口可以被访问,在启动时候需要使用 `docker run -p <out port>:<expose port>`   


  9. WORKDIR  
    该指令指定工作目录,相当于`shell`命令里面的 `cd`,指定工作目录之后,后续的`COPY`,  `RUN`, `CMD`, `ENTRYPOINT` 等命令都是在当前目录下完成  


# USAGE  &  CONTEXT  
  当执行 `docker build` 的时候需要一个 `Dockerfile` 文件和一个 `context`, `context` 的涵义是指包含一些列文件的`PATH`或者`URL`,这里的 `PATH` 代表了文件系统的目录, `URL` 则代表了 `Git` 仓库地址.  

  这里文件系统的目录是包含下面的子目录以及子目录中的文件,也就是 `whole directory ` 都会被作为上下文发送给 ` docker daemon`.  

  `docker build ` 构建的时候不是在CLI(命令行界面)构建而是把 `当前目录` 作为 `context` 发送给 ` docker daemon`, 也就是docker的守护进程,所以说不能发送过大的目录,特别是不要在根目录执行 `docker build`, 官方推荐是使用一个空目录作为 `context` 来存放 `Dockerfile` ,仅仅添加 `Dockerfile` 需要的文件.  

  这里遇到一些问题,执行 `docker build` 命令的时候会将当前目录作为 `context` 发送给守护进程, 但是 `Dockerfile` 不能直接使用这些文件,官方说明为:  
  > To use a file in the build context, the Dockerfile refers to the file specified in an instruction, for example, a COPY instruction  

  翻译过来就是--`要使用 context 中使用某个文件, Dockerfile 指定一个命令来引用这个文件,例如: COPY 命令`, 换句话说,就是这些文件发送给守护进程,但是不能直接使用,得通过命令来使用(后面会说明碰到的问题).  


  
  


# 制作  

## 学习了dockerfile和指令之后,我想到应该有两种方式制作镜像  
- 方法一: 使用gradle构建项目,然后在使用dockerfile把jar包制作成镜像: 这种方法简单,但是感觉没什么意义啊,不过随后我还真的在[springboot官方指导](https://spring.io/guides/gs/spring-boot-docker/)下找到了这个方法.  
  1. 第一步,执行gradle构建项目gradle build -x test  
  2. 第二步,编写dockerfile  

        ```
            FROM openjdk:11
            LABEL maintainer="dengbojing@qq.com"  
            ARG JAR_FILE=build/libs/*.jar
            COPY ${JAR_FILE} app.jar
            EXPOSE 8090
            ENTRYPOINT ["java","-jar","/app.jar"]
        ```
  3. ` docker build -t dengbojing/gateway .` 点代表把当前目录作为`context`发送给`dockerdeamon`

- 方法二: 把方法一的第一步放在Dockerfile里面,这样就少执行一步命令,
  1. 进入项目目录,新建一个空白的 `Dockerfile` 文件,填写如下内容:  
        ```
        FROM openjdk:11  
        LABEL maintainer="dengbojing@qq.com"  
        COPY . .
        RUN ./gradlew build -x test  
        EXPOSE 8090  
        ENTRYPOINT ["java","-jar","build/libs/service-gateway-0.0.1-SNAPSHOT.jar"]  
        ```

        ps: 第一次写命令时候不了解 `Dockerfile` 和 `context` 的工作原理,觉得将当前工作目录发送给`docker daemon` 就能直接使用了,没有写`COPY . . `, 结果就是怎么都运行不过去,找不到`gradlew` 文件.后面 `Google` 之,看到这种写法,一脸懵,后来请教群里大神,加上仔细阅读文档,最终解惑.  
  2. 这种方法有一个弊端,就是构建之后的镜像会比较大,因为 `gradle` 构建项目阶段所需要的额外的文件最终也被添加到镜像中了, 所以官方提供了多阶段构建. 例: 
        ```
        FROM openjdk:11 AS build  
        LABEL maintainer="dengbojing@qq.com"  
        COPY . .
        RUN ./gradlew build -x test  

        FROM openjdk:11 AS final
        WORKDIR /app
        COPY --from=build build/libs/service-gateway-0.0.1-SNAPSHOT.jar app.jar
        EXPOSE 8090  
        ENTRYPOINT ["java","-jar","app.jar"] 
        ```

        可以对比一下两种不同方式构建的镜像最后的大小, 如下图: ![compare-v1-v2.png](https://i.loli.net/2020/08/04/6MwQ8Vtqpye1Wdn.png)  
        可以看到,v1是通过非多阶段构建的,构建之后有1.16g大小,而通过多阶段构建,抛弃了 `gradle` 文件,只留下需要的项目jar包, 只有652M,好处显而易见.   
        ps: 如果还想那个精简,那么可以使用 `jre` 而非 `jdk`; 我这里是使用的自己的一个`spring-cloud-gateway`项目进行学习的.  
- 方法三: 以上的方法,是我直观能想到的方法,但是通过学习,找到了更简便的方法,那就是 `gradle插件` ,编写 `gradle构建脚本` ,生成 `docker` 镜像, 具体文档, 点击[这里](https://bmuschko.github.io/gradle-docker-plugin/#spring_boot_application_plugin)  
    
    
     
    
# 后记
  目前方法三还处于理论阶段,文档是看懂了,但是没有实质性的操作过.而且在项目构建过程中涉及到网络通信, `spring-cloud` 所有的项目都应该在注册中心注册, 我采用的 `zookeeper` 作为注册中心和配置中心, 这就涉及到了两个容器之间相互通信, 目前还没有学会, 目前做法是在宿主机启动 `zookeeper`, 然后找到 `docker` 虚拟网卡, 找到宿主机相对于 `docker` 的 `ip address` , 将镜像里面的 zk 地址改为宿主机相对于容器的ip, 这种方法很不容器化, 所以接着学习, 学会很容器化的方式方法.  
  > 骐骥一跃,不能十步;驽马十驾,功在不舍.

    
    
