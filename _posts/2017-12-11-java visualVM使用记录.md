---
title: java visualVM 以及其他jvm性能监控工具使用
description: Null
categories:
 - others
photos:
tags:
- Blog
---

> 关于visualVM以及其他jvm性能监控工具使用的一些使用记录
![TITLE]({{ site.url }}/assets/images/201712/1211_top.png)

## visualVM远程连接jvm
- 添加JMX连接<br/>
关于jmx：JMX的全称为Java Management Extensions. <br/>顾名思义，是管理Java的一种扩展。这种机制可以方便的管理、监控正在运行中的Java程序。常用于管理线程，内存，日志Level，服务重启，系统环境等<br/>
1：确保jvm启动时的参数<br/>
```
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=xxxx <- jmx连接时的端口
-Dcom.sun.management.jmxremote.ssl=false <- ssl安全连接,默认为是
-Dcom.sun.management.jmxremote.authenticate=false <- 不需要认证
```
2：使用visualvm连接<br/>
先建立对应的远程连接，之后添加JMX连接<br/>

- 添加jstatd连接
1：修改$JAVA_HOME/jre/lib/security/java.policy中，在最后一行添加
```
permission java.security.AllPermission;
```
2：启动jstatd服务
```
jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.logCalls=true -J-Djava.rmi.server.hostname=139.199.182.196
```
3：添加jstatd连接，visualVM会自动连接

## jconsole
查询jvm的cpu&内存<br/>
需要jvm启动时加入命令<br/>
```
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=xxxx <- jmx连接时的端口
-Dcom.sun.management.jmxremote.ssl=false <- ssl安全连接,默认为是
-Dcom.sun.management.jmxremote.authenticate=false <- 不需要认证
```

## jstat
![TITLE]({{ site.url }}/assets/images/201712/jvmtools1.png)
从左到右代表值：<br/>
第一个sur的容量，第二个sur的容量，第一个sur已使用空间，第二个sur已使用空间<br/>
eden容量，eden已使用空间，old代容量，old代已使用空间<br/>
方法区大小，方法区使用大小，压缩类空间大小，压缩类空间似乎用大小<br/>
年轻代gc次数，年轻代gc时间，old代gc次数，old代gc时间，gc总时间<br/>

## jmap


## jstack
实例:发现某个进程cpu过高,想找到对应的线程,打印线程栈
1：先查找对应的进程`top`
1：再通过top查找对应的线程`top -H -p pid`
2：通过jstack打印对应线程栈信息`jstack pid`
遇到的问题:
```
10555: Unable to open socket file: target process not responding or HotSpot VM not loaded
The -F option can be used when the target process is not responding
```
