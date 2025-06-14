---
layout: post
title: Springboot框架学习笔记
description: 记录
tag: 笔记
---



## SpringBoot概念与功能

**概念**

- SpringBoot提供一种快速使用Spring的方式
- 基于约定优于配置的思想
- 不必在配置与逻辑业务之间进行思维切换，全身心投入到逻辑业务的代码编写

**功能**

- **自动配置**：SpringBoot的自动配置是一个运行时（准确来说是程序启动时）的过程，这些过程均由SpringBoot自动完成
- **起步依赖**：将具备某种功能的坐标打包到一起，并提供一些默认的功能

- **辅助功能**：提供一些大型项目中的非功能性特性，如嵌入式服务器、安全、指标等





## SpringBoot配置

### 配置文件分类

SpringBoot是基于约定的，**很多配置都有默认值**，如果想**使用自己的配置替换默认配置**的话，可以使用**application.properties或者application.yml(application.yaml)**进行配置

- properties

```properties
server.port=8080
```

- yml

```yaml
server
 port: 8080
```



- SpringBoot提供了2种配置文件类型：**properties和yml/yaml**
- 默认配置文件名称：application
- **在同一级目录下优先级为：properties > yml > yaml**



### yaml文件

yaml全称是**YAML Ain't Markup Language**。yaml是一种直观的能被电脑识别的数据序列化格式，并容易理解阅读。



**yaml基本语法**

- 大小写敏感
- **数据值前必须有空格，作为分隔符**
- **使用缩进表示层级关系**
- 缩进时不允许使用Tab键，只允许使用空格（不同操作系统Tab对应的空格数可能不同，导致层级混乱）
- **所进的空格数目不重要，只要相同层级的元素左侧对齐即可**
- #表示注释，从这个字符一直到行尾，都会被解析器忽略

```yaml
server
	port: 8080
	address: 127.0.0.1
	
name: abc
```



**yaml数据格式**

- 对象(map)：键值对的集合

```yaml
person:
	name: zhangsan
# 行内写法
person: {name: zhangsan}
```

- 数组：一组**按次序排列**的值

```yaml
address:
	- beijing
	- shanghai
# 行内写法
address: [beijing,shanghai]
```

- 纯量：单个的、不可再分的值（可理解为常量）

```yaml
msg1: 'hello \n world' # 单引号忽略转义字符
msg2: "hello \n world" # 双引号识别转义字符
```

- 参数引用

```yaml
name: lisi

person:
	name: ${name} # 引用上面定义的name的值
```





### 读取配置文件内容

- @Value

```java
@Value("${name}")
private String name;

@Value("${person.name}")
private String name2;
```

- Environment

```java
@Autowired
private Environment env;

System.out.println(env.getProperty("person.name"));
System.out.println(env.getProperty("address[0]"));
```

- @ConfigurationProperties

**创建对象与yml配置文件的内容进行绑定**

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private int age;
    private String[] address;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String[] getAddress() {
        return address;
    }

    public void setAddress(String[] address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

```yaml
person:
  name: zhangsan
  age: 20
  address:
    - beijing
    - shanghai
```





### profile

在开发SpringBoot应用时，通常同一套程序会被安装到不同环境，比如开发、测试、生产等。不同环境下的配置不同，若每次打包时都要修改配置文件会非常麻烦。**profile功能就是来进行动态配置切换的**

- profile配置方式

  - **多profile文件方式**：提供多个配置文件，每个代表一个环境

    - application-dev.properties/yml  开发环境
    - application-test.properties/yml  测试环境
    - application-pro.properties/yml  生产环境

  - **yml多文档方式**

    - 在yml中使用 --- 分隔不同配置

    ```yaml
    ---
    server:
      port: 8081
    
    spring:
      profiles: dev
    ---
    server:
      port: 8082
    
    spring:
      profiles: test
    ---
    server:
      port: 8083
    
    spring:
      profiles: pro
    ---
    spring:
      profiles:
        active: dev
    ```

    

- profile激活方式

  - 配置文件：在配置文件中配置

  ```yaml
  spring.profiles.active=dev
  ```

  - 虚拟机参数：在VM options指定

  ```yaml
  -Dspring.profiles.active=dev
  ```

  - 命令行参数

  ```yaml
  java -jar xxx.jar --spring.profiles.active=dev
  ```

  



### 内部配置加载顺序

SpringBoot程序启动时，会从以下位置加载配置文件

- file:./config/：当前项目的/config目录下
- file:./：当前项目的根目录
- classpath:/config/：classpath的/config目录
- classpath:/：classpath的根目录

**加载顺序为上文的排列顺序，高优先级配置的属性会生效**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvOTI4OTUzLzIwMTkwNi85Mjg5NTMtMjAxOTA2MTMxMTM2NTk0NjEtMTE3MjA3ODc4LnBuZw?x-oss-process=image/format,png) 



​                                                                                                  



### 外部配置加载顺序

参考[Springboot加载外部配置文件的方法_new defaultresourceloader().getresource-CSDN博客](https://blog.csdn.net/qq_41665121/article/details/105674208)





## SpringBoot原理分析

### SpringBoot自动配置

#### Condition

通过Conditon这个功能，可以实现选择性的创建Bean操作

- **自定义条件**
  - 定义条件类：自定义类实现Condition接口，重写matches方法，在matches方法中进行逻辑判断，返回boolean值。**以下为matches方法的两个参数**
    - **context：上下文对象，可以获取属性值，获取类加载器，获取BeanFactory等。具体来说，通过context对象，可以获取环境、IoC容器、ClassLoader对象等等**
    - **metadata：元数据对象，用于获取注解属性**
    
    ```java
    //获取注解属性值value
    Map<String, Object> map = 			 metadata.getAnnotationAttributes(ConditionOnClass.class.getName());
    ```
    
  - 判断条件：在初始化Bean时，**使用@Conditional(条件类.class)注解**
- SpringBoot提供的常用条件注解
  - **ConditionalOnProperty**：判断配置文件中是否有对应属性和值才初始化Bean
  - **ConditionalOnClass**：判断环境中是否有对应字节码文件才初始化Bean
  - **ConditionalOnMissingBean**：判断环境中没有对应Bean才初始化Bean





#### 切换内置web服务器

SpringBoot的web环境中默认使用tomcat作为内置服务器，其实SpringBoot提供了4种内置服务器供选择，我们可以修改maven工程中的pom配置文件进行很方便的切换

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!--排除tomcat依赖-->
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

    <!--引入jetty的依赖-->
<dependency>
    <artifactId>spring-boot-starter-jetty</artifactId>
    <groupId>org.springframework.boot</groupId>
</dependency>
```





#### @Enable*注解

SpringBoot提供了很多Enable开头的注解，这些注解都是用于动态启用某些功能的。**其底层原理是使用@Import注解导入一些配置类**，实现动态加载

在一个工程中想使用其他工程定义的bean时，可以采用以下几种方法

- **使用@ComponentScan扫描bean对应的包**

```java
@ComponentScan("com.itheima.config")
```

- **使用@Import注解，加载类。这些类都会被Spring创建并放入IoC容器**

```java
@Import(UserConfig.class)
```

- **可以对Import注解进行封装**

自定义封装的EnableUser注解类

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(UserConfig.class)
public @interface EnableUser {
}
```

调用@EnableUser注解

```java
@EnableUser
public class SpringbootEnableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

        Object user = context.getBean("user");
        System.out.println(user);

    }
}
```





#### @Import注解

@Enable*底层依赖于@Import注解导入一些类，使用@Import导入的类会被Spring加载到IoC容器中。而@Import提供4种用法

- **导入Bean**

下面的代码直接导入Userbean

```java
@Import(User.class)
public class SpringbootEnableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

        // 按类型获取，不能按名称获取，若使用按名称获取可能会找不到
        User user = context.getBean(User.class);
        System.out.println(user);

    }
}
```



- **导入配置类**

```java
@Import(UserConfig.class)
public class SpringbootEnableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

		// UserConfig配置类中有两个bean：User和Role，均能被获取到
        User user = context.getBean(User.class);
        System.out.println(user);

        Role role = context.getBean(Role.class);
        System.out.println(role);
    }
}
```

**对于这种导入方法，配置类UserConfig上面的注解@Configuration是可以省略的**

```java
//@Configuration
public class UserConfig {

    @Bean
    public User user() {
        return new User();
    }

    @Bean
    public Role role() {
        return new Role();
    }
}
```



- **导入ImportSelector实现类，一般用于加载配置文件中的类**

实现类如下

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 复写接口方法，导入两个bean：User和Role
        return new String[]{"com.itheima.domain.User", "com.itheima.domain.Role"};
    }
}
```

再导入实现类即可

```java
@Import(MyImportSelector.class)
public class SpringbootEnableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

		// MyImportSelector实现类中有两个bean：User和Role，均能被获取到
        User user = context.getBean(User.class);
        System.out.println(user);

        Role role = context.getBean(Role.class);
        System.out.println(role);
    }
}
```

**这种方式和上一种方式类似，但这种方式在配置实现类中是以字符串实现的，所以可以用这个方式加载一些配置文件的bean**



- **导入ImportBeanDefinitionRegistrar实现类**

创建ImportBeanDefinitionRegistrar的实现类

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        // 获取User类的定义
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(User.class).getBeanDefinition();
        // 注册以user为名称，User类的定义为定义的bean
        registry.registerBeanDefinition("user", beanDefinition);
    }
}
```

导入ImportBeanDefinitionRegistrar实现类

```java
@Import({MyImportBeanDefinitionRegistrar.class})
public class SpringbootEnableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringbootEnableApplication.class, args);

		// ImportBeanDefinitionRegistrar中只有User的bean，没有Role的bean
        // 所以输出role时会报错
        User user = context.getBean(User.class);
        System.out.println(user);

        Role role = context.getBean(Role.class);
        System.out.println(role);
    }
}
```





#### @EnableAutoConfiguration注解

- @EnableAutoConfiguration注解内部使用@Import(AutoConfigurationImportSelector.class)来加载配置类
- 配置文件位置：**META-INF/spring.factories**，该配置文件中定义了大量的配置类，当SpringBoot应用启动时，会自动加载这些配置类，初始化bean
- 并不是所有的bean都会被初始化，在配置类中使用Condition来加载满足条件的bean

**说明：spring.factories是SpringBoot中用于自动配置的一种机制，位于META-INF目录下。spring.factories文件采用了 Java properties 文件的格式，每一行表示一个自动配置类及其对应的条件**





### SpringBoot监听机制

**SpringBoot的监听机制，其实是对java提供的事件监听机制的封装**



**java监听机制**

- java中的事件监听机制定义了以下几个角色

  - 事件：Event，继承java.util.EventObject类的对象

  - 事件源：Source，任意对象Object

  - 监听器：Listener，实现java.util.EventListener对象



**SpringBoot监听机制**

SpringBoot在项目启动时，会对几个监听器进行回调，**我们可以实现这些监听器接口，在项目启动时完成一些操作**

监听器接口：ApplicationContextInitializer、SpringApplicationRunListener、CommandLineRunner、ApplicationRunner

- **ApplicationContextInitializer**
  - 这个接口允许在应用程序上下文被创建之前对其进行定制和配置
  - 可以用来做一些初始化的工作，例如添加自定义属性源、激活特定的Spring配置文件等
  - 通过实现这个接口，可以在Spring应用程序上下文被创建之前对其进行自定义设置

```
补充：在java中，"上下文对象"通常是指一种数据结构或对象，用于存储和传递在特定环境中的相关信息或状态。这个概念在不同的技术和框架中有不同的具体实现和用法。

在一些情况下，上下文对象可以是一个普通的 Java 对象，用于存储和传递方法调用或处理过程中的相关信息。例如，在多线程编程中，可以创建一个包含线程状态、线程局部变量等信息的上下文对象，以便在方法调用之间传递这些信息。

在一些框架和技术中，上下文对象可能具有更具体的含义和功能。例如：
1.Servlet 上下文对象：在 Java Web 开发中，Servlet 上下文对象是用于存储 Servlet 环境相关信息的对象。它可以通过 ServletContext 接口来访问，其中包含了 Servlet 容器的一些配置信息、Servlet 的初始化参数等。

2.Spring 应用程序上下文对象：在 Spring 框架中，应用程序上下文对象是 Spring IoC 容器中的一个重要组成部分，用于管理和维护 Bean 的定义、依赖关系等信息。它可以通过 ApplicationContext 接口来访问，其中包含了应用程序中所有 Bean 的定义、配置信息等。

3.数据库连接上下文对象：在使用 JDBC 进行数据库操作时，可以创建一个数据库连接上下文对象，用于管理数据库连接的状态、事务信息等。

总的来说，上下文对象是一种在特定环境中存储和传递相关信息的通用机制，在不同的技术和框架中具有不同的实现和用法。它可以帮助组织和管理程序执行过程中的状态和信息，提高代码的灵活性和可维护性。
```



- **SpringApplicationRunListener**
  - 这个接口**用于监听SpringBoot应用程序的启动过程中的事件**，例如应用程序启动、运行失败等
  - 可以用来实现一些高级的应用程序启动时的逻辑，例如记录日志、发送通知等
  - 通过实现这个接口，可以在应用程序启动的各个阶段添加自定义行为



- **CommandLineRunner**
  - 这是一个函数式接口，**用于在SpringBoot应用程序启动后立即执行一些逻辑**
  - 它提供了一个**'run'**方法，该方法会在SpringBoot应用程序启动后立即执行，**且可以访问应用程序启动时的命令行参数**
  - **可以用来实现一些与命令行相关的初始化工作**，例如加载数据、执行特定任务等

```
补充：函数式接口(Functional Interface)是指仅包含一个抽象方法的接口。Java 8 引入了函数式接口的概念，以支持函数式编程风格和Lambda表达式。函数式接口通常用于表示可以作为Lambda表达式传递的类型，或者作为方法引用的目标类型。

函数式接口具有以下特征：
1.只包含一个抽象方法：函数式接口只能包含一个未实现的抽象方法。它可以包含多个默认方法或静态方法，但只能有一个抽象方法。
2.可以使用@FunctionalInterface注解：虽然不是必须的，但通常将函数式接口标记为@FunctionalInterface注解，以便编译器检查它是否符合函数式接口的要求。如果一个接口标记为@FunctionalInterface，但它不符合函数式接口的要求（例如包含多个抽象方法），编译器将产生错误。
3.Lambda表达式和方法引用的目标类型：函数式接口可以被用作Lambda表达式的类型。Lambda表达式提供了一种简洁的语法来实现函数式接口的抽象方法。同样，函数式接口也可以是方法引用的目标类型。
```



- **ApplicationRunner**
  - 与**'CommandLineRunner'**类似，**也是用于在SpringBoot应用程序启动后立即执行一些逻辑的接口**
  - 不同之处在于，**'ApplicationRunner'**的**'run'**方法接受的参数是**'ApplicationArguments'**，而不是简单的字符串数组
  - **'ApplicationArguments'**提供了更丰富的访问应用程序启动时的参数信息的能力，例如访问非标准选项、参数值的类型转换等



**要使上面四种监听器接口中的ApplicationContextInitializer和SpringApplicationRunListener生效，需要在配置META-INF文件夹和其中的spring.factories**





### SpringBoot启动流程分析

参考[12-SpringBoot流程分析-初始化_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Lq4y1J77x?p=29&vd_source=548ea3473b15a87db5000cd98a1411b1)的P29-P30







## SpringBoot监控

SpringBoot自带监控功能Actuator，可以帮助实现对程序内部运行情况监控，比如监控状况、bean加载情况、配置属性、日志信息等



**使用步骤**

- 导入依赖坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 访问

  ```
  http://localhost:8080/actuator
  ```







## SpringBoot项目部署

SpringBoot项目开发完毕后，支持两种方式部署到服务器：

- jar包
- war包

