---
layout: post
title: SpringBoot中数据访问JPA的使用
date: 2017-09-14 21:30:00
tags: [SpringBoot, JPA]
---

Spring Data JPA除了上文提到的使用固定的名称查找数据, 还有其他很多种查询数据或者删除数据的高阶语法可以使用.

在开始使用一些其他JPA的API之前, 我们先看一下JpaRepository接口的定义, 以便更好的了解JPA接口中实现的功能

### JpaRepository接口源码

JpaRepository 接口如下:


	package org.springframework.data.jpa.repository;

	import java.io.Serializable;
	import java.util.List;
	import org.springframework.data.domain.Example;
	import org.springframework.data.domain.Sort;
	import org.springframework.data.repository.NoRepositoryBean;
	import org.springframework.data.repository.PagingAndSortingRepository;
	import org.springframework.data.repository.query.QueryByExampleExecutor;

	@NoRepositoryBean
	public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
	    List<T> findAll();

	    List<T> findAll(Sort var1);

	    List<T> findAll(Iterable<ID> var1);

	    <S extends T> List<S> save(Iterable<S> var1);

	    void flush();

	    <S extends T> S saveAndFlush(S var1);

	    void deleteInBatch(Iterable<T> var1);

	    void deleteAllInBatch();

	    T getOne(ID var1);

	    <S extends T> List<S> findAll(Example<S> var1);

	    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
	}

其中定义了很多基本的增删改查的功能. 比如查找所有记录、按照某个字段排序查找、批量保存数据、清空缓存(commit数据)、保存并提交数据、批量删除数据、批量全部删除、根据Id获取数据等等功能, 我们的Repository只需要继承该接口, 即可实现上面接口中定义的所有功能。

### 定义查询方法

#### 根据属性名查找

Spring Jpa支持通过定义在Repository接口中的方法名来定义查询, 方法名是根据实体类的属性名来确定的。

1、根据属性名来定义查询方法, 如下:

	package com.terrylmay.springboot.usermodule.repository;

	import com.terrylmay.springboot.usermodule.entity.UserEntity;
	import org.springframework.data.jpa.repository.JpaRepository;

	public interface UserRepository extends JpaRepository<UserEntity, Integer> {

		# 根据UserEntity中的username来查找数据, username接到findBy后面使用驼峰命名法即可
		# 相当于hql select * from user u where u.username=?1
	    public UserEntity findByUsername(String username);

	    # 根据多个属性名来查询 使用And来连接
	    # 相当于hql select * from user u where u.username=?1 and u.email=?2
	    UserEntity findByUsernameAndEmail(String userName, String email);

	    # 相当于hql select * from user u where u.username like ?1
	    List<UserEntity> findByUsernameLike(String username);

	}

从代码中可以看出, 使用findBy、And、Like等字样就可以分别匹配Sql语句中的查询关键字.同时, findBy也可以使用find、read、queryBy、 get、 getBy等关键字来代替。

2、限制结果数量使用Top或者First关键字实现

比如实现查找前10个数据

	List<UserEntity> findTop10ByUsernameLike(String userName);

	List<UserEntity> findFirst10ByUsername(String userName);

3、使用JPA的NamedQuery查询

JPA允许用户使用NamedQuery来定义查询方法即通过一个名字映射一个SQL语句
我们在UserEntity上加上如下注释:

	@Entity(name = "user")
	@NamedQuery(name="UserEntity.password", query = "select * from user u where password=?1")
	public class UserEntity

然后在UserRepository中加入如下方法:

	UserEntity password(String password);

这里就会调用NamedQuery中的SQL语句, 并且返回UserEntity相关实例

4、使用@Query查询

@Query查询有多种方式, 使用参数索引方式查询、使用占位符与参数名绑定查询、Specification查询

使用参数索引查询方式代码如下:

	@Query(value = "select * from user where username=?1")
    UserEntity findByUsername(String username);

 该代码中, ?1相当于占位符 表示下面方法中的第一个参数就是我要指定的数据, 假设方法传入的参数为"terry", 那么相应的程序就会执行SQL select * from user where username='terry'

 使用占位符与参数名绑定查询

 	@Query(value = "select * from user where username=:username and password=:password")
    UserEntity findByUserNameAndPassword(@Param(value = "username") String username, @Param(value = "password") String password);

通过@Param中的value来表示sql语句中:后面的参数需要指定的值。这样写法的好处就是可以直观的看到哪个变量代表了哪个sql条件, 同时函数的参数顺序是可以随意调整的。

5、排序查询

	List<UserEntity> findAllByUsername(String username, Sort sort);

使用该接口的时候

	userRepository.findAllByUsername("terrylmay", new Sort(Direction.ASC, "id"));

6、分页查询

	Page<UserEntity> findAllByUsername(String username, Pageable pageable);

使用该接口的时候:

	userRepository.findAllByUsername("terrylmay", new PageRequest(0, 10);

查找第0页的10个数据。

7、比较复杂的查询方法Specification

JPA提供了基于准则查询的方式, 即Criteria查询.同时Spring Data JPA提供了一个Specification接口让我们更加方便的构造准则查询。

首先, 我们的接口中必须实现JpaSpecificationExecutor接口, 代码如下:

	public interface UserRepository extends JpaRepository<UserEntity, Integer>, JpaSpecificationExecutor<UserEntity> {}

然后 我们需要定义Criterial查询

	package com.terrylmay.springboot.usermodule;

	import com.terrylmay.springboot.usermodule.entity.UserEntity;
	import org.springframework.data.jpa.domain.Specification;

	import javax.persistence.criteria.CriteriaBuilder;
	import javax.persistence.criteria.CriteriaQuery;
	import javax.persistence.criteria.Predicate;
	import javax.persistence.criteria.Root;

	public class CriteriaSpecification {

	    public static Specification<UserEntity> usernameEqual(final String username) {
	        return new Specification<UserEntity>() {
	            public Predicate toPredicate(Root<UserEntity> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
	                return criteriaBuilder.equal(root.get("username"), username);
	            }
	        };
	    }
	}

CriteriaBuilder、CriteriaQuery中的方法对应的sql语句中的关键字, 到后面用到的时候再找相应的对应关系。

到这里基本上JPA的查询相关操作就已经差不多了。

