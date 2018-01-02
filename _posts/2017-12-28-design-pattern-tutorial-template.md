---
layout: post
title: 设计模式系列之模板方法
date: 2017-12-28 21:49:00
tags: [设计准则, 模板方法]
---

### 模板方法

定义一个操作中的算法框架, 而将一些步骤延迟到子类中。 使得子类可以在不改变一个算法的结构即可重新定义该算法的某些特性的步骤.
模板方法基本上是仅次于单例模式以及原型模式的最简单也最容易理解的设计模式. 我们来看一下模板方法类图:

![模板方法类图](/assets/images/2017-12-28-design-pattern-template.png)

该类图中有4个模块:

IProduct: 定义该产品应该有行为

AbstractBackboneProduct: 定义该产品行为的骨架实现

ConcreteProductA: 具体的产品子类, 并且实现骨架实现中的部分细节

ConcreteProductB: 功能同 ConcreteProductA

### 应用场景

模板方法模式在实际工程开发过程中, 使用的场景已经该是很多的. 比如在项目中 有一个使用MQ的需求, 同时MQ的消费者有多个, 分别负责消费不同Topic下的消息; 我们可以根据该场景进行建模; 因为消费者的行为就是消费消息, 我们先定义一个消费接口IMessageConsumer

    interface IMessageConsumer {
        void consume(Message message);
    }

然后, 我们借助于RocketMQ的客户端来完成我们抽象消费者类的创建:

    class AbstractMessageConsumer implements IMessageConsumer {

    public AbstractMessageConsumer() {
        this.init();
    }

    public void init() {
        DefaultMQPushConsumer consumer =
                new DefaultMQPushConsumer("PushConsumer");
        consumer.setNamesrvAddr("192.168.58.163:9876");
        try {
            //订阅PushTopic下Tag为push的消息
            consumer.subscribe(this.getConsumerTopic(), this.getTag());
            //程序第一次启动从消息队列头取数据
            consumer.setConsumeFromWhere(
                    ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
            consumer.registerMessageListener(
                    new MessageListenerConcurrently() {
                        public ConsumeConcurrentlyStatus consumeMessage(
                                List<MessageExt> list,
                                ConsumeConcurrentlyContext Context) {
                            Message msg = list.get(0);
                            this.consumeMessage(msg);
                            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                        }
                    }
            );
            consumer.start();
        } catch (Exception e) {

        }
    }

    protected abstract String getConsumerTopic();

    protected abstract String getTag();

}

最后, 我们需要分别实现TopicA下的消息的监听处理 与 TopicB下的消息的监听处理 以及消息的处理, 即ConcreteTopicConsumer类:

    class ConcreteTopicConsumer extends AbstractMessageConsumer {
        @Override
        protected String getConsumerTopic() {
            return "TopicA";
        }

        @Override
        protected String getTag() {
            return "push";
        }

        public void consume(Message message) {
            System.out.println(message.toString());
        }
    }

### 实践心得 

模板方法在以下几个场景下可以使用:

1、多个子类有共同的方法, 并且实现逻辑基本上相同. 就如上面代码中初始化Consumer的过程逻辑基本上一样
2、如果有实现逻辑不太一样的, 也可以在子类中通过实现一个boolean类型返回值的函数, 然后在父类逻辑代码中调用子类的方法, 完成一些简单的逻辑控制
3、对于一些有if else的逻辑, 如果基本流程相同也可以使用模板方法来完成. 比如使用用户名、邮箱登录或者手机号任何一种方式登录.我们既可以再入口处判断是否是邮箱登录、或者手机号登录, 然后抽象出 ```用户``` 、 ```authToken```等登录属性, 同时抽象出 ```validUserName``` ```verifyAuthToken```等抽象方法供子类实现, 而抽象模板类中只需要挨个调用抽象方法即可.