---
title: spring-boot+elk
date: 2023-01-22 22:24:58
tags: [java,spring-boot,elk]
---

spring boot 结合 ELK

<!--more-->
# elasticsearch
## install elasticsearch 


0. `apt-get upgrade`

1. `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`

2. `apt-get install apt-transport-https`

3. `apt-get update`

4. `apt-get install elasticsearch`

## configurate elasticsearch

1. `vim  /etc/elasticsearch/elasticsearch.yml`
```
  network.host: 0.0.0.0  
  port: 9200  
```
[其他配置](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html)

2. 启动: `systemctl start elasticsearch.service`

3. 测试: `curl -X GET "localhost:9200"`

# kibana

## install kibana

1. `apt-get install kibana`

## Configure Kibana

1. `vim etc/kibana/kibana.yml`
```
  server.port: 5601  
  server.host: "localhost"
  elasticsearch.hosts: ["http://localhost:9200"]
```
[其他配置](https://www.elastic.co/guide/en/kibana/7.17/get-started.html)

2. 启动: `systemctl start kibana`

3. 开机启动(可选): `systemctl enable kibana`

4. 开放防火墙, 浏览器测试访问 `http://localhost:5601`

# Logstash

## install logstash

0. 主要作用: input 收集, filter过滤, output输出

1. `apt-get install logstash`

2. 开机启动(可选): `systemctl enable logstash`

## Configure Logstash

1. `vim /etc/logstash/conf.d/logstash.conf` 
[其他配置](https://www.elastic.co/guide/en/logstash/7.17/introduction.html)


2. 配置说明: 以`beats`为例,下面使用`filebeat`
```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}" 
  }
}
```



# FileBeat(可选)

## install fileBeat

1. ```apt-get install filebeat```

## configure filebeat

1. ```vim /etc/filebeat/filebeat.yml```  
注释es配置,打开logstash配置

```
# output.elasticsearch:
   # Array of hosts to connect to.
   # hosts: ["localhost:9200"]
output.logstash
   hosts: ["localhost:5044"]

```
2. (可选)启动一个module(预设的一些日志处理模块): `filebeat modules enable system` system module在linux系统下会对/var/logs下面所有日志传输  
[其他配置](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html)

3. 加载日志: `filebeat setup -e`

4. 其他通用日志设置修改: 
```
- type: filestream

  # Unique ID among all inputs, an ID is required.
  id: my-filestream-id

  # Change to true to enable this input configuration.
  enabled: false

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /var/log/*.log
    #- c:\programdata\elasticsearch\logs\*
```
enable 设置为ture, paths设置为日志文件目录

5. 启动: `systemctl start filebeat`

6. 开机启动(可选): systemctl enable filebeat


# spring-boot

## add dependency

```
  implementation group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '7.2'
```

## configure logstash
input moule选择tcp, 其他不变, [详见文档](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-tcp.html)

```
input {
  tcp {
    id => "tcp_1"
    port => 9250
    mode => ["server"]
    codec => json_lines
  }
}
```


## set log file

logback.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <property name="LOGS" value="./logs" />

    <appender name="Console"
              class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %red(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %yellow(%C{1.}): %msg%n%throwable
            </Pattern>
        </layout>
    </appender>
    <appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:9250</destination>
        <!-- encoder is required -->
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
    </appender>

    <appender name="RollingFile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOGS}/spring-boot-logger.log</file>
        <encoder
                class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
        </encoder>

        <rollingPolicy
                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- rollover daily and when the file reaches 10 MegaBytes -->
            <fileNamePattern>${LOGS}/archived/spring-boot-logger-%d{yyyy-MM-dd}.%i.log
            </fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>

    <!-- LOG everything at INFO level -->
    <root level="info">
        <appender-ref ref="stash" />
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </root>

    <!-- LOG "com.lotus*" at TRACE level -->
    <logger name="com.lotus" level="trace" additivity="false">
        <appender-ref ref="stash" />
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </logger>

</configuration>

```