---
title: @Entity注解
author: Zeeny
comments: false
date: 2017-05-14
updated: 2017-05-14
tags: [Java]
categories: Java
summary: @Entity注解
---


# @Entity注解

* 必须与@Id注解 结合使用 

* 否则 No identifier specified for entity。

* name 属性  

<p>(可选)实体名称。 缺省为实体类的非限定名称。</p>   
<p>该名称用于引用查询中的实体。</p>  
<p>该名称不能是Java持久性查询语言中的保留字面值。 </p>

* 不与@Table结合的话 表名 默认为 SnakeCaseStrategy(命名策略 )为表名  
* 若使用 name属性 且没有与@Table结合 则表名为 name值的SnakeCaseStrategy(命名策略 )  
* 例如：

```  
@Entity  
public class UserEntity{...} 表名 user_entity  

@Entity(name="UE")  
public class UserEntity{...} 表名 ue  

@Entity(name="UsEntity")  
public class UserEntity{...} 表名 us_entity  
```

## 说明
 
* Java Persistence API定义了一种定义，可以将常规的普通Java对象（有时被称作POJO）映射到数据库。这些普通Java对象被称作Entity Bean。
        
* 除了是用Java Persistence元数据将其映射到数据库外，Entity Bean与其他Java类没有任何区别。事实上，创建一个Entity Bean对象相当于新建一条记录，删除一个Entity Bean会同时从数据库中删除对应记录，修改一个Entity Bean时，容器会自动将Entity Bean的状态和数据库同步。

* Java Persistence API还定义了一种查询语言（JPQL），具有与SQL相类似的特征，只不过做了裁减，以便处理Java对象而非原始的关系表。

```
@Entity 
 public class Employee implements Serializable { 
    private static final long serialVersionUID = 1L; 
    @Id 
    private Long id; 
    private String name; 
    private int age; 
    private String addree; 
    
   // Getters and Setters 
 }
```

*  如果没有 @javax.persistence.Entity 和 @javax.persistence.Id 这两个注解的话，它完全就是一个典型的 POJO 的 Java 类，现在加上这两个注解之后，就可以作为一个实体类与数据库中的表相对应。


## 映射规则：

1. 实体类必须用 @javax.persistence.Entity 进行注解；

2. 必须使用 @javax.persistence.Id 来注解一个主键；

3. 实体类必须拥有一个 public 或者 protected 的无参构造函数，之外实体类还可以拥有其他的构造函数；

4. 实体类必须是一个顶级类（top-level class）。一个枚举（enum）或者一个接口（interface）不能被注解为一个实体；

5. 实体类不能是 final 类型的，也不能有 final 类型的方法；

6. 如果实体类的一个实例需要用传值的方式调用（例如，远程调用），则这个实体类必须实现（implements）java.io.Serializable 接口。
