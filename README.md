# ELK_docker
docker 安装部署elk  并集成springboot项目


## ELK原理自行百度
* ELK是三个开源软件的缩写，分别表示：Elasticsearch , Logstash, Kibana , 它们都是开源软件。新增了一个FileBeat，它是一个轻量级的日志收集处理工具(Agent)，Filebeat占用资源少，适合于在各个服务器上搜集日志后传输给Logstash，官方也推荐此工具。

## ELK

### WINDOWS本地下搭建ELK(7.XX版本):---------------最后会报一个错（修改jvm参数之类的，暂时不知道怎么解决（linux路径））
* 官网教程：（本地各种问题，采用docker）
  https://www.elastic.co/cn/downloads/elasticsearch
  https://www.cnblogs.com/Wolfmanlq/p/5976246.html
* (1)Es7.xx安装：(因为上边是2.xx版本，安装命令不一样)
* ==>启动命令：elasticsearch.bat
* ==>报错修改配置:
org.elasticsearch.bootstrap.StartupException: ElasticsearchException[X-Pack is not supported and Mac..........
（在config/elasticsearch.yml添加一条配置：xpack.ml.enabled: false）
https://blog.csdn.net/fanrenxiang/article/details/81358332
==>启动成功：

*安装head插件(es5以上：)


### 我们在docker容器中搭建ELK:(两个方法，步骤四建议采用)

* (1)Win10安装docker：
https://www.cnblogs.com/5bug/p/8506085.html
安装之前确保电脑任务管理器，已启用虚拟化（否则重启电脑进入bios界面修改参数配置，不同电脑不同快捷键，自行百度）

![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%871.png)

* （2）安装kitematic（运行容器的图形化界面）
https://jingyan.baidu.com/article/fcb5aff768d8eeedaa4a71f8.html
![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%872.png)


* （3）安装部署ELK：（一次性安装，快速）
https://www.cnblogs.com/soar1688/p/6849183.html
1.在power-shell界面执行命令：docker pull sebp/elk 拉取elk镜像  
2.docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -e ES_MIN_MEM=128m  -e ES_MAX_MEM=1024m -it --name elk sebp/elk 将镜像运行为容器，由于我本机内存不符合安装要求，为了保证ELK能够正常运行，加了-e参数限制使用最小内存及最大内存 ----等待启动。。。

3.在浏览器输入 localhost:5601  看kibana启动情况
![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%873.png)

 
 命令使用：
 ![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%874.png)
 ![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%875.png)
Linux 和docker 常用命令：
https://www.cnblogs.com/mq0036/p/8520605.html
Docker常用命令（全）
https://www.cnblogs.com/cblogs/p/dockerCommand.html
Linux常用命令（全）
https://www.cnblogs.com/yjd_hycf_space/p/7730690.html

### （4）、Docker安装elk并集成springboot（单独拉取各个镜像安装）-------我采用此方法成功搭建！！

* 参考连接：https://www.cnblogs.com/sxdcgaq8080/p/10442696.html

** Logstash与springboot整合得所需依赖以及相关配置：

        <!-- logback 推送日志文件到logstash -->
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>6.0</version>
        </dependency>
        <!-- Your project must also directly depend on either logback-classic or logback-access.  For example: -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>

** 日志的配置文件logback-spring.xml

 <?xml version="1.0" encoding="UTF-8"?>
<!--该日志将日志级别不同的log信息保存到不同的文件中 -->
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />

    <springProperty scope="context" name="springAppName"
                    source="spring.application.name" />

    <!-- 日志在工程中的输出位置 -->
    <property name="LOG_FILE" value="${BUILD_FOLDER:-build}/${springAppName}" />

    <!-- 控制台的日志输出样式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />

    <!-- 控制台输出 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <!-- 日志输出编码 -->
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- 为logstash输出的JSON格式的Appender -->
    <appender name="logstash"
              class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>192.168.2.12:5044</destination>
        <!-- 日志输出编码 -->
        <encoder
                class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        "severity": "%level",
                        "service": "${springAppName:-}",
                        "trace": "%X{X-B3-TraceId:-}",
                        "span": "%X{X-B3-SpanId:-}",
                        "exportable": "%X{X-Span-Export:-}",
                        "pid": "${PID:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "rest": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>

    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="console" />
        <appender-ref ref="logstash" />
    </root>

</configuration>


官网查看：
https://github.com/logstash/logstash-logback-encoder
logstash 控制台不能输出日志，就查看配置文件 logstash.conf是不是对的（按步骤操作）
![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%876.png)

之前input里面的  tcp  我用的是beats所以会报一个错：（因为不同的输入方式，beats需要用Filebeats,如果没有安装配置就会报此错）
Caused by: org.logstash.beats.BeatsParser$InvalidFrameProtocolException: Invalid Frame Type, received: 64
at org.logstash.beats.BeatsParser.decode(BeatsParser.java:92) ~[logstash-input-beats-5.1.6.jar:?]

配置完后，logstash 控制台可以正常打印项目的日志了（json格式）

![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%877.png)
但是kibana还是不行，看不出来，之前的错误原因是，创建索引，没有按照logastash的配置来，

```
input{
        tcp {

                port => 5044
                codec => json_lines
        }
}
output{
        elasticsearch{
                hosts=>["http://192.168.92.130:9200"]
                index => "user-%{+YYYY.MM.dd}"
                }
        stdout{codec => rubydebug}
}
```
在kibana中创建索引，要按照这个格式创建  user-*,然后去discover中就能看到我的项目打印的日志了。（要和logstash.conf配置文件对应）
![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%878.png)
![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%879.png)
![image](https://github.com/17661977890/ELK_docker/blob/master/image/%E5%9B%BE%E7%89%880.png)





ELK原理介绍：
https://www.cnblogs.com/aresxin/p/8035137.html


Logstash工作原理：
Logstash事件处理有三个阶段：inputs → filters → outputs。是一个接收，处理，转发日志的工具。支持系统日志，webserver日志，错误日志，应用日志，总之包括所有可以抛出来的日志类型。
Input：输入数据到logstash。
Filters：数据中间处理，对数据进行操作。
Outputs：outputs是logstash处理管道的最末端组件。一个event可以在处理过程中经过多重输出，但是一旦所有的outputs都执行结束，这个event也就完成生命周期

input 就是在那收集数据，output就是输出到哪，filter代表一个过滤规则意思是什么内容会被收集。
上面的配置(tcp)是直接从项目输入到logastash,有的还会配redis beats等等



关于Beats了解：
https://www.cnblogs.com/cjsblog/p/9445792.html





ES集群配置：
9200作为Http协议，主要用于外部通讯
9300作为Tcp协议，jar之间就是通过tcp协议通讯
ES集群之间是通过9300进行通讯
