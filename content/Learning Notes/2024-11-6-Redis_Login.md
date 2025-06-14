---
layout: post
title: 基于Redis的用户短信登陆功能
description: 记录
tag: 笔记
---

## 1.发送验证码功能实现

前端会给后端传入 String 和一个 HttpSession ，然后对应的 controller 调用 service 层的 `sendCode` 方法：

```java
/**
 * 发送手机验证码
 */
@PostMapping("code")
public Result sendCode(@RequestParam("phone") String phone, HttpSession session) {
    return userService.sendCode(phone, session);
}
```

通过 `RandomUtil` 实现随机生成验证码并保存到 redis 中。**此处的返回验证码仅仅是模拟返回过程**，并不是真实返回。

```java
/**
 * 发送验证码
 * @param phone
 * @param session
 * @return
 */
@Override
public Result sendCode(String phone, HttpSession session) {
    // 1.校验手机号
    if (RegexUtils.isPhoneInvalid(phone)) {
        // 2.如果不符合
        return Result.fail("手机号格式错误！");
    }
    // 3.符合，生成验证码
    String code = RandomUtil.randomNumbers(6);
    // 4.保存验证码到 redis
    stringRedisTemplate.opsForValue().set(LOGIN_CODE_KEY + phone, code, LOGIN_CODE_TTL, TimeUnit.MINUTES);
    // 5.返回验证码
    log.debug("发送短信验证码成功，验证码：{}", code);
    return Result.ok();
}
```



## 2.登录功能实现

前端会给后端传入 LoginFormDTO 和一个 HttpSession ，然后对应的 controller 调用 service 层的 `login` 方法：

```java
/**
 * 登录功能
 * @param loginForm 登录参数，包含手机号、验证码；或者手机号、密码
 */
@PostMapping("/login")
public Result login(@RequestBody LoginFormDTO loginForm, HttpSession session){
    return userService.login(loginForm, session);
}
```

用户登录需要两个信息：手机号与验证码。手机号由用户前端输入提供，验证码由后端生成并存入 redis 数据库里，在用户登录时再访问 redis 查询比对，两者都符合就允许登录。

验证码正确后会到后端数据库内查询是否存在该用户，如果不存在就创建新用户。

有了 user 对象后，先生成随机 token ，然后将 user 转换为 userDTO 存放到 redis 数据库中（该内容需要频繁查询，使用 redis 提高系统的访问速度），注意转换成 userDTO 的小细节，最后返回 token

```java
/**
 * 用户登录
 * @param loginForm
 * @param session
 * @return
 */
@Override
public Result login(LoginFormDTO loginForm, HttpSession session) {
    // 校验手机号
    String phone = loginForm.getPhone();
    if (RegexUtils.isPhoneInvalid(phone)) {
        return Result.fail("手机号格式错误");
    }

    // 从 redis 获取并校验验证码
    String cacheCode = stringRedisTemplate.opsForValue().get(LOGIN_CODE_KEY + phone);
    String code = loginForm.getCode();
    if (cacheCode == null || !cacheCode.equals(code)) {
        // 不一致，直接报错
        return Result.fail("验证码错误");
    }

    // 一致，根据手机号查询用户
    User user = query().eq("phone", phone).one();

    // 判断用户是否存在
    if (user == null) {
        // 不存在，创建新用户并保存
        user = createUserWithPhone(phone);
    }

    // 随机生成 token
    String token = UUID.randomUUID().toString(true);
    UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);

    // 此处 userDTO 转换为 Map 类型时，userDTO 内有 Long 类型无法转成 String 类型，而 redis 存放的是 String-String 类型
    // 所以需要用工具类手动的将 userDTO 的所有属性转换为 String 类型
    Map<String, Object> userMap = BeanUtil.beanToMap(userDTO, new HashMap<>(),
            CopyOptions.create()
                    .setIgnoreNullValue(true)
                    .setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString()));
    String tokenKey = LOGIN_USER_KEY + token;

    // 此处如果 userMap 没有进行转换的话，其中有 Long 类型无法直接存到 redis 数据库中
    stringRedisTemplate.opsForHash().putAll(tokenKey, userMap);
    stringRedisTemplate.expire(tokenKey, LOGIN_USER_TTL, TimeUnit.MINUTES);
    // 返回 token
    return Result.ok(token);
}
```



## 3.如何维持用户的登录状态？

在 redis 中，每个键值对都对应着存活时间 TTL ，我们需要根据用户的操作实时刷新 redis 中对应用户 token 的 TTL 。同时，我们也要对当前用户进行判断，是否在 redis 数据库中，如果不在，说明用户并没有登录，**有部分功能非登录用户不能访问**，需要进行拦截。这个功能可以通过拦截器实现。

我们设置两个拦截器：

- 第一个拦截器拦截所有路径，获取 session 里的 token ，然后到  redis 里查询是否有这个用户，从而判断当前用户有没有登陆。如果登陆了，就保存到 ThreadLocal 里供后端使用，并刷新 token 有效期，实现维持用户登陆状态。

```java
public class RefreshTokenInterceptor implements HandlerInterceptor {

    private StringRedisTemplate stringRedisTemplate;

    // 此处的 StringRedisTemplate 只能通过构造函数进行注入，因为当前 LoginInterceptor 对象并不是 spring 管理的 bean 对象
    public RefreshTokenInterceptor(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    // 前置拦截器实现用户登录校验
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("authorization");
        if (StrUtil.isBlank(token)) {
            // 直接放行，拦截功能在 LoginInterceptor 里完成
            return true;
        }

        // 基于 token 获取 redis 中的用户
        String key = RedisConstants.LOGIN_USER_KEY + token;
        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(key);

        // 判断用户是否存在
        if (userMap.isEmpty()) {
            // 直接放行，拦截功能在 LoginInterceptor 里完成
            return true;
        }

        // 将查询到的 Hash 数据转为 UserDTO 对象
        UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);
        // 保存用户信息到 ThreadLocal
        UserHolder.saveUser(userDTO);
        // 刷新 token 有效期
        stringRedisTemplate.expire(key, RedisConstants.LOGIN_USER_TTL, TimeUnit.MINUTES);
        // 放行
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 移除用户，防止内存泄漏
        UserHolder.removeUser();
    }
}
```

- 第二个拦截器拦截需要登陆的路径，由第一个拦截器放行的内容来判断当前用户是否登录，如果没有登陆就拦截。

```java
public class LoginInterceptor implements HandlerInterceptor {

    // 前置拦截器实现用户登录校验
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // ThreadLocal 中没有用户，拦截
        if (UserHolder.getUser() == null) {
            response.setStatus(401);
            return false;
        }
        // ThreadLocal 中有用户，直接放行
        return true;
    }
}
```

- 拦截器配置类

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/voucher/**",
                        "/upload/**",
                        "/shop-type/**",
                        "/shop/**",
                        "/blog/hot",
                        "/user/code",
                        "/user/login"
                ).order(1);
        registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate)).addPathPatterns("/**").order(0);
    }
}
```

