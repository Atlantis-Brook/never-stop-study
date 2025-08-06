#es #elasticsearch
# ES9新特性 todo
 25年04月15日 Elastic推出[Elasticsearch9.0]()版本
- Full-Text Search
- RAG ,使用 Elasticsearch 的简单 RAG 系统 <调研ES支持什么样的向量数据库>
  ![Components of a simple RAG system using Elasticsearch](https://www.elastic.co/docs/solutions/images/elasticsearch-reference-rag-schema.svg)
- semantic_text https://www.bilibili.com/video/BV1j25TzKEft/?vd_source=29e582d27bf2c4eb8b68757fde2921b3
# ES核心概念 todo

集群：
## 节点（Node）
- **数据节点（Data Node）**：负责存储数据（主分片和副本分片），执行索引、搜索、聚合等操作。配置为 node.data: true。
- **主节点（Master Node）**：负责集群管理任务，如分片分配、节点加入/退出、索引创建等。配置为 node.master: true。
- **协调节点（Coordinating Node）**：接收客户端请求，协调查询或写入操作，分发任务到数据节点并汇总结果。默认情况下，所有节点都可以充当协调节点，除非明确禁用（node.data: false, node.master: false）。
- **摄入节点（Ingest Node）**：用于预处理文档（如数据转换、过滤）。配置为 node.ingest: true。
一个节点可以同时扮演多个角色（例如数据节点和协调节点），但在生产环境中通常分离角色以优化性能。

## 分片（Shard）
分片是索引的物理存储单元，将索引数据分割成较小的部分，分布在集群的不同节点上。分片分为两种类型：
- **主分片（Primary Shard）**：负责处理写入操作，并将数据复制到副本分片。
- **副本分片（Replica Shard）**：主分片的拷贝，用于提高读性能和容错能力。

## 段（Segment）
段是 Lucene（Elasticsearch 的底层库）中用于存储数据的物理文件。每个分片由多个段组成，每个段是一个倒排索引（Inverted Index），存储文档和搜索相关的元数据。
**段合并（Segment Merging）**：

- **背景**：每次写入（索引或更新文档）都会生成新的段。随着时间推移，段数量增加可能导致性能下降。
- **合并过程**：Elasticsearch 定期将小段合并为较大的段，减少段数量，优化查询性能。
- **影响**：
    - 段合并是 CPU 和 I/O 密集型操作，可能影响节点性能。
    - 合并后的段是不可变的（Immutable），新写入的文档会生成新的段。
- **配置**：可以通过 index.merge.policy 参数调整合并策略，但通常建议保留默认设置。

找到ES书中的图
# ES核心原理  todo
> 从宏观到微观进行讲解 一个文档创建 -》 写入〉 -〉 存储 -〉生成索引
## 倒排索引底层原理
**什么是搜索引擎：**
- 全文搜索引擎，自然语言处理、爬虫、网页处理、大数据处理如，谷歌、百度、必应等
- 垂直搜索引擎，有明确的搜索行为，各大电商网站、OA、站内搜索、视频网站等。
**搜索引擎应该具备哪些能力**
1. 查询速度快
	- 高效的压缩算法
	- 快速的编码和解码速度
2. 结果准确：相关度评分算法
	- BM25
	- TF-IDF
3. 检索结果丰富：召回率
### 倒排索引的数据结构
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250718025221-a2812a-20250718025221034.png)
- **倒排表（Posting List）**：记录包含某个词项的所有文档的列表，通常包含文档 ID 和其他元数据。
- **词项字典（Term dictionary）**：文档经过分词（tokenization）后生成的最小索引单元，通常是单词或短语。
- **词项索引（Term index）**：存储所有词项的集合，通常使用高效的数据结构（如 B+ 树或哈希表）以支持快速查找。
### 倒排索引核心算法
#### 倒排表的压缩算法
**压缩，就是尽可能降低每个数据占用的空间，同时又能让信息不失真，能够还原回来。**
##### FOR：Frame Of Reference
稠密数据
Delta-encode（增量编码）
##### RBM：Roaring Bit Map
稀疏数组

> 稀疏的数组不适合使用FOR压缩算法
#### 词项索引的检索原理
所有计算机软件的本质都是状态机，软件的九成以上的逻辑都是储存和更改各种数据的状态
而ES就采用xxx状态机进行数据压缩
FSM
FSA


Finite State Transducers，FST
## ES读写原理及调优
### ES文档的写入过程
#### ES支持的写入操作
- create，只创建，不覆盖
- delete，懒删除（“标记删除”）机制，不是立即从磁盘中移除
- index，创建或覆盖文档
- update，更新文档中的部分字段（不会覆盖整个文档）
>以文档为基本单位进行操作。
#### ES数据写入流程
ES中数据写入均发生在Primary Shard，当数据在Primary写入完成之后，会同步到相应的Replica Shard，ES的数据写入流程：
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250715013247-762e07-20250715013246598.png)
1. 客户端发起写入请求至node 4
2. node 4通过文档 id 在路由表中的映射信息确定当前数据的位置为分片0，分片0的主分片位于node 5，并将数据转发至node 5。
3. 数据在node 5写入，写入成功之后将数据的同步请求转发至其副本所在的node 4和node 6上面，等待所有副本数据写入成功之后node 5将结果报告node 4，并由node 4将结果返回给客户端，报告数据写入成功。
在这个过程中，接受用户请求的节点是不固定的，上图中node 4发挥了协调节点和客户端节点的作用，将数据负载均衡到对应节点和接收并返回用户请求。
>数据在由 node4 转发至 node5的时候，是通过以下公式来计算，指定的文档具体在那个分片的
>`shard_num = hash(_routing) % num_primary_shards`
>其中，\_routing 的默认值是文档的 id。
#### 写一致性策略 （怎么讲解？）todo
一致性策略由 `wait_for_active_shards` 参数控制：
即确定客户端返回数据之前必须处于active 的分片分片数（包括主分片和副本），默认为 wait_for_active_shards = 1，即只需要主分片写入成功，设置为 `all`或任何正整数，最大值为索引中的分片总数 ( `number_of_replicas + 1` )。如果当前 active 状态的副本没有达到设定阈值，写操作必须等待并且重试，默认等待时间30秒，直到 active 状态的副本数量超过设定的阈值或者超时返回失败为止。
执行索引操作时，分配给执行索引操作的主分片可能不可用。造成这种情况的原因可能是主分片当前正在从网关恢复或正在进行重定位。默认情况下，索引操作将在主分片上等待最多 1 分钟，然后才会失败并返回错误。
### ES写入原理
![Image from Fynotefile.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717234604-d999a6-Image%20from%20Fynotefile.png)
  
Memory Buffer是为了保证数据高性能写入
OS Cache是为了保证数据高性能检索
Translog是为了保证数据安全性，临时的
最终同步到磁盘保存数据
#### Translog todo
类似于mysql中的redolog，为了数据安全性引入的概念
#### Refresh
为了把Memory Buffer给清空生成segment文件，立马同步到OS Cache中，此时将segment文件打开，数据就可以被检索了
#### Flush todo
为了把数据从OS Cache中刷到OS Disk中，将索引文件同步到磁盘中
#### Merge
当segment文件产生较多的时候的一个合并过程，在合并的时候回提交一个commit point来标记哪些segment是可用的 ，删除旧的标记。
在合并过程中，标记为删除的数据不会写入新分段，当合并过程结束，旧的分段数据被删除，标记删除的数据才从磁盘中删除
### 写入性能调优
生产经常面临的写入可以分为两种情况：
**高频低量**：高频的创建或更新索引或文档一般发生在 处理 C 端业务的场景下。
**低频高量**：一般情况为定期重建索引或批量更新文档数据。
在搜索引擎的业务场景下，用户一般并不需要那么高的写入实时性。写入的文档需要建立索引，但数据并不会马上被写入索引，而是等待要写入的数据达到一定数量之后，批量写入。这种操作优点类似于快递场景，只有当快递数量装满整车的时候，快递车才会发车。

> 以下为常见 数据写入的调优手段，写入调优均以提升写入吞吐量和并发能力为目标，而非提升写入实时性。
#### 增加refresh_interval的参数值
目的是减少segment文件的创建，减少segment的merge次数，merge是发生在JVM中的，有可能导致full GC，增加refresh会降低搜索的实时性。
#### 增加Buffer大小
本质也是减小refresh的时间间隔，因为导致segment文件创建的原因不仅有时间阈值，还有buffer空间大小，写满了也会创建。 默认最小值 48MB< 默认值 JVM 空间的10% < 默认最大无限制
#### 关闭副本
当需要单次写入大量数据的时候，建议关闭副本，暂停搜索服务，或选择在检索请求量谷值区间时间段来完成。
> 这种场景一般都是对内的，比如每晚需要更新索引、重建索引，此时可以把副本先 关掉，等索引更新完毕之后再打开副本。 
#### 禁用swap
> swap，属于虚拟存储技术，当内存满时，操作系统使用页面置换算法（如LRU）选择候选页，然后swap-out将内存页写入磁盘的swap区，而当进程再次访问此页时执行swap-in，触发缺页中断，操作系统将页从swap区读取回内存。

由于Swap 是磁盘上划分或文件形式的**虚拟内存扩展区域**，当系统物理 RAM 不足时，暂存不常用内存页到这里，以释放 RAM，虽能避免 OOM，却会因 I/O 较慢而影响性能。

![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250716032702-bab087-20250716032701498.png)
>交换对性能和节点稳定性非常不利，应该不惜一切代价避免。它可能导致垃圾收集持续几分钟而不是几毫秒，并且可能导致节点响应缓慢甚至与集群断开连接。在Elastic分布式系统中，让操作系统杀死节点更有效。
- 频繁 swap 不仅降低性能，还会引入不稳定性，例如 **响应延迟、节点掉线甚至集群分裂** 。
- 因此 ES 更倾向于在内存紧张时让系统直接 **OOM 杀掉节点**，而不是让关键 JVM 堆被边缘化。
#### 使用多个工作线程
为了使用集群的所有资源，应该从多个线程或进程发送数据。除了更好地利用集群的资源外，还有助于降低每个 fsync 的成本。
确保注意 TOO_MANY_REQUESTS (429)响应代码（EsRejectedExecutionException使用 Java 客户端），这是 Elasticsearch 告诉我们它无法跟上当前索引速度的方式。发生这种情况时，应该在重试之前暂停索引，最好使用随机指数退避算法，有效避免压垮集群。
#### 避免使用稀疏数据
**稀疏数据** 是指在同一个索引下，大量文档的字段不一致，很多字段对于大多数文档都是空值或不存在。这会导致即使字段为空，Lucene也会在内部为每个文档预留存储空间，浪费磁盘和内存资源并且写入与刷新时开销以及查询性能下降。
**建议**拆分索引、禁止不必要字段索引避免mapping膨胀。
### 查询性能调优
#### 避免单次召回大量数据
搜索引擎最擅长的事情是从海量数据中查询少量相关文档，而非单次检索大量文档。非常不建议动辄查询上万数据。如果有这样的需求，建议使用滚动查询。
#### 避免单个文档过大
默认http.max_content_length设置为 100MB，Elasticsearch 将拒绝索引任何大于该值的文档。虽然可以修改配置来增加大小，但 Lucene 仍然有大约 2GB 的限制。
#### 数据建模
避免嵌套与父子关联。Nested 可以使查询慢几倍，Join 会使查询慢数百倍。两种类型的使用场景应该是：Nested针对字段值为非基本数据类型的时候，而Join则用于当子文档数量级非常大的时候。
推荐使用 denormalized（扁平）结构以提高查询效率
#### 使用 filter 代替 query
在查询过程中，query是要对查询的每个结果计算相关性得分的，而filter不会。另外filter有相应的缓存机制，可以提高查询效率。
#### 避免深度分页
避免单页数据过大。ES提供两种解决方案 scroll search 和 search after。后面会详细讲解关于深度分页及其优化。
#### 避免使用脚本
Scripting是Elasticsearch支持的一种专门用于复杂场景下支持自定义编程的脚本功能。相对于 DSL 而言，脚本的性能更差，DSL能解决 80% 以上的查询需求，如非必须，尽量避免使用 Script
## 深度分页问题 (Deep Paging)
### 什么是深度分页
ES中正常的分页查询
```json
GET order_2290w/_search
{
  "from": 0,
  "size": 5
}
```
但是如果我们查询的数据页数特别大，达到当`from + size`大于`10000`的时候，就会出现问题，如下图报错信息所示：
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717164430-495380-20250717164429958.png)
分布式系统都面临着同一个问题，数据的排序不可能在同一个节点完成。
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717183218-73b3dd-20250717183218100.png)
一共10w条数据存储在ES数据节点中，我们想取10000～10100条ES检索数据时，我们从任意一个分片中取出的10100条数据，都不一定是全部数据的前10100条。而唯一的解决办法是从每个分片中取出当前分片的前10100条，然后汇总成50500条数据再次排序，然后从排序后的这50500条中查询前10100条数据，此时才能保证一定是整个索引中的前10100条数据。

每次有序的查询都会在每个分片中执行单独的查询，然后进行数据的二次排序，而这个二次排序的过程是发生在heap中的，如果想查询的是第10001-10100这一百条数据，ES必须将前10100全部取出进行二次查询。因此，如果查询的数据排序越靠后，就越容易导致OOM（Out Of Memory）情况的发生，频繁的深分页查询会导致频繁的FGC。
### max_result_window参数
`max_result_window`是分页返回的最大数值，默认值为10000。max_result_window本身是对JVM的一种保护机制，通过设定一个合理的阈值，避免分页查询时由于单页数据过大而导致OOM。

**永远不要暴力提升** `max_result_window`, 很可能导致的后果就是频繁的发生OOM而且很难找到原因。
### 深度分页解决方案
#### 避免使用深度分页
解决深度分页的最有效手段就是避免使用深度分页，可能有人会有疑问作为技术怎么能要求客户做什么呢，通过妥协用户体验来解决技术问题？
众多优秀大型搜索引擎是如何面对深度分页场景的：
全文搜索，Google / Baidu 等所有词条召回结果中均没有跳页功能；
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717014204-d148e4-20250717014202825.png)
垂直搜索，淘宝 / 京东 等检索商品结果中最多仅展示100页，本质和ES中的`max_result_window`作用是一样的。
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717014111-4f14bb-20250717014109607.png)
#### 滚动查询 Scroll Search
官方ES7之后已不再推荐使用滚动查询进行深度分页查询，因为无法保存索引状态。
##### 适合场景
单个滚动搜索请求中检索大量结果，即**非C端**应用场景
##### 使用
```json
GET <index>/_search?scroll=1m
{
  "size": 100
}
```
`scroll`表示Elasticsearch 应该保持“搜索上下文”多长时间，`scroll`超时后，搜索上下文会自动删除。


```json
 POST /_search/scroll
{
	"scroll": "1m",
	"scroll _id":
	 "DXF1ZХJ5QW5kRmV®Y2gBAАAAААABY08WN3Q4dDJjcVVRQ0NBb]1GMmFqN®ZVZw=="
}
```
##### 特点
- 可处理亿级数据的全量查询，适合批量读取所有文档，如导出任务等
- 不能用于实时场景，占用内存上下文
#### 深度翻页 Search After
##### 适合场景
需要稳定、高性能分页场景，如用户行为连续浏览，即**C端**应用场景
##### 如何使用
使用当前页最后一条文档的排序字段值（如时间戳、唯一 ID）作为游标
```json
GET <index>/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "sort": [{ "timestamp": "asc" }, { "_id": "asc" }],
  "search_after": [1625022345000, "doc123"]
}
```
##### 特点
- 不支持向前搜索
- 每次只能向后搜索1页数据
- 适用于C端业务
# 项目预告
Q3 ES平台建设项目重新启动，各业务中心在日常使用ES搜索的CASE中的问题，在中台侧推动各业务中心接入并升级ES时，统一协助解决。重新规范化整合ES资源，并解决历史遗留问题。
