#Arthas
## Arthas介绍

&emsp;&emsp;Arthas是一个线上监控诊断工具，通过全局视角实时查看应用load、内存、GC、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断，包括查看方法调用的出入参、异常、监测方法执行耗时，类加载信息等，大大提升线上问题排查效率。**Arthas旨在解决这些问题。开发人员可以在线解决生产问题。无需JVM重启，无需代码更改。Arthas作为观察者永远不会暂停正在运行的线程。**

官方参考文档：[Arthas官方文档](https://arthas.aliyun.com/doc/)

```sh
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.  
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-' 
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-. 
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----' 
```

## Arthas能做什么

0. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
1. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
2. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
3. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
4. 是否有一个全局视角来查看系统的运行状况？
5. 有什么办法可以监控到 JVM 的实时运行状态？
6. 怎么快速定位应用的热点，生成火焰图？
7. 怎样直接从 JVM 内查找某个类的实例？

## Arthas Install

使用`arthas-boot`（推荐）下载`arthas-boot.jar`JAR包

```shell
curl -O https://arthas.aliyun.com/arthas-boot.jar
```

检查下载`arthas-boot.jar`是否成功

```sh
[root@basic-beidou-test-bf7f95ddc-vbttv data]# ll
total 106093
-rw-r--r-- 1 root root    141899 9月  11 17:16 arthas-boot.jar
drwxr-xr-x 2 root root      4096 9月  11 17:16 arthas-output
drwxrwxrwx 4 root root      4096 6月  27 11:07 logs
-rw-r--r-- 1 root root 108485876 9月  11 14:16 ROOT.jar
```

通过使用`java -jar`的方式启动

```sh

[root@basic-beidou-test-bf7f95ddc-vbttv data]# java -jar arthas-boot.jar
[INFO] JAVA_HOME: /usr/local/jdk1.8.0_181/jre
[INFO] arthas-boot version: 3.7.2
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 1 /data/ROOT.jar
```

选择想要监控的Java进程

```sh
1
[INFO] arthas home: /root/.arthas/lib/3.7.2/arthas
[INFO] The target process already listen port 3658, skip attach.
[INFO] arthas-client connect 127.0.0.1 3658
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.  
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-' 
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-. 
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----' 

wiki       https://arthas.aliyun.com/doc
tutorials  https://arthas.aliyun.com/doc/arthas-tutorials.html
version    3.7.2
main_class
pid        1
time       2024-09-11 17:16:47

[arthas@1]$
```

## Arthas idea plugin

&emsp;&emsp;**Arthas命令生成插件**

&emsp;&emsp;直接去idea 插件仓库搜索`arthas idea`下载安装 ,也可以直接在idea 中安装插件

&emsp;&emsp;将光标放置在具体的类、字段、方法上面 右键选择需要执行的命令，部分会有窗口弹出、根据界面操作获取命令；部分直接获取命令复制到了剪切板 ，自己启动arthas 后粘贴命令即可执行。

## 命令列表

详细命令用法查阅官方文档：[命令列表](https://arthas.aliyun.com/doc/commands.html)

### 基础用法

#### sc

在Arthas的命令行界面中输入`sc`命令。这将列出所有已加载的类的基本信息。要查看类的成员变量信息，需同时使用`-d`和`-f`参数。

```sh
[arthas@1]$ sc com.edianyun.beidou.provider.service.*
com.edianyun.beidou.provider.service.ApplicationService
com.edianyun.beidou.provider.service.DepartmentService
com.edianyun.beidou.provider.service.EmployeeMessageService
com.edianyun.beidou.provider.service.SystemService
com.edianyun.beidou.provider.service.impl.ApplicationServiceImpl
com.edianyun.beidou.provider.service.impl.ApplicationServiceImpl$$EnhancerBySpringCGLIB$$23830009
com.edianyun.beidou.provider.service.impl.DepartmentServiceImpl
com.edianyun.beidou.provider.service.impl.DepartmentServiceImpl$$EnhancerBySpringCGLIB$$2b9096db
com.edianyun.beidou.provider.service.impl.DepartmentServiceImpl$$EnhancerBySpringCGLIB$$2b9096db$$FastClassBySpringCGLIB$$6a0d0f54
com.edianyun.beidou.provider.service.impl.DepartmentServiceImpl$$FastClassBySpringCGLIB$$7a583827
com.edianyun.beidou.provider.service.impl.DepartmentServiceImpl$$Lambda$1488/1750883128
com.edianyun.beidou.provider.service.impl.EmployeeMessageServiceImpl
com.edianyun.beidou.provider.service.impl.EmployeeMessageServiceImpl$$EnhancerBySpringCGLIB$$90f9010e
com.edianyun.beidou.provider.service.impl.EmployeeMessageServiceImpl$$EnhancerBySpringCGLIB$$90f9010e$$FastClassBySpringCGLIB$$a37236c1
com.edianyun.beidou.provider.service.impl.EmployeeMessageServiceImpl$$FastClassBySpringCGLIB$$5f5d1958
com.edianyun.beidou.provider.service.impl.EmployeeMessageServiceImpl$$Lambda$1644/71329748
com.edianyun.beidou.provider.service.impl.SystemServiceImpl
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$a962d9c4
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$a962d9c4$$FastClassBySpringCGLIB$$67299049
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$FastClassBySpringCGLIB$$5994df2a
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1504/1927995381
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1511/1883129367
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1512/1783440433
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1513/1090811768
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1514/335440839
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1639/874146782
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1640/1123361944
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1643/1800263128
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1645/1247556693
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1646/1830405338
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1647/1169832235
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1649/2058080283
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1650/2013446374
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1651/1453094191
com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$Lambda$1652/2088549043
Affect(row-cnt:35) cost in 20 ms.
```

#### dashboard

当前系统的实时数据面板，按 ctrl+c 退出。当运行在 Ali-tomcat 时，会显示当前 tomcat 的实时信息，如 HTTP 请求的 qps, rt, 错误数, 线程池信息等等。

```sh
[arthas@1]$ dashboard
ID       NAME                                                     GROUP                       PRIORITY           STATE              %CPU               DELTA_TIME         TIME              INTERRUPTED        DAEMON             
-1       VM Periodic Task Thread                                  -                           -1                 -                  0.0                0.000              0:27.530          false              true
62       redisson-timer-4-1                                       main                        5                  TIMED_WAITING      0.0                0.000              0:16.352          false              false             
78       redisson-netty-2-16                                      main                        5                  RUNNABLE           0.0                0.000              0:15.230          false              false             
64       redisson-netty-2-2                                       main                        5                  RUNNABLE           0.0                0.000              0:15.017          false              false             
116      DestroyJavaVM                                            main                        5                  RUNNABLE           0.0                0.000              0:12.999          false              false             
-1       C2 CompilerThread6                                       -                           -1                 -                  0.0                0.000              0:12.992          false              true
-1       C2 CompilerThread7                                       -                           -1                 -                  0.0                0.000              0:12.381          false              true
-1       C2 CompilerThread4                                       -                           -1                 -                  0.0                0.000              0:11.568          false              true
-1       C2 CompilerThread3                                       -                           -1                 -                  0.0                0.000              0:10.363          false              true
-1       C2 CompilerThread0                                       -                           -1                 -                  0.0                0.000              0:10.276          false              true
-1       C2 CompilerThread5                                       -                           -1                 -                  0.0                0.000              0:10.041          false              true
-1       C2 CompilerThread2                                       -                           -1                 -                  0.0                0.000              0:9.782           false              true
-1       C2 CompilerThread1                                       -                           -1                 -                  0.0                0.000              0:9.730           false              true
Memory                                          used             total           max             usage           GC                                                                                                               
heap                                            169M             989M            989M            17.13%          gc.ps_scavenge.count                                     60                                                      
ps_eden_space                                   86M              314M            315M            27.50%          gc.ps_scavenge.time(ms)                                  656
ps_survivor_space                               7M               8M              8M              85.10%          gc.ps_marksweep.count                                    4                                                       
ps_old_gen                                      75M              667M            667M            11.34%          gc.ps_marksweep.time(ms)                                 536
nonheap                                         158M             166M            -1              95.34%          
code_cache                                      50M              51M             240M            21.09%
metaspace                                       96M              102M            -1              93.94%
compressed_class_space                          11M              12M             1024M           1.12%
direct                                          80K              80K             -               100.00%
mapped                                          0K               0K              -               0.00%
Runtime                                                                                                                                                                                                                           
os.name                                                                                                          Linux
os.version                                                                                                       5.10.134-15.al8.x86_64
java.version                                                                                                     1.8.0_181
java.home                                                                                                        /usr/local/jdk1.8.0_181/jre
systemload.average                                                                                               2.23
processors                                                                                                       16
timestamp/uptime                                                                                                 Thu Sep 12 14:57:06 CST 2024/87600s

```

#### thread

查看当前线程信息，查看线程的堆栈

```sh
[arthas@1]$ thread
Threads Total: 143, NEW: 0, RUNNABLE: 53, BLOCKED: 0, WAITING: 41, TIMED_WAITING: 21, TERMINATED: 0, Internal threads: 28
ID         NAME                                                            GROUP                           PRIORITY             STATE                %CPU                  DELTA_TIME           TIME                 INTERRUPTED           DAEMON
267        arthas-command-execute                                          system                          5                    RUNNABLE             0.3                   0.000                0:0.151              false                 true
-1         C1 CompilerThread8                                              -                               -1                   -                    0.19                  0.000                0:2.094              false                 true
-1         VM Periodic Task Thread                                         -                               -1                   -                    0.03                  0.000                0:27.045             false                 true
62         redisson-timer-4-1                                              main                            5                    TIMED_WAITING        0.01                  0.000                0:16.076             false                 false               
2          Reference Handler                                               system                          10                   WAITING              0.0                   0.000                0:0.104              false                 true
3          Finalizer                                                       system                          8                    WAITING              0.0                   0.000                0:0.158              false                 true
4          Signal Dispatcher                                               system                          9                    RUNNABLE             0.0                   0.000                0:0.000              false                 true
253        Attach Listener                                                 system                          9                    RUNNABLE             0.0                   0.000                0:0.005              false                 true
255        arthas-timer                                                    system                          9                    WAITING              0.0                   0.000                0:0.000              false                 true
258        arthas-NettyHttpTelnetBootstrap-3-1                             system                          5                    RUNNABLE             0.0                   0.000                0:0.026              false                 true
259        arthas-NettyWebsocketTtyBootstrap-4-1                           system                          5                    RUNNABLE             0.0                   0.000                0:0.000              false                 true
260        arthas-NettyWebsocketTtyBootstrap-4-2                           system                          5                    RUNNABLE             0.0                   0.000                0:0.000              false                 true
261        arthas-shell-server                                             system                          9                    TIMED_WAITING        0.0                   0.000                0:0.042              false                 true
262        arthas-session-manager                                          system                          9                    TIMED_WAITING        0.0                   0.000                0:0.037              false                 true
264        arthas-NettyHttpTelnetBootstrap-3-2                             system                          5                    RUNNABLE             0.0                   0.000                0:0.144              false                 true
387        arthas-NettyHttpTelnetBootstrap-3-3                             system                          5                    RUNNABLE             0.0                   0.000                0:0.006              false                 true
388        arthas-NettyHttpTelnetBootstrap-3-4                             system                          5                    RUNNABLE             0.0                   0.000                0:0.285              false                 true
425        arthas-NettyHttpTelnetBootstrap-3-5                             system                          5                    RUNNABLE             0.0                   0.000                0:0.012              false                 true
426        arthas-NettyHttpTelnetBootstrap-3-6                             system                          5                    RUNNABLE             0.0                   0.000                0:0.032              false                 true
433        arthas-NettyHttpTelnetBootstrap-3-7                             system                          5                    RUNNABLE             0.0                   0.000                0:0.005              false                 true
434        arthas-NettyHttpTelnetBootstrap-3-8                             system                          5                    RUNNABLE             0.0                   0.000                0:0.033              false                 true
3002       logback-1                                                       system                          5                    WAITING              0.0                   0.000                0:0.001              false                 true
3003       arthas-Net
```

```sh
[arthas@1]$ thread 267
"arthas-command-execute" Id=267 RUNNABLE
    at sun.management.ThreadImpl.dumpThreads0(Native Method)
    at sun.management.ThreadImpl.getThreadInfo(ThreadImpl.java:448)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.processThread(ThreadCommand.java:233)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.process(ThreadCommand.java:120)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.process(AnnotatedCommandImpl.java:82)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.access$100(AnnotatedCommandImpl.java:18)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:111)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:108)
    at com.taobao.arthas.core.shell.system.impl.ProcessImpl$CommandProcessTask.run(ProcessImpl.java:385)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:748)
```

#### memory

查看 JVM 内存信息。

```sh
[arthas@1]$ memory
Memory                                                                                                        used                                total                                max                                 usage
heap                                                                                                          232M                                992M                                 992M                                23.43%
ps_eden_space                                                                                                 155M                                317M                                 317M                                48.85%
ps_survivor_space                                                                                             2M                                  8M                                   8M                                  25.39%
ps_old_gen                                                                                                    75M                                 667M                                 667M                                11.31%
nonheap                                                                                                       148M                                155M                                 -1                                  95.48%
code_cache                                                                                                    45M                                 46M                                  240M                                19.14%
metaspace                                                                                                     91M                                 97M                                  -1                                  94.45%
compressed_class_space                                                                                        11M                                 12M                                  1024M                               1.08%
direct                                                                                                        80K                                 80K                                  -                                   100.00%
mapped                                                                                                        0K                                  0K                                   -                                   0.00%
```

#### watch

函数执行数据观测,能观察到的范围为：`返回值`、`抛出异常`、`入参`，通过编写 OGNL 表达式进行对应变量的查看。

```sh
[arthas@1]$ watch com.edianyun.beidou.provider.service.impl.SystemServiceImpl getOwnerJumpLink '{params, returnObj, throwExp}' -n 3 -x 4
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 185 ms, listenerId: 14
method=com.edianyun.beidou.provider.service.impl.SystemServiceImpl.getOwnerJumpLink location=AtExit
ts=2024-09-12 14:55:16; [cost=15.038565ms] result=@ArrayList[
    @Object[][
        @Long[10021],
    ],
    @ArrayList[
        @OwnerJumpLinkVO[
            ownerType=@Integer[0],
            dingTalkJumpLink=@String[dingtalk://dingtalkclient/page/profile?corp_id=dinge30ff797ab2976d8&staff_id=312318701550241],
            userName=@String[wangjiahui],
            employeeName=@String[王佳辉],
            companyEmail=@String[wangjiahui@edianyun.com],
            department=@String[#总裁室#研发中心#研发部#产品中心研发组#产品中心研发组员工],
        ],
        @OwnerJumpLinkVO[
            ownerType=@Integer[1],
            dingTalkJumpLink=@String[dingtalk://dingtalkclient/page/profile?corp_id=dinge30ff797ab2976d8&staff_id=2230218756574030],
            userName=@String[wuqingqing],
            employeeName=@String[吴晴晴],
            companyEmail=@String[wuqingqing@edianyun.com],
            department=@String[#总裁室#研发中心#产品中心#产品中心员工],
        ],
    ],
    null,
]
method=com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444.getOwnerJumpLink location=AtExit
ts=2024-09-12 14:55:16; [cost=16.509336ms] result=@ArrayList[
    @Object[][
        @Long[10021],
    ],
    @ArrayList[
        @OwnerJumpLinkVO[
            ownerType=@Integer[0],
            dingTalkJumpLink=@String[dingtalk://dingtalkclient/page/profile?corp_id=dinge30ff797ab2976d8&staff_id=312318701550241],
            userName=@String[wangjiahui],
            employeeName=@String[王佳辉],
            companyEmail=@String[wangjiahui@edianyun.com],
            department=@String[#总裁室#研发中心#研发部#产品中心研发组#产品中心研发组员工],
        ],
        @OwnerJumpLinkVO[
            ownerType=@Integer[1],
            dingTalkJumpLink=@String[dingtalk://dingtalkclient/page/profile?corp_id=dinge30ff797ab2976d8&staff_id=2230218756574030],
            userName=@String[wuqingqing],
            employeeName=@String[吴晴晴],
            companyEmail=@String[wuqingqing@edianyun.com],
            department=@String[#总裁室#研发中心#产品中心#产品中心员工],
        ],
    ],
    null,
]
method=com.edianyun.beidou.provider.service.impl.SystemServiceImpl.getOwnerJumpLink location=AtExceptionExit
ts=2024-09-12 14:55:24; [cost=2.273002ms] result=@ArrayList[
    @Object[][
        @Long[1],
    ],
    null,
    java.lang.RuntimeException: 系统ID: 1不存在!请重新检查!
        at com.edianyun.beidou.provider.service.impl.SystemServiceImpl.getOwnerDomainBySystemId(SystemServiceImpl.java:203)
        at com.edianyun.beidou.provider.service.impl.SystemServiceImpl.getOwnerJumpLink(SystemServiceImpl.java:178)
        at com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$FastClassBySpringCGLIB$$5994df2a.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:771)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)
        at org.springframework.cache.interceptor.CacheInterceptor.lambda$invoke$0(CacheInterceptor.java:53)
        at org.springframework.cache.interceptor.CacheAspectSupport.invokeOperation(CacheAspectSupport.java:366)
        at org.springframework.cache.interceptor.CacheAspectSupport.execute(CacheAspectSupport.java:421)
        at org.springframework.cache.interceptor.CacheAspectSupport.execute(CacheAspectSupport.java:346)
        at org.springframework.cache.interceptor.CacheInterceptor.invoke(CacheInterceptor.java:61)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:691)
        at com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444.getOwnerJumpLink(<generated>)
        at com.edianyun.beidou.provider.controller.SystemController.getHeaderJumpLink(SystemController.java:96)
        at sun.reflect.GeneratedMethodAccessor205.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138)
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:878)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:792)
        at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040)
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943)
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
        at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:626)
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:733)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at com.edianyun.filter.DecryptFilter.doFilter(DecryptFilter.java:61)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:93)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
        at org.apache.catalina.valves.RemoteIpValve.invoke(RemoteIpValve.java:747)
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
        at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:373)
        at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
        at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:868)
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1589)
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)
,
]
Command execution times exceed limit: 3, so command will exit. You can set it with -n option.
```

#### trace

方法内部调用路径，并输出方法路径上的每个节点上耗时,`trace` 命令能主动搜索 `class-pattern`／`method-pattern` 对应的方法调用路径，渲染和统计整个调用链路上的所有性能开销和追踪调用链路。

`----skipJDKMethod <value>`skip Jdk method trace, default value true.默认情况下，trace 不会包含 jdk 里的函数调用，如果希望 trace jdk 里的函数，需要显式设置`--skipJDKMethod false`。

```sh
[arthas@1]$ trace com.edianyun.beidou.provider.service.impl.SystemServiceImpl getOwnerJumpLink -n 5
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 187 ms, listenerId: 12
`---ts=2024-09-12 14:49:36;thread_name=http-nio-8080-exec-2;id=104;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@cfbc8e8
    `---[46.388448ms] com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444:getOwnerJumpLink() [throws Exception]
        +---[99.75% 46.271632ms ] org.springframework.cglib.proxy.MethodInterceptor:intercept() [throws Exception]
        |   `---[98.74% 45.690184ms ] com.edianyun.beidou.provider.service.impl.SystemServiceImpl:getOwnerJumpLink() [throws Exception]
        |       +---[99.46% 45.445057ms ] com.edianyun.beidou.provider.service.impl.SystemServiceImpl:getOwnerDomainBySystemId() #178 [throws Exception]
        |       `---throw:java.lang.RuntimeException #203 [系统ID: 100不存在!请重新检查!]
        `---throw:java.lang.RuntimeException #203 [系统ID: 100不存在!请重新检查!]

`---ts=2024-09-12 14:49:39;thread_name=http-nio-8080-exec-1;id=103;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@cfbc8e8
    `---[3.43423ms] com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444:getOwnerJumpLink() [throws Exception]
        +---[99.22% 3.407534ms ] org.springframework.cglib.proxy.MethodInterceptor:intercept() [throws Exception]
        |   `---[93.94% 3.200952ms ] com.edianyun.beidou.provider.service.impl.SystemServiceImpl:getOwnerJumpLink() [throws Exception]
        |       +---[93.50% 2.992808ms ] com.edianyun.beidou.provider.service.impl.SystemServiceImpl:getOwnerDomainBySystemId() #178 [throws Exception]
        |       `---throw:java.lang.RuntimeException #203 [系统ID: 10064不存在!请重新检查!]
        `---throw:java.lang.RuntimeException #203 [系统ID: 10064不存在!请重新检查!]

`---ts=2024-09-12 14:49:44;thread_name=http-nio-8080-exec-3;id=105;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@cfbc8e8
    `---[12.362222ms] com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444:getOwnerJumpLink()
        `---[99.71% 12.326102ms ] org.springframework.cglib.proxy.MethodInterceptor:intercept()
            `---[96.81% 11.933207ms ] com.edianyun.beidou.provider.service.impl.SystemServiceImpl:getOwnerJumpLink()
                +---[38.15% 4.552794ms ] com.edianyun.beidou.provider.service.impl.SystemServiceImpl:getOwnerDomainBySystemId() #178
                +---[50.08% min=2.726272ms,max=3.249623ms,total=5.975895ms,count=2] com.edianyun.beidou.provider.service.EmployeeMessageService:getOwnerInfoByUserName() #181
                +---[0.14% min=0.004492ms,max=0.012352ms,total=0.016844ms,count=2] com.edianyun.beidou.provider.vo.OwnerJumpLinkVO:<init>() #183
                +---[9.46% min=0.2413ms,max=0.887668ms,total=1.128968ms,count=2] org.springframework.beans.BeanUtils:copyProperties() #184
                +---[0.12% min=0.004133ms,max=0.009822ms,total=0.013955ms,count=2] com.edianyun.beidou.provider.dao.po.EmployeeMessagePO:getDingTalkId() #185
                +---[0.10% min=0.004567ms,max=0.007617ms,total=0.012184ms,count=2] com.edianyun.beidou.provider.utils.DingTalkUrlUtil:dingTalkJumpLink() #185
                +---[0.08% min=0.003163ms,max=0.00585ms,total=0.009013ms,count=2] com.edianyun.beidou.provider.vo.OwnerJumpLinkVO:setDingTalkJumpLink() #185
                `---[0.07% min=0.002951ms,max=0.004987ms,total=0.007938ms,count=2] com.edianyun.beidou.provider.vo.OwnerJumpLinkVO:setOwnerType() #186

`---ts=2024-09-12 14:49:45;thread_name=http-nio-8080-exec-9;id=111;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@cfbc8e8
    `---[0.165261ms] com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444:getOwnerJumpLink()
        `---[84.68% 0.139946ms ] org.springframework.cglib.proxy.MethodInterceptor:intercept()

`---ts=2024-09-12 14:49:45;thread_name=http-nio-8080-exec-5;id=107;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@cfbc8e8
    `---[0.165684ms] com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444:getOwnerJumpLink()
        `---[85.17% 0.141109ms ] org.springframework.cglib.proxy.MethodInterceptor:intercept()

Command execution times exceed limit: 5, so command will exit. You can set it with -n option.
```

#### stack

输出当前方法被调用的调用路径。

```sh
[arthas@1]$ stack com.edianyun.beidou.provider.service.impl.SystemServiceImpl getOwnerJumpLink -n 2
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 223 ms, listenerId: 15
ts=2024-09-12 15:01:25;thread_name=http-nio-8080-exec-8;id=110;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@cfbc8e8
    @com.edianyun.beidou.provider.service.impl.SystemServiceImpl.getOwnerJumpLink()
        at com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$FastClassBySpringCGLIB$$5994df2a.invoke(<generated>:-1)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:771)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)
        at org.springframework.cache.interceptor.CacheInterceptor.lambda$invoke$0(CacheInterceptor.java:53)
        at org.springframework.cache.interceptor.CacheAspectSupport.invokeOperation(CacheAspectSupport.java:366)
        at org.springframework.cache.interceptor.CacheAspectSupport.execute(CacheAspectSupport.java:421)
        at org.springframework.cache.interceptor.CacheAspectSupport.execute(CacheAspectSupport.java:346)
        at org.springframework.cache.interceptor.CacheInterceptor.invoke(CacheInterceptor.java:61)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:749)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:691)
        at com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444.getOwnerJumpLink(<generated>:-1)
        at com.edianyun.beidou.provider.controller.SystemController.getHeaderJumpLink(SystemController.java:96)
        at sun.reflect.GeneratedMethodAccessor205.invoke(null:-1)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138)
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:878)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:792)
        at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040)
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943)
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
        at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:626)
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:733)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at com.edianyun.filter.DecryptFilter.doFilter(DecryptFilter.java:61)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:93)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
        at org.apache.catalina.valves.RemoteIpValve.invoke(RemoteIpValve.java:747)
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
        at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:373)
        at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
        at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:868)
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1589)
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

ts=2024-09-12 15:01:25;thread_name=http-nio-8080-exec-8;id=110;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@cfbc8e8
    @com.edianyun.beidou.provider.service.impl.SystemServiceImpl$$EnhancerBySpringCGLIB$$36093444.getOwnerJumpLink()
        at com.edianyun.beidou.provider.controller.SystemController.getHeaderJumpLink(SystemController.java:96)
        at sun.reflect.GeneratedMethodAccessor205.invoke(null:-1)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
        at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138)
        at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:878)
        at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:792)
        at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
        at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040)
        at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943)
        at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
        at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:626)
        at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
        at javax.servlet.http.HttpServlet.service(HttpServlet.java:733)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at com.edianyun.filter.DecryptFilter.doFilter(DecryptFilter.java:61)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:93)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
        at org.apache.catalina.valves.RemoteIpValve.invoke(RemoteIpValve.java:747)
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
        at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:373)
        at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
        at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:868)
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1589)
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:748)

Command execution times exceed limit: 2, so command will exit. You can set it with -n option.
```

#### tt

TimeTunnel方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

```sh
[arthas@1]$ tt -t com.edianyun.beidou.provider.controller.SystemController getHeaderJumpLink
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 141 ms, listenerId: 16
 INDEX         TIMESTAMP                          COST(ms)          IS-RET        IS-EXP        OBJECT                    CLASS                                               METHOD                                             
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 1000          2024-09-12 15:22:27                82.737622         true          false         0x573965e0                SystemController                                    getHeaderJumpLink
 1001          2024-09-12 15:22:28                0.169218          true          false         0x573965e0                SystemController                                    getHeaderJumpLink
 1002          2024-09-12 15:22:33                8.303633          true          false         0x573965e0                SystemController                                    getHeaderJumpLink
 1003          2024-09-12 15:22:37                2.691117          false         true          0x573965e0                SystemController                                    getHeaderJumpLink
 1004          2024-09-12 15:22:37                2.369208          false         true          0x573965e0                SystemController                                    getHeaderJumpLink
 1005          2024-09-12 15:22:37                2.300203          false         true          0x573965e0                SystemController                                    getHeaderJumpLink
```

**查看调用信息**，对于具体一个时间片的信息而言，你可以通过 `-i` 参数后边跟着对应的 `INDEX` 编号查看到他的详细信息。

```sh
[arthas@1]$ tt -i 1000
 INDEX          1000
 GMT-CREATE     2024-09-12 15:22:27
 COST(ms)       82.737622
 OBJECT         0x573965e0
 CLASS          com.edianyun.beidou.provider.controller.SystemController
 METHOD         getHeaderJumpLink
 IS-RETURN      true
 IS-EXCEPTION   false
 PARAMETERS[0]  @Long[10021]
 RETURN-OBJ     @Result[
                    serialVersionUID=@Long[274478677502831445],
                    code=@Integer[0],
                    message=@String[成功],
                    data=@ArrayList[isEmpty=false;size=2],
                ]
Affect(row-cnt:1) cost in 1 ms.
```

**重做一次调用**，`tt` 命令由于保存了当时调用的所有现场信息，所以我们可以自己主动对一个 `INDEX` 编号的时间片自主发起一次调用，从而解放你的沟通成本。此时你需要 `-p` 参数。通过 `--replay-times` 指定 调用次数，通过 `--replay-interval` 指定多次调用间隔(单位 ms, 默认 1000ms)

```sh
[arthas@1]$ tt -i 1000 -p --replay-times 3
 RE-INDEX       1000
 GMT-REPLAY     2024-09-12 15:31:06
 OBJECT         0x573965e0
 CLASS          com.edianyun.beidou.provider.controller.SystemController
 METHOD         getHeaderJumpLink
 PARAMETERS[0]  @Long[10021]
 IS-RETURN      true
 IS-EXCEPTION   false
 COST(ms)       0.22644
 RETURN-OBJ     @Result[
                    serialVersionUID=@Long[274478677502831445],
                    code=@Integer[0],
                    message=@String[成功],
                    data=@ArrayList[isEmpty=false;size=2],
                ]
Time fragment[1000] successfully replayed 1 times.

 RE-INDEX       1000
 GMT-REPLAY     2024-09-12 15:31:07
 OBJECT         0x573965e0
 CLASS          com.edianyun.beidou.provider.controller.SystemController
 METHOD         getHeaderJumpLink
 PARAMETERS[0]  @Long[10021]
 IS-RETURN      true
 IS-EXCEPTION   false
 COST(ms)       0.315634
 RETURN-OBJ     @Result[
                    serialVersionUID=@Long[274478677502831445],
                    code=@Integer[0],
                    message=@String[成功],
                    data=@ArrayList[isEmpty=false;size=2],
                ]
Time fragment[1000] successfully replayed 2 times.

 RE-INDEX       1000
 GMT-REPLAY     2024-09-12 15:31:08
 OBJECT         0x573965e0
 CLASS          com.edianyun.beidou.provider.controller.SystemController
 METHOD         getHeaderJumpLink
 PARAMETERS[0]  @Long[10021]
 IS-RETURN      true
 IS-EXCEPTION   false
 COST(ms)       0.325094
 RETURN-OBJ     @Result[
                    serialVersionUID=@Long[274478677502831445],
                    code=@Integer[0],
                    message=@String[成功],
                    data=@ArrayList[isEmpty=false;size=2],
                ]
Time fragment[1000] successfully replayed 3 times.
```



**注意事项**

- tt 命令的实现是：把函数的入参/返回值等，保存到一个`Map<Integer, TimeFragment>`里，默认的大小是 100。
- tt 相关功能在使用完之后，需要手动释放内存，否则长时间可能导致OOM。退出 arthas 不会自动清除 tt 的缓存 map。

**删除tt缓存**

- 通过索引删除指定tt记录

  ```sh
  tt -d 1001
  ```

- 清除所有的tt记录

  ```sh
  tt --delete-all
  ```

#### jad

反编译指定已加载类的源码

```sh
[arthas@1]$ jad com.edianyun.beidou.provider.controller.ActuatorController

ClassLoader:                                                                                                                                                                                                                      
+-org.springframework.boot.loader.LaunchedURLClassLoader@6615435c
  +-sun.misc.Launcher$AppClassLoader@18b4aac2
    +-sun.misc.Launcher$ExtClassLoader@343f4d3d

Location:                                                                                                                                                                                                                         
file:/data/ROOT.jar!/BOOT-INF/classes!/                                                                                                                                                                                           

       /*
        * Decompiled with CFR.
        * 
        * Could not load the following classes:
        *  org.springframework.web.bind.annotation.GetMapping
        *  org.springframework.web.bind.annotation.RequestMapping
        *  org.springframework.web.bind.annotation.ResponseBody
        *  org.springframework.web.bind.annotation.RestController
        */
       package com.edianyun.beidou.provider.controller;

       import org.springframework.web.bind.annotation.GetMapping;
       import org.springframework.web.bind.annotation.RequestMapping;
       import org.springframework.web.bind.annotation.ResponseBody;
       import org.springframework.web.bind.annotation.RestController;

       @RestController
       @RequestMapping(value={"/actuator"})
       public class ActuatorController {
           @ResponseBody
           @GetMapping(value={"/health"})
           public String health() {
/*21*/         return "ok";
           }
       }

Affect(row-cnt:1) cost in 470 ms.
```

### 进阶用法

[jad/mc/redefine 热更新](https://hengyun.tech/arthas-online-hotswap/)

[其他特性](https://arthas.aliyun.com/doc/advanced-use.html)