---
layout: post
title: 字符串包含实现
date: 2016-02-21 10:35:05
tags: [字符串]
---

Hello World中是否包含这样的字符串HelloWd,一般会使用多次循环来遍历判断是否存在某个字符.下面我通过使用正则表达式的思想来实现这个功能

	public static void main(String args[]) {  
	    String stringStr = "CDMter";  
	    String key = "CaseDoesMatter";  
	    judgeSequence(stringStr, key);  
	}  

	public static void judgeSequence(String s, String t) {  
	    String regex = "";  
	    for (int i = 0; i < s.length() - 1; i++) {  
	        regex = regex + s.charAt(i) + "[a-zA-Z]*";  
	    }  
	    regex += s.charAt(s.length() - 1);  
	    Pattern pt = Pattern.compile(regex);  
	    Matcher mch = pt.matcher(t);  
	    if (mch.find()) {  
	        System.out.println("YES");  
	    } else {  
	        System.out.println("NO");  
	    }  
	}  
