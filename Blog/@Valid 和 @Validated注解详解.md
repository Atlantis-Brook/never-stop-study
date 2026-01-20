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

``` java
@PostMapping("/user")
public String createUser(@Validated @RequestBody User user) {
    // 触发 User 类中定义的注解校验，如 @NotNull, @Email 等
    return "success";
}
```

#### B. 分组校验 (仅 `@Validated` 支持)

场景：同一个 DTO，在“新增”时不需要 ID，在“修改”时必须有 ID。

你可以定义两个接口（标记类）：Create.class 和 Update.class。

``` java
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

``` java
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

``` java
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

### 3. 常见坑点总结



### 总结建议

- **在 Controller 校验单个简单对象**：`@Valid` 或 `@Validated` 随你挑。
    
- **需要分组校验**：必须用 `@Validated`。
    
- **需要嵌套校验（校验对象里的对象）**：内部对象属性上必须加 `@Valid`。
    
- **在 Service 层或普通 Bean 里校验**：类上加 `@Validated`。
    

在 Spring Boot 中，当 `@Valid` 或 `@Validated` 校验失败时，系统会抛出异常。如果不手动处理，前端会收到一个 500 错误或者一个格式非常乱的默认错误响应。

统一处理校验异常的核心在于使用 **`@RestControllerAdvice`** 配合 **`@ExceptionHandler`**。

---

### 1. 校验失败抛出的异常类型

在编写处理器之前，你需要知道校验失败会触发哪种异常：

1. **`MethodArgumentNotValidException`**：当你在 Controller 方法参数上使用 `@RequestBody` 且校验失败时抛出（最常见）。
    
2. **`ConstraintViolationException`**：当你在 Service 类上使用 `@Validated`，或者在 Controller 路径参数/查询参数上校验失败时抛出。
    
3. **`BindException`**：当表单提交（非 JSON）校验失败时抛出。
    

---

### 2. 核心代码实现：全局异常处理器

我们可以创建一个通用的结果类 `Result`，并编写全局处理器：

Java

```
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理 JSON 请求参数校验失败 (MethodArgumentNotValidException)
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<String> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        // 从异常对象中拿到所有的错误信息
        List<String> errors = e.getBindingResult().getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.toList());
        
        // 拼接成一个字符串或者返回 List 都可以
        String message = String.join(", ", errors);
        return Result.fail(400, "参数校验失败: " + message);
    }

    /**
     * 处理 单个参数/路径变量 校验失败 (ConstraintViolationException)
     */
    @ExceptionHandler(ConstraintViolationException.class)
    public Result<String> handleConstraintViolationException(ConstraintViolationException e) {
        String message = e.getConstraintViolations().stream()
                .map(ConstraintViolation::getMessage)
                .collect(Collectors.joining(", "));
        return Result.fail(400, "参数非法: " + message);
    }
}
```

---

### 3. 处理流程示意图

当请求进入系统，整个校验与异常处理的链路如下：

1. **Request** 进入 Controller 接口。
    
2. **Hibernate Validator** 根据注解（如 `@NotBlank`）执行校验。
    
3. 校验不通过，**抛出异常**（中断业务逻辑）。
    
4. **DispatcherServlet** 捕获异常并寻找匹配的 `@RestControllerAdvice`。
    
5. **ExceptionHandler** 格式化错误信息并返回 **JSON**。
    

---

### 4. 进阶技巧：自定义错误提示内容

你可以在 DTO 的注解中自定义 `message`，异常处理器会自动抓取它：

Java

```
public class UserDTO {
    @NotBlank(message = "用户名不能为空")
    private String username;

    @Min(value = 18, message = "年龄不能小于18岁")
    private Integer age;
}
```

此时，当前端发送一个 age 为 10 的请求时，全局异常处理器会捕获并返回： `{"code": 400, "msg": "参数校验失败: age: 年龄不能小于18岁", "data": null}`

---

### 5. 关于内存 OOM 的特别提醒

在你目前的 3G Pod 环境下，如果频繁发生大量参数校验失败，虽然不会直接导致 OOM，但要注意：

- **异常堆栈追踪（Stacktrace）**：抛出异常是非常消耗 CPU 和内存的操作（需要填充堆栈）。
    
- **优化建议**：在高并发场景下，如果参数校验失败率很高，建议在生产环境**关闭异常堆栈打印**，只记录简要的错误信息，以减轻 JVM 的压力。