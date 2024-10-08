---
layout: post
title: 通过AOP和反射实现不同表的公共字段自动填充
description: 记录
tag: 笔记
---

## 项目问题场景

当前端页面涉及到**新增或更新**数据操作时，在后端数据库中会有更新某些公共字段的需求，比如创建时间、更新时间、创建人等等。

比如下面的 `service` 层代码：

```java
/**
 * 新增员工
 * @param employeeDTO
 */
public void save(EmployeeDTO employeeDTO) {
    System.out.println("当前线程的 id：" + Thread.currentThread().getId());
    Employee employee = new Employee();

    // 对象属性拷贝，将前者的属性拷贝到后者的属性上，前提是两者的属性名一致
    BeanUtils.copyProperties(employeeDTO, employee);

    // 设置账号状态，默认正常状态 1表示正常，0表示锁定
    employee.setStatus(StatusConstant.ENABLE); // 用常量类来设置，不用具体的数字（硬编码）

    // 设置密码，默认为 123456，同样使用常量类来设置
    employee.setPassword(DigestUtils.md5DigestAsHex(PasswordConstant.DEFAULT_PASSWORD.getBytes()));

    // 设置当前记录的创建时间和修改时间
    employee.setCreateTime(LocalDateTime.now());
    employee.setUpdateTime(LocalDateTime.now());

    // 设置当前记录创建人 id 和修改人 id
    employee.setCreateUser(BaseContext.getCurrentId());
    employee.setUpdateUser(BaseContext.getCurrentId());

    employeeMapper.insert(employee);
}

/**
 * 编辑员工信息
 * @param employeeDTO
 */
public void update(EmployeeDTO employeeDTO) {
    Employee employee = new Employee();
    BeanUtils.copyProperties(employeeDTO, employee);

    employee.setUpdateTime(LocalDateTime.now());
    employee.setUpdateUser(BaseContext.getCurrentId()); // 获取操作人 id，此处是通过拦截器的 thread 获取对应信息

    employeeMapper.update(employee);
}
```

**其中更新 `updateTime` 、`setUpdateUser` 等操作在重复的调用相同的 `set` 方法**，会增加代码的冗余且不便于后期维护。

于是，这里可以采用 AOP 和反射的方法来实现对于不同表的公共字段的自动填充。





## 具体实现

要使用 AOP ，我们需要有一个切入点 `cutPoint` ，在代码执行到对应的 `service` 层方法时，将其拦截下来，利用 `Java` 的反射特性进行公共字段的填充。

- 首先自定义一个注解 `AutoFill`，用于标识某个方法需要进行功能字段自动填充处理（代码中的 `OperationType` 是枚举类，存放的 `UPDATE` 和 `INSERT`）

```java
/**
 * 自定义注解，用于标识某个方法需要进行功能字段自动填充处理
 */
@Target(ElementType.METHOD) // 作用在方法上的注解
@Retention(RetentionPolicy.RUNTIME) // 生命周期是程序运行时
public @interface AutoFill {
    // 数据库操作类型：UPDATE INSERT
    OperationType value();
}
```

- 然后自定义一个切面类，**确定切入点并确认通知类型为前置通知（因为要在更新或新增操作之前将表中的公共字段进行填充）**，最后在该类中实现公共字段自动填充逻辑，具体看注释

```java
/**
 * 自定义切面，实现公共字段自动填充逻辑
 */
@Aspect
@Component
@Slf4j
public class AutoFillAspect {

    /**
     * 切入点
     */
    @Pointcut("execution(* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)")
    public void autoFillPointCut() {}

    /**
     * 前置通知，在通知中进行公共字段的赋值
     */
    @Before("autoFillPointCut()")
    public void autoFill(JoinPoint joinPoint) {
        log.info("开始进行公共字段自动填充...");

        // 获取当前被拦截的方法上的数据库操作类型
        MethodSignature signature = (MethodSignature) joinPoint.getSignature(); // 获取方法签名对象
        AutoFill autoFill = signature.getMethod().getAnnotation(AutoFill.class); // 获得方法上的注解对象
        OperationType operationType = autoFill.value(); // 获取数据库操作类型

        // 获取到当前被拦截的方法参数--实体对象
        Object[] args = joinPoint.getArgs();
        if (args == null || args.length == 0) { // 特判获取的对象是否为空
            return;
        }

        Object entity = args[0]; // 获取到的实体对象（拦截到的方法可能有多个参数，此处要保证实体参数在第一个位置）

        // 准备赋值的数据
        LocalDateTime now = LocalDateTime.now();
        Long currentId = BaseContext.getCurrentId();

        // 根据当前不同的操作类型，为对应的属性通过反射来赋值
        if (operationType == OperationType.INSERT) {
            // 为 4 个公共字段赋值
            try {
                Method setCreateTime = entity.getClass().getDeclaredMethod(AutoFillConstant.SET_CREATE_TIME, LocalDateTime.class);
                Method setCreateUser = entity.getClass().getDeclaredMethod(AutoFillConstant.SET_CREATE_USER, Long.class);
                Method setUpdateTime = entity.getClass().getDeclaredMethod(AutoFillConstant.SET_UPDATE_TIME, LocalDateTime.class);
                Method setUpdateUser = entity.getClass().getDeclaredMethod(AutoFillConstant.SET_UPDATE_USER, Long.class);

                // 通过反射为对象属性赋值
                setCreateTime.invoke(entity, now);
                setCreateUser.invoke(entity, currentId);
                setUpdateTime.invoke(entity, now);
                setUpdateUser.invoke(entity, currentId);
            } catch (Exception e) {
                e.printStackTrace();
            }

        } else if (operationType == OperationType.UPDATE) {
            // 为 2 个公共字段赋值
            try {
                Method setUpdateTime = entity.getClass().getDeclaredMethod(AutoFillConstant.SET_UPDATE_TIME, LocalDateTime.class);
                Method setUpdateUser = entity.getClass().getDeclaredMethod(AutoFillConstant.SET_UPDATE_USER, Long.class);

                // 通过反射为对象属性赋值
                setUpdateTime.invoke(entity, now);
                setUpdateUser.invoke(entity, currentId);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }


    }
}

```

- 最后在 `mapper` 层对应的方法上添加自定义的 `@AutoFill` 注解，确认切入点为当前方法

  - EmployeeMapper

  ```java
  /**
   * 插入员工数据
   * @param employee
   */
  @Insert("insert into employee (name, username, password, phone, sex, id_number, create_time, update_time, create_user, update_user, status) " +
          "value " +
          "(#{name},#{username},#{password},#{phone},#{sex},#{idNumber},#{createTime},#{updateTime},#{createUser},#{updateUser},#{status})")
  @AutoFill(value = OperationType.INSERT)
  void insert(Employee employee);
  
  /**
   * 根据主键动态修改属性
   * @param employee
   */
  @AutoFill(value = OperationType.UPDATE)
  void update(Employee employee);
  ```

  - CategoryMapper

  ```java
  /**
   * 插入数据
   * @param category
   */
  @Insert("insert into category(type, name, sort, status, create_time, update_time, create_user, update_user)" +
          " VALUES" +
          " (#{type}, #{name}, #{sort}, #{status}, #{createTime}, #{updateTime}, #{createUser}, #{updateUser})")
  @AutoFill(value = OperationType.INSERT)
  void insert(Category category);
  
  /**
   * 根据id修改分类
   * @param category
   */
  @AutoFill(value = OperationType.UPDATE)
  void update(Category category);
  ```





