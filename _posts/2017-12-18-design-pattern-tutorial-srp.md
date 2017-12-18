---
layout: post
title: 设计模式系列之单一责任原则
date: 2017-12-18 19:40:00
tags: [设计准则, 单一责任原则]
---

设计模式虽然很久以前就已经知道了并且在项目中也会用到一些基本的设计模式, 比如 单例、享元、模板方法、抽象工厂等. 但是感觉还是要记录并整理一下在这方面的体会与实践, 以便加深自己对设计模式的理解。

###  单一责任原则

单一责任原则英文名称Single Responsibility Principle 简称SRP。对于单一责任原则有一个比较好的评判标准或者说是依据: 对于一个类或者接口 有且仅有一个变化的因素可以引起类或者接口的改变。我觉得在单一责任原则指导下, 我们能够尽量做到接口定义的行为符合单一责任原则、类的定义符合单一责任 以及 方法逻辑定义符合单一责任, 那么整个系统的可读性与可维护性就已经非常不错了。

### 接口的单一责任原则

接口的单一责任原则在进行接口的行为定义的时候需要考虑业务场景; 比如我们有一个连接数据库的需求, 那么我们首先定义一个数据库的相关接口:

     public interface IDBConnection {
        String getDBDriverName();
        String getDBUrl();
        String getDBUserName();
        String getDBPassword();
        Connection getDBConnection();
        String closeConnection();

        Object executeSQL(String sql)
    }

基本上该接口就不符合接口的单一责任原则, 我们可以看出 当数据库类型的改变会引起接口实现类的变化, 当数据库用户名密码改变也会引起实现类的改变, 数据库安装机器的改变也会引起实现类的变化; 那么该接口基本上是需要重新设计的; 我们基于单一责任原则对接口进行重新设计; 通过分析发现驱动程序与数据库类型相关, 而其他的基本与数据库类型无关 所以可以将getDBDriverName方法单独抽取到一个接口中, 而getDBUrl是与数据库类型和数据库实例所在的IP相关的, 所以 也可以单独抽出来作为接口, 用户名密码因为属于认证部分, 也可以单独出来成为接口, 而getDBConnection() 和 closeConnection()属于数据库连接管理的一部分, 也应该单独抽出来作为接口;  执行SQL语句接口也可以单独抽取出来，那么我们就有了5个接口;

** 数据库驱动类接口IDBDriver **

    public interface IDBDriver {
        String getDriverName();
    }

** 数据库实例访问URL类IDBUrl **

    public interface IDBUrl {
        String getDBUrl();
    }

** 数据库认证接口 IDBAuth**

    public interface IDBAuth {
        String getUserName();
        String getPassword();
    }

** 数据库管理接口 IDBConnectionManager**

    public interface IDBConnectionManager {
        Connection getDBConnection();
        void closeConnection();
    }

** SQL 语句执行接口ISQLExecutor **

    public interface ISQLExecutor {
        Object executeSQL(String sql);
    }

这样我们可以看到一个接口由刚开始的多个责任经过重构, 每一个接口的职责都非常清晰; 但是单一责任原则是最好理解, 但是又比较难实施的, 因为每一个阶段对于业务的理解不一样或者需求不一样, 对于责任的划分也是不尽相同的; 定义的某一些职责在开发的时候看的是比较合适的, 但是随着需求的变更, 也可能接口需要合并或者再次分离; 同时对于在接口上应用单一责任原则的时候 也要避免责任划分的太细而造成的 不必要的接口以及实现类数量的急剧上升。

### 类的单一责任原则

如果上述接口 都被一个类实现, 那么显然该类也是不符合单一责任原则的. 所以在实现的时候也需要根据情况来定义多个类实现上述接口;
假设当 用户定义好接口, 同时接口符合单一责任原则. 类仅仅实现了一个接口, 那么我们能说类是符合单一责任原则的么?这个也不一定, 为什么呢? 因为类中也会存在在接口中没有定义的私有方法, 用于做一些简单的功能;

比如现在有一个文件操作类, 该类主要完成文件 以及 目录创建工作；那么我们开始写代码了:

    public static class FileUtils {
        public static boolean createFile(String fileName) throws IOException {

            if (fileName == null && fileName.length() == 0) {
                return false;
            }

            File file = new File(fileName);
            boolean isSuccess = false;
            if (!file.exists()) {
                isSuccess = file.createNewFile();
            }

            return isSuccess;
        }

        public static boolean createDir(String path) throws IOException {
            if (path == null || path.length() == 0) {
                return false;
            }

            File file = new File(path);

            if (file.exists()) {
                return false;
            }

            return file.mkdirs();
        }
    }

这时候我们发现, 在两个方法中都有对字符串的是否为null 以及 是否为空字符 进行判断, 那么我们根据代码重用原则 对代码进行重构, 抽取一个公共方法, 提取对字符串的判断

    public static class FileUtils {
        public static boolean createFile(String fileName) throws IOException {

            if (checkIsEmpty(fileName)) {
                return false;
            }

            File file = new File(fileName);
            boolean isSuccess = false;
            if (!file.exists()) {
                isSuccess = file.createNewFile();
            }

            return isSuccess;
        }

        private static boolean checkIsEmpty(String fileName) {
            if (fileName == null && fileName.length() == 0) {
                return true;
            }
            return false;
        }

        public static boolean createDir(String path) throws IOException {
            if (checkIsEmpty(path)) {
                return false;
            }

            File file = new File(path);

            if (file.exists()) {
                return false;
            }

            return file.mkdirs();
        }
    }

那么我们的文件操作类里面就有了对于字符串的判断语句, 这样也就同样违背了单一责任原则; 所以我们需要将对字符串的判断移出到StringUtils类中; 

    public static class StringUtils {
        public static boolean isEmpty(Object str) {
            return str == null || "".equals(str);
        }
    }

这样就完成了类责任的单一性;

### 方法的单一责任原则

对于方法的单一责任原则, 就更好理解了; 一个方法仅做一件事; 比如在进行数据库操作的时候, 如果我们想定义数据的存储 与 更新功能, 大部分人会定义两个接口, 一个是插入操作、另外一个是更新操作;但是也有一部分人会这样写代码:

    public class UserRepository {
        public UserEntity saveOrUpdateUser(UserEntity userEntity) {

        }
    }

我们能够看到, 该方法中完成了几个操作? 最少是3个不同的功能, 第一 插入操作; 第二 更新操作 第三 查询操作; 这种代码相比于定义3个接口来说 更容易出现Bug 而且 对于后面的维护也不是很容易; 当然有人会说这么一个小功能 很好维护;是的!这仅仅是几行代码就能完成的功能相对来说还能维护, 但是当一个方法的职责很多 而且代码很多 并且有很多if语句判断的时候, 对于代码阅读、代码维护、代码扩展、函数单个功能变更都将是灾难性的;

所以, 在方法的实现上也遵从单一责任原则也是非常有必要的.

### 单一责任原则的有点

    1、类的复杂性降低, 实现的职责都有很清晰明确的定义
    2、可读性提高
    3、可维护性提高
    4、变更引起的风险降低, 不会因为对一个功能的修改影响到另外的一个或者几个功能点

对于单一责任原则的如何应用: 根据引起变化的原因来定义接口与类, 一个方法仅做一件事;