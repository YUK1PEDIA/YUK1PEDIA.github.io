---
layout: post
title: Spring Cache
description: 记录
tag: 笔记
---

## 什么是Spring Cache？

Spring Cache 是一个框架，实现了基于**注解**的缓存功能。

Spring Cache 提供了一层抽象，底层可以切换不同的缓存实现，例如：

- EHCache
- Caffeine
- Redis

要使用 Spring Cache 时，需要在工程的 pom 文件中导入相应的坐标：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>
	<version>2.7.3</version>
</dependency>
```



指定缓存中间件时，需要导入对应的中间件坐标，比如 Redis：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```





## 常用注解

| 注解           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| @EnableCaching | 开启缓存注解功能，通常加在启动类上                           |
| @Cacheable     | 在方法执行前先查询缓存中是否有数据，如果有数据，则直接返回缓存数据；如果没有缓存数据，调用方法并将方法返回值放到缓存中 |
| @CachePut      | 将方法的返回值放到缓存中                                     |
| @CacheEvict    | 将一条或多条数据从缓存中删除                                 |



### `@CachePut` 

```java
@PostMapping
// 使用Spring Cache缓存数据，生成的 key 为：“userCache::user.id"
@CachePut(cacheNames = "userCache", key = "#user.id")
public User save(@RequestBody User user){
    userMapper.insert(user);
    return user;
}
```

调用该方法后，redis 中生成的 key-value 为：

![1.png](https://s2.loli.net/2024/09/23/ry1cbsT9dLRoSqn.png)

key 的具体值是根据 `user` 对象的 `id` 属性计算来的

有一个 [Empty] 文件夹是因为每个冒号 ":" 代表一层关系，这里有两个冒号



### `@Cacheable` 

```java
@GetMapping
@Cacheable(cacheNames = "userCache", key = "#id")
public User getById(Long id){
    User user = userMapper.getById(id);
    return user;
}
```

调用该方法获取 user 对象时，若此时 redis 缓存中有对应的数据，则不会去查询数据库，直接由缓存返回。



### 

### `@CacheEvict`

```java
@DeleteMapping
@CacheEvict(cacheNames = "userCache", key = "#id")
public void deleteById(Long id){
    userMapper.deleteById(id);
}
```

删除数据库中对应数据的同时，清除 redis 中对应的缓存数据。