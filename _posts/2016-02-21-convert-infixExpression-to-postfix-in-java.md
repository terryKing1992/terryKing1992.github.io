---
layout: post
title: 中缀转后缀代码实现
date: 2016-02-21 10:01:29
tags: [表达式求值]
---

最近上一篇说到了关于算术表达式求值问题，上面已经大致有了思路，因为关于IOS的实现是在公司电脑上面写的，基本上已经知道怎么实现了,IOS可以实现关于多位数的运算，而下面是两种语言的实现。不过都只是实现了关于10以内的数字的运算.包括java实现以及c++语言的实现。

java实现：

	Stack stack = new Stack();
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

C++实现：

	#include "stdafx.h"  
	#include <stack>  
	#include<iostream>  
	#include <ctype.h>  
	using namespace std;  
	void change(stack<char> & stack1,char * c,int size);  
	bool isoperator(char c);  
	int priorityLevel(char c);  
	int _tmain(int argc, _TCHAR* argv[])  
	{  
	    stack<char> stack1;  
	    stack<char> stack2;  
	    char * ch="(1+2)*3+4*6=";  
	    change(stack1,ch,strlen(ch));  
	    return 0;  
	}  
	void change(stack<char> & stack1,char * ch ,int size)  
	{  
	    for(int i=0;i<size;i++)  
	    {  
	        char c=ch[i];  
	        if(isdigit(c))  
	        {  
	         cout<<c<<" ";  
	        }else if(isoperator(c))  
	        {  
	         if(stack1.empty())  
	         {  
	            stack1.push(c);  
	         }else if(priorityLevel(c)<=priorityLevel(stack1.top()))  
	         {  
	           cout<<stack1.top()<<" ";  
	           stack1.pop();  
	           stack1.push(c);  
	          }else if(priorityLevel(c)>priorityLevel(stack1.top()) && priorityLevel(c)!=4 )  
	         {  
	            stack1.push(c);  
	          }else if(priorityLevel(c)>priorityLevel(stack1.top()) && priorityLevel(c)==4 )  
	         {  
	            while(!stack1.empty() && stack1.top()!='(')  
	            {  
	                cout<<stack1.top()<<" ";  
	                stack1.pop();  
	            }  
	            if(stack1.empty())  
	            {  
	                exit(0);  
	            }  
	          }  
	       }else  
	       {  
	          while(!stack1.empty())  
	            {  
	                cout<<stack1.top()<<" ";  
	                stack1.pop();  
	            }  
	       }  
	    }  
	}  
	bool isoperator(char c)  
	{  
	    if(c=='+'|| c=='-'|| c=='*' || c=='/' || c=='(' ||c==')')  
	    {  
	        return true;  
	    }else  
	    {  
	        return false;  
	    }  
	}  
	int priorityLevel(char c)  
	{  
	    switch(c)  
	    {  
	    case '+':  
	    case '-':  
	        return 1;  
	    case '*':  
	    case '/':  
	        return 2;  
	    case '(':  
	        return 0;  
	    case ')':  
	        return 4;  
	    default:  
	        return -1;  
	    }  
	}  

回头可以考虑一下关于多位数运算java和c语言的实现。IOS能够实现主要是有一个系统函数：[123+3/2 doubleValue] 就可以返回123这个,而不用自己去解析之后再处理.
