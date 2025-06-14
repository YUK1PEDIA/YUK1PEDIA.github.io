---
layout: post
title: 分布式锁
description: 记录
tag: 笔记
---

## 基本原理和实现方式

- 分布式锁：满足分布式系统或集群模式下多进程可见并且互斥的锁。分布式锁的核心思想就是让大家都使用同一把锁，只要大家使用的是同一把锁，那么我们就能锁住线程，不让线程进行，**让程序串行执行**。

![1.webp](https://s2.loli.net/2024/11/09/gkcmltHMzDTF8oW.webp)

- 那么分布式锁他应该满足的条件呢？
  - **可见性**：多个线程都能看到相同的结果，注意：这个地方说的可见性并不是并发编程中指的内存可见性，只是说多个进程之间都能感知到变化的意思
  - **互斥**：互斥是分布式锁的最基本的条件，使得程序串行执行
  - **高可用**：程序不易崩溃，时时刻刻都保证较高的可用性
  - **高性能**：由于加锁本身就让性能降低，所有对于分布式锁本身需要他就较高的加锁性能和释放锁性能
  - **安全性**：安全也是程序中必不可少的一环
- 常见的分布式锁有三种：
  - **Mysql**：mysql本身就带有锁机制，但是由于mysql性能本身一般，所以采用分布式锁的情况下，其实使用mysql作为分布式锁比较少见
  - **Redis**：redis作为分布式锁是非常常见的一种使用方式，现在企业级开发中基本都使用redis或者zookeeper作为分布式锁，利用setnx这个方法，如果插入key成功，则表示获得到了锁，如果有人插入成功，其他人插入失败则表示无法获得到锁，利用这套逻辑来实现分布式锁
  - **Zookeeper**：zookeeper也是企业级开发中较好的一个实现分布式锁的方案，zookeeper中实现分布式锁主要依赖于它的临时节点（ephemeral node）和顺序节点（sequential node）特性。



## Redis 分布式锁的实现核心思路

实现分布式锁时需要实现的两个基本方法：

- 获取锁：
  - 互斥：确保只能有一个线程获取锁
  - 非阻塞：尝试一次，成功返回 true，失败返回 false
- 释放锁：
  - 手动释放
  - 超时释放：获取锁时添加一个超时时间

**实现核心思路**：

我们利用 redis 的 `setnx` 方法，当有多个线程进入时，我们就利用该方法，第一个线程进入时，redis 中就有这个 key 了，返回了1，如果结果是1，则表示他抢到了锁，那么他去执行业务，然后再删除锁，退出锁逻辑，没有抢到锁的线程，等待一定时间后重试即可。

![1.webp](https://s2.loli.net/2024/11/09/HC4dWSmnLX9sFER.webp)



## 实现基于 redis 的分布式锁

- 分布式锁的基本接口

  ```java
  /**
   * 分布式锁
   */
  public interface ILock {
  
      /**
       * 尝试获取锁
       * @param timeoutSec
       * @return
       */
      boolean tryLock(long timeoutSec);
      
      /**
       * 释放锁
       */
      void unlock();
  }
  ```

- 简单分布式锁的实现

  ```java
  public class SimpleRedisLock implements ILock {
  
      private String name; // 业务名
      private StringRedisTemplate stringRedisTemplate;
  
      public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
          this.name = name;
          this.stringRedisTemplate = stringRedisTemplate;
      }
  
      private static final String KEY_PREFIX = "lock:";
      private static final String ID_PREFIX = UUID.randomUUID().toString(true) + "-";
  
      /**
       * 获取锁
       * @param timeoutSec
       * @return
       */
      @Override
      public boolean tryLock(long timeoutSec) {
          // 获取线程标识
          String threadId = ID_PREFIX + Thread.currentThread().getId();
          // 获取锁，redis 中以线程标识作为 value
          Boolean success = stringRedisTemplate.opsForValue()
                  .setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
          // 封装类的自动拆箱可能造成空指针异常，这里用 equals 方法进行判断
          return Boolean.TRUE.equals(success);
      }
  
      /**
       * 释放锁
       */
      @Override
      public void unlock() {
          // 获取当前线程标识和分布式锁里存放的线程标识
          String threadId = ID_PREFIX + Thread.currentThread().getId();
          String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
          // 两个 id 相同才能释放锁
          if (threadId.equals(id)) {
              stringRedisTemplate.delete(KEY_PREFIX + name);
          }
      }
  }
  ```

### redis 分布式锁误删情况：

- **具体情况**：持有锁的线程在锁的内部出现了阻塞，**导致他的锁自动释放**，这时其他线程，线程 2 来尝试获得锁，就拿到了这把锁，然后线程 2 在持有锁执行过程中，线程 1 从阻塞状态恢复过来，继续执行，而线程 1 执行过程中，走到了删除锁逻辑，**此时就会把本应该属于线程 2 的锁进行删除，这就是误删其他线程的锁的情况**。

- **解决方案**：解决方案就是在每个线程释放锁的时候，**去判断一下当前这把锁是否属于自己**，如果属于自己，则不进行锁的删除，假设还是上边的情况，线程 1 卡顿，锁自动释放，线程 2 进入到锁的内部执行逻辑，此时线程 1 从阻塞状态恢复过来，然后尝试删除锁，但是线程 1 发现当前这把锁不属于自己，于是不进行删除锁逻辑，当线程 2 走到删除锁逻辑时，如果没有卡过自动释放锁的时间点，则判断当前这把锁是属于自己的，于是删除这把锁。

![1.webp](https://s2.loli.net/2024/11/09/hx4JoF1P9rUsnqg.webp)

根据上面的分析，我们可以修改之前的代码，在线程获取到锁的同时向 redis 中写入自己的线程标识，在释放锁之前检查当前 redis 中的线程标识是不是自己的，再选择是否删除当前锁。

```java
public class SimpleRedisLock implements ILock {

    private String name; // 业务名
    private StringRedisTemplate stringRedisTemplate;

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    private static final String KEY_PREFIX = "lock:";
    private static final String ID_PREFIX = UUID.randomUUID().toString(true) + "-";

    /**
     * 获取锁
     * @param timeoutSec
     * @return
     */
    @Override
    public boolean tryLock(long timeoutSec) {
        // 获取线程标识
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        // 获取锁，redis 中以线程标识作为 value
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
        // 封装类的自动拆箱可能造成空指针异常，这里用 equals 方法进行判断
        return Boolean.TRUE.equals(success);
    }

    /**
     * 释放锁
     */
    @Override
    public void unlock() {
        // 获取当前线程标识和分布式锁里存放的线程标识
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
        // 两个 id 相同才能释放锁
        if (threadId.equals(id)) {
            stringRedisTemplate.delete(KEY_PREFIX + name);
        }
    }
}
```

### 分布式锁的原子性问题

线程 1 现在持有锁之后，在执行业务逻辑过程中，他正准备删除锁，**而且已经走到了条件判断的过程中**，比如他已经拿到了当前这把锁确实是属于他自己的，正准备删除锁，此时线程 1 发生了阻塞，**并且他的锁此时到期了**，被迫释放。那么此时线程 2 进来，发现可以获取锁，于是线程 2 获取到分布式锁，但是线程 1 他会接着往后执行，当他阻塞结束后，他直接就会执行删除锁那行代码，**相当于条件判断并没有起到作用，这就是删锁时的原子性问题**，之所以有这个问题，是因为线程 1 的拿锁，比锁，删锁，实际上并不是原子性的，我们要防止刚才的情况发生。

![1.webp](https://s2.loli.net/2024/11/09/B3zRnvGXVYa761Z.webp)

原子性问题可以使用 **lua 脚本**解决，在 java 中我们一般会用 redission 这个第三方库来实现分布式锁，这时候就不需要我们上面手写的分布式锁了。



## 基于 redisson 的分布式锁实现

### setnx 实现的分布式锁存在的问题

基于 redis 的 setnx 方法实现的分布式锁存在以下问题：

- **重入问题**：重入问题是指**获得锁的线程可以再次进入到相同的锁的代码块中**，可重入锁的意义在于防止死锁，比如 HashTable 这样的代码中，他的方法都是使用 synchronized 修饰的，**假如他在一个方法内，调用另一个方法，那么此时如果是不可重入的，不就死锁了吗**？所以可重入锁他的主要意义是防止死锁，我们的 synchronized 和 Lock 锁都是可重入的。
- **不可重试**：是指目前的分布式锁只能尝试一次，我们认为合理的情况是：当线程在获得锁失败后，他应该能再次尝试获得锁。
- **超时释放：**我们在加锁时增加了过期时间，这样的我们可以防止死锁，但是如果卡顿的时间超长，虽然我们采用了 lua 表达式防止删锁的时候，误删别人的锁，但是毕竟没有锁住，有安全隐患。
- **主从一致性：** 如果 redis 提供了主从集群，当我们向集群写数据时，主机需要异步的将数据同步给从机，而万一在同步过去之前，主机宕机了，就会出现死锁问题。

### 使用 redisson 实现分布式锁

Redisson 是一个 Redis 的 Java 客户端，它不仅提供了对 Redis 基本命令的支持，还扩展了许多高级功能，如分布式对象、集合、锁、消息队列等。Redisson 的设计目标是让开发者能够更加方便地在 Java 应用程序中使用 Redis 的强大功能，而无需关心底层的通信细节。

**主要特性**

1. **分布式对象**：Redisson 提供了一系列分布式对象，如 Map、Set、List、Queue 等，这些对象可以在多个 JVM 之间共享，并且支持原子操作。
2. **分布式锁**：包括可重入锁（Reentrant Lock）、公平锁（Fair Lock）、多锁（MultiLock）、红锁（Red Lock）、读写锁（ReadWrite Lock）等。
3. **分布式集合**：提供了多种分布式的集合类型，如 Set、SortedSet、List、Map 等。
4. **分布式消息队列**：支持发布/订阅模式（Pub/Sub）、主题（Topic）、队列（Queue）、延迟队列（Delayed Queue）等。
5. **分布式服务**：提供了分布式远程服务调用（Remote Service）、分布式执行器服务（Executor Service）等。
6. **分布式集合缓存**：支持本地缓存、近似缓存等多种缓存策略。
7. **分布式计数器**：提供了一个高并发的分布式计数器。
8. **分布式ID生成器**：支持 Snowflake 算法的分布式 ID 生成器。
9. **分布式任务调度**：支持定时任务和周期任务的调度。

我们先在工程中引入 redisson 的 maven 依赖：

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.28.0</version>
</dependency>
```

然后新建一个配置类，用于配置 redisson 客户端，这个配置类里的方法返回的 RedissonClient 对象将被 spring 容器管理：

```java
@Configuration
public class RedissonConfig {

    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379").setPassword("123456");
        return Redisson.create(config);
    }
}
```

于是我们原来的业务方法中就可以使用 RedissonClient 来实现

```java
/**
 * 优惠券秒杀下单
 * @param voucherId
 * @return
 */
@Override
public Result seckillVoucher(Long voucherId) {
    SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
    if (voucher.getBeginTime().isAfter(LocalDateTime.now()) || voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("不在秒杀的时间范围内！");
    }
    if (voucher.getStock() < 1) {
        return Result.fail("库存不足！");
    }

    Long userId = UserHolder.getUser().getId();
    // 悲观锁 synchronized 只能在单机系统里解决线程安全问题，不能解决分布式系统的线程安全问题
//        synchronized (userId.toString().intern()) {
//            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
//            return proxy.createVoucherOrder(voucherId);
//        }

    // 分布式锁实现
    // SimpleRedisLock 是自己实现的简单分布式锁
//        SimpleRedisLock lock = new SimpleRedisLock("order" + userId, stringRedisTemplate);

    // Rlock 是 redisson 框架提供的分布式锁
    RLock lock = redissonClient.getLock("lock:order:" + userId);

    boolean isLock = lock.tryLock();
    if (!isLock) {
        return Result.fail("不允许重复下单！");
    }

    try {
        // 获取 spring 事务的代理对象，如果直接调用 createVoucherOrder 方法可能会造成 spring 事务失效
        IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
        // spring 的事务实现是通过动态代理技术实现的，拿到代理对象来调用方法能保证事务的有效性
        return proxy.createVoucherOrder(voucherId);
    } finally {
        lock.unlock();
    }
}

@Transactional
public Result createVoucherOrder(Long voucherId) {
    // 通过拦截器获取登录用户名
    Long userId = UserHolder.getUser().getId();
    // 优惠券一个用户最多购买一次
    int count = query().eq("user_id", userId).eq("voucher_id", voucherId).count();
    if (count > 0) {
        return Result.fail("用户已经购买过一次！");
    }

    // 扣减库存
    boolean success = seckillVoucherService.update()
            .setSql("stock = stock - 1")
            // 第二个 gt 是加了一个乐观锁，判断现在查询到的库存和之前的库存是否一样，从而来判断数据库是否被修改过
            // 能解决高并发下的线程安全问题
            .eq("voucher_id", voucherId).gt("stock", 0)
            .update();
    if (!success) {
        // 扣减库存失败
        return Result.fail("库存不足！");
    }

    // 接下来创建订单
    VoucherOrder voucherOrder = new VoucherOrder();
    // 生成全局唯一订单 id
    long orderId = redisIdWorker.nextId("order");
    voucherOrder.setId(orderId);
    voucherOrder.setUserId(userId);
    // 设置优惠券 id
    voucherOrder.setVoucherId(voucherId);
    save(voucherOrder);

    // 返回订单 id
    return Result.ok(orderId);
}
```

- redisson 中的 `tryLock` 方法：
  - `tryLock()`：它会使用默认的超时时间和等待机制。具体的超时时间是由 Redisson 配置文件或者自定义配置决定的。
  - `tryLock(long time, TimeUnit unit)`：它会在指定的时间内尝试获取锁（等待time后重试），如果获取成功则返回 true，表示获取到了锁；如果在指定时间内（Redisson内部默认指定的）未能获取到锁，则返回 false。
  - `tryLock(long waitTime, long leaseTime, TimeUnit unit)`：指定等待时间为watiTime，如果超过 leaseTime 后还没有获取锁就直接返回失败;

#### 分布式锁-redisson 可重入锁原理：

![1.webp](https://s2.loli.net/2024/11/09/TCnNgASlvHLf4XM.png)

redisson 可重入机制测试：

```java
@SpringBootTest
@Slf4j
public class RedissonTest {
 
        @Resource
        private RedissonClient redissonClient;
 
        private RLock lock;
        /**
         * 方法1获取一次锁
         */
        @Test
        void method1() {
            boolean isLock = false;
            // 创建锁对象
            lock = redissonClient.getLock("lock");
            try {
                isLock = lock.tryLock();
                if (!isLock) {
                    log.error("获取锁失败，1");
                    return;
                }
                log.info("获取锁成功，1");
                method2();
            } finally {
                if (isLock) {
                    log.info("释放锁，1");
                    lock.unlock();
                }
            }
        }
 
        /**
         * 方法二再获取一次锁
         */
        void method2() {
            boolean isLock = false;
            try {
                isLock = lock.tryLock();
                if (!isLock) {
                    log.error("获取锁失败, 2");
                    return;
                }
                log.info("获取锁成功，2");
            } finally {
                if (isLock) {
                    log.info("释放锁，2");
                    lock.unlock();
                }
            }
        }
 
}
```

**核心就是用一个哈希结构来存线程和获取锁的次数 。**



#### 分布式锁-redisson 锁重试和 WatchDog 机制

**不可重试问题 : 获取锁一次就返回false , 没有重试机制** 。

上面有提到 tryLock 方法的一种参数组合：

`tryLock(long waitTime, long leaseTime, TimeUnit unit)`：指定等待时间为 waitTime，在等待时间内进行获取锁的多次尝试 ，直到等待时间结束（超过 leaseTime）还没有获取锁成功 ，才会返回 false,加了这个参数就变成了一个可重试的锁了；

**WatchDog 机制**：

WatchDog 机制是 Redisson 的一个重要特性，用于自动延长锁的有效时间，防止因客户端崩溃或网络问题导致的锁提前释放。当客户端成功获取锁后，Redisson 会在后台启动一个 WatchDog，定期检查锁的状态并自动续期。

**如何使用 WatchDog 机制？**

1. **自动续期**：当客户端成功获取锁后，Redisson 会自动启动一个 WatchDog，定期发送续期请求，确保锁不会因为超时而被自动释放。
2. **续期间隔**：默认情况下，WatchDog 的续期间隔是锁的持有时间的一半。例如，如果锁的持有时间是 10 秒，那么 WatchDog 每 5 秒会自动发送一次续期请求。

```java
RLock lock = redissonClient.getLock("myLock");

try {
    // 尝试获取锁，最多等待10秒，上锁以后30秒自动解锁
    boolean isLocked = lock.tryLock(10, 30, TimeUnit.SECONDS);
    if (isLocked) {
        System.out.println("Lock acquired!");
        // 执行业务逻辑
        Thread.sleep(5000); // 模拟耗时操作
    } else {
        System.out.println("Failed to acquire lock.");
    }
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    // 释放锁
    lock.unlock();
    System.out.println("Lock released.");
}
```

