---
layout: post
title: spring boot中集成mybatis通过注解方式增删改查
date: 2019-08-21
author: xiepl1997
tags: springboot
---

springboot集成了mybatis后，就不需要繁琐的xml配置了，只需要一个Mapper类。  

增删改查对应的注解分别为@Insert、@Delete、@Update、@Select。  
```java
/**
 *@Mapper将pojo声明为一个Mapper接口
 *@Results是结果映射列表，@Result中property是pojo类的属性名，column是数据库表的字段名
 *@Select、@Insert分别代表执行了真实的SQL
 */
@Mapper
public interface UserMapper{
	
	@Select("select * from user where id = #{id}")
	public User findById(@Param("id") String id);

	//当pojo类中属性与数据库中字段名不一致时使用@Results
	@Select("select name, age from user")
	@Results({
		@Result(property = "name", column = "name"),
		@Result(property = "age", column = "age")
	})
	List<User> findAll();

	@Insert("insert into user(name, age) values(#{name}, #{age})")
	int insert(@Param("name") String name, @Param("age") Integer age);

	//以实体类型接受参数
	@Insert("insert into user(name, age) values(#{name}, #{age})")
	int insertByUser(User user);

	//以map形式接受参数
	@Insert("insert into user(name, age) values(#{name, jdbcType=VARCHAR}, #{age, jdbcType=INTEGER})")
	int insertByMap(Map<String, Integer> map);

	@Update("update user set age = #{age} where name = #{name}")
	int update(@Param("age") Integer age, @Param("name") String name);

	@Delete("delete from user where id = #{id}")
	int deleteById(@Param("id") String id);

}
```