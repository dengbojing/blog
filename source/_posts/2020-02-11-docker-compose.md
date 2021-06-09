---
title: docker-compose
date: 2020-02-11 17:36:28
tags: 
  - docker
  - gradle
  - springboot
---
what? 扫把独立日? 
<!--more-->
# 引言  
使用gradle插件构建镜像和docker-compose连接容器
> # Talk is cheap, show me the code

# 准备  
打开docker远程访问端口,`docker desktop for windows` GUI 界面有一个`expose daemon on tcp://localhost:2375`, 只需要勾选,然后重启就行;当然这个是提供一个loopback的访问,实际上你并没有真正的暴露一个局域网或者广域网访问的端口,不过对于`docker-gradle` 使用已经足够,如果非要提供一个远程访问端口请参考[微软文档](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon).  
操作很简单就是在 `C:\ProgramData\Docker\config\daemon.json` 添加一行`"hosts": ["tcp://0.0.0.0:2375"]`(ps:需要关闭`expose daemon on tcp://localhost:2375`), 这里需要注意的就是你 `docker desktop for windows` 是以windows container启动的而不是linux container 方式启动, 不然不起作用, 如果非要以linux container方式启动并且提供外网访问, 也不是没有办法具体操作参考[这个issue](https://github.com/docker/for-win/issues/314)(ps:未实验)

# Gradle插件构建镜像    
1. 在 `build.gradle` 中添加 `docker-plugin` 插件,然后编写脚本具体文档参考[这里](https://bmuschko.github.io/gradle-docker-plugin/#remote_api_plugin), 最终 `build.gradle` 就是像下面这样:  
    ```
    plugins {
        ...
        id 'com.bmuschko.docker-spring-boot-application' version '6.1.3'
        ...
    }
    dependencies{...}
    docker {
        springBootApplication {
            baseImage = 'openjdk:11'
            ports = [8090]
            maintainer = 'dengbojing@qq.com'
            images = ['dengbojing/gateway:v3']
            jvmArgs = ['-Dspring.profiles.active=production', '-Xmx2048m']
            mainClassName = 'com.yichen.ServiceGatewayApplication'
        }
    }
    ```
    `baseImage` 基于哪个基础镜像构建.  
    `ports` 需要暴露的端口.  
    `maintainer` 对应 `dockerfile` 中已经废弃的 `maintainer` 命令, 添加一些元信息.  
    `images`  构建出来的额镜像名称.  
    `jvmArgs` 对应 `dockerfile` `ENTRYPOINT` 命令中的启动参数.  
    `mainClassName`  对应 `dockerfile` `ENTRYPOINT` 命令中的启动类.    

2. 执行 `gradle dockerBuildImage`, 该命令就会使用 `docker -H tcp://127.0.0.1:2375 build` 来构建镜像, 所以要先开启 `2375端口`;  
 当然该插件也能提供远程构建,具体请看[官方文档](https://bmuschko.github.io/gradle-docker-plugin/#remote_api_plugin);   
 如果使用开发工具可以在开发工具`gradle`插件里面找到对应的执行的`task`;   
 然后会在 `${classpath}/build/docker` 下面看到生成的 `dockerfile`,这里并不是打 `jar` 包的方式, 而是用完整的`lib` 和 `classes` 制作镜像,然后用`java -cp` 指定设置 `classpath` 然后启动脚本写的 `mianClass`;  
 此时执行 `docker images` 就可以看到制作的镜像.  

3. 插件提供了4个 `task`, 分别是 `dockerPushImage`--推送镜像到镜像仓库,需要指定用户名密码,具体参看[官方文档](https://bmuschko.github.io/gradle-docker-plugin/#remote_api_plugin) , `dockerBuildImage`--构建镜像 , `dockerCreateDockerfile`--创建`dockerfile` , `dockerSyncBuildContext`--将代码同步到`docker context` , 前面的 `task` 总是依赖后面的 `task`.


# Docker-compose介绍 & 简单使用  

1. `docker-compose` 是官方提供的编排项目工具, 主要是应对***单机***多容器之间连接通信.
2. 使用 `docker-compose.yml` 作为模版文件  
3. 模版文件开头使用 `version` 来指定 `docker-compose` 文件格式,目前最新版本为3.7,具体对照关系可以参考[这里](https://docs.docker.com/compose/compose-file/)  
4. 一个简单的 `docker-compose.yml` 示例:
```yaml  
version: '3'
services: 
    web: 
        build: .
        image: dengbojing/gateway:v4
        ports: 
            - "8090:8090"
        networks: 
            - gateway
    zookeeper: 
        image: zookeeper
        networks: 
            - gateway
networks: 
    gateway: 
```
`version`: compose的版本号,具体对应关系可以查看[文档](https://docs.docker.com/compose/compose-file/)  
`services`: 需要启动的服务,一个服务对应一个容器,也就是说在执行该文件之后,会多出两个容器  
`web`,`zookeeper`: 服务名称, 最终创建的容器名称会以启动时候指定的 `${project_name}` 加上服务的名称为完整的容器名称
`build`: 指定构建的 `docker-context` 和 `dockerfile`, 此处都在当前目录; 详细指定格式为:
```yaml
...
build: 
    context: .
    dockfile: ./dockerfile
... 
```
ps: 这里可以指定 `docker-gradle`生成的 `dockerfile` 作为构建脚本.   
`image: ` 指定构建完成之后的镜像名称,也就说执行完该文件会多一个以 `dengbojing/gateway:v4` 为名称的镜像; 下面的`image`为以`zookeeper`镜像为基础创建一个容器;  
`ports`: 对外暴露的端口  

5. `networks`: 定义一个网络, 然后在 `services` 中使用, 此时 `web` 服务容器就可以通过下面的服务名--`zookeeper` 来访问下面的服务容器, 例:  
```yaml
spring:
  cloud:
    zookeeper:
      connect-string: zookeeper:2181
```
ps: `gateway` 项目为`spring-cloud-gateway` 项目, 使用了 `zookeeper` 作为注册中心和配置中心, 所以需要访问 `zookeeper`.  
6. 写到这里,简单的使用 `docker-compose` 就完成了, 不难发现对于微服务来说这种方式是非常有用的, 我们可以把我们几个服务包装成一个 `docker-compose` 项目,然后进行部署,它还支持从一个镜像启动多个容器实例--毕竟容器实例就是类的对象,有对个对象当然是没问题的; 这样以来那么一个文件里面就可以包含多个相同功能的服务,一个注册中心,一个配置中心,完美实现了集群功能.   
当然在实际生产过程中,注册中心和配置中心肯定在不同的服务器上, 所以这种方式有一定的局限性.
 # 后记
简单的学习了一下 `docker-compose` , 其中有很多配置和命令之后实战的时候才会真的用到. 怎么说呢:    
> 路漫漫其修远兮, 吾将上下而求索



    
