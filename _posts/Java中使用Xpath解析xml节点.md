---
title: Java中使用Xpath解析xml节点
date: 2016-02-21 10:50:01
tags: [XML, XPath]
---

	DocumentBuilderFactory domFactory = DocumentBuilderFactory  
	                .newInstance();  
	DocumentBuilder builder = domFactory.newDocumentBuilder();  
	domFactory.setNamespaceAware(false);//这句话可有可无,但是不知道为什么设置成true就无法解析了,可能跟命名空间有关吧..以后再研究 

	Document doc = builder.parse("config1.xml");  
	XPathFactory factory = XPathFactory.newInstance();  
	XPath xpath = factory.newXPath();  
	XPathExpression expr = xpath.compile("//name/text()");  
	Object result = expr.evaluate(doc, XPathConstants.NODESET);  
	NodeList nodes = (NodeList) result;  

	for (int i = 0; i < nodes.getLength(); i++) {  
	    System.out.println(nodes.item(i).getNodeValue());  
	}  