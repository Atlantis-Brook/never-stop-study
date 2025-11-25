#Spring #Transactional #Dynamic-datasource
**为什么 @Transactional 会导致多数据源 @DS 失效？为什么两个数据源有相同表结构时更容易踩坑？如何正确解决？**

---

# 🎯 关键结论（先告诉你答案）

> **在使用 dynamic-datasource 时，`@Transactional` 默认只绑定到“主数据源”**  
> 这会导致：
> 
> - `@DS` **失效**
>     
> - SQL 全部落到一个数据源
>     
> - 尤其在两个库“表结构一致”时更无感，容易误以为正常
>     

**原因：事务 AOP 的切面执行顺序高于数据源路由切面，导致 DS 不能生效。**

解决方案见后面。

---

# 🧨 为什么 @Transactional 会导致 @DS 失效？

## 1）Spring 事务注解优先级比 @DS 高（重要！）

Spring 的切面执行顺序：

|切面|执行先后|
|---|---|
|**事务切面（@Transactional）**|**先执行**|
|数据源切面（DSDynamicDataSourceAspect）|后执行|

也就是说：

### 当进入一个 `@Transactional` 方法时：

- Spring **已经决定使用哪个数据源的事务管理器**
    
- 事务开始后 `@DS` 才执行，这时候切换数据源已经 **来不及了**
    
- 因为：一个事务只能绑定到 1 个 DataSource
    

因此：

> **有事务时，数据源由事务管理器决定，不由 @DS 决定。**

---

# 🌋 为什么“两个数据源表结构一样”时更容易误判？

因为你看不出来它其实根本没有切换数据源，例如：

- archery 库和 default 库都有表：`t_user`
    
- 由于事务绑定到了 default
    
- 所有查询都跑 default
    
- 结果照样查出来，不报错
    

你会误以为：

> “@DS 正常工作”

实际根本没切！

---

# 🔍 复现现象（你肯定遇到过）

```java
@Service
public class MyService {

    @DS("archery")
    @Transactional
    public void queryArchery() {
        archeryMapper.selectById(1); // ❌ 实际查询的是 default
    }
}
```

结果：**查的不是 archery，而是默认数据源**

如果 archery 和 default 结构相同 → 你根本察觉不到。

---

# 🚀 正确解决方式（按优先级推荐）

---

## ✅ **方案 1：事务也使用动态数据源管理器（推荐）**

使用 dynamic-datasource 提供的 **DynamicDataSourceTransactionManager**

你需要让事务管理器支持多数据源，否则 Spring 默认使用主数据源事务管理器。

**如果你用的是 starter：**

```yaml
spring:
  datasource:
    dynamic:
      primary: default
      strict: false
      datasource:
        default:
          url: ...
        archery:
          url: ...
```

它会自动启用 `DynamicDataSourceTransactionManager`  
→ **@Transactional 将自动跟随 @DS 切换数据源**  
→ 不会再失效

📌 **前提：不要自己定义 DataSourceTransactionManager！**

---

## ✅ **方案 2：给方法加 @DSTransactional（官方提供）**

动态数据源提供了一个替代方案：

```java
@DSTransactional
@DS("archery")
public void test(){}
```

特点：

- 事务和数据源绑定在一起
    
- 正确地先决定数据源，再开启事务
    

---

## ⛔ 错误方式：把 @Transactional 放在 Service 类上

比如：

```java
@Service
@Transactional
public class MyService { ... }
```

这样：

- 所有方法共享同一个数据源事务管理器（通常是主库）
    
- @DS 根本无法生效
    

---

# 🎯 建议你按如下规则使用

|场景|用法|
|---|---|
|单个方法切换数据源并需要事务|用 `@DSTransactional` + `@DS`|
|整个 service 读写都属于某个数据源|在 Service 类上用 `@DS`，方法上用 `@Transactional`|
|有多个数据源并带事务|确保使用 DynamicDataSourceTransactionManager|

---

# 🧪 如果你愿意，我可以帮你检查你的配置

如果你把你的：

- `application.yml`（动态数据源部分）
    
- 自己有没有声明事务管理器的配置类
    
- 你使用的 @DS + @Transactional 代码示例
    

发出来，我可以告诉你 **到底是不是因为事务管理器配置错误导致的**（90% 的公司都会配错）。


下面我把 **@DSTransactional 的内部原理** 用最清晰、最工程化的方式讲透。

> 结论先说：  
> **@DSTransactional 的本质是“一个整合了 @DS 与 @Transactional 的组合注解 + 一个自定义 Transaction Advisor（切面）”，它在事务开启之前先执行数据源切换，从而保证事务绑定的是正确的数据源。**

这是为什么 @Transactional + @DS 会失效，而 @DSTransactional 不会。

---

# 🧱 一、为什么 @Transactional + @DS 会失效（理解原理的前提）

Spring 的 AOP 优先级是：

```
事务切面（@Transactional） > 动态数据源切面（@DS）
```

也就是说：

### 进入方法时发生的顺序

1. **@Transactional 先执行** → 事务管理器已经绑定主数据源
    
2. @DS 想切换数据源，但已经晚了（事务已经开始）
    

所以事务期间数据源不会变，统一是 `primary` 数据源。

---

# 🚀 二、@DSTransactional 是怎么解决这个顺序问题的？

Dynamic-datasource 额外定义了一个 **特殊的 Spring 切面（Advisor）**

> **它的优先级比事务切面更高**

因此执行顺序变成：

```
@DSTransactional（动态事务切面） → 切换数据源
      ↓
真正进入 Spring 的事务切面 → 开启正确数据源的事务
```

---

# 📦 三、@DSTransactional 的源码机制（核心原理）

下面是 dynamic-datasource 核心的执行逻辑（简化解释）：

---

## 1) @DSTransactional 不是简单的标记，而是一个复合注解

源码位置（版本 3.x）：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Transactional
@DS
public @interface DSTransactional {
    String value() default "";
}
```

关键点：

- 它 **同时包含了 @Transactional 和 @DS**
    
- 但它不是简单叠加，而是 **会被专门的切面增强处理**
    

---

## 2) Dynamic-datasource 给 @DSTransactional 定义了一个独立的事务 Advisor

源码（简化）：

```java
public class DynamicLocalTransactionAdvisor extends AbstractPointcutAdvisor {

    public DynamicLocalTransactionAdvisor(...) {
        this.advice = new DynamicLocalTransactionInterceptor(...);
    }
}
```

这个 Advisor：

### 优先级高于 Spring 事务切面（@Transactional）

通过：

```java
@Order(Ordered.LOWEST_PRECEDENCE - 1)
```

Dynamic-datasource 的切面实际上排到了事务切面前面。

---

## 3) 这个 Advisor 会做两件事：

---

### **① 先根据 @DSTransactional 的值切换数据源**

```java
DynamicDataSourceContextHolder.push(dsName);
```

这一步完成数据源切换。

放在事务开启前执行 → 关键所在。

---

### **② 再调用 Spring 的事务逻辑（即 @Transactional）**

内部最终仍然调用：

```java
TransactionInterceptor.invoke()
```

但是因为数据源已经切换了，所以事务绑定正确的数据源。

---

# 🔥 四、用一句话总结核心原理

> **@DSTransactional 通过一个优先级比事务切面更高的 AOP 切面，在事务开启之前强制切换数据源，从而确保事务真正使用 @DS 指定的数据源。**

这就是它能解决事务中 @DS 失效的根本原因。

---

# 🎯 五、为什么两个数据源表结构相同的时候更容易“看不出问题”

因为：

- 当 @Transactional + @DS 失效时，实际查询的是主库
    
- 两个库表结构一样
    
- 查询仍然能正常返回
    

所以你会误以为：

> 数据源切换成功了。

实际上并没有。

---

# 📘 六、什么时候必须使用 @DSTransactional？

|场景|是否需要|
|---|---|
|方法内需要数据源切换并开启事务|✔ 必须|
|方法内只有一个数据源，但是非主数据源|✔ 建议|
|不需要事务|✘ 不用|
|使用 @DS 在 Service 类上 + 方法上使用 @Transactional|✔ 可行（因为切面顺序正确）|

---

# 📌 七、你可能在问的附加问题

如果你需要，我也可以进一步讲：

- @DS 的切面执行顺序（DSDynamicDataSourceAspect 原理）
    
- 为什么 @DS 放在 Mapper 不如放 Service 层
    
- 多数据源事务如何回滚
    
- 多数据源混合事务是否支持分布式事务
    

可以继续问。
