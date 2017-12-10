---
layout: post
title: 使用Actuator监控与度量SpringBoot应用程序
date: 2017-12-09 20:30:00
tags: [Actuator, Springboot]
---

SpringBoot Actuator为应用程序提供了很多Web服务, 通过这些服务我们可以了解到SpringBoot应用程序的运行时的内部状况。有了Actuator我们可以知道Bean是组合组装在一起的，获取运行时信息的快照等等。

### 一、如何在SpringBoot工程中配置使用Actuator

Maven项目中, 在pom的依赖项中增加

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
        <version>1.5.6.RELEASE</version>
    </dependency>

版本看使用SpringBoot的版本情况。

### Actuator的配置类的服务访问

##### Actuator 通过/beans获得Bean的装配信息

通过[http://localhost:8080/beans](http://localhost:8080/beans)接口获得每一个Bean的情况

    [
        {
            "context": "application:9080",
            "parent": null,
            "beans": [
                {
                    "bean": "springApplicationStart",
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.terrylmay.springboot.config.SpringApplicationStart$$EnhancerBySpringCGLIB$$529ae26c",
                    "resource": "null",
                    "dependencies": []
                },
                {
                    "bean": "org.springframework.boot.autoconfigure.internalCachingMetadataReaderFactory",
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.core.type.classreading.CachingMetadataReaderFactory",
                    "resource": "null",
                    "dependencies": []
                },
                {
                    "bean": "asyncDemo",
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.terrylmay.springboot.usermodule.controller.AsyncDemo",
                    "resource": "file [/Users/terrylmay/Github/SpringBootDemo/target/classes/com/terrylmay/springboot/usermodule/controller/AsyncDemo.class]",
                    "dependencies": []
                }
            ]
        }
    ]

会得到如上的JSON格式的Bean信息。

    * bean:表示Spring应用程序中bean的名字
    * aliases: 表示Bean的别名
    * scope: 表示bean的作用域, 默认是单例模式, 如果指定了Bean的@Scope("property")。那么每次声明都会创建一个Bean的实例
    * type: 表示所属的类
    * resource: 表示.class文件的物理位置, 这个会随着构建方式、运行方式的变化而变化
    * dependencies: 表示当前Bean注入的Bean名称列表

##### Actuator 通过/autoconfig提供Conditional注解的Bean是否创建成功

通过/autoconfig清晰的看到Bean的初始化是否成功[http://localhost:8080/autoconfig](http://localhost:8080/autoconfig)

    {
        "positiveMatches": {
            ...
            "JdbcTemplateAutoConfiguration#jdbcTemplate": [
                {
                    "condition": "OnBeanCondition",
                    "message": "@ConditionalOnMissingBean (types: org.springframework.jdbc.core.JdbcOperations; SearchStrategy: all) did not find any beans"
                }
            ]
            ...
        },
        ...
        "negativeMatches": {
            
            "ActiveMQAutoConfiguration": {
                "notMatched": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory'"
                    }
                ],
                "matched": []
            }
        }
        ...
    }

从上面的信息我们可以清晰的看到, jdbcTemplate这个Bean的创建条件就是当Spring容器中没有JdbcOperations该接口类型的Bean时自动创建, 而消息则说明没有找到任何JdbcOperations类型的Bean, 符合创建条件。所以JdbcTemplateAutoConfiguration#jdbcTemplate的Bean在positiveMatches列表中。

而在negativeMatches列表中, 举例: 因为在Classpath中没有ConnectionFactory、ActiveMQConnectionFactory这两个类, 所以ActiveMQAutoConfiguration类型的Bean没有创建。

##### 查看SpringBoot的配置属性

通过[http://localhost:8080/env](http://localhost:8080/env)可以查看应用程序可用的所有环境属性的列表.

    {
        "profiles": [],
        "server.ports": {
            "local.server.port": 9080
        },
        "servletContextInitParams": {},
        "systemProperties": {
            "com.sun.management.jmxremote.authenticate": "false",
            "java.runtime.name": "Java(TM) SE Runtime Environment",
            "spring.output.ansi.enabled": "always",
            "sun.boot.library.path": "/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib",
            "java.vm.version": "25.40-b25",
            "gopherProxySet": "false",
            "java.vm.vendor": "Oracle Corporation",
            "java.vendor.url": "http://java.oracle.com/",
            "java.rmi.server.randomIDs": "true",
            "path.separator": ":",
            "java.vm.name": "Java HotSpot(TM) 64-Bit Server VM",
            "file.encoding.pkg": "sun.io",
            "user.country": "CN",
            "sun.java.launcher": "SUN_STANDARD",
            "sun.os.patch.level": "unknown",
            "PID": "42844",
            "com.sun.management.jmxremote.port": "62246",
            "java.vm.specification.name": "Java Virtual Machine Specification",
            "user.dir": "/Users/terrylmay/Github/SpringBootDemo",
            "java.runtime.version": "1.8.0_40-b27",
            "java.awt.graphicsenv": "sun.awt.CGraphicsEnvironment",
            "org.jboss.logging.provider": "slf4j",
            "java.endorsed.dirs": "/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/jre/lib/endorsed",
            "os.arch": "x86_64",
            "java.io.tmpdir": "/var/folders/2b/kjrx2wl92vd3bt63ng8c4sb00000gn/T/",
            "line.separator": "\n",
            "java.vm.specification.vendor": "Oracle Corporation",
            "os.name": "Mac OS X",
            "sun.jnu.encoding": "UTF-8",
            "spring.beaninfo.ignore": "true",
        },
        "systemEnvironment": {
            "PATH": "/Library/Frameworks/Python.framework/Versions/3.6/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/X11/bin:/usr/local/sonar-runner-2.4/bin:/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home/bin:/usr/local/mysql/bin:/Users/terrylmay/Software/apache-maven-3.3.9/bin:/usr/local/Cellar/git/2.13.0/bin:/usr/local/opt/npm/libexec/bin:/usr/local/Cellar/node/8.9.1/bin",
            "JAVA_MAIN_CLASS_42844": "com.terrylmay.springboot.config.SpringApplicationStart",
            "GIT_HOME": "/usr/local/Cellar/git/2.13.0",
            "SHELL": "/bin/bash",
            "SONAR_RUNNER_HOME": "/usr/local/sonar-runner-2.4/bin",
            "JAVA_HOME": "/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home",
            "USER": "terrylmay",
            "VERSIONER_PYTHON_PREFER_32_BIT": "no",
            "TMPDIR": "/var/folders/2b/kjrx2wl92vd3bt63ng8c4sb00000gn/T/",
            "SSH_AUTH_SOCK": "/private/tmp/com.apple.launchd.zRUrkHmX8q/Listeners",
            "DISPLAY": "/private/tmp/com.apple.launchd.26KAsOXYNg/org.macosforge.xquartz:0",
            "SONAR_HOME": "/usr/local/sonarqube-5.1",
            "XPC_FLAGS": "0x0",
            "MAVEN_HOME": "/Users/terrylmay/Software/apache-maven-3.3.9",
            "MYSQL_HOME": "/usr/local/mysql",
            "VERSIONER_PYTHON_VERSION": "2.7",
            "__CF_USER_TEXT_ENCODING": "0x1F5:0x19:0x34",
            "Apple_PubSub_Socket_Render": "/private/tmp/com.apple.launchd.zDxwjPXzep/Render",
            "LOGNAME": "terrylmay",
            "LC_CTYPE": "zh_CN.UTF-8",
            "PWD": "/Users/terrylmay/Github/SpringBootDemo",
            "XPC_SERVICE_NAME": "com.jetbrains.intellij.13772",
            "HOME": "/Users/terrylmay"
        },
        "applicationConfig: [classpath:/application.properties]": {
            "spring.jpa.show-sql": "true",
            "spring.datasource.username": "root",
            "spring.datasource.driverClassName": "com.mysql.cj.jdbc.Driver",
            "server.port": "9080",
            "management.security.enabled": "false",
            "spring.datasource.password": "******",
            "spring.datasource.url": "jdbc:mysql://127.0.0.1:3306/spring_data_jpa_test"
        }
    }

从上面的信息中我们可以看到3大类型的数据, 系统属性、系统环境变量以及应用配置。

同时我们还可以使用[http://localhost:8080/env/{属性名}](http://localhost:8080/env/{属性名})
如在浏览器中输入[http://localhost:8080/env/spring.jpa.show-sql](http://localhost:8080/env/spring.jpa.show-sql)即可访问show-sql的值, 我们会得到如下结果

    {"spring.jpa.show-sql":"true"}

##### 通过/configprops查看应用的属性配置

通过[http://localhost:8080/configprops](http://localhost:8080/configprops)查看配置的一些具体信息

    {
    
        "serverProperties": {
            "prefix": "server",
            "properties": {
                "contextParameters": {},
                "address": null,
                "maxHttpPostSize": 0,
                "contextPath": null,
                "error": {
                    "path": "/error",
                    "includeStacktrace": "NEVER"
                },
                "ssl": null,
                "serverHeader": null,
                "useForwardHeaders": null,
                "port": 9080,
                "maxHttpHeaderSize": 0,
                "servletPath": "/",
                "jspServlet": null,
                "jetty": {
                    "maxHttpPostSize": 0,
                    "acceptors": null,
                    "selectors": null
                },
                "connectionTimeout": null
            }
        }
    }

我们可以看到server的一些配置, 包括server的前缀。正如我们在application.properties中定义的server.port 以及 server.contextPath一样。同时该配置项也能够提供快速找到哪些属性可以配置。

##### 使用/mappings查看应用中的服务都有哪些

通过[http://localhost:8080/mappings](http://localhost:8080/mappings)查看当前web应用中的服务都有哪些

    {
        "/webjars/**": {
            "bean": "resourceHandlerMapping"
        },
        "/**": {
            "bean": "resourceHandlerMapping"
        },
        "/**/favicon.ico": {
            "bean": "faviconHandlerMapping"
        },
        "{[/services/sayHello/{name}],methods=[GET]}": {
            "bean": "requestMappingHandlerMapping",
            "method": "public java.lang.String com.terrylmay.springboot.config.SpringApplicationStart.sayHello(java.lang.String)"
        },
        "{[/services/getPersonList/{personNum}],methods=[GET]}": {
            "bean": "requestMappingHandlerMapping",
            "method": "public java.util.List<java.util.Map<java.lang.String, java.lang.Object>> com.terrylmay.springboot.config.SpringApplicationStart.getPersonList(java.lang.Integer)"
        },
        "{[/error]}": {
            "bean": "requestMappingHandlerMapping",
            "method": "public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)"
        },
        "{[/error],produces=[text/html]}": {
            "bean": "requestMappingHandlerMapping",
            "method": "public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)"
        },
        "{[/auditevents || /auditevents.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public org.springframework.http.ResponseEntity<?> org.springframework.boot.actuate.endpoint.mvc.AuditEventsMvcEndpoint.findByPrincipalAndAfterAndType(java.lang.String,java.util.Date,java.lang.String)"
        },
        "{[/metrics/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.MetricsMvcEndpoint.value(java.lang.String)"
        },
        "{[/metrics || /metrics.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/trace || /trace.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/dump || /dump.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/health || /health.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.HealthMvcEndpoint.invoke(javax.servlet.http.HttpServletRequest,java.security.Principal)"
        },
        "{[/mappings || /mappings.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/configprops || /configprops.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/heapdump || /heapdump.json],methods=[GET],produces=[application/octet-stream]}": {
            "bean": "endpointHandlerMapping",
            "method": "public void org.springframework.boot.actuate.endpoint.mvc.HeapdumpMvcEndpoint.invoke(boolean,javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse) throws java.io.IOException,javax.servlet.ServletException"
        },
        "{[/autoconfig || /autoconfig.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/info || /info.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/env/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EnvironmentMvcEndpoint.value(java.lang.String)"
        },
        "{[/env || /env.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/beans || /beans.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        },
        "{[/loggers/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.LoggersMvcEndpoint.get(java.lang.String)"
        },
        "{[/loggers/{name:.*}],methods=[POST],consumes=[application/vnd.spring-boot.actuator.v1+json || application/json],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.LoggersMvcEndpoint.set(java.lang.String,java.util.Map<java.lang.String, java.lang.String>)"
        },
        "{[/loggers || /loggers.json],methods=[GET],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}": {
            "bean": "endpointHandlerMapping",
            "method": "public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()"
        }
    }

该数据中除了我们定义的服务外, 还可以额外的看到actuator提供的接口以及如何调用。

### Actuator运行时指标类服务

##### 查看应用程序的度量信息

为了了解应用程序的内存、堆栈信息, actuator提供了/metrics 服务用来提供应用的快照信息

    {
        "mem": 356749,
        "mem.free": 198931,
        "processors": 4,
        "instance.uptime": 3582611,
        "uptime": 3588813,
        "systemload.average": 1.63134765625,
        "heap.committed": 300544,
        "heap.init": 65536,
        "heap.used": 101612,
        "heap": 932352,
        "nonheap.committed": 57432,
        "nonheap.init": 2496,
        "nonheap.used": 56206,
        "nonheap": 0,
        "threads.peak": 44,
        "threads.daemon": 26,
        "threads.totalStarted": 64,
        "threads": 28,
        "classes": 8243,
        "classes.loaded": 8243,
        "classes.unloaded": 0,
        "gc.ps_scavenge.count": 12,
        "gc.ps_scavenge.time": 216,
        "gc.ps_marksweep.count": 2,
        "gc.ps_marksweep.time": 203,
        "httpsessions.max": -1,
        "httpsessions.active": 0,
        "datasource.primary.active": 0,
        "datasource.primary.usage": 0,
        "gauge.response.beans": 48,
        "gauge.response.mappings": 30,
        "gauge.response.env": 58,
        "gauge.response.autoconfig": 30,
        "gauge.response.dump": 354,
        "gauge.response.env.name": 4,
        "gauge.response.configprops": 154,
        "gauge.response.star-star": 35,
        "counter.status.200.mappings": 1,
        "counter.status.200.beans": 1,
        "counter.status.200.env.name": 1,
        "counter.status.200.configprops": 1,
        "counter.status.404.env.name": 2,
        "counter.status.404.star-star": 1,
        "counter.status.200.autoconfig": 2,
        "counter.status.200.dump": 1,
        "counter.status.200.env": 1
    }

提供了垃圾收集器、内存、堆、类加载器、系统、线程池、数据源、http等相关信息

    * gc.*: 包含已经发生的垃圾收集次数、垃圾收集所耗费的事件(来自于java.lang.management.GarbageCollectorMXBean)
    * mem.*: 分配给应用程序的内存数量和空闲内存数量(来自于java.lang.Runtime)
    * heap.*: 当前内存使用量(来自于java.lang.management.MemoryUsage)
    * processors: 处理器数量(来自于java.lang.Runtime)
    * uptime: 运行时间(来自于java.lang.management.RuntimeMXBean)
    * instance.uptime: 平均负载(java.lang.management.OperatingSystemMXBean)
    * threads.*: 线程、守护线程的数量 以及JVM启动后的线程数量峰值(来自于java.lang.management.ThreadMXBean)
    * datasource.*: 数据源连接数量(仅当Spring应用中存在DatasourceBean的时候才会有这个信息)
    * counter.*: 多种应用服务Http请求的度量值与计数器

同时我们也可以使用[http://localhost:8080/metrics/mem.free](http://localhost:8080/metrics/mem.free)来获取可用内存数量

    {"mem.free":197443}

##### 使用/trace记录WEB请求的细节

尽管/metrics可以提供Web请求计数器和计时器，可是度量信息中缺少了更加详细的信息。 使用[http://localhost:8080/trace](http://localhost:8080/trace)查看包括请求方法、路径、时间戳、请求响应的头信息。

    [
        {
            "timestamp": 1512912854713,
            "info": {
                "method": "GET",
                "path": "/metrics/mem.free",
                "headers": {
                    "request": {
                        "host": "localhost:9080",
                        "connection": "keep-alive",
                        "user-agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36",
                        "upgrade-insecure-requests": "1",
                        "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
                        "accept-encoding": "gzip, deflate, br",
                        "accept-language": "zh-CN,zh;q=0.8,en;q=0.6"
                    },
                    "response": {
                        "X-Application-Context": "application:9080",
                        "Content-Disposition": "inline;filename=f.txt",
                        "Content-Type": "application/vnd.spring-boot.actuator.v1+json;charset=UTF-8",
                        "Transfer-Encoding": "chunked",
                        "Date": "Sun, 10 Dec 2017 13:34:14 GMT",
                        "status": "200"
                    }
                },
                "timeTaken": "29"
            }
        }
    ]

该接口能够返回最近100次http请求的详细信息。

##### 使用/dump 服务获取程序线程的快照

    [
        {
            "threadName": "DestroyJavaVM",
            "threadId": 55,
            "blockedTime": -1,
            "blockedCount": 0,
            "waitedTime": -1,
            "waitedCount": 0,
            "lockName": null,
            "lockOwnerId": -1,
            "lockOwnerName": null,
            "inNative": false,
            "suspended": false,
            "threadState": "RUNNABLE",
            "stackTrace": [],
            "lockedMonitors": [],
            "lockedSynchronizers": [],
            "lockInfo": null
        },
        ...
    ]

##### 使用/health 服务查看应用程序的健康情况

    {
        "status": "UP",
        "diskSpace": {
            "status": "UP",
            "total": 120120541184,
            "free": 14053470208,
            "threshold": 10485760
        },
        "db": {
            "status": "UP",
            "database": "MySQL",
            "hello": 1
        }
    }


以上基本上就是actuator提供的基本的服务 以及 服务的搭建过程。

