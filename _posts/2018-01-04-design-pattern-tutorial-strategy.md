---
layout: post
title: 设计模式系列之策略模式
date: 2018-01-04 21:43:00
tags: [设计准则, 策略模式]
---

### 策略模式

策略模式是一种简单的设计模式, 也叫政策模式; 策略模式主要是封装一系列的算法, 并使他们之间可以互换.

我们看一下策略模式的通用类图:

![策略模式类图](/assets/images/2018-01-04-design-pattern-strategy.png)

该类图中包含3部分:

1、Strategy接口: 负责定义算法的调用接口
2、ConcreteStrategy: 负责实现具体的算法.
3、Context类: 负责隐藏具体策略的实现细节, 屏蔽高层模块对于策略类的直接访问。

### 应用场景

一个最基本的加减法需求, 用户输入两个数字 以及 “+” 或 “-” 符号, 然后输出计算结果.

还是按照我们最原始的冲动来开发程序, 这个简单, 定义一个方法然后根据符号来判断就可以了
   
    ```java
    class ClientDemo {
        public static void main(String[] args) {
            int leftValue = 2;
            int rightValue = 10;
            String operator = "-";
            int result = calculate(leftValue, rightValue, operator);
            System.out.println("打印结果为:" + result);
        }

        public static int calculate(int leftValue, int rightValue, String operator) {
            if ("+".equals(operator)) {
                return leftValue + rightValue;
            }

            if ("-".equals(operator)) {
                return leftValue - rightValue;
            }

            return 0;
        }
    }
    ```

就这样我们很快完成了程序的开发, 现在需求变了, 我们不仅需要 “+” “-”运算, 还需要 “*” “/”运算. 如果我们要实现该需求, 那么我们需要修改calculate方法. 增加```if```判断, 这样显然不符合开闭原则. 那么我们尝试引入策略模式试试看?

首先定义策略接口Calculator

    public interface Calculator {
        int calculate(int leftValue, int rightValue);
    }

同时实现加法策略 以及 减法策略:

    public class AddCalculatorStrategy implements Calculator {

        public int calculate(int leftValue, int rightValue) {
            return leftValue + rightValue;
        }
    }

    public class MinusCalculatorStrategy implements Calculator {
        public int calculate(int leftValue, int rightValue) {
            return leftValue - rightValue;
        }
    }

同时, 我们提供一个上下文Context类用于封装上层代码对于策略类接口的直接访问

    public class CalculatorContext {
        private Calculator calculator;

        public CalculatorContext(Calculator calculator) {
            this.calculator = calculator;
        }
        
        public int calculate(int leftValue, int rightValue) {
            return this.calculator.calculate(leftValue, rightValue);
        }
    }

那么我们客户端应该如何调用呢?

    public class Client {
        public static void main(String[] args) {
            Calculator calculator = new AddCalculatorStrategy();
            CalculatorContext calculatorContext = new CalculatorContext(calculator);
            int result = calculatorContext.calculate(1, 2);
            System.out.println("计算结果为:" + result);
        }
    }

如果我们要增加乘除运算怎么办呢? 我们直接增加DivCalculatorStrategy 以及 MultiCalculatorStrateg并且实现相关算法就可以了. 对于扩展是非常方便的. 

但是我们回过头来想一想, 本来策略模式中的Context上下文类主要是想要屏蔽高层对于策略类的直接调用. 那么我们现在可以看到, 如果客户端直接调用```Calculator```的```calculate```方法也是可以的. 根本不需要```CalculatorContext```这个类:

    public class Client {
        public static void main(String[] args) {
            Calculator calculator = new AddCalculatorStrategy();
            int result = calculator.calculate(1, 2);
            System.out.println("计算结果为:" + result);
        }
    }

这样也是可以达到相同的效果的. 那么是否Context角色就可以不要了呢? 当然不是. 而刚开始也提到了Context类主要是想要屏蔽客户端对于具体策略类的直接访问. 但是在代码层面却没有做到这一点, 仍然会给调用者比较迷惑的地方, 我既然可以不使用Context类, 那我为什么要用?
而换句话说, 我们的Context类并没有实现```屏蔽上层调用对于具体策略类的直接访问``` 这个功能. 那么我们应该如何做才能让上层代码根本不用关心自己到底调用的哪个策略就可以得到结果呢? 我们可以跟简单工厂一起使用来完成该工作:

下面我们先定义一个简单工厂类, 并且把具体的策略类的public修饰符去掉

    public class CalculatorStrategyFactory {
    
        public Calculator getCalculator(String operator) {
            if ("+".equals(operator)) {
                return new AddCalculatorStrategy();
            }
            
            if ("-".equals(operator)) {
                return new MinusCalculatorStrategy();
            }
            
            return null;
        }
    }

我们看一下客户端如何实现:

    public class Client {
        public static void main(String[] args) {
            Calculator calculator = CalculatorStrategyFactory.getCalculator("+");
            CalculatorContext calculatorContext = new CalculatorContext(calculator);
            int result = calculatorContext.calculate(1, 2);
            System.out.println("计算结果为:" + result);
        }
    }

如果客户端与具体策略类不在同一个包下, 那么客户端绝对是看不到具体策略类都有哪些的, 它只需要传入业务的逻辑, 并且得到计算结果就可以了.

### 实践心得

策略模式在开发过程中, 我们应该会经常使用到, 但是我们并没有意识到自己使用的是策略模式. 但是如我们上面需求的演化过程, 策略模式基本上不会单独被使用到, 因为这样子的话, 所有的策略都需要公开给客户端使用, 这样显然对于客户端来说是不友好的. 所以一般我们会使用简单工厂或者工厂方法对策略进一步的封装之后将工厂类暴露出去. 从而达到隐藏策略类的目的.

其实在整个开发过程中, 我都有一点疑惑的地方在于: 既然简单工厂也可以完全屏蔽具体的策略, 那为什么还需要Context类呢? 如果哪位知道,麻烦留言告知一下, 谢谢~
