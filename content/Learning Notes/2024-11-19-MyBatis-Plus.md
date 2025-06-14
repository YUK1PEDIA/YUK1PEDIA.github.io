---
layout: post
title: MyBatis-plus
description: 记录
tag: 笔记
---

## 简介

MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具包，它在 MyBatis 的基础上进行了一些优化和提升，旨在简化开发，提高开发效率，减少冗余代码



## 提升与特性

### 1.无侵入式增强

MyBatis-Plus 是在 MyBatis 基础上的增强，不需要修改现有 MyBatis 的配置和代码，最大程度地保持兼容性和灵活性。



### 2.增强的 CRUD 操作

MyBatis-Plus 提供了内置的通用 Mapper 和 Service，开发者不再需要编写常见的增删改查（CRUD）方法。通过继承 `BaseMapper` 接口，开发者可以直接调用现成的方法来进行数据库操作。

**常见的 CRUD 方法：**

- `insert()`
- `updateById()`
- `selectById()`
- `deleteById()`
- `selectList()`
- `selectOne()`
- `selectCount()`

举个例子：

假设有一个 `User` 实体类，包含用户信息的字段。

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

`Mapper` 接口：MyBatis-Plus 提供了 `BaseMapper` 接口，只需让自己的 `Mapper` 接口继承 `BaseMapper`，就能自动拥有常用的 CRUD 方法。

```java
public interface UserMapper extends BaseMapper<User> {
    // 你可以根据需要添加自定义的方法
}
```

1. 插入操作：

```java
@Autowired
private UserMapper userMapper;

public void insertUser() {
    User user = new User();
    user.setName("John");
    user.setAge(30);
    user.setEmail("john.doe@example.com");

    // 插入用户
    userMapper.insert(user);
}
```

**注意：** 插入时会自动返回 `user` 的主键值，如果数据库使用的是自增主键。



2. 根据 ID 更新：`updateById()` 方法用于根据 `id` 更新某条记录。

```java
public void updateUserById() {
    User user = new User();
    user.setId(1L);  // 更新 ID 为 1 的用户
    user.setAge(31);  // 更新年龄为 31

    // 根据 id 更新记录
    userMapper.updateById(user);
}
```



3. 根据 ID 查询：`selectById()` 方法根据 `id` 查询单条记录。

```java
public void selectUserById() {
    Long userId = 1L;
    
    // 根据 id 查询用户
    User user = userMapper.selectById(userId);
    System.out.println(user);
}
```



4. 查询所有记录：`selectList()` 方法可以查询所有记录，当然也可以通过 `QueryWrapper` 对查询条件进行过滤。

```java
public void selectAllUsers() {
    // 查询所有用户
    List<User> users = userMapper.selectList(null);  // 传 null 表示不加查询条件
    users.forEach(System.out::println);
}
```



5. 条件查询：使用 `QueryWrapper` 来构建查询条件。以下示例查询年龄大于 18 且名字为 "John" 的用户。

```java
public void selectUsersByCondition() {
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.gt("age", 18)  // 年龄大于 18
           .eq("name", "John");  // 名字为 "John"

    List<User> users = userMapper.selectList(wrapper);
    users.forEach(System.out::println);
}
```



6. 删除操作：`deleteById()` 方法根据 ID 删除记录。

```java
public void deleteUserById() {
    Long userId = 1L;

    // 根据 ID 删除用户
    userMapper.deleteById(userId);
}
```



7. 批量插入：`insertBatchSomeColumn()` 方法用于批量插入数据。注意，这个方法是 MyBatis-Plus 在 3.x 版本后才引入的，如果需要批量插入，可以使用该方法。

```java
public void insertBatchUsers() {
    List<User> users = new ArrayList<>();
    
    users.add(new User(null, "Alice", 25, "alice@example.com"));
    users.add(new User(null, "Bob", 28, "bob@example.com"));
    
    // 批量插入用户
    userMapper.insertBatchSomeColumn(users);
}
```



8. 分页查询：MyBatis-Plus 提供了分页查询支持，通过 `Page` 对象来进行分页查询。

```java
public void selectUsersByPage() {
    // 当前页和每页条数
    Page<User> page = new Page<>(1, 10);  // 第1页，每页10条

    // 进行分页查询
    Page<User> userPage = userMapper.selectPage(page, null);

    // 获取分页结果
    List<User> users = userPage.getRecords();
    users.forEach(System.out::println);

    // 获取分页信息
    long total = userPage.getTotal();  // 总记录数
    System.out.println("Total users: " + total);
}
```



9. 逻辑删除：MyBatis-Plus 支持逻辑删除，通过在实体类中使用 `@TableLogic` 注解，MyBatis-Plus 将自动进行逻辑删除，而不是物理删除。

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
    
    @TableLogic
    private Integer deleted;  // 逻辑删除字段
}
```

使用 `deleteById()` 时，默认会根据 `@TableLogic` 注解标记的字段进行逻辑删除。

```java
public void deleteUserLogic() {
    Long userId = 1L;

    // 执行逻辑删除
    userMapper.deleteById(userId);
}
```





### 3.条件构造器（Wrapper）

MyBatis-Plus 提供了 `QueryWrapper` 和 `UpdateWrapper` 类，用于构建查询条件和更新条件。条件构造器支持链式调用，开发者可以更直观地构建查询和更新条件，减少了手动拼接 SQL 的复杂度。

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("age", 25).lt("score", 100);
List<User> users = userMapper.selectList(wrapper);
```

`QueryWrapper` 是用于查询的条件构造器，支持多种条件查询，允许通过链式调用来添加各种查询条件。

1. 构建查询方法

**常用方法**

- `eq()`：等于
- `ne()`：不等于
- `gt()`：大于
- `lt()`：小于
- `ge()`：大于等于
- `le()`：小于等于
- `like()`：模糊匹配
- `between()`：区间查询
- `in()`：集合查询
- `isNull()`：为空
- `orderByAsc()` / `orderByDesc()`：排序
- `groupBy()`：分组
- `or()`：逻辑 OR 条件
- `likeLeft()`：左模糊匹配
- `likeRight()`：右模糊匹配

**示例**

假设我们有一个 `User` 实体类，包含 `id`, `name`, `age`, 和 `email` 字段

```java
// 实体类
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

**查询年龄大于30且名字为"John"的用户**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.gt("age", 30)  // 年龄大于30
            .eq("name", "John");  // 名字为"John"

// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
users.forEach(System.out::println);
```

**模糊查询名字包含"Jo"且邮箱为特定值**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.like("name", "Jo")  // 名字包含"Jo"
            .eq("email", "john.doe@example.com");  // 邮箱为特定值

// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
users.forEach(System.out::println);
```

**查询年龄在25到30岁之间的用户**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.between("age", 25, 30);  // 年龄在25到30之间

// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
users.forEach(System.out::println);
```

**查询名字为"John"或"Jane"的用户**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.in("name", "John", "Jane");  // 名字为"John"或"Jane"

// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
users.forEach(System.out::println);
```

**查询年龄大于30，并按年龄升序排序**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.gt("age", 30)  // 年龄大于30
            .orderByAsc("age");  // 按年龄升序排序

// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
users.forEach(System.out::println);
```



2. 构建更新条件

**常用方法**

- `set()`：设置更新的字段值
- `eq()`：等于
- `ne()`：不等于
- `gt()`：大于
- `lt()`：小于
- `ge()`：大于等于
- `le()`：小于等于
- `like()`：模糊匹配
- `isNull()`：为空
- `in()`：集合查询

**更新名字为"John"的用户的年龄为35**

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper.eq("name", "John")  // 条件：名字为"John"
             .set("age", 35);  // 更新：将年龄设置为35

// 执行更新
userMapper.update(null, updateWrapper);  // 第一个参数为null表示不更新实体类中的数据
```

**批量更新多个用户的邮箱**

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper.in("id", 1L, 2L, 3L)  // 条件：id为1, 2, 3
             .set("email", "new.email@example.com");  // 更新邮箱

// 执行更新
userMapper.update(null, updateWrapper);
```

**将年龄大于30的用户的状态更新为"已成年"**

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper.gt("age", 30)  // 条件：年龄大于30
             .set("status", "已成年");  // 更新：将状态设置为"已成年"

// 执行更新
userMapper.update(null, updateWrapper);
```



3. 组合条件查询

在 MyBatis-Plus 中，`QueryWrapper` 和 `UpdateWrapper` 支持灵活的逻辑组合条件（如 AND、OR、NOT）。使用 `and()`、`or()`、`nor()` 等方法，可以实现更复杂的查询。

**查询条件的 AND 组合**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("name", "John")  // 名字为"John"
            .and(wrapper -> wrapper.lt("age", 30).or().gt("age", 40));  // 年龄小于30或大于40

// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
users.forEach(System.out::println);
```

**查询条件的 OR 组合**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("name", "John")  // 名字为"John"
            .or().eq("name", "Jane");  // 或者名字为"Jane"

// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
users.forEach(System.out::println);
```



### 4.分页查询

MyBatis-Plus 内置了分页插件，使得分页查询更加简单。通过配置分页插件，可以直接使用 `Page` 对象来进行分页查询。

```java
Page<User> page = new Page<>(1, 10);  // 第1页，每页10条
Page<User> userPage = userMapper.selectPage(page, null);
```





### 5.自动填充功能

MyBatis-Plus 支持字段的自动填充，例如在插入或更新时自动填充某些字段的值（如创建时间、修改时间等）。通过实现 `MetaObjectHandler` 接口来配置自动填充逻辑。

```java
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        // 插入时自动填充字段 "createTime"，值为当前时间
        this.setFieldValByName("createTime", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        // 更新时自动填充字段 "updateTime"，值为当前时间
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}

```

**自动填充的工作原理**

- 当 MyBatis-Plus 执行插入（`insert()`）或更新（`update()`）操作时，会先检查实体类中是否有需要自动填充的字段。通常，这些字段是一些例如 `createTime`（创建时间）和 `updateTime`（更新时间）等字段。
- 在 `insert()` 操作时，`insertFill()` 方法会被调用，允许你在插入数据之前设置需要填充的字段（如设置 `createTime` 字段为当前时间）。
- 在 `update()` 操作时，`updateFill()` 方法会被调用，允许你在更新数据时自动填充某些字段（如设置 `updateTime` 字段为当前时间）。

以上面的代码为例：

- `insertFill(MetaObject metaObject)` 方法
  - `metaObject` 参数是 MyBatis-Plus 用来封装实体类的对象，它提供了对实体类字段的访问接口。通过 `setFieldValByName()` 方法设置实体类中某个字段的值。
  - `setFieldValByName("createTime", new Date(), metaObject)` 这行代码的作用是：通过**反射机制**，自动将实体类中 `createTime` 字段的值设置为当前的日期和时间（`new Date()`）。
- `updateFill(MetaObject metaObject)` 方法
  - `metaObject` 参数同样用于封装实体类的对象，通过 `setFieldValByName()` 方法来设置字段值。
  - `setFieldValByName("updateTime", new Date(), metaObject)` ：通过反射机制，自动将实体类中 `updateTime` 字段的值设置为当前的日期和时间（`new Date()`）。







### 6.乐观锁插件

MyBatis-Plus 提供了乐观锁插件，通过在实体类中添加版本号字段（如 `version`），并在数据库操作时自动校验版本号，从而实现对数据并发操作的控制。

```java
@Version
private Integer version;
```





### 7.代码生成器

MyBatis-Plus 提供了强大的代码生成器，可以根据数据库表结构自动生成实体类、Mapper 接口、Mapper XML 文件、Service 类等，极大提高了开发效率，避免了重复造轮子。





### 8.自定义 SQL 支持

虽然 MyBatis-Plus 提供了很多通用的操作，但如果需要自定义复杂的 SQL 查询或操作，仍然可以通过 `@Select`、`@Update` 等注解来编写 SQL，也可以直接使用 MyBatis 的原生方式。





### 9.性能分析

MyBatis-Plus 提供了 SQL 性能分析插件，可以在开发阶段输出 SQL 语句和执行时间，帮助开发者发现性能瓶颈，优化查询。





### 10.分库分表支持（Sharding）

MyBatis-Plus 提供了对分库分表的支持，方便进行水平拆分和数据库分布式管理。

**分库分表** 是数据库水平扩展（sharding）的一种常见技术，目的是通过将数据分散到多个数据库或表中，以提高数据库的性能、可扩展性和容错能力。在大型分布式系统中，单一数据库可能面临性能瓶颈或者数据量过大导致无法处理的情况，分库分表通过将数据切分成多个部分，分散到多个存储单元中，解决了这些问题。





### 11.补充

先看下面两段代码

```java
User user = query().eq("phone", phone).one();
List<Blog> blogs = query().in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list();
```

虽然这两段代码调用了 `query()` 方法，但是 **它们查询的是不同的数据库表**，这主要是通过 **`query()` 方法所在的服务类** 来决定的。



1. **`query()` 方法所在的服务类**

在 MyBatis-Plus 中，`query()` 方法是 `IService<T>` 接口中的一个默认方法，它实际上是通过 `BaseMapper` 执行数据库查询操作。因此，**`query()` 方法是基于当前实例的 `BaseMapper` 来执行查询的**。不同的服务类会使用不同的实体类和 `BaseMapper`，从而查询不同的数据库表。

- **`User user = query().eq("phone", phone).one();`**
  - 假设 `query()` 方法是 `UserService` 类中的方法，这时 `UserService` 会注入 `UserMapper`，而 `UserMapper` 是与 `user` 表进行交互的。
  - 在 `query()` 方法调用时，`BaseMapper`（**这里是 `UserMapper`**）会执行对应 `user` 表的查询。
- **`List<Blog> blogs = query().in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list();`**
  - 同样，假设这行代码是在 `BlogService` 类中调用的 `query()`，那么此时 `query()` 返回的就是与 `Blog` 实体相关联的 `BlogMapper`。
  - `BlogMapper` 会执行对 `blog` 表的查询。



2. **`BaseMapper` 和 `query()` 方法的关联**

每个实体类（如 `User`、`Blog`）通常会有一个对应的 **Mapper**，该 **Mapper** 是通过 `BaseMapper<T>` 进行扩展的，它提供了常用的 CRUD 操作。而 `IService<T>` 接口中的 `query()` 方法依赖于 `BaseMapper` 来执行实际的查询操作。

- **`BaseMapper<T>`**：提供了常用的数据库操作方法，例如 `selectList()`, `selectOne()`, `selectById()` 等。
- **`query()` 方法**：返回一个 `QueryChainWrapper`，用于构建查询条件，最终会调用对应的 `BaseMapper`（如 `UserMapper` 或 `BlogMapper`）执行查询。



3. **如何判断查询哪个数据库？**

MyBatis-Plus 在执行查询时是通过 **`BaseMapper`** 来决定的，具体如下：

- **`UserService` 类**：
  - 如果 `query()` 方法在 `UserService` 中被调用，那么 `UserService` 的 `BaseMapper` 就是 `UserMapper`，`UserMapper` 与 `user` 表对应，查询将发生在 `user` 表中。
  - 这里的 `query()` 方法调用的是 `UserMapper` 的方法。
- **`BlogService` 类**：
  - 如果 `query()` 方法在 `BlogService` 中被调用，那么 `BlogService` 的 `BaseMapper` 就是 `BlogMapper`，`BlogMapper` 与 `blog` 表对应，查询将发生在 `blog` 表中。
  - 这里的 `query()` 方法调用的是 `BlogMapper` 的方法。

每个服务类（如 `UserService`、`BlogService`）都具有自己的 `BaseMapper`，而 `BaseMapper` 的 `selectList()`、`selectOne()` 等方法会查询不同的数据库表。

