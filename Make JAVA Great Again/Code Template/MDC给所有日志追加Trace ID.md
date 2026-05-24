在前后端分离的项目中，使用 **MDC（Mapped Diagnostic Context）** 给所有日志追加 `traceId` 是非常优雅且标准的做法。

由于 Spring Boot 默认使用 **Logback** 作为日志框架，最直接、最无侵入性的实现方式是：**使用 `HandlerInterceptor`（拦截器）在请求进来时生成 traceId 存入 MDC，并在请求结束时销毁。**

下面是完整的、拿来即用的可执行代码片段。

### Step 1: 编写拦截器（捕捉并注入 traceId）

新建一个拦截器类。我们通常优先尝试从前端请求头获取 `trace-id`（方便微服务全链路追踪），如果没有，后端就自动用 `UUID` 生成一个。
``` java
import org.slf4j.MDC;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.UUID;

@Component
public class LogInterceptor implements HandlerInterceptor {

    // 定义 MDC 中存储的 key，需要和日志配置文件（logback.xml）中的保持一致
    private static final String TRACE_ID = "traceId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 1. 尝试从前端传入的请求头中获取 traceId（兼容微服务/网关透传）
        String traceId = request.getHeader(TRACE_ID);
        
        // 2. 如果前端没传，后端自己生成一个干净的 UUID（去掉横杠）
        if (!StringUtils.hasText(traceId)) {
            traceId = UUID.randomUUID().toString().replace("-", "");
        }
        
        // 3. 将 traceId 塞入当前线程的 MDC 上下文中
        MDC.put(TRACE_ID, traceId);
        
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 4. 【非常重要】请求结束时，必须清除当前线程的 MDC
        // 否则在 Tomcat 线程池复用线程时，会导致日志 traceId 错乱、甚至内存泄漏
        MDC.remove(TRACE_ID);
    }
}
```

### Step 2: 注册拦截器

将刚刚写好的拦截器注册到 Spring MVC 的配置流中（正好可以放在你之前的 `WebMvcConfigurer` 里面）。
```java
package com.example.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private LogInterceptor logInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 拦截所有请求，让所有接口的日志都能带上 traceId
        registry.addInterceptor(logInterceptor).addPathPatterns("/**");
    }
}
```

### Step 3: 修改日志配置文件（关键步骤）

光把 `traceId` 存进 MDC 还不够，你必须告诉 Logback 怎么把它打印出来。

在 `src/main/resources` 下找到（或新建）**`logback-spring.xml`**，在 `pattern`（日志格式）中加入 **`[%X{traceId}]`**：
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- 注意看这里的 %X{traceId}，它会自动去 MDC 里面抓取对应的变量 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - [%X{traceId}] - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>

</configuration>
```

### 💡 避坑与进阶提示

1. **异步线程失效问题：** MDC 的底层是基于 `ThreadLocal`（线程本地变量）实现的。这意味着如果你在 Controller 里面用了 **`@Async` 异步注解**、或者自己开了一个 `new Thread()` 或线程池，**子线程是拿不到主线程的 `traceId` 的**。
    
2. **异步解决方案：** 如果项目中大量使用了线程池，需要重写线程池的 `TaskDecorator`（任务装饰器），在提交任务给线程池时，手动复制一份主线程的 MDC 上下文给子线程。
    
3. **前端顺手牵羊：** 你可以在拦截器的 `preHandle` 里通过 `response.setHeader("traceId", traceId);` 把追踪码顺便返回给前端。这样前端报错时，用户直接截图前端的 `traceId`，你复制去日志系统（如 ELK）一搜，整条请求链路的日志一目了然！