非常好的问题，这种现象在 Spring Boot / Spring Cloud Alibaba + Nacos 配置绑定中**非常常见**：  
👉 顶层的 `enable` 可以正常注入，但嵌套的 `crm`、`urlMap` 都是 `null`。

我们来系统分析一下原因。

---

## 🧩 一、你当前的类结构

```java 
@Configuration("decryptWhitelistConfig")
@Data
@ConfigurationProperties(prefix = "decrypt-whitelist")
public class DecryptWhitelistConfig {

    private Boolean enable;
    private Crm crm;
    private Map<String, List<String>> urlMap;

    @Data
    public class Crm {
        private String getUserIsSaleByEmailUrl;
    }
}

```

---

## 🧠 二、Nacos 配置示例（已知）
```yml
decrypt-whitelist:
  enable: true
  crm:
    getUserIsSaleByEmailUrl: growth.edianzu.cn/api/growth/directmarketing/lob/customer/outside/user/getUserIsSaleByEmail?email=
  urlList:
    user: /
    telemarket: /api/growth/directmarketing/lob/operations/business/stat/customer/detail, ...

```


---

## 🚨 三、问题总结

### ✅ 能注入的：

`enable: true` → ✅ 正常注入。

### ❌ 不能注入的：

`crm`、`urlList` → ❌ `null`

---

## ⚙️ 四、原因分析（关键点）

### 🧩 1️⃣ 内部类 `Crm` 未声明为 `static`

这是**最核心原因**。

你写的是：
```java
@Data
public static class Crm {
    private String getUserIsSaleByEmailUrl;
}
```
它是一个**非静态内部类**。  
在 Spring Boot 的 `@ConfigurationProperties` 绑定机制中，  
非静态内部类 **需要依附于外部类的实例** 才能被创建，  
而在属性绑定阶段（Bean 初始化早期），Spring 并**不会自动创建外部类的实例**去初始化内部类字段。

➡️ 所以 `crm` 对象无法被实例化，自然是 `null`。

✅ **正确写法：**
```java
@Data
public static class Crm {
    private String getUserIsSaleByEmailUrl;
}
```
**必须加 `static`。**