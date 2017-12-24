---
layout: post
title: 设计模式系列之开闭原则
date: 2017-12-23 09:49:00
tags: [设计准则, 开闭原则]
---

### 开闭原则

开闭原则就是字面理解的意思: 软件实体对扩展开放, 对修改关闭; 那么什么是软件实体呢? 通常在代码开发过程中, 我们能接触到的就是1、模块: 比如maven 模块、 或者任意一个系统的子模块如(电商App的购物车模块、详情展示模块等等) 2、类: 在面向对象的编程语言中, 如果想要实现一个系统必须是由最基本的类组成的 3、方法: 粒度更小的组成单元。

开闭原则是在设计代码架构时所需要优先考虑的原则, 而前面记录的5大原则 则是对开闭原则如何实现的具体的实践方式; 通常意义来说: 开闭原则是指导思想, 其他基本原则是实现该思想的具体手段. 同样也可以理解为开闭原则是抽象概念, 而其他基本原则是具体实现.

### 场景示例

假设在开发一个推送系统过程中, 我们需要有3中方式的推送: 按照使用系统本身的推送(MQTT), 使用Google Android官方推送GCM, 使用iOS官方推送APNS, 那么在实现过程中。如果不加思考的就开始编码, 我们可能写出来如下的代码:

    package com.terrylmay.springboot;

    import java.util.UUID;

    public class PushDemo {
        public static void main(String[] args) {
            String pushType = "mqtt";
            PushClient pushClient = new PushClient();
            PushMessage pushMessage = new PushMessage("Hello, Welcome", UUID.randomUUID().toString());

            if ("mqtt".equals(pushType)) {
                pushClient.pushByMQTT(pushMessage);
            }else if("apns".equals(pushType)) {
                pushClient.pushByAPNS(pushMessage);
            }else if("gcm".equals(pushType)) {
                pushClient.pushByGCM(pushMessage);
            }

        }
    }

    class PushClient {

        void pushByMQTT(PushMessage pushMessage) {
            System.out.println("推送的类型为: MQTT, 要推送的设备Id:" + pushMessage.getDeviceToken() + ",推送的消息为:" + pushMessage.getMessage());
        }

        void pushByAPNS(PushMessage pushMessage) {
            System.out.println("推送的类型为: APNS, 要推送的设备Id:" + pushMessage.getDeviceToken() + ",推送的消息为:" + pushMessage.getMessage());
        }

        void pushByGCM(PushMessage pushMessage) {
            System.out.println("推送的类型为: GCM, 要推送的设备Id:" + pushMessage.getDeviceToken() + ",推送的消息为:" + pushMessage.getMessage());
        }
    }

    class PushMessage {
        private String message;
        private String deviceToken;

        public PushMessage(String message, String deviceToken) {
            this.message = message;
            this.deviceToken = deviceToken;
        }

        public String getMessage() {
            return message;
        }

        public void setMessage(String message) {
            this.message = message;
        }

        public String getDeviceToken() {
            return deviceToken;
        }

        public void setDeviceToken(String deviceToken) {
            this.deviceToken = deviceToken;
        }
    }


当然从上面的程序来说, 感觉还可以。我们对于不同的消息都有单独的方法来处理, 满足方法层面的单一责任原则. 但是如果我们后面要扩展, 那么就需要在原有类的基础上修改. 比如我要添加一个小米推送的策略以及百度推送的策略呢? 那么需要在原有的类的基础上增加```pushByXM``` 和 ```pushByBaiDu``` 的方法. 这样做虽然也符合对扩展开放, 但仅仅是在方法层面的开放, 但是在类层面显然不符合对修改封闭的原则 以及 对扩展开放的原则. 同时在高层调用的时候, 我们同样需要修改```if else ```的代码, 那么如何才能够做到同时对扩展开放 对修改关闭呢?

我们总是听说需要将面向对象编程的思想 转换到 面向接口编程, 面向接口编程具有易维护、易扩展、健壮性强等特点, 所以我们也将上述代码修改为面向接口编程的例子:

首先, 因为推送具有多个通道类型的推送, 那么我们先将消息进行重构, 因为推送消息都需要有消息体才可以、设备ID等. 消息推送通道类型在我们这个例子中也是需要的. 所以我们经过分析开发出如下代码:

    class PushMessage {
        private String message;
        private String deviceToken;
        private PushChannelType pushChannelType;

        public PushMessage(String message, String deviceToken) {
            this.message = message;
            this.deviceToken = deviceToken;
        }

        public String getMessage() {
            return message;
        }

        public void setMessage(String message) {
            this.message = message;
        }

        public String getDeviceToken() {
            return deviceToken;
        }

        public void setDeviceToken(String deviceToken) {
            this.deviceToken = deviceToken;
        }

        public PushChannelType getPushChannelType() {
            return pushChannelType;
        }

        public void setPushChannelType(PushChannelType pushChannelType) {
            this.pushChannelType = pushChannelType;
        }
    }

    class APNSPushMessage extends PushMessage {
        public APNSPushMessage(String message, String deviceToken) {
            super(message, deviceToken);
            this.setPushChannelType(PushChannelType.APNS);
        }
    }

    class MQTTPushMessage extends PushMessage {
        public MQTTPushMessage(String message, String deviceToken) {
            super(message, deviceToken);
            this.setPushChannelType(PushChannelType.MQTT);
        }
    }

    class GCMPushMessage extends PushMessage {
        public GCMPushMessage(String message, String deviceToken) {
            super(message, deviceToken);
            this.setPushChannelType(PushChannelType.GCM);
        }
    }

我们通过分析创建出了3中不同的推送消息, 这样当有另外一种消息需要扩展的时候, 我们只需要创建一个子类就可以了. 后面我们要对 推送消息的客户端进行重构。

    interface IPushClient {
        void pushMessage(PushMessage pushMessage);
    }

    class MQTTPushClient implements IPushClient {

        public void pushMessage(PushMessage pushMessage) {
            System.out.println("推送的类型为: MQTT, 要推送的设备Id:" + pushMessage.getDeviceToken() + ",推送的消息为:" + pushMessage.getMessage());

        }
    }

    class APNSPushClient implements IPushClient {
        public void pushMessage(PushMessage pushMessage) {
            System.out.println("推送的类型为: APNS, 要推送的设备Id:" + pushMessage.getDeviceToken() + ",推送的消息为:" + pushMessage.getMessage());
        }
    }

    class GCMPushClient implements IPushClient {
        public void pushMessage(PushMessage pushMessage) {
            System.out.println("推送的类型为: GCM, 要推送的设备Id:" + pushMessage.getDeviceToken() + ",推送的消息为:" + pushMessage.getMessage());

        }
    }

我们对PushClient进行重构, 基于面向接口编程, 有子类实现具体的方法. 如果有另外一种比如小米推送或者百度推送的支持, 那么再重新写一个IPushClient的实现类就可以了.这样我们做到了对类的修改关闭, 对类的扩展开放了.这样看来基本上实现了开闭原则, 我们来看一下客户端代码应该怎么写:

    import java.util.UUID;

    public class PushDemo {
        public static void main(String[] args) {
            IPushClient pushClient = new MQTTPushClient();
            PushMessage pushMessage = new MQTTPushMessage("Hello, Welcome", UUID.randomUUID().toString());
            pushClient.pushMessage(pushMessage);
        }
    }

基于上述上层代码, 如果我们要使用APNS的推送, 应该怎么做呢? 将MQTT相关的东西(PushMessage、PushClient)改为APNS的即可.这样看来, 当我们新增一种消息通道, 需要修改代码的两行就能搞定了. 这样的代码在我看来已经非常不错了, 具有很高的扩展性. 但是根据最小知识原则, 上层代码都不需要知道到底初始化哪个推送客户端就可以完成推送的功能. 我们又引入了BeanContainer的概念. 这个地方只是模拟Spring ComponentScan之后的bean的状态;

    class BeanContainer {
        private final static Map<String, IPushClient> beanContainerMap = new HashMap<String, IPushClient>();

        static {
            beanContainerMap.put(PushChannelType.MQTT.getChannelType(), new MQTTPushClient());
            beanContainerMap.put(PushChannelType.APNS.getChannelType(), new APNSPushClient());
            beanContainerMap.put(PushChannelType.GCM.getChannelType(), new GCMPushClient());
        }

        public static IPushClient getPushClientByPushType(String pushType) {
            return beanContainerMap.get(pushType);
        }
    }

同时客户端代码调整为:

    import java.util.HashMap;
    import java.util.Map;
    import java.util.UUID;

    public class PushDemo {
        public static void main(String[] args) {
            PushMessage pushMessage = new MQTTPushMessage("Hello, Welcome", UUID.randomUUID().toString());
            IPushClient pushClient = BeanContainer.getPushClientByPushType(pushMessage.getPushChannelType().getChannelType());
            pushClient.pushMessage(pushMessage);
        }
    }

 这样客户端当改变推送类型的时候,只需要修改一行代码, 即只需要知道我要推送的消息是哪种类型的, 然后你们给我我需要的PushClient, 让我能完成推送功能即可. 其他的上层调用方完全不用关系. 同时代码也具有了很高的扩展性. 符合开闭原则。

 ### 开闭原则的心得

开闭原则是所有基本原则 与 23种设计模式的更高层抽象; 实现开闭原则的方式多种多样, 并不限于5大基本原则 + 23种设计模式. 只要能够对于纷繁复杂的需求变化能够做到在增加需求的时候不会影响到其他模块 同时也不会影响到原始功能的使用. 那么基本上系统的健壮性、易维护性、易扩展性就都可以得到很好的保证. 同时, 在开发过程中 也不要忘记我们的最后的安全网 单元测试 与 集成测试.



