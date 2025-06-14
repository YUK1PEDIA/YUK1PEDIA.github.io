---
layout: post
title: 泛型与函数式编程
description: 记录
tag: 笔记
---

## 泛型与函数式编程

- 函数式编程：在Java中，函数式编程是一种编程范式，它把计算视为数学上的函数求值，并且避免了状态和可变数据。这意味着在函数式编程中，函数是“一等公民”，**它们可以作为参数传递给其他函数，也可以作为结果从函数中返回**。此外，函数式编程鼓励使用不可变对象来减少副作用，从而提高代码的可读性和可维护性。
- 泛型：泛型是Java语言的一个重要特性，它允许你在定义类、接口和方法时使用**类型参数**。这样做的好处是**可以重用相同的代码来处理不同的数据类型**，同时还能获得编译时的类型安全检查，避免运行时出现类型错误。简单来说，泛型就是一种参数化类型的机制，即所操作的数据类型可以被指定为一个参数，这种参数类型可以在使用时确定。



## 应用

我们以下面的需求为例子：

![1.png](https://s2.loli.net/2024/11/07/sWRkoalcH5EjJKQ.png)

#### 方法 1：

方法 1 需要将任意 java 对象序列化为 json 并存储在 string 类型的 key 中，还需要设置 TTL 过期时间。显然这个方法并不需要返回值，我们可以复用 `StringRedisTemplate` 的 `set` 方法，并结合 `JSONUtil` 工具包的 `toJsonStr` 方法实现该功能。

由于需要存储在 string 类型的 key 中，所以需要传入一个 String 类型的变量 key ；由于是任意 java 对象，因此 value 定义成 Object 类型；对于设置 TTL 过期时间，我们可以仿照 Spring 的风格，传入一个 Long 类型的 time 和一个 TimeUnit（时间单位）。

```java
public void set(String key, Object value, Long time, TimeUnit unit) {
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value), time, unit);
}
```

#### 方法 2：

在方法 1 的基础上需要添加一个**逻辑过期时间**，这里逻辑过期时间的作用是防止 redis 出现**缓存击穿**的情况，缓存击穿、穿透、雪崩的介绍在：https://yuk1pedia.github.io/2024/11/Cache-penetration,-cache-avalanche,-and-cache-breakdown/

为了实现这个需求，我们额外定义一个 `RedisData` 类：

```java
@Data
public class RedisData {
    private LocalDateTime expireTime; // 逻辑过期时间
    private Object data;
}
```

在这个类中，将具体的数据封装到 `Object` 对象 data 里，另外用 `LocalDateTime` 封装逻辑过期时间。

于是我们就可以把用户传进来的数据封装到 `RedisData` 对象里了：

```java
public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit) {
    // 设置逻辑过期
    RedisData redisData = new RedisData();
    redisData.setData(value);
    redisData.setExpireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)));
    // 写入 redis
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));
}
```



上述两个存储缓存的方法并不需要用到函数式编程和泛型，下面查询缓存的方法就需要这两种编程技巧了。

#### 方法 3：

为了解决缓存穿透的问题，我们需要将查询到空值的 id 写入 redis 缓存中，那么下次查询相同的 id 时，访问就会落到缓存上，不会给数据库带来过大的压力。

redis 中存放的**对象类型很多**，但 redis 是基于 `key-value` 的 NoSQL 数据库，查询的逻辑都是通过 key 值来找到对应的 value 值，**所以这里考虑用同一套查询逻辑实现对不同类型对象的查询**。

在 redis 中，对象一般是以 json 字符串的形式存放，我们从中拿到需要的数据后要进行反序列化，但问题是**我们要将 json 字符串反序列化成什么类型的对象**？很明显，这个信息在后端工程里无法得到，需要用户指定。因此这里就可以利用到 java 的**泛型**特点，我们将方法 3 的返回类型指定成泛型，用户传入需要查询的对象类型后执行一套查询逻辑，将查询结果返回给用户。

查询前我们需要得知用户想根据哪个 key 查询，考虑到 redis 的 NoSQL 特性，传入的这个 key 不一定是 Long 类型或者 Int 类型，**因此还需要定义另外一个传入参数的泛型**。

当用户传入了需要查询的对象类型和 key 后，我们可以调用 `StringRedisTemplate` 的 `get` 方法到缓存数据库中查询。如果缓存命中，就直接返回；如果缓存没有命中，这个访问就落到后端数据库上，**我们就需要进行后端数据库的查询操作**。

但这里有一个问题，**我们并不知道如何查询后端数据库**，因为我们编写的是一个工具类，并不会在这个类里注入持久层的对象进行数据库的查询，于是这里就需要用到**函数式编程**的特性：将查询的方法作为参数，传入工具类的查询方法里。

```java
// 解决缓存穿透的根据 id 查询方法
public <R, ID> R queryWithPassThrough(
        String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {
    String key = CACHE_SHOP_KEY + id;
    // 根据 id 查询商铺缓存
    String Json = stringRedisTemplate.opsForValue().get(key);
    if (StrUtil.isNotBlank(Json)) {
        // redis 中存在，直接返回（isNotBlank只有在字符串为"abc"这种时才返回 true，空字符串、空格回车都返回 false）
        return JSONUtil.toBean(Json, type);
    }
    // 判断命中的是否为空值
    if (Json != null) {
        return null;
    }

    // redis 中不存在，查询后端数据库
    R r = dbFallback.apply(id);
    if (r == null) {
        // 往 redis 里写入空值，防止缓存穿透
        stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
        return null;
    }
    // 后端数据库存在对应店铺，就写入 redis 缓存中
    this.set(key, r, time, unit);
    return r;
}
```

### 方法 4：

方法 4 和方法 3 的思路相同，也要用到函数式编程和泛型，具体不再赘述。可以看看方法 4 是如何使用逻辑过期来解决缓存击穿问题的。

```java
// 使用逻辑过期解决缓存击穿问题
public <R, ID> R queryWithLogicalExpire(
        String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {
    String key = CACHE_SHOP_KEY + id;
    // 根据 id 查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(key);
    if (StrUtil.isBlank(shopJson)) {
        // 没有命中就返回（命中了还需要查询是否逻辑过期）
        return null;
    }

    // 命中，需要判断过期时间
    RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
    R r = JSONUtil.toBean((JSONObject) redisData.getData(), type);
    LocalDateTime expireTime = redisData.getExpireTime();

    // 判断是否过期
    if (expireTime.isAfter(LocalDateTime.now())) {
        // 没有过期
        return r;
    }

    // 已经过期，进行缓存重建
    String lockKey = LOCK_SHOP_KEY + id;
    boolean isLock = tryLock(lockKey);
    // 判断获取锁是否成功
    if (isLock) {
        // 获取锁成功，开启独立线程实现缓存重建
        CACHE_REBUILD_EXECUTOR.submit(() -> {
            try {
                // 查询数据库
                R r1 = dbFallback.apply(id);
                // 写入 redis
                this.setWithLogicalExpire(key, r1, time, unit);
            } catch (Exception e) {
                throw new RuntimeException(e);
            } finally {
                unLock(lockKey);
            }
        });
    }

    return r;
}
```

## 完整工具类和调用过程代码

工具类：

```java
@Component
@Slf4j
public class CacheClient {

    private final StringRedisTemplate stringRedisTemplate;

    public CacheClient(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    public void set(String key, Object value, Long time, TimeUnit unit) {
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value), time, unit);
    }

    public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit) {
        // 设置逻辑过期
        RedisData redisData = new RedisData();
        redisData.setData(value);
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)));
        // 写入 redis
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));
    }

    // 解决缓存穿透的根据 id 查询方法
    public <R, ID> R queryWithPassThrough(
            String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {
        String key = CACHE_SHOP_KEY + id;
        // 根据 id 查询商铺缓存
        String Json = stringRedisTemplate.opsForValue().get(key);
        if (StrUtil.isNotBlank(Json)) {
            // redis 中存在，直接返回（isNotBlank只有在字符串为"abc"这种时才返回 true，空字符串、空格回车都返回 false）
            return JSONUtil.toBean(Json, type);
        }
        // 判断命中的是否为空值
        if (Json != null) {
            return null;
        }

        // redis 中不存在，查询后端数据库
        R r = dbFallback.apply(id);
        if (r == null) {
            // 往 redis 里写入空值，防止缓存穿透
            stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return null;
        }
        // 后端数据库存在对应店铺，就写入 redis 缓存中
        this.set(key, r, time, unit);
        return r;
    }


    // 线程池
    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);
    // 使用逻辑过期解决缓存击穿问题
    public <R, ID> R queryWithLogicalExpire(
            String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit) {
        String key = CACHE_SHOP_KEY + id;
        // 根据 id 查询商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        if (StrUtil.isBlank(shopJson)) {
            // 没有命中就返回（命中了还需要查询是否逻辑过期）
            return null;
        }

        // 命中，需要判断过期时间
        RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
        R r = JSONUtil.toBean((JSONObject) redisData.getData(), type);
        LocalDateTime expireTime = redisData.getExpireTime();

        // 判断是否过期
        if (expireTime.isAfter(LocalDateTime.now())) {
            // 没有过期
            return r;
        }

        // 已经过期，进行缓存重建
        String lockKey = LOCK_SHOP_KEY + id;
        boolean isLock = tryLock(lockKey);
        // 判断获取锁是否成功
        if (isLock) {
            // 获取锁成功，开启独立线程实现缓存重建
            CACHE_REBUILD_EXECUTOR.submit(() -> {
                try {
                    // 查询数据库
                    R r1 = dbFallback.apply(id);
                    // 写入 redis
                    this.setWithLogicalExpire(key, r1, time, unit);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                } finally {
                    unLock(lockKey);
                }
            });
        }

        return r;
    }

    private boolean tryLock(String key) {
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
        return BooleanUtil.isTrue(flag);
    }

    private void unLock(String key) {
        stringRedisTemplate.delete(key);
    }
}
```

在应用层的调用：

```java
@Resource
private CacheClient cacheClient;

/**
 * 根据缓存实现 id 查询商铺
 * @param id
 * @return
 */
@Override
public Result queryById(Long id) {
    // 防止缓存穿透的查询方式
//        Shop shop = cacheClient
//                .queryWithPassThrough(CACHE_SHOP_KEY, id, Shop.class, this::getById, CACHE_SHOP_TTL, TimeUnit.MINUTES);

    // 互斥锁解决缓存击穿
//        Shop shop = queryWithMutex(id);

    // 逻辑过期解决缓存击穿
    Shop shop = cacheClient
            .queryWithLogicalExpire(CACHE_SHOP_KEY, id, Shop.class, this::getById, CACHE_SHOP_TTL, TimeUnit.MINUTES);

    if (shop == null) {
        return Result.fail("店铺不存在！");
    }
    return Result.ok(shop);
}
```

