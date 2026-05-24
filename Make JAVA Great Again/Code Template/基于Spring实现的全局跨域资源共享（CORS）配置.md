#cors

解决全局跨域问题
> 允许前端（例如 Vue、React 等独立部署的项目）跨域访问后端接口，并解决因跨域带来的安全限制（如 Cookie 丢失问题）。
```java 
import org.springframework.context.annotation.Configuration;  
import org.springframework.web.servlet.config.annotation.CorsRegistry;  
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;  
  
/**  
 * 全局跨域配置  
 */  
@Configuration  
public class CorsConfig implements WebMvcConfigurer {  
  
    @Override  
    public void addCorsMappings(CorsRegistry registry) {  
        // 覆盖所有请求  
        registry.addMapping("/**")  
                // 允许发送 Cookie                .allowCredentials(true)  
                // 放行哪些域名（必须用 patterns，否则 * 会和 allowCredentials 冲突）  
                .allowedOriginPatterns("*")  
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")  
                .allowedHeaders("*")  
                .exposedHeaders("*");  
    }  
}
```

**这段配置的作用可以概括为：全开、全放。**
- **优点：** 极其方便。在前后端分离开发（特别是本地开发测试阶段），一配置上它，前端就绝对不会再报任何 `CORS error`（跨域错误），开发体验极顺畅。
- **缺点（生产环境隐患）：** **安全性较低**。由于它接收来自任何域名的请求（`*`），且允许带上 Cookie，这就给 **CSRF（跨站请求伪造）** 攻击留下了隐患。
- **优化建议：** 建议在项目上线（生产环境）时，将 `.allowedOriginPatterns("*")` 修改为**具体的、受信任的前端域名**，例如：`{java icon}  .allowedOriginPatterns("https://admin.yourcompany.com", "https://www.yourcompany.com")`