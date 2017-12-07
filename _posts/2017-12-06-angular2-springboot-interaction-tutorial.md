---
layout: post
title: 使用Angular2中的proxy访问后台服务
date: 2017-12-06 21:30:00
tags: [Angular2, Springboot]
---

前篇记录了关于如何快速搭建一个Angular2 SPA前端模块, 有了前端模块我们就需要与后台进行通讯, 同样Angular2提供了这样的机制, 下面我们来看一下如何使用Proxy与后端模块进行通信。

### 一、搭建一个最简单的SpringMVC应用

请参考 [快速搭建SpringBoot 应用](http://www.terrylmay.com/2017/09/springboot-frameworkstarter/
)

### 二、搭建一个最简单的Angular2应用

请参考[本地如何搭建Angular2前端SPA框架](http://www.terrylmay.com/2017/12/angular2-tutorial/)

### 三、开发一个最简单的获取人员列表信息的服务

#### 1、创建一个PersonModel的类

	package com.terrylmay.springboot;

	public class PersonModel {
	    private String userName;
	    private Integer age;
	    private String email;
	    private String address;

	    public PersonModel(String userName, Integer age, String email, String address) {
	        this.userName = userName;
	        this.age = age;
	        this.email = email;
	        this.address = address;
	    }

	    public String getUserName() {
	        return userName;
	    }

	    public void setUserName(String userName) {
	        this.userName = userName;
	    }

	    public Integer getAge() {
	        return age;
	    }

	    public void setAge(Integer age) {
	        this.age = age;
	    }

	    public String getEmail() {
	        return email;
	    }

	    public void setEmail(String email) {
	        this.email = email;
	    }

	    public String getAddress() {
	        return address;
	    }

	    public void setAddress(String address) {
	        this.address = address;
	    }
	}

#### 2、在创建一个RestController并且返回一个PersonModel列表

	在类的注解上加上@RequestMapping(value = "/services")

	@RequestMapping(value = "/getPersonList/{personNum}", method = RequestMethod.GET)
    public List<Map<String, Object>> getPersonList(@PathVariable(name = "personNum") Integer personNum) {
       List<Map<String, Object>> personList = new ArrayList<Map<String, Object>>();
       for (int index = 0; index < personNum; index++) {
           PersonModel personModel = new PersonModel("Hello" + index, 25, "15756394" + index + "@qq.com", "广东省深圳市");
           personList.add(BeanMap.create(personModel));
       }
       
       return personList;
    }

我们通过浏览器访问, 能够得到如下结果:

![用户列表截图](/assets/images/2017-12--6-person-list-result.png)

### 四、配置Angular的更高级的特性

#### 1、增加高级模块依赖

	1、primeng组件: 里面提供了丰富的UI组件;
	$ cnpm install primeng -S
	2、http模块: 对于http的封装, 简单易用
	$ cnpm install http -S
	3、annimation组件
	$ cnpm install @angular/animations --save
	4、font-awesome依赖
	$ cnpm install font-awesome -S

#### 2、在.angular-cli.json文件的styles模块中增加primeng的css依赖

	{
		...,
		"styles": [
			"styles.css",
	        "../node_modules/_primeng@5.0.2@primeng/resources/themes/omega/theme.css",
	        "../node_modules/_primeng@5.0.2@primeng/resources/primeng.min.css",
	        "../node_modules/_font-awesome@4.7.0@font-awesome/css/font-awesome.min.css"
		],
		...
	}

其中具体的版本换成已安装好的

#### 3、在项目根目录下创建一个proxy.config.json的文件, 并将如下内容拷贝到文件中

	{
		"/services": {
			"target": "http://127.0.0.1:9080",
			"secure": false
		}
	}

这样当我们在Angular程序中使用http访问/services/getPersonList/10这个访问路径的时候, angular2的proxy模块就会帮我们把请求转发到真实的http服务器。

### 五、开发用户列表显示功能

#### 1、在AppModule中增加Primeng的声明

	import {DataTableModule,SharedModule} from 'primeng/primeng';

并且在imports中加入DataTableModule 以及 SharedModule模块

	import { BrowserModule } from '@angular/platform-browser';
	import {BrowserAnimationsModule} from '@angular/platform-browser/animations';
	import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

	import { DataTableModule, SharedModule } from 'primeng/primeng';

	import { HttpModule } from '@angular/http';
	import { AppComponent } from './app.component';
	import { PersonService } from './person.service';

	@NgModule({
	  declarations: [
	    AppComponent
	  ],
	  imports: [
	    BrowserModule,
	    HttpModule,
	    DataTableModule,
	    SharedModule
	  ],
	  providers: [PersonService],
	  bootstrap: [AppComponent],
	  schemas: [CUSTOM_ELEMENTS_SCHEMA]
	})
	export class AppModule { }

因为使用自定义的元素, 所以需要将schema设置为CUSTOM_ELEMENTS_SCHEMA

#### 2、创建一个person-model.ts的文件

	$ ng generate class PersonModel

并将如下内容拷贝到文件中

	export class PersonModel {
	    constructor(public userName: string, public age: number,
	                public email: string, public address: string)
	    {}

	}

#### 3、创建一个person.service.ts文件

	$ ng generate service person

如果上面创建service命令报错, 那么就手工创建一个文件,并将如下内容拷贝到文件中

	import { Injectable } from '@angular/core';
	import { Http, Response } from '@angular/http';
	import { PersonModel } from './person-model';

	@Injectable()
	export class PersonService {

	    constructor(private http: Http) {}

	    getPersonList() {
	        return this.http.get('/services/getPersonList/10')
	                    .toPromise()
	                    .then(res => <PersonModel[]> res.json())
	                    .then(data => data);
	    }
	}

#### 4、将如下内容拷贝到app.component.ts中

	import { Component } from '@angular/core';
	import { PersonModel } from './person-model';
	import { PersonService } from './person.service';

	@Component({
	  selector: 'app-root',
	  templateUrl: './app.component.html',
	  styleUrls: ['./app.component.css']
	})
	export class AppComponent {
	  title = 'app';

	  personList: PersonModel[];

	  constructor(private personService: PersonService) { }

	  ngOnInit() {
	      this.personService.getPersonList().then(persons => {
	        this.personList = persons;
	        console.log(persons);
	      });
	  }
	}


#### 5、将界面元素拷贝到app.component.html中

	<div>
	    <p-dataTable [value]="personList">
	        <p-column field="userName" header="userName"></p-column>
	        <p-column field="age" header="age"></p-column>
	        <p-column field="email" header="email"></p-column>
	        <p-column field="address" header="address"></p-column>
	    </p-dataTable>
	</div>


### 六、运行程序查看效果

![运行截图](/assets/images/2017-12-07-person-list-ui-result.png)

到这里，基本上一个在本地可运行的Angular以及SpringBoot应用的交互框架就已经搭建完成了。









