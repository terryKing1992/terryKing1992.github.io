---
layout: post
title: 深入理解Redis系列之SpringBoot集成
date: 2018-11-06 22:30:00
tags: [redis, springboot]
---

>前面一篇文章已经写了如何搭建一个单机版Redis服务, 那么我们应该怎么在现有的系统中集成进来呢? 由于笔者使用的编程语言是Java, 所以本篇文章主要描述SpringBoot如何集成单Redis节点完成数据的增删改查.

# SpringBoot环境 #

## 快速搭建一个SpringBoot工程 ##
 
进入 `https://start.spring.io` 网站, 使用该网站初始化一个SpringBoot工程
 
![1706159233-5bdc6766067a1.png](https://upload-images.jianshu.io/upload_images/964723-a924ad71ce9cbfc4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 添加相关依赖 ##
因为使用spring initializer已经帮我们把Redis的依赖建立好了; 但是由于我们要使用Jedis客户端访问Redis, 所以还需要添加Jedis的依赖;

```xml
     <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version> //版本号可以放在properties中作为属性, 这边用${jedis.version}来依赖
     </dependency>
```
## 配置Redis节点信息 ##
打开application.properties文件, 初始化的文件是空的; 我们将spring redis最基本的信息加入进去

```text
spring.redis.host=localhost
spring.redis.port=6379
```
## 将Redis信息读入到程序中 ##
新建一个Java类命名为`StandaloneRedisConfig.java`， 放在`com.xxx.example.config`包下

```java
package com.terrylmay.redis.example.config;

import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "spring.redis")
@ConditionalOnProperty(name = {"spring.redis.host"})
public class StandaloneRedisConfig {

    String host;

    int port;

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public int getPort() {
        return port;
    }

    public void setPort(int port) {
        this.port = port;
    }
}
```
上面配置中的`@ConditionalOnProperty(name = {"spring.redis.host"})` 如果只是单机的Redis则不需要添加该属性; 但是为了后面一套代码兼容多个Redis部署模式, 使用该属性作为是否创建Bean的条件; 如果是集群模式那么就不会使用`spring.redis.host`来作为连接字符串了;

## 配置Jedis的连接池 ##
将Redis连接对象放入到Spring容器中进行管理

```
package com.terrylmay.redis.example;

import com.terrylmay.redis.example.config.StandaloneRedisConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;

@SpringBootApplication(scanBasePackages = {"com.terrylmay.redis.example"})
public class RedisExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisExampleApplication.class, args);
    }

    @Autowired
    StandaloneRedisConfig standaloneRedisConfig;

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    @Bean
    @ConditionalOnBean(value = {StandaloneRedisConfig.class})
    public RedisConnectionFactory standaloneRedisConnectionFactory() {
        JedisConnectionFactory factory = new JedisConnectionFactory(new RedisStandaloneConfiguration(standaloneRedisConfig.getHost(), standaloneRedisConfig.getPort()));
        return factory;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate() {
        return new StringRedisTemplate(redisConnectionFactory);
    }
}
```
这里的`@ConditionalOnBean(value = {StandaloneRedisConfig.class})`与上面的`ConditionalOnProperty` 是一个道理

这里的`scanBasePackages = {"com.terrylmay.redis.example"}` 是为了以后将Redis的客户端独立出一个工程而做的, 当然独立出来的工程base包名还要是这个才可以;

因为还没有看Redis支持的数据结构, 那么现在只是把Redis字符串模板类放到Spring 容器中, 后续再增加其他数据类型的支持;

## 创建操作Redis的接口 以及实现 ##

创建`ICacheProvider.java`接口:

```java
package com.terrylmay.redis.example.provider;

public interface ICacheProvider {
    void setString(String key, String value);

    String getString(String key);
}
```

Jedis版本的实现：

```java
package com.terrylmay.redis.example.provider.impl;

import com.terrylmay.redis.example.provider.ICacheProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

@Component
public class JedisCacheProvider implements ICacheProvider {

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @Override
    public void setString(String key, String value) {
        stringRedisTemplate.opsForValue().set(key, value);
    }

    @Override
    public String getString(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }
}
```

这样基本上一个可以操作Redis的Java程序就已经就绪了; 那么我们需要验证一下, 当然如果在主工程中写一个类去验证也是没有问题的, 比如创建一个Bean, 并且放到被`PostContruct`注解的方法里面;

但是更加专业的做法是写一个测试程序来测试, 下面看一下该测试程序应该怎么写

## UT测试程序可用性
因为创建工程的时候, 就已经有一个测试类在test目录下面了, 我们增加我们想要的功能

```java
package com.terrylmay.redis.example;

import com.terrylmay.redis.example.provider.ICacheProvider;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {RedisExampleApplication.class})
public class RedisExampleApplicationTests {

    @Autowired
    ICacheProvider jedisCacheProvider;

    @Test
    public void contextLoads() {
        jedisCacheProvider.setString("name", "terrylmay");
        System.out.println(jedisCacheProvider.getString("name")); 
        Assert.assertEquals("terrylmay", jedisCacheProvider.getString("name"));
    }
}
```

`注: 程序中不要有打印, 使用Logger或者直接断言来处理`  （本来想用markdown语法来标红的, 但是发现简书竟然不支持html的写法; 没办法只能用``来搞定了）

## 开发过程中遇到的问题 ## 
一、在写好所有的程序之后, 跑测试用例, 但是始终都是报NoSuchBeanException
```text
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException:
No qualifying bean of type 'com.terrylmay.redis.example.config.StandaloneRedisConfig' available: expected at least 1 bean which qualifies as autowire candidate. 
Dependency annotations: {
@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

原因共有三点：

1、写了`scanBasepackages`来扫描包下面的bean, 扫描的包与类所在的包不一样， 只有一个字符之差 `com.terrylmay.redis.example` 与 `com.terrlmay.redis.example`, 当然这时候`idea`会报错， 只是我不认识那个错而已;  Idea报错如图所示: 

![image.png](https://upload-images.jianshu.io/upload_images/964723-be5e43d881953bcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、按照网上的application.properties属性的读取方式, 只使用了一个注解:
`@ConfigurationProperties(prefix = "spring.redis")` 但是点进该注解里面看, 它其实并没有Component注解的功能; 所以增加了`@Configuration`注解

3、第三个原因不仔细断然不会发现这个错误

![image.png](https://upload-images.jianshu.io/upload_images/964723-46958b7c21587b65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我理解的是只要工程里面父工程是`spring-boot-starter-parent`, 那么就不应该存在这类Jar包没有依赖的问题,  打开文档

![image.png](https://upload-images.jianshu.io/upload_images/964723-91f1edb748412ada.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

依赖可粘贴版:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

到这里, 一个能够使用Redis的Java工程就已经就绪了; 最终的代码全部在[Github的spring-redis-example仓库下](https://github.com/terryKing1992/spring-redis-example), 欢迎star与PR





