---
layout: post
title: Log4j、Log4j2以及Logback性能对比
date: 2018-08-23 22:30:00
tags: [日志框架, Logback, log4j, log4j2]
---

### 前言

原本以前都是开发一些管理端的功能, 并发量少的出奇, 仅仅关注了功能的完善性与易用性; 并没有太注意在并发下日志系统的性能问题; 但近期在开发一个需要较高并发系统的时候, 日志框架的性能问题才逐渐的展现在我的面前; 以前的日志框架使用的logback, 其本身也是SpringBoot中自带的一个框架; 但是通过测试发现, 在并发量较高的情况下, logback框架即使是异步打印日志的方式, 也会存在某一个时间点突然卡那么1-3s钟, 表现为:本来接口的调用仅仅在50ms左右就能够完成; 但是并发量高的情况下会出现某一次调用就出现3000+ms的性能缺陷. 通过查看日志发现, 1、有时候日志出现断层发生在调用函数栈切换的时候, 即: 在A函数中调用B函数, 但在调用B函数之前打印的日志与进入到B之后打印的日志时间相差1-3s钟； 2、有时候直接在同一个函数的两个连续打印语句中出现时间断层. 这让我不得不怀疑是日志框架的问题了.于是就有了这篇对比日志框架性能差异的文章.


### 环境配置

1、JDK1.8版本
2、MAC Air电脑
3、Intellij Idea

### 测试指标

1、吞吐量: 在特定的时间范围内, 能够记录日志条数; 包括峰值吞吐量以及最大持续产量(即平均吞吐量)
2、响应时间延时: 记录消息所需的时间

### Logback的配置

Logback的体系结构足够通用，以便在不同情况下应用。目前，logback分为三个模块：logback-core，logback-classic和logback-access。

logback-core模块为其他两个模块奠定了基础。logback-classic模块可以被同化为log4j的显着改进版本。此外，logback-classic本身实现了SLF4J API，因此您可以在logback和其他日志框架（如log4j或java.util.logging（JUL））之间来回切换。

logback-access模块​​与Servlet容器（如Tomcat和Jetty）集成，以提供HTTP访问日志功能。请注意，您可以在logback-core之上轻松构建自己的模块。

首先, 我们创建一个maven 工程, 并且添加相关Logback的依赖;

```xml
<properties>
    <log4j2.version>2.11.1</log4j2.version>
    <slfj.version>1.7.25</slfj.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slfj.version}</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <version>1.2.3</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

下面配置Logback的xml文件:

```xml
<configuration>
    <property name="LOG_NAME" value="../logs"></property>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="RollingFile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/log_info.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>

        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_NAME}/info/%d{yyyy-MM-dd_HH}-%i.log.log</fileNamePattern>
            <maxHistory>10</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>100MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>

        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n
            </pattern>
        </encoder>
    </appender>
    <root level="INFO">
        <!--<appender-ref ref="STDOUT"/>-->
        <appender-ref ref="RollingFile"/>
    </root>
</configuration>
```

我们首先通过最简单的循环打印的方式看下logback打印200万条数据耗时多少? 新建一个HelloWorldController类

```java
package com.terrylmay.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.CountDownLatch;

public class HelloWorldController {

    private static final Logger LOGGER = LoggerFactory.getLogger(HelloWorldController.class);

    public static void logging() {
        LOGGER.info("HelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorld");
    }

    public static void main(String[] args) throws InterruptedException {
        Long startTime = System.currentTimeMillis();
        int threadCount = 1;
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int index = 0; index < threadCount; index++) {
            new Thread(new Runnable() {
                public void run() {
                    for (int i = 0; i < 2000000; i++) {
                        logging();
                    }
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        Long endTime = System.currentTimeMillis();
        LOGGER.info("total cost time is:{}", (endTime - startTime));
    }
}


```

经过测试发现, logback连续打印200万条数据, 每条记录162个字符, 总耗时为 ```15233ms```;


### Log4j2的配置与测试

```xml
<dependencies>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.25</version>
    </dependency>

    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.11.1</version>
    </dependency>

    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.11.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.11.1</version>
    </dependency>
</dependencies>
```

对于这种maven依赖, 为了方便管理版本; 最好优化为下面的配置:

```xml
<properties>
    <log4j2.version>2.11.1</log4j2.version>
    <slfj.version>1.7.25</slfj.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slfj.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>${log4j2.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>${log4j2.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>${log4j2.version}</version>
    </dependency>
</dependencies>

```

我们首先通过最简单的循环打印的方式看下log4j2打印200万条数据耗时多少? 新建一个HelloWorldController类

```java
package com.terrylmay.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.CountDownLatch;

public class HelloWorldController {

    private static final Logger LOGGER = LoggerFactory.getLogger(HelloWorldController.class);

    public static void logging() {
        LOGGER.info("HelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorldHelloWorld");
    }

    public static void main(String[] args) throws InterruptedException {
        Long startTime = System.currentTimeMillis();
        int threadCount = 1;
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int index = 0; index < threadCount; index++) {
            new Thread(new Runnable() {
                public void run() {
                    for (int i = 0; i < 2000000; i++) {
                        logging();
                    }
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        Long endTime = System.currentTimeMillis();
        LOGGER.info("total cost time is:{}", (endTime - startTime));
    }
}


```

然后, 需要我们配置一下log4j2的配置文件, 放入到resources目录下即可;

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="OFF">

    <Properties>
        <property name="LOG_NAME">./logs</property>
        <property name="LOG_LEVEL">info</property>
    </Properties>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>
        </Console>

        <RollingFile name="randomAccessFile" fileName="${LOG_NAME}/log_info.log" filePattern="${LOG_NAME}/info/log-info-%d{yyyy-MM-dd}.%i.log">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy modulate="true" interval="1" />
                <SizeBasedTriggeringPolicy size="100M" />
            </Policies>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="info">
            <!--<AppenderRef ref="Console" />-->
            <AppenderRef ref="randomAccessFile" />
        </Root>
    </Loggers>
</configuration>
```
经过测试发现, log4j2连续打印200万条数据, 每条记录162个字符, 总耗时为 ```14817ms```;

基本上在同步情况下, 一个线程在性能上表现不出来较大差异, 下面我们看一下两个线程日志性能的情况;

修改Java程序中的threadCount=2;测试其表现
经测试发现, logback时间消耗在: ```23894ms```, 而log4j2时间消耗在: ```24420ms```

修改Java程序中的threadCount=4;测试其表现
经测试发现, logback时间消耗在: ```48866ms```, 而log4j2时间消耗在: ```56178ms```

修改Java程序中的threadCount=8; 由于机器的性能与硬盘不够, 只能将线程中的for循环减小为10W测试其表现
经测试发现, logback时间消耗在: ```9868ms```, 而log4j2时间消耗在: ```12405ms```

总的来说, Log4j2在同步输出日志上性能要稍微逊色于logback一点;


