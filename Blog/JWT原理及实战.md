## 1. 什么是JWT

JSON Web Token (JWT) is an open standard ([RFC 7519](https://tools.ietf.org/html/rfc7519)) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the **HMAC** algorithm) or a public/private key pair using **RSA** or **ECDSA**.

1. **翻译**

- 官网地址：[https://jwt.io/introduction](https://jwt.io/introduction)
- 翻译：json web token(JWT) 是一个开放标准（RFC 7519），它定义了一种紧凑的、自包含的方式，用于在各方之间以JSON对象安全地传输信息。此信息可以验证和信任，因为它是数字签名的。JWT可以使用秘密（使用HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名

1. **通俗解释**
JWT简称JSON Web Token，也就是通过JSON形式作为Web应用中的令牌，用于在各方之间安全地将信息作为JSON对象传输。在数据传输过程中还可以完成数据加密、签名等相关处理。

## 2. JWT能做什么

1. **授权**
   这是使用JWT的最常见方案。一旦用户登录，每个后续请求将包括JWT，从而允许用户访问该令牌允许的路由，服务和资源。单点登录是当今广泛使用JWT的一项功能，因为它的开销很小并且可以在不同的域中轻松使用。
2. **信息交换**
   JSON Web Token是在各方之间安全地传输信息的好方法。因为可以对JWT进行签名（例如，使用公钥/私钥对），所以您可以确保发件人是他们所说的人。此外，由于签名是使用标头和有效负载计算的，因此可以验证内容是否遭到篡改。

## 3. 为什么是JWT

### 基于传统的Session认证

1. **认证方式**
   我们知道，http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登陆的信息，这份登录的信息会在响应时传递给浏览器，告诉其保存为Cookie，以便下次请求时发送给我们的应用，这样我们的应用就能识别到请求来自哪个用户了，这就是传统的基于session的认证。
2. **认证流程**
   客户端 ----------------认证----------------->	Web Application

​				<---------------cookie---------------

​		请求应用携带cookie，找到对应的session
3. **暴露问题**
   - 每个用户经过我们的应用认证之后，我们的应用都要在服务器端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大
   - 用户认证后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上，这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。
   - 因为是基于cookie来进行用户识别的，cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。
   - 在前后端分离系统中就更加痛苦，前后端分离在应用解耦后增加了部署的复杂性。通常用户一次请求就要多次转发。如果用session每次携带sessionid到服务器，服务器还要查询用户信息。同时如果用户很多。这些信息存储在服务器内存中，给服务器增加负担。还有就是CSRF（跨站伪造请求攻击）攻击，session是基于cookie进行用户识别的，cookie如果被截获，用户就会很容易受到跨站请求伪造攻击。还有就是sessionid就是一个特征值，表达的信息不够丰富。不容易扩展。而且如果你后端应用是多节点部署。那么就需要实现session共享机制。不方便集群应用。

### 基于JWT认证

1. **认证流程**
   - 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输（https协议），从而避免敏感信息被嗅探。
   - 后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload（负载），将其与头部分别进行Base64编码拼接后签名，形成一个JWT(Token)。形成的JWT就是一个形同xxx.yyy.zzz的字符串。token head. payload. singurater
   - 后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果**保存在localStorage或sessionStorage上**，退出登录时前端删除保存的JWT即可。
   - 前端在每次请求时将JWT**放入HTTP Header中的Authorization位**。（解决XSS和XSRF问题）
   - 后端检查是否存在，如果存在验证JWT的有效性。例如，检查签名是否正确；检查Token是否过期；检查Token的接收方是否是自己。
   - 验证通过后后端使用JWT中包含的用户信息进行其他逻辑操作，返回相应结果。
2. **JWT优势**
   - 简洁（Compact）:可以通过URL, POST参数或者在HTTP header发送，因为数据量小，传输速度也很快
   - 自包含（Self-contained）:负载中包含了所有用户所需要的信息，避免了多次查询数据库
   - 因为Token是以JSON加密的形式保存在客户端的，所以JWT是跨语言的，原则上任何web形式都支持
   - 不需要在服务端保存会话信息，特别使用于分布式微服务

## 4. JWT的结构是什么

1. **令牌组成**
   - 标头（Header）
   - 有效载荷（Payload）
   - 签名（Signature）
     因此，JWT通常如下所示：xxx.yyy.zzz	Header.Payload.Signature

2. **Header**
   - 标头通常由两个部分组成：令牌的类型（即JWT）和所使用的签名算法，例如HMAC、SHA256或RSA。它会使用Base64 编码组成JWT结构的第一部分。
   - 注意：Base64是一种编码，也就是说，它是可以被翻译回原来的样子的。它并不是一种加密过程。

   ```json
   {
       "alg": "HS256",
       "type": "JWT"
   }
   ```

3.  **Payload**

   令牌的第二部分是有效负载，其中包含声明。声明是有关实体（通常是用户）和其他数据的声明。同样的，他会使用Base64 编码组成JWT结构的第二部分

   ```json
   {
       "sub": "1234567890",
       "name": "John Doe",
       "admin": "true",
   }
   ```

4. **Signature**

   &emsp;&emsp;前面两部分都是使用Base64进行编码的，即前端可以解开知道里面的信息。Signature需要使用编码后的header和payload以及我们提供的一个密钥，然后使用header中指定的签名算法（HS256）进行签名。签名的作用是保证JWT没有被篡改过。

   如，`HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret);`

   - **签名的目的**

     最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被篡改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时使用的密钥的话，得出来的签名也是不一样的。

   - **信息安全的问题**

     在这里有一个疑问：Base64是一种编码，是可逆的，那么我的信息不就被暴露了嘛？

     是的，所以在JWT中，不应该在负载里面加入任何敏感的数据。在上面的例子中，我们传输的是用户的User ID。这个值实际上不是什么敏感内容，一般情况下被知道也是安全的。但是像密码这样的内容就不能被放在JWT中了。如果将用户的密码放在了JWT中，那么怀有恶意的第三方通过Base64解码就能很快知道你的密码了。因此JWT适合用于向Web应用传递一些非敏感信息。JWT还经常用于设计用户认证和授权系统，甚至实现Web应用的单点登录。

5. **放在一起**

   - 输出是三个由点分隔的Base64-URL字符串，可以在HTML和Http环境中轻松传递这些字符串，与基于XML的标准例如SAML相比，它更紧凑。

   - 简介（Compact）
     可以通过URL，POST参数或者在**HTTP header**发送，因为数据量小，传输速度快
   - 自包含（Self-Contained）
   - 负载中包含了所有用户所需要的信息，避免了多次查询数据库

    ```
    eyJhbGciOiJIUzUxMiJ9.
    eyJzdWIiOiIyMDEyIiwiaWF0IjoxNzI0MTUwNjQzLCJleHAiOjE3MjY3NDI2NDN9.
    ouVI3IUe2kdcQJfk8R4LUpwgpMARRs-9_qqbka2vxmSfmWYS0qyr_zLwoL1f-LEN_-9ZFN2Yd0VUhHfn3389pQ
    ```
   
## 5. 使用JWT

1. **引入依赖**

   ```xml
   <dependency>
       <groupId>io.jsonwebtoken</groupId>
       <artifactId>jjwt</artifactId>
       <version>0.9.1</version>
   </dependency>
   ```

2. 生成token

   ```java
   /**
    * 使用默认过期时间（7天），生成一个JWT
    *
    * @param username 用户名
    * @param claims   JWT中的数据
    * @return
    */
   public String createToken(String username, Map<String, Object> claims) {
       return createToken(username, claims, defaultExpire);
   }
   
   /**
    * 生成token
    *
    * @param username 用户名
    * @param claims   请求体数据
    * @param expire   过期时间 单位：毫秒
    * @return token
    */
   public String createToken(String username, Map<String, Object> claims, Long expire) {
       JwtBuilder builder = Jwts.builder();
       Date now = new Date();
       // 生成token
       builder.id("rQRk$yN:7%*Bw}A_A-]M~4#;yGa:a_F{") //id 这个可以不填，但是建议填
               .issuer("Galaxy") //签发者
               .claims(claims) //数据 
               .subject(username) //主题
               .issuedAt(now) //签发时间
               .expiration(new Date(now.getTime() + expire)) //过期时间
               .signWith(key); //签名方式
       builder.header().add("JWT", "JSpWdhuPGblNZApVclmX");
       return builder.compact();
   }
   ```

3. **根据令牌和签名解析数据**

   ```JAVA
   /**
    * 解析token
    *
    * @param token jwt token
    * @return Claims
    */
   public Claims claims(String token) {
       try {
           return Jwts.parser()
                   .verifyWith(key)
                   .build()
                   .parseSignedClaims(token)
                   .getPayload();
       } catch (Exception e) {
           if (e instanceof ExpiredJwtException) {
               //现在不需要使用 claims.getExpiration().before(new Date());
               // 判断JWT是否过期了 如果过期会抛出ExpiredJwtException异常
               throw new RunException("token已过期");
           }
           if (e instanceof JwtException) {
               throw new RunException("token已失效");
           }
           logger.error("jwt解析失败" + e);
           throw new RunException("token解析失败");
       }
   }
   ```

4. **常见异常信息**

   - `SignatureVerificationException`		签名不一致异常
   - `TokenExpiredException`     			          令牌过期异常
   - `AlgorithmMismatchException`                算法不匹配异常
   - `InvalidClaimException`                           失效的payload异常

## 6. 封装工具类

```java 
package com.edianyun.echat.server.service.biz;

/**
 * jwt工具类
 */
@Slf4j
@Component
public class JwtUtil {

    private static final Logger logger = LoggerFactory.getLogger(JwtUtil.class);
    @Value("${jwt.secret}")
    private String jwtSecret;
    private final long defaultExpire = 1000 * 60 * 60 * 24 * 7L;//7天
   
    private JwtUtil() {
    }

    /**
     * 使用默认过期时间（7天），生成一个JWT
     *
     * @param username 用户名
     * @param claims   JWT中的数据
     * @return
     */
    public String createToken(String username, Map<String, Object> claims) {
        return createToken(username, claims, defaultExpire);
    }

    /**
     * 生成token 
     *
     * @param username 用户名
     * @param claims   请求体数据
     * @param expire   过期时间 单位：毫秒
     * @return token
     */
    public static  String createToken(String username, Map<String, Object> claims, Long expire) {
        JwtBuilder builder = Jwts.builder();
        Date now = new Date();
        // 生成token
        builder.id("rQRk$yN:7%*Bw}A_A-]M~4#;yGa:a_F{") //id 这个可以不填，但是建议填
                .setIssuer("Galaxy") //签发者
                .setClaims(claims) //数据
                .setSubject(username) //主题
                .setIssuedAt(now) //签发时间
                .setExpiration(new Date(now.getTime() + expire)) //过期时间
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
        builder.header().add("JWT", "JSpWdhuPGblNZApVclmX");
        return builder.compact();
    }

    /**
     * 解析token
     *
     * @param token jwt token
     * @return Claims
     */
    public static Claims parseJWT(String token) {
        try {
            return Jwts.parser()
                	.setSigningKey(jwtSecret)	
                    .parseClaimsJws(token)
                    .getBody();
        } catch (Exception e) {
            if (e instanceof ExpiredJwtException) { 
                //现在不需要使用 claims.getExpiration().before(new Date());
                // 判断JWT是否过期了 如果过期会抛出ExpiredJwtException异常
                throw new RunException("token已过期");
            }
            if (e instanceof JwtException) {
                throw new RunException("token已失效");
            }
            logger.error("jwt解析失败" + e);
            throw new RunException("token解析失败");
        }
    }
}
```

## 7. 整合Spring boot

### 创建拦截器

```java
@Component
public class JWTInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpSevletResponse response, Object handler) throws Exception {
        Map<String, Object> map = new HashMap<>;
    	String token = request.getHeader("Authorization");
        try{
            JWTUtils.parseJWT(token); // 验证令牌
            return true;	// 用户已登录，继续处理请求
        } catch (SignatureVerficationException e) {
            e.printStackTrace();
            map.put("msg", "无效签名！");
        } catch (TokenExpiredException e) {
            e.printStackTrace();
            map.put("msg", "token过期！");
        } catch (AlgorithmMismatchException e) {
            e.printStackTrace();
            map.put("msg", "token算法不一致！");
        } catch (Exception e) {
            e.printStackTrace();
            map.put("msg", "token无效！");
        }
        map.put("state", false);
        
       	// 用户未登录，返回错误响应或重定向到登录页面 
         sendErrorResponse(response);
    	
        return false;
    }
    
    // 将map 转为json jackson 把拦截信息写入响应中返回前端
    private void sendErrorResponse(HttpServletResponse response) throws IOException {
        response.setContentType("application/json;charset=UTF-8");
        // 设置响应状态码为401 Unauthorized。
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.getWriter().write(new ObjectMapper().writeValueAsString(map));
    }	
   
}
```

### 注册拦截器

```java
/**
* 拦截器注册
* 实现WebMvcConfigurer接口，推荐使用，提供灵活的配置方式，保留Spring Boot的自动配置功能
* 继承WebMvcConfigurationSupport类，适用于需要完全接管Spring MVC配置的场景，但会禁用Spring Boot的自动配置
*/
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
 	
    @Autowired
    private JWTInterceptor jwtInterceptor;
 	   
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtInterceptor);
        		.addPathPatterns("/**")
                .excludePathPatterns("/user/**");
    }
}
```
