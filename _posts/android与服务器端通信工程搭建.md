---
title: android与服务器端通信工程搭建
date: 2016-02-21 10:46:37
tags: [android, 服务器端, 通信]
---

最近要做一个项目,所以进行了一个小测试.在网上看到的很多Demo，全部是在onCreate()方法中进行连接以及请求服务器端的数据并且在控件中显示的,但是请教了别人之后才知道这种方法现在已经不能用了，必须在主线程中开启一个线程，并且使用Handler这个对象来实现数据的异步请求.

<!-- more -->

然后当请求响应完成之后才会在界面中更新数据,这应该就是android中的异步请求机制吧..下面是整个服务器端以及客户端的代码.

首先我服务器端是通过struts来完成的.所以关于struts的jar包以及Json包所依赖的jar包都要导入到web程序中去.以下是服务器端的包图
web.xml文件中要配置过滤器,我是把所有请求都通过struts中的action进行处理的,所以下面是web.xml中的代码

	<?xml version="1.0" encoding="UTF-8"?>  
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
	    xmlns="http://java.sun.com/xml/ns/javaee"  
	    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
	    id="WebApp_ID" version="3.0">  
	    <display-name>YltxServer</display-name>  
	    <welcome-file-list>  
	        <welcome-file>index.html</welcome-file>  
	        <welcome-file>index.htm</welcome-file>  
	        <welcome-file>index.jsp</welcome-file>  
	        <welcome-file>default.html</welcome-file>  
	        <welcome-file>default.htm</welcome-file>  
	        <welcome-file>default.jsp</welcome-file>  
	    </welcome-file-list>  
	    <!-- 定义Struts2的核心控制器：FilterDispatcher -->  
	    <filter>  
	        <!-- 定义核心Filter的名称 -->  
	        <filter-name>struts2</filter-name>  
	        <!-- 定义Filter的实现类 -->  
	        <filter-class>org.apache.struts2.dispatcher.FilterDispatcher</filter-class>  
	    </filter>  

	    <filter-mapping>  
	        <filter-name>struts2</filter-name>  
	        <url-pattern>/*</url-pattern>  
	    </filter-mapping>  
	</web-app>  

同时按照struts中的规则，创建一个struts.xml文件，并注册自己写的action

	package com.maylor.action;  

	import javax.servlet.http.HttpServletRequest;  
	import javax.servlet.http.HttpServletResponse;  

	import net.sf.json.JSONArray;  
	import net.sf.json.JSONObject;  

	import org.apache.struts2.interceptor.ServletRequestAware;  
	import org.apache.struts2.interceptor.ServletResponseAware;  

	import com.opensymphony.xwork2.ActionSupport;  

	public class LoginAction extends ActionSupport implements ServletRequestAware,  
	        ServletResponseAware {  
	    /** 
	     *  
	     */  
	    private static final long serialVersionUID = 1L;  
	    private HttpServletRequest request;  
	    private HttpServletResponse response;  

	    public void login() {  
	        try {  
	            // HttpServletRequest request =ServletActionContext.getRequest();  
	            // HttpServletResponse response=ServletActionContext.getResponse();  
	            this.response.setContentType("text/html;charset=utf-8");  
	            this.response.setCharacterEncoding("UTF-8");  
	            // 将要返回的实体对象进行json处理  
	            // JSONObject json=JSONObject.fromObject(this.getUsername());  
	            // 输出格式如：{"id":1, "username":"zhangsan", "pwd":"123"}  
	            // System.out.println(json);  

	            // this.response.getWriter().write(json.toString());  

	            JSONObject json = new JSONObject();  
	            json.put("username", "username");  
	            json.put("password", "password");  
	            JSONObject json1 = new JSONObject();  
	            json1.put("phone", "phone");  
	            json1.put("tel", "tel");  
	            JSONArray array = new JSONArray();  
	            array.add(json);  
	            array.add(json1);  
	            byte[] jsonBytes = array.toString().getBytes("utf-8");  
	            response.setContentLength(jsonBytes.length);  
	            response.getOutputStream().write(jsonBytes);  

	            /** 
	             * JSONObject json=new JSONObject(); json.put("login", "login"); 
	             * byte[] jsonBytes = json.toString().getBytes("utf-8"); 
	             * response.setContentType("text/html;charset=utf-8"); 
	             * response.setContentLength(jsonBytes.length); 
	             * response.getOutputStream().write(jsonBytes); 
	             * response.getOutputStream().flush(); 
	             * response.getOutputStream().close(); 
	             **/  

	        } catch (Exception e) {  
	            e.printStackTrace();  
	        }  
	        // return null;  
	    }  

	    @Override  
	    public void setServletResponse(HttpServletResponse arg0) {  
	        // TODO Auto-generated method stub  
	        this.response = arg0;  
	    }  

	    @Override  
	    public void setServletRequest(HttpServletRequest arg0) {  
	        // TODO Auto-generated method stub  
	        this.request = arg0;  
	    }  
	}  


然后就是客户端的请求了.

下面是android客户端的代码,记住要在AndroidMainifest.xml中加入Internet访问权限

	<uses-permission android:name="android.permission.INTERNET">

	</uses-permission>

下面就是客户端的代码部分了.

	package com.maylor.demo;  

	import java.io.BufferedReader;  
	import java.io.IOException;  
	import java.io.InputStream;  
	import java.io.InputStreamReader;  

	import org.apache.http.HttpEntity;  
	import org.apache.http.HttpResponse;  
	import org.apache.http.client.ClientProtocolException;  
	import org.apache.http.client.HttpClient;  
	import org.apache.http.client.methods.HttpGet;  
	import org.apache.http.impl.client.DefaultHttpClient;  
	import org.json.JSONArray;  
	import org.json.JSONException;  
	import org.json.JSONObject;  

	import android.annotation.SuppressLint;  
	import android.app.Activity;  
	import android.os.Bundle;  
	import android.os.Handler;  
	import android.os.Message;  
	import android.util.Log;  
	import android.view.Menu;  
	import android.widget.TextView;  

	public class MainActivity extends Activity {  
	    private static String URL = "http://192.168.2.52:8080/YltxServer/login";  
	    public Handler mHandler;  

	    @SuppressLint("HandlerLeak")  
	    @Override  
	    protected void onCreate(Bundle savedInstanceState) {  
	        super.onCreate(savedInstanceState);  
	        setContentView(R.layout.activity_main);  
	        getPDAServerData();  
	        // Button button = (Button) findViewById(R.id.button1);  
	        // button.setOnClickListener(new OnClickListener() {  
	        //  
	        // @Override  
	        // public void onClick(View arg0) {  
	        // // TODO Auto-generated method stub  
	        // Intent intent = new Intent(MainActivity.this,  
	        // PoiSearchActivity.class);  
	        // startActivity(intent);  
	        // }  
	        // });  
	        mHandler = new Handler() {  
	            @Override  
	            public void handleMessage(Message msg) {  
	                if (msg.what == 1000) {  
	                    TextView view = (TextView) findViewById(R.id.textview);  
	                    String str = "";  
	                    try {  
	                        JSONArray array = new JSONArray(msg.obj.toString());  
	                        JSONObject object = array.getJSONObject(0);  
	                        str = object.getString("username");  
	                    } catch (JSONException e) {  
	                        // TODO Auto-generated catch block  
	                        e.printStackTrace();  
	                    }  
	                    Log.e("wangchao", str);  
	                    view.setText(str);  
	                }  
	            }  
	        };  
	    }  

	    /** 
	     * 请求服务 
	     *  
	     * @param url 
	     */  
	    private void getPDAServerData() {  
	        new Thread() {  
	            public void run() {  
	                HttpClient client = new DefaultHttpClient();  
	                HttpGet request;  
	                String msg = "";  
	                try {  
	                    request = new HttpGet(URL);  
	                    HttpResponse response = client.execute(request);  
	                    // 判断请求是否成功  
	                    if (response.getStatusLine().getStatusCode() == 200) {  
	                        HttpEntity entity = response.getEntity();  
	                        if (entity != null) {  
	                            InputStream in = entity.getContent();  

	                            BufferedReader buff = new BufferedReader(  
	                                    new InputStreamReader(in));  
	                            String line = "";  
	                            while ((line = buff.readLine()) != null) {  
	                                msg += line;  
	                            }  
	                            Message msg1 = mHandler.obtainMessage();  
	                            msg1.what = 1000;  
	                            msg1.obj = msg;  
	                            mHandler.sendMessage(msg1);  
	                        }  
	                    }  
	                } catch (ClientProtocolException e) {  
	                    e.printStackTrace();  
	                } catch (IOException e) {  
	                    e.printStackTrace();  
	                }  
	            }  
	        }.start();  
	    }  

	    @Override  
	    public boolean onCreateOptionsMenu(Menu menu) {  
	        // Inflate the menu; this adds items to the action bar if it is present.  
	        getMenuInflater().inflate(R.menu.main, menu);  
	        return true;  
	    }  
	}

这个服务就算是全部完成了..整个工程