在 Java 中，`@Valid` 和 `@Validated` 都是用来触发 **Bean Validation**（如 Hibernate Validator）进行参数校验的注解，但它们的设计背景和应用场景有所不同。

简单来说：**`@Valid` 是标准 JSR-303 规范**，而 **`@Validated` 是 Spring 提供的增强版**。

---

### 1. 核心区别对比

|**特性**|**@Valid (标准 JSR)**|**@Validated (Spring)**|
|---|---|---|
|**来源**|`javax.validation` 或 `jakarta.validation`|`org.springframework.validation.annotation`|
|**分组校验**|不支持|**支持** (核心优势)|
|**使用位置**|方法参数、方法、**成员属性**|类、方法参数、方法|
|**嵌套校验**|**支持** (必须加在成员属性上)|不支持（不能加在成员属性上）|

---

### 2. 使用场景详解

#### A. 基础校验 (Controller 层)

这是最常见的用法，两者在简单的 Controller 参数校验上效果一致。

Java

```
@PostMapping("/user")
public String createUser(@Validated @RequestBody User user) {
    // 触发 User 类中定义的注解校验，如 @NotNull, @Email 等
    return "success";
}
```

#### B. 分组校验 (仅 `@Validated` 支持)

场景：同一个 DTO，在“新增”时不需要 ID，在“修改”时必须有 ID。

你可以定义两个接口（标记类）：Create.class 和 Update.class。

Java

```
public class UserDTO {
    @NotNull(groups = Update.class) // 仅在更新组校验
    private Long id;

    @NotBlank(groups = {Create.class, Update.class})
    private String name;
}

@RestController
public class UserController {
    // 此时只会校验标记了 Create 组的字段
    @PostMapping("/add")
    public String add(@Validated(Create.class) @RequestBody UserDTO user) {
        return "added";
    }
}
```

#### C. 嵌套校验 (必须配合 `@Valid`)

场景：一个 User 对象包含一个 Address 对象，你也想校验 Address 内部的字段。

注意：@Validated 不能用在成员属性上，所以嵌套校验必须使用 @Valid。

Java

```
public class User {
    @NotBlank
    private String name;

    @NotNull
    @Valid // 必须使用 @Valid 才能触发嵌套校验
    private Address address;
}

public class Address {
    @NotBlank
    private String city;
}
```

#### D. 普通 Service 类校验

在非 Controller 类（如 Service）中，Spring 默认不自动执行校验。你需要在**类上**加上 `@Validated`，然后在参数前加校验注解。

Java

```
@Service
@Validated // 告诉 Spring 这个类需要进行方法级别校验
public class UserService {
    
    public void updateUser(@Valid User user) { 
        // 逻辑...
    }
    
    public void deleteUser(@Min(1) Long id) {
        // 逻辑...
    }
}
```

---

### 3. 常见坑点总结

1. **嵌套校验失效**：如果你在 User 类里的 Address 属性上写了 `@Validated`，校验是不会生效的，必须写 `@Valid`。
    
2. **分组校验失效**：如果你在 Controller 参数上用了 `@Valid`，那么你在 DTO 里写的 `(groups = ...)` 将被忽略。
    
3. 集合校验：如果参数是 List<User>，直接加 @Valid 可能失效。在 Spring 较高版本中，可以这样写：
    
    public String save(@RequestBody @Valid List<User> users)。
    
    如果失效，通常需要在类上加 @Validated，并在参数前加 @Valid。
    

---

### 总结建议

- **在 Controller 校验单个简单对象**：`@Valid` 或 `@Validated` 随你挑。
    
- **需要分组校验**：必须用 `@Validated`。
    
- **需要嵌套校验（校验对象里的对象）**：内部对象属性上必须加 `@Valid`。
    
- **在 Service 层或普通 Bean 里校验**：类上加 `@Validated`。
    

**你想了解如何统一处理校验失败后的错误提示（异常处理器）吗？**