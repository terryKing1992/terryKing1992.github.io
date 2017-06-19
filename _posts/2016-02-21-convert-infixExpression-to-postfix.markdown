---
layout: post
title: 中缀表达式转后缀表达式
date: 2016-02-21 09:55:33
tags: [中缀转后缀, 表达式求值]
---

中缀表达式到后缀表达式的转换、后缀表达式求值 思路 以及 Java代码实现
要把表达式从中缀表达式的形式转换成用后缀表示法表示的等价表达式，必须了解操作符的优先级和结合性。优先级或者说操作符的强度决定求值顺序；优先级高 的操作符比优先级低的操作符先求值。


如果所有操作符优先级一样，那么求值顺序就取决于它们的结合性。操作符的结合性定义了相同优先级操作符组合的顺序（从右至左或从左至右）。

转换过程包括用下面的算法读入中缀表达式的操作数、操作符和括号：

	1. 初始化一个空堆栈，将结果字符串变量置空。
	2. 从左到右读入中缀表达式，每次一个字符。
	3. 如果字符是操作数，将它添加到结果字符串。
	4. 如果字符是个操作符，弹出（pop）操作符，直至遇见开括号（opening parenthesis）、优先级较低的操作符或者同一优先级的右结合符号。把这个操作符压入（push）堆栈。
	5. 如果字符是个开括号，把它压入堆栈。
	6. 如果字符是个闭括号（closing parenthesis），在遇见开括号前，弹出所有操作符，然后把它们添加到结果字符串。
	7. 如果到达输入字符串的末尾，弹出所有操作符并添加到结果字符串。

后缀表达式求值

对后缀表达式求值比直接对中缀表达式求值简单。在后缀表达式中，不需要括号，而且操作符的优先级也不再起作用了。您可以用如下算法对后缀表达式求值：

	1. 初始化一个空堆栈
	2. 从左到右读入后缀表达式
	3. 如果字符是一个操作数，把它压入堆栈。
	4. 如果字符是个操作符，弹出两个操作数，执行恰当操作，然后把结果压入堆栈。如果您不能够弹出两个操作数，后缀表达式的语法就不正确。
	5. 到后缀表达式末尾，从堆栈中弹出结果。若后缀表达式格式正确，那么堆栈应该为空。

前面理论知识引用自：http://blog.csdn.net/arduousbonze/article/details/3128084
最近在做IOS，刚刚入门，想动手学习一下关于UICollectionView的使用，就自己写了一个关于计算器的实现，不过因为公司的各种限制，代码拿不出来，又没办法写博客，只好晚上回家写了.下面给出java关于算术表达式的实现：

	```java
	Stack<Character> stack = new Stack<Character>();  
	  String str = "3+4+5/6";  
	  StringBuffer stringBuffer = new StringBuffer();  
	  for (int i = 0; i < str.length(); i++) {  
	    if(Character.isDigit(str.charAt(i))) {  
	        stringBuffer.append(str.charAt(i));  
	    }else {  
	        boolean isHighLevel = str.charAt(i) == '*' || str.charAt(i) =='/';  
	        Character ch = null;  
	        if(stack.isEmpty()) {  
	            stack.push(str.charAt(i));  
	        }else if(isHighLevel && ((ch =stack.pop()) == '+' || ch =='-')){  
	            stack.push(ch);  
	            stack.push(str.charAt(i));  
	        }else {  
	            stringBuffer.append(stack.pop());  
	            stack.push(str.charAt(i));  
	        }  
	    }  
	  }  

	  while (!stack.isEmpty()) {  
	    stringBuffer.append(stack.pop());  
	  }  
	  System.out.println(stringBuffer.toString());  

	  String afterString = stringBuffer.toString();  
	  Stack<Double> intStack = new Stack<Double>();  
	  double result=0;  
	  for (int i = 0; i < afterString.length(); i++) {  
	    if(Character.isDigit(afterString.charAt(i))) {  
	        intStack.push(Double.valueOf(afterString.charAt(i)+""));  
	    }else {  
	        Double top = intStack.pop();  
	        Double second = intStack.pop();  

	        switch(afterString.charAt(i))  
	        {  
	        case '*':  
	            result = top * second;  
	            break;  
	        case '/':  
	            result = second / top;  
	            break;  
	        case '+':  
	            result = top + second;  
	            break;  
	        case '-':  
	            result = second - top;  
	            break;  
	        }  
	        intStack.push(result);  
	    }  
	  }  
	  System.out.println(intStack.pop());  
	}
	```

不过这种写法只能够解决输入为整数的情况，输入为double的情况还没有考虑，不过实现大同小异，只不过可能IOS里面没有栈的实现，可能要自己通过NSMutableArray来实现上面的方法了，初步设想是通过NSMutableArray 和NSNumber来实现，不过具体情况还得做过了再看.
