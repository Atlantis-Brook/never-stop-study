#es #elasticsearch
# 1、 单机服务的问题？
- 性能有限
- 可用性差
- 难以扩展
# 2、分布式的好处
-  高可用：集群可以容忍部分节点宕机而保持服务的可用性和数据的完整性
-  易扩展：当集群性能不满足业务需要时，可以方便快速的扩容集群，而无需停止服务
-  高性能：集群通过负载均衡器分摊并发请求压力，可以大大提高集群的吞吐能力和并发能力
# 3、集群环境的选择
## 自动发现
集群中节点之间通过9300端口通信，并发现彼此的存在。ES不需要修改任何的配置，节点之间即可发现彼此的存在从而形成集群。
## 核心配置
- `cluster.name`：集群名称，节点根据集群名称确定是否是同一个集群。
- `node.name`：节点名称，集群内唯一。
- `node.roles`：节点的角色配置，不同的角色以字符串数组方式配置。
- `network.host`：节点对外提供服务的地址以及集群内通信的IP地址，如果配置方式为本地地址，那么当前节点仅会在本地发现其他节点。另外修改此配置会触发生产模式并执行引导检查。但是生产环境一般是每个服务器一个节点，每个节点都要和其他服务器形成发现，所以一般配置为本机的内网IP。
- `network.port`：对外提供服务的端口号，默认9200-9299
- `transport.port`：节点之间通讯端口号，默认9300-9399
- `discovery.seed_host`：集群初始化的种子节点，可配置部分或全部节点，大型集群可通过嗅探器发现剩余节点。
- `cluster.initial_master_nodes`：节点初始`active master`节点，必须是有master角色的节点，即必须是候选节点，但并不是必须配置所有候选节点。生产模式下启动新集群时，必须明确列出应在第一次选举中计算其选票的候选节点。第一次成功形成集群后，`cluster.initial_master_nodes`从每个节点的配置中删除设置。重启启动集群或向现有集群添加新节点时，请勿使用此配置。
## 开发模式和生产模式
- **开发模式：** 开发模式是默认配置（未配置集群发现是设置），如果用户只是处于需学习的目的，而引导检查会把很多用户挡在门外，所以ES提供了设置项`{properties icon} discovery.type: single-node`。此配置为指定节点为单节点发现以绕过引导检查。
- **生产模式：** 当用户修改了有关集群的相关配置会触发生产模式，在生产模式下，服务启动会触发ES的引导检查/启动检查（bootstrap check），所谓引导检查是在服务启动之前对一些重要的配置项进行检查，检查其配置值是否是合理的。引导检查包括对JVM大小、内存锁、虚拟内存、最大线程数、集群发现相关配置等相关的检查，如果某一项或几项配置的不合理，ES会拒绝启动服务，并且在开发模式下的某些告警信息会升级成错误信息输出。引导检查十分严格，为了防止用户在对ES的基本使用不了解的前提下启动服务而导致的后期性能问题无法解决。一旦服务以某种不合理的配置启动，随着使用时间的增加很大概率会产生较大的性能问题，但此时集群已变得难以维护，ES为避免这种情况而做出了引导检查的设置。
## 单节点模式
单节点启动，节点会选举自己成为active master节点，会绕过引导检查 
```yml title:elasticsearch.yaml
discovery.type: single-node
```
## 引导检查——Bootstrap Checks
在启动生产模式时，节点启动之前ES会自动对节点的相关配置逐项检查，目的是避免开发者在对齐配置项不了解的前提下做出不合理的配置。如果配置不符合性能或者兼容性要求，ES会阻止服务启动以保证服务的性能和可用性。
检查项：
- **堆大小检查**
- **文件描述符检查**
- **内存锁检查**
- **最大线程数检查**
- **最大文件大小检查**
- 虚拟内存检查
- 文件系统映射数检查
- 客户端JVM检查
- 串行收集器检查
- 系统调用过滤器检查
- OnError和OnOOMError检查
- 早期访问检查
- 所有权限检查
- **发现配置检查**
# 4、主从模式
Elasticsearch为什么使用主从模式？Elasticsearch使用的主从架构模式，除此之外，还可以使用分布式哈希表（DHT），其区别在于：
- 主从模式适合节点数量不多，并且节点的状态改变（加入集群或者离开集群）不频繁的情况。
- 分布式哈希表支持每小时数千个节点的加入或离开，响应约为4-10跳
ES的应用场景一般来说单个集群中不会有太多的节点（一般来说不会超过1k个），节点的数量远远小于单个节点（主节点）所能维护的连接数。并且通常主节点不必经常处理节点的加入和离开，处于相对稳定的对等网络中，因此使用主从模式。
# 5、节点
## 获选节点/投票节点（master- eligible，有时候也叫master节点）
默认情况下

# 6、ES常见模块：Modules
## Cluster
cluster模块是Master节点执行集群管理的封装实现，管理集群状态，维护集群级（除了集群级，还有索引级分片级等级别）的配置信息。其主要功能包括：
- 管理集群状态，将新生成的集群状态发布到集群的所有节点
- 调用allocation模块执行分片分配感知，决策分片分配行为
- 在集群各个节点直接迁移分片，保证数据平衡，shard rebalance
## Allocation
此模块是实现了对节点分片的分配感知策略，新节点加入和离开、动态扩容都需要分片分配感知，此模块由主节点调用，常见的使用场景如：跨机架强制感知实现高可用，冷热集群架构设计等。
## Bootstrap
引导检查模块
## Ingest
预处理模块负责数据索引之前的一些预处理操作，比如数据类型处理、数据结构的转换等，很多场景下可代替logstash处理管道消息
## Monitor
监控功能提供了一种方式来了解Elasticsearch集群的运行状况和性能
## Discovery
发现模块负责管理如发现集群中新加入的节点，或者节点退出之后将状态信息移除，起作用类似于ZooKeeper。发现模块是用于elasticsearch的内置发现模块默认值。它提供单播发现，但可以扩展到支持云环境和其他形式的发现
## Gateway
负责对收到Master广播下来的集群状态数据的持久化存储，并在集群完全重启时恢复他们
## Indices
索引模块管理全局级索引配置，不包括索引及索引以下级。集群启动阶段需要主副分片恢复就是在这个模块完成的
## HTTP
Http模块允许通过JSON over HTTP的方式访问ES的API，HTTP模块本质哈桑是完全异步的，这一位置没有阻塞线程等待响应。使用异步通信进行HTTP的好处是解决了C10k的问题。
## Transport
传输模块用于集群内部节点的通信，传输模块使用TCP协议，每个节点都与其他节点维持若干个TCP长连接，通信的本质也是完全异步的。

# 7、分片：Shard
Shard即数据分片，是ES的数据载体。在ES中数据分为primary shard（主分片）和replica shard（副本分片），每一个primary shard承载单个索引的一部分数据，分布于各个节点，replica为某个primary的副本，即备份。分片分配的原则是尽量均匀的分配在集群中的各个节点，以最大程度降低部分shard在出现意外时对整个集群乃至服务造成影响。
每个分片就是一个Lucene的实例，具有完整的功能。
## 7.1 分片的创建策略
分片产生的目的是为了实现分布式，而分布式的好处之一就是实现“高可用性”（还包括高性能如提高吞吐量等），分片的分配策略极大程度上都是围绕如何提高可用性而来的，如**分片分配感知、强制感知**等。
互联网开发没有“银弹”，分片的数量分配也没有适用于所有场景的最佳值，创建分片策略的最佳方法是使用在生产环境中看到的相同查询和索引负载在生产硬件上对生产数据进行基准测试。分片的分配策略主要从两个指标来衡量：即数量和单个分片的大小。
### 7.1.1 分片分配策略
- 一个索引包含一个或多个分片，在7.0之前默认五个主分片，每格主分片一个副本；在7.0之后默认一个主分片。副本可在索引创建之后修改数量，但是主分片的数量一旦确认不可修改，只能重新创建索引，即Reindex
- 每个分片都是一个Lucene实例，有完整的创建索引和处理请求的能力
- ES会自动在nodes上做分片负载均衡shard rebalance 
- 一个doc不可能同时存在于多个主分片中，但是当每个主分片的副本数量不唯一时，可以同时存在于多个副本中。
- 每个主分片和其副本分片或者相同的副本不能同时存在于同一个节点上。 
### 7.1.2 分片的数量
- **避免分片过多**：大多数搜索会命中多个分片。每个分片在单个CPU线程上运行搜索。虽然分片可以运行多个并发搜索，但是跨大量分片的搜索会耗尽节点的搜索线程池。这会导致低吞吐量和缓慢的搜索速度。
- **分片越少越好**：每个分片都使用内存和CPU资源。在大多数情况下，一小组大分片比许多小分片使用更少的资源。
### 7.1.3 分片的大小决策
- **分片的合理容量**：10GB-50GB。虽然不是硬性限制，但10GB-50GB之间的分片往往效果很好。根据网络和用例，也许可以使用更大的分片。在索引的生命周期管理中，一般设置50GB为单个索引的最大阈值。
- **堆内存容量和分片数量的关联**：小于20分片/GB堆内存，一个节点可以容纳的分片数量与节点的堆内存成正比。例如，一个拥有30GB堆内存的节点最多应该有600个分片。如果节点超过每GB20个分片，考虑添加另一个节点。
查询当前节点堆内存大小：
```json
GET _cat/nodes?v=true&h=heap.current
```
- 避免重负载节点：如果分配给特定节点的分片过多，会造成当前节点为**重负载节点**
## 7.2 重要的配置

ES写入流程
![Pasted image 20250417024153.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250701143657-a39605-Pasted%20image%2020250417024153.png)