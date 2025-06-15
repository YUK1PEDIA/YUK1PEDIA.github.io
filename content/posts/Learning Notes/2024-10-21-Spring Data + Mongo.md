---
layout: post
title: Spring Data + Mongo
description: 记录
tag: 笔记
---

## 1.什么是 Spring Data？

Spring Data 是 Spring 框架中的一部分，专注于简化数据访问层的开发。其核心目标是减少开发人员编写的数据访问层代码，并为常见的数据访问模式提供开箱即用的解决方案。Spring Data 支持多种数据存储技术，如关系数据库（通过 JPA）、NoSQL 数据库（如 MongoDB、Cassandra 等）。

**主要功能如下：**

1. **简化的 CRUD 操作**：通过 Spring Data，开发者可以轻松实现基本的 CRUD（增删查改）操作，无需编写复杂的 SQL 语句。

2. **Repository 接口**：定义简单的 Repository 接口，并通过 Spring Data 自动提供实现。例如，可以定义一个继承 `JpaRepository` 的接口，Spring Data JPA 会自动生成常见的 CRUD 方法。

3. **自定义查询方法**：通过方法名称来定义查询，Spring Data 会自动解析方法名并生成相应的 SQL。例如，方法名为 `findByName` 会自动生成一个按 `name` 字段查询的 SQL 语句。

4. **分页和排序**：通过 `Pageable` 接口，轻松实现分页查询和结果排序。

5. **支持事务管理**：Spring Data 与 Spring 的事务管理无缝集成，开发者可以使用注解来管理事务，确保数据的安全性和一致性。





## 2.什么是 JPA？

JPA 是 Java 中的一个标准规范，用于**管理 Java 对象与关系数据库之间的映射**。JPA 规范本身并不是具体的实现，而是定义了一套用于**对象-关系映射（ORM）**的 API。

**JPA 核心概念：**

1. **实体（Entity）**：一个被映射到数据库表的普通 Java 类。通过 `@Entity` 注解将类声明为一个 JPA 实体，并通过其他注解指定其与数据库表之间的关系。

2. **实体管理器（EntityManager）**：JPA 的核心接口，负责实体的生命周期管理，如持久化、更新、删除、查询等操作。

3. **JPQL（Java Persistence Query Language）**：一种面向对象的查询语言，类似于 SQL，但操作的是 Java 对象而不是数据库表。

4. **注解映射**：

   - `@Entity`：标识一个类为实体类。

   - `@Table`：定义实体类对应的数据库表。

   - `@Id`：定义实体类的主键。

   - `@GeneratedValue`：定义主键的生成策略。

   - `@ManyToOne`，`@OneToMany`，`@ManyToMany`，`@OneToOne`：定义实体类之间的关系。



## 3.Spring Data JPA

Spring Data JPA 是 Spring Data 项目的一部分，它是对 JPA 的进一步封装和扩展。它简化了 JPA 的使用，开发者可以通过 Repository 接口的方式直接与数据库交互，而无需手动编写 EntityManager 相关的代码。

**Spring Data JPA 核心特点：**

1. **简化的 Repository 模型**：通过继承 `JpaRepository` 或 `CrudRepository`，可以轻松获取基本的 CRUD 操作和分页、排序功能。

2. **动态查询**：使用**方法名称**查询，可以通过定义以 `findBy` 开头的方法，并指定实体类的字段来实现动态查询。

3. **自定义查询**：通过 `@Query` 注解，可以编写 JPQL 或原生 SQL 来进行自定义查询。

4. **与事务的集成**：通过 `@Transactional` 注解，开发者可以对方法的事务行为进行管理。

以下面的代码为例：

- 定义实体类

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
    
    // getters and setters
}
```

- 定义 Repository 接口

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByName(String name);
}
```

- 在服务中使用

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;
    
    public List<User> getUsersByName(String name) {
        return userRepository.findByName(name);
    }
    
    public void saveUser(User user) {
        userRepository.save(user);
    }
}
```



## 4.什么是 Mongo？

MongoDB 是一个 NoSQL 数据库，具有高性能、灵活的数据模型和高可用性，适用于存储海量、半结构化或非结构化数据。MongoDB 采用 BSON（二进制 JSON）格式存储数据，支持丰富的查询功能、内嵌文档和多种索引方式。**MongoDB 写入数据的速度相较于 MySQL要快非常多。**

**Mongo 主要特点**

1. **无模式（Schema-less）**：MongoDB 不要求预先定义数据模式，数据的结构可以随着应用需求变化。

2. **文档存储**：MongoDB 以 BSON 文档的形式存储数据，每条记录相当于一个 JSON 对象，非常灵活且支持嵌套数据。

3. **高扩展性**：支持水平扩展，容易进行数据分片（Sharding），使其可以轻松处理海量数据和高并发请求。

4. **内嵌文档和数组**：允许文档内嵌其他文档或数组，减少了跨集合查询和联表查询的需求。

5. **丰富的查询功能**：支持复杂的查询、过滤、排序、分页等操作，还支持地理空间查询和全文搜索。

6. **强大的索引功能**：支持创建多种索引，如单字段索引、复合索引、地理空间索引、全文索引等。



## 5.Spring Data Mongo

Spring Data MongoDB 是 Spring Data 项目的一部分，专门为 MongoDB 提供数据访问的支持。它封装了对 MongoDB 数据库的访问，使得开发者可以使用类似 JPA 的方式与 MongoDB 交互。

**Spring Data MongoDB 的主要功能:**

1. **Repository 支持**：通过定义 Repository 接口，Spring Data MongoDB 会自动生成常见的 CRUD 方法，类似于 Spring Data JPA 的 `JpaRepository`。
2. **MongoTemplate**：提供了灵活的模板类，可以执行自定义的查询、更新、删除等操作。与 Repository 的自动生成方式不同，`MongoTemplate` 更加灵活，可以编写复杂的查询逻辑。
3. **方法查询派生**：通过方法名派生查询功能，开发者可以定义以 `findBy` 开头的方法，Spring Data MongoDB 会自动根据方法名生成查询语句。例如，`findByUsername` 会自动生成查询 `username` 字段的 MongoDB 查询。
4. **自定义查询**：可以通过 `@Query` 注解编写自定义的 MongoDB 查询，支持使用 JSON 格式或者 Spring 的查询表达式来构建查询。
5. **分页与排序**：通过 `Pageable` 接口，Spring Data MongoDB 提供了分页和排序功能，简化了处理大量数据的场景。
6. **聚合查询**：Spring Data MongoDB 支持 MongoDB 的聚合查询功能，可以执行复杂的数据统计和汇总操作。
7. **事务支持**：在 MongoDB 4.x 及以上版本，Spring Data MongoDB 支持多文档的事务操作，确保数据的一致性。

#### Spring Data MongoDB 主要组件

1. **Repository 接口**：类似于 Spring Data JPA，开发者只需要定义接口，并继承 `MongoRepository`，就可以自动生成 CRUD 操作。
2. **MongoTemplate**：提供了对 MongoDB 的底层操作，如插入、更新、删除、查询等。适用于需要更复杂查询逻辑或自定义操作的场景。
3. **@Document** 注解：将 Java 对象映射为 MongoDB 文档，类似于 JPA 的 `@Entity` 注解。



以下面的代码为例：

- 定义实体类

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "users")  // 指定 MongoDB 中的集合名称
public class User {
    @Id
    private String id;
    private String username;
    private String email;
    
    // Getters and setters
}
```

- 定义 Repository 接口

```java
import org.springframework.data.mongodb.repository.MongoRepository;
import java.util.List;

public interface UserRepository extends MongoRepository<User, String> {
    List<User> findByUsername(String username);  // 方法名派生查询
}
```

- 使用 MongoRepository 的服务类

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;
    
    public List<User> getUsersByUsername(String username) {
        return userRepository.findByUsername(username);
    }
    
    public void saveUser(User user) {
        userRepository.save(user);
    }
    
    public void deleteUser(String id) {
        userRepository.deleteById(id);
    }
}
```

- 使用 MongoTemplate 进行复杂查询

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;

@Service
public class UserCustomService {

    @Autowired
    private MongoTemplate mongoTemplate;
    
    public User getUserByEmail(String email) {
        Query query = new Query();
        query.addCriteria(Criteria.where("email").is(email));
        return mongoTemplate.findOne(query, User.class);
    }
}
```

