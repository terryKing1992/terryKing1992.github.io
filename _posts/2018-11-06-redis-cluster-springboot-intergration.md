---
layout: post
title: 深入理解Redis系列之SpringBoot集群集成
date: 2018-11-06 22:30:00
tags: [redis, springboot]
---

>上一篇文章写了关于集群搭建的步骤、master节点切换的相关内容, 有了集群肯定需要程序访问, 那么，今天就看一下SpringBoot如何访问Redis集群的；

##  继续Spring-redis-example框架 ##

由于前面访问单机版Redis已经写好了一个`maven`工程, 后面对于集群的访问也都在该工程上面进行了; 不了解的可以移步[深入理解Redis之SpringBoot集成](https://www.jianshu.com/p/3aff3c56f0ad);

我们在原来工程的基础上稍微改造一下, 就能够同时支持单节点与集群模式的Redis访问了.

```
package com.terrylmay.redis.example.config;

import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "spring.redis.cluster")
@ConditionalOnProperty(name = {"spring.redis.cluster.nodes"})
public class ClusterRedisConfig {

    private String nodes;

    private String password;

    private int maxRedirects;

    public String getNodes() {
        return nodes;
    }

    public void setNodes(String nodes) {
        this.nodes = nodes;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getMaxRedirects() {
        return maxRedirects;
    }

    public void setMaxRedirects(int maxRedirects) {
        this.maxRedirects = maxRedirects;
    }
}
```
新增一个关于集群信息的配置类, 方便后面创建`JedisConnectionFactory`的时候用到. 修改`Application类`

```
package com.terrylmay.redis.example;

import com.terrylmay.redis.example.config.ClusterRedisConfig;
import com.terrylmay.redis.example.config.StandaloneRedisConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.Arrays;
import java.util.Collections;

@SpringBootApplication(scanBasePackages = {"com.terrylmay.redis.example"})
public class RedisExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisExampleApplication.class, args);
    }
    
    @Autowired(required = false) 
    StandaloneRedisConfig standaloneRedisConfig;

    @Autowired(required = false)
    ClusterRedisConfig clusterRedisConfig;

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        JedisConnectionFactory factory = null;
        if (standaloneRedisConfig != null) {
            factory = new JedisConnectionFactory(new RedisStandaloneConfiguration(standaloneRedisConfig.getHost(), standaloneRedisConfig.getPort()));
            return factory;
        }

        if (clusterRedisConfig != null) {
            JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration(Arrays.asList(clusterRedisConfig.getNodes().split(",")));
            redisClusterConfiguration.setMaxRedirects(clusterRedisConfig.getMaxRedirects());
            redisClusterConfiguration.setPassword(clusterRedisConfig.getPassword());
            factory = new JedisConnectionFactory(redisClusterConfiguration, jedisPoolConfig);
        }

        return factory;
    }

    @Bean
    public StringRedisTemplate stringRedisTemplate() {
        return new StringRedisTemplate(redisConnectionFactory);
    }
}
```
这两个类`StandaloneRedisConfig,ClusterRedisConfig` 的注解之所以用`required=false` 是因为这些类的Bean只有在满足配置文件中有特定的属性Key的时候才会生成, 所以对于某一个特定环境, 只可能使用一种Redis模式; 在创建`RedisConnectionFactory ` 根据Bean是否存在, 创建出来不同模式的集群访问类;

最后, 对工程进行一下完善, 因为原来是打算把所有模式下的`redis`配置信息放到一个配置文件中, 然后通过注释的方式做个演示; 后面发现多创建几个`application-xxx.properties`文件更加方便;

创建出来的不同模式下的Redis配置信息文件如下:

`application.properties` 里面只放置当前运行时的`spring.profiles.active` 信息

``` text
spring.profiles.active=cluster
 ```

`application-standalone.properties` 文件内容如下:

```
spring.redis.host=localhost
spring.redis.port=6379
```

`application-cluster.properties` 文件内容如下:

```
spring.redis.cluster.nodes=127.0.0.1:6379,127.0.0.1:6380,127.0.0.1:6381,127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381,
spring.redis.cluster.password=
spring.redis.cluster.max-redirects=12
```

这样, 如果测试的时候, 只需要切换`application.properties`文件中的`spring.profiles.active`属性值即可; 可选项有: `standalone|cluster|sentinel`

代码写完之后, 可以运行一下单元测试看是否能通过; 所有关于该项目的代码在[Github spring-redis-example仓库中](https://github.com/terryKing1992/spring-redis-example) 欢迎 star  & PR