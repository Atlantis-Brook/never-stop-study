## Sentinel介绍

&emsp;&emsp;Sentinel 是面向分布式、多语言异构化服务架构的流量治理组件，主要以流量为切入点，从流量路由、流量控制、流量整形、熔断降级、系统自适应过载保护、热点流量防护等多个维度来帮助开发者保障微服务的稳定性。

具体原理参考：[Sentinel官方文档](https://sentinelguard.io/zh-cn/docs/introduction.html)

## 限流算法

Sentinel采用滑动窗口的方式进行限流，滑动窗口协议是传输层进行流控的一种措施，接收方通过通告发送方自己的窗口大小，从而控制发送方的发送速度，从而达到防止发送方发送速度过快而导致自己被淹没的目的。

**参考如下网址提供的动态效果**[https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/selective-repeat-protocol/index.html](https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/selective-repeat-protocol/index.html)

微服务中的其他限流算法：令牌桶，漏桶，计数器。

## Sentinel使用

### 流量控制规则（FlowRule）

- 流量规则的定义

    `resource`：资源名，即限流规则的作用对象。

    `count`：限流的阈值。

    `grade`：QPS的统计类型，其中，0代表根据并发数量来限流，1代表根据QPS来进行流量控制。

    `strategy`：根据调用关系选择策略。

    `controlBehavior`：流控效果，直接拒绝，排队等待，慢启动模式，默认直接拒绝。

- 通过代码定义流量控制规则

    引入Sentinel依赖

    ```xml
    <dependency>
        <groupId>com.alibaba.csp</groupId>
        <artifactId>sentinel-core</artifactId>
    </dependency>
    ```
    
    通过调用`FlowRuleManager.loadRules()`方法通过硬编码方式定义流量控制规则
    
    ```java
    private static void initFlowQpsRule() {
        List<FlowRule> rules = new ArrayList<>();
        FlowRule rule1 = new FlowRule();
        rule1.setResource(resource);
        // Set max qps to 20
        rule1.setCount(20);
        rule1.setGrade(RuleConstant.FLOW_GRADE_QPS);
        rule1.setLimitApp("default");
        rules.add(rule1);
        FlowRuleManager.loadRules(rules);
    }
    ```

## 针对网关流量控制

> Sentinel支持对Spring Cloud Gateway、Zuul等主流API Gateway进行限流。

### 网关限流规则（GatewayFlowRule）

&emsp;&emsp;针对API Gateway的场景定制的限流规则，可以针对不同的route或自定义的API分组进行限流，支持针对请求中的参数、Header、来源IP等进行定制化的限流。

### 用户自定义API定义分组（ApiDefinition）

&emsp;&emsp;可以看做是一些URL匹配的组合。比如我们可以定义一个API叫做`my_api`，请求path模式为`/foo/**`和`/baz/**`的都归到`my_api`这个API分组下面。限流的时候可以针对这个自定义的API分组维度进行限流。

### Spring Cloud Gateway

使用时需引入一下模块

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
</dependency>
```

使用时只需注入对应的`SentinelGatewayFilter`实例以及`SentinelGatewayBlcokExceptionHandler`实例即可。

```java
@Configuration
public class GatewayConfiguration {

    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;

    public GatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                ServerCodecConfigurer serverCodecConfigurer) {
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
        // Register the block exception handler for Spring Cloud Gateway.
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }

    @Bean
    @Order(-1)
    public GlobalFilter sentinelGatewayFilter() {
        return new SentinelGatewayFilter();
    }
}
```

如我们在Spring Cloud Gateway中配置了以下路由：

```yml
env: dev
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: azure-distribution-dispatch
          uri: lb://azure-distribution-dispatch
          predicates:
            - Path=/api/openai/**
          filters:
            - StripPrefix=2
            
        - id: api-v1
          uri: lb://azure-distribution-dispatch
          predicates:
            - Path=/v1/**
          filters:
            - StripPrefix=1
        
        - id: azure-distribution-log
          uri: lb://azure-distribution-log
          predicates:
            - Path=/api/log/**
          filters:
            - StripPrefix=2
```

同时自定义一些API分组：

```java
/**
 * 自定义限流API资源
 */
private void initCustomizedApis() {
    Set<ApiDefinition> definitions = new HashSet<>();
    ApiDefinition api = new ApiDefinition("api-v1")
            // 设置路径匹配规则
            .setPredicateItems(new HashSet<ApiPredicateItem>() {{
                // 监控echat的访问接口
                add(new ApiPathPredicateItem().setPattern("/v1/**")
                        .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
                // 监控api调用的接口
                add(new ApiPathPredicateItem().setPattern("/api/openai/**")
                        // 设置匹配策略，使用前缀匹配
                        .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
            }});
    definitions.add(api);
    GatewayApiDefinitionManager.loadApiDefinitions(definitions);
    log.info("GatewayApiDefinitionManager Api definition {}", GatewayApiDefinitionManager.getApiDefinition("api-v1"));
}
```

那么这里面的 API name（如 `api-v1`）都会被标识为 Sentinel 的资源。比如访问网关的 URL 为 `http://localhost:8080/v1/foo/22` 的时候，对应的统计会加到 `api-v1` 这个资源上面。

通过调用`GatewayFlowRuleManager.loadRules()`方法通过硬编码方式定义网关流量控制规则：

```java
/**
 * 针对单网关节点，从redis中获取限流参数，对自定义资源限流
 * @param param
 */
public void initGatewayRules(GatewayFlowRuleParam param) {
    Set<GatewayFlowRule> rules = new HashSet<>();
    if(param != null) {
        if(param.getCountSec() != null)
            myStringRedisTemplate.opsForValue().set(GATEWAY_SENTINEL_MAX_REQUEST_COUNT_SECOND, String.valueOf(param.getCountSec()));

        if(param.getCountSec() != null)
            myStringRedisTemplate.opsForValue().set(GATEWAY_SENTINEL_MAX_REQUEST_COUNT_MINUTE, String.valueOf(param.getCountMin()));
    }

    String countSecStr = myStringRedisTemplate.opsForValue().get(GATEWAY_SENTINEL_MAX_REQUEST_COUNT_SECOND);
    String countMinStr = myStringRedisTemplate.opsForValue().get(GATEWAY_SENTINEL_MAX_REQUEST_COUNT_MINUTE);

    log.info("GATEWAY_SENTINEL_MAX_REQUEST_COUNT_SECOND: {},GATEWAY_SENTINEL_MAX_REQUEST_COUNT_MINUTE: {}  from Redis.", countSecStr, countMinStr);
    if(countSecStr != null)
        rules.add(new GatewayFlowRule("api-v1")
                .setResourceMode(SentinelGatewayConstants.RESOURCE_MODE_CUSTOM_API_NAME)  // 使用自定义API名称作为资源
                .setCount(Double.parseDouble(countSecStr))    // 每秒最多请求个数
                .setIntervalSec(1)  // 统计时间窗口为多少1s
                .setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_DEFAULT)  // 流控行为控制，支持快速失败和匀速排队模式，默认快速失败
                .setMaxQueueingTimeoutMs(600)  // 匀速排队模式下的最长排队时间，单位是ms，仅在匀速排队模式下生效
                .setBurst(2)    // 突发流量控制，允许的突发请求数
                .setParamItem(new GatewayParamFlowItem()        // 针对参数限流配置
                        .setParseStrategy(SentinelGatewayConstants.PARAM_PARSE_STRATEGY_CLIENT_IP)  //从请求中提取来源IP的策略
                        .setFieldName("X-Forwarded-For") // 若提取策略是header或URL则需指定参数模式，使用X-Forwarded-For头部获取客户端IP
                        .setMatchStrategy(SentinelGatewayConstants.PARAM_MATCH_STRATEGY_EXACT)  // 参数值匹配策略，精确匹配模式
//                        .setPattern("192.168.1.100")  // 针对特定IP地址进行限流。
                )
        );
    if(countMinStr != null)
        rules.add(new GatewayFlowRule("api-v1")
                .setResourceMode(SentinelGatewayConstants.RESOURCE_MODE_CUSTOM_API_NAME)  // 使用自定义API名称作为资源
                .setCount(Double.parseDouble(countMinStr))    // 每分钟最多请求个数
                .setIntervalSec(60)  // 统计时间窗口为60s
                .setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_DEFAULT)  // 流控行为控制，支持快速失败和匀速排队模式，默认快速失败
                .setMaxQueueingTimeoutMs(600)  // 匀速排队模式下的最长排队时间，单位是ms，仅在匀速排队模式下生效
                .setBurst(2)    // 突发流量控制，允许的突发请求数
                .setParamItem(new GatewayParamFlowItem()        // 针对参数限流配置
                        .setParseStrategy(SentinelGatewayConstants.PARAM_PARSE_STRATEGY_CLIENT_IP)  //从请求中提取来源IP的策略
                        .setFieldName("X-Forwarded-For") // 若提取策略是header或URL则需指定参数模式，使用X-Forwarded-For头部获取客户端IP
                        .setMatchStrategy(SentinelGatewayConstants.PARAM_MATCH_STRATEGY_EXACT)  // 参数值匹配策略，精确匹配模式
//                        .setPattern("192.168.1.100")  // 针对特定IP地址进行限流。
                )
        );
    GatewayRuleManager.loadRules(rules);
    log.info("GatewayRuleManage All Rules: {}", GatewayRuleManager.getRules());
}
```

在`GatewayCallbackManager`注册回调进行定制：

- `setBlockHandler`：%% 注册函数用于实现自定义的逻辑处理被限流的请求 %%，对应接口为`BlockRequestHandler`。默认实现为 `DefaultBlockRequestHandler`，当被限流时会返回类似于下面的错误信息：`Blocked by Sentinel: FlowException`。

```java
@PostConstruct
public void initBlockHandlers() throws Exception {
    log.info("PostConstruct init gateway rules and customized api");
 	// 针对单网关节点规则配置
    initCustomizedApis();
    initGatewayRules(null);

    // 自定义限流响应处理器
    BlockRequestHandler blockRequestHandler = (exchange, e) -> {

        // 加log
        String ipAddress = exchange.getRequest().getHeaders().getFirst("X-Forwarded-For");
        log.info("Sentinel-Gateway too many request from IP:{}, notes[ServerWebExchange: {}]", ipAddress, exchange.getRequest().getHeaders(), e);
        HashMap<String, Object> map = new HashMap<>();
        map.put("code", "429");
        map.put("message", "请求过快，请稍后重试！");
        map.put("data", null);

        return ServerResponse.status(HttpStatus.TOO_MANY_REQUESTS)
                .contentType(MediaType.APPLICATION_JSON)
                .body(BodyInserters.fromValue(map));
    };
    // 注册自定义限流响应处理器
    GatewayCallbackManager.setBlockHandler(blockRequestHandler);
}
```

Demo实例：[sentinel-demo-spring-cloud-gateway](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo/sentinel-demo-spring-cloud-gateway)

## 动态规则扩展

手动修改规则通过API`FlowRuleManager.loadRules(List<FlowRule> rules); GatewayFlowRuleManager.loadRules(Set<GatewayFlowRule> rules);`一般用于测试和演示，生产环境上一般通过动态规则源的方式来动态管理规则。

**客户端实现** `ReadableDataSource` **接口端监听规则中心实时获取变更**

`DataSource` 扩展常见的实现方式有:

- **拉模式**：客户端主动向某个规则管理中心定期轮询拉取规则，这个规则中心可以是 RDBMS、文件，甚至是 VCS 等。这样做的方式是简单，缺点是无法及时获取变更；
- **推模式**：规则中心统一推送，客户端通过注册监听器的方式时刻监听变化，比如使用 Nacos、Zookeeper 等配置中心。这种方式有更好的实时性和一致性保证。

Sentinel 目前支持以下数据源扩展：

- Pull-based: 动态文件数据源、[Consul](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-datasource-consul), [Eureka](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-datasource-eureka)
- Push-based: [ZooKeeper](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-datasource-zookeeper), [Redis](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-datasource-redis), [Nacos](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-datasource-nacos), [Apollo](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-datasource-apollo), [etcd](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-datasource-etcd)

**注册数据源，根据nacos配置中心配置文件，动态管理规则**，只需要调用以下方法将数据源注册到指定的规则管理器中：

  ```java 
  ReadableDataSource<String, List<FlowRule>> flowRuleDataSource = new NacosDataSource<>(remoteAddress, groupId, dataId, parser);
  FlowRuleManager.register2Property(flowRuleDataSource.getProperty());
  ```

#### 普通限流

引入依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <version>1.8.8</version>
</dependency>
```

注册动态数据源

```java
private static void loadMyNamespaceRules() throws Exception {
    Properties properties = new Properties();
    properties.put(PropertyKeyConst.SERVER_ADDR, remoteAddress);
    properties.put(PropertyKeyConst.NAMESPACE, NACOS_NAMESPACE_ID);
    ReadableDataSource<String, List<FlowRule>> flowRuleDataSource = new NacosDataSource<>(properties, groupId, dataId,
            source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>() {
            }));
    log.info("路由规则: {}", JSON.toJSONString(flowRuleDataSource.loadConfig()));
    FlowRuleManager.register2Property(flowRuleDataSource.getProperty());
    log.info("成功注册路由规则: {}", JSON.toJSONString(FlowRuleManager.getRules()));
}
```

初始化

```java
@PostConstruct
public void initBlockHandlers() throws Exception {
    log.info("PostConstruct init gateway rules and customized api");
    // 针对单网关节点规则配置，方案弃用，采用动态规则扩展方式。
    initCustomizedApis();
    // 动态规则扩展方式。
    loadMyNamespaceRules();
    log.info("动态规则扩展配置成功！");

}
```

Demo实例：[sentinel-demo-nacos-datasource](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo/sentinel-demo-nacos-datasource)

#### Gateway 限流

**注意**：

- Sentinel 网关流控默认的粒度是 route 维度以及自定义 API 分组维度，默认**不支持 URL 粒度**。若通过 Spring Cloud Alibaba 接入，请将 `spring.cloud.sentinel.filter.enabled` 配置项置为 false（若在网关流控控制台上看到了 URL 资源，就是此配置项没有置为 false）。
- 若使用 Spring Cloud Alibaba Sentinel 数据源模块，需要注意网关流控规则数据源类型是 `gw-flow`，若将网关流控规则数据源**指定为 flow 则不生效**。

根据注册规则监听器，动态监听限流规则配置。

```java
private static final String remoteAddress = "dev-nacosv1.edianzu.cn:80";
    // nacos group
    private static final String groupId = "DEFAULT_GROUP";
    // nacos dataId
    private static final String dataId = "sentinel-dynamic-rules.json";
    // nacos namespace id
    private static final String NACOS_NAMESPACE_ID = "e4bc392d-14cc-4874-92e7-45faba81942a";

    private static void loadMyNamespaceRules() throws Exception {
        Properties properties = new Properties();
        properties.put(PropertyKeyConst.SERVER_ADDR, remoteAddress);
        properties.put(PropertyKeyConst.NAMESPACE, NACOS_NAMESPACE_ID);
        ReadableDataSource<String, Set<GatewayFlowRule>> gatewayflowRuleDataSource = new NacosDataSource<>(properties, groupId, dataId,
                source -> JSON.parseObject(source, new TypeReference<Set<GatewayFlowRule>>() {
                }));
        log.info("路由规则: {}", JSON.toJSONString(gatewayflowRuleDataSource.loadConfig()));
        GatewayRuleManager.register2Property(gatewayflowRuleDataSource.getProperty());
        log.info("成功注册路由规则: {}", JSON.toJSONString(GatewayRuleManager.getRules()));
    }
```
在Nacos配置中心配置sentinel 网关限流规则`sentinel-dynamic-rules.json`
```json
[
  {
    "resource": "api-v1",     // 资源名称
    "resourceMode": 1,        // 使用自定义API名称作为资源
    "grade": 1,
    "count": 20,              // 最多请求阈值
    "intervalSec": 60,        // 统计时间窗口
    "controlBehavior": 0,     // 流控行为，支持快速失败和匀速排队模式
    "burst": 2,               // 突发流量控制
    "maxQueueingTimeoutMs": 600,  // 排队模式下最长的排队时间
    "paramItem": {            // 针对参数限流配置
      "index": 0,
      "parseStrategy": 0,     // 从请求中提取来源IP
      "fieldName": "X-Forwarded-For",     // 如果提取策略是header或URL则需指定参数模式
      "pattern": null,        // 针对特定IP地址进行限流
      "matchStrategy": 0      // 匹配策略，精确匹配
    }
  },
  {
    "resource": "api-v1",
    "resourceMode": 1,
    "grade": 1,
    "count": 2,
    "intervalSec": 1,
    "controlBehavior": 0,
    "burst": 2,
    "maxQueueingTimeoutMs": 600,
    "paramItem": {
      "index": 1,
      "parseStrategy": 0,
      "fieldName": "X-Forwarded-For",
      "pattern": null,
      "matchStrategy": 0
    }
  }
]
```

初始化

```java
@PostConstruct
public void initBlockHandlers() throws Exception {
    log.info("PostConstruct init gateway rules and customized api");
    // 针对单网关节点规则配置，方案弃用，采用动态规则扩展方式。
    initCustomizedApis();
    // 动态规则扩展方式。
    loadMyNamespaceRules();
    log.info("动态规则扩展配置成功！");
}
```
