> 开场：我也是初学者，对ES理论的理解相对浅薄，今天的分享主要是抛砖引玉的效果，做一个ES的简单入门，加深一下大家对ES的理解，在后续工作中进行深度研究。
# ES9新特性 717
 25年04月15日 Elastic推出[Elasticsearch9.0]()版本
- Full-Text Search
- RAG ,使用 Elasticsearch 的简单 RAG 系统 <调研ES支持什么样的向量数据库>
  ![Components of a simple RAG system using Elasticsearch](https://www.elastic.co/docs/solutions/images/elasticsearch-reference-rag-schema.svg)
- semantic_text https://www.bilibili.com/video/BV1j25TzKEft/?vd_source=29e582d27bf2c4eb8b68757fde2921b3

# ES核心概念 718
节点
索引
分片
- 数据分片
- 数据副本
分段
词项
找到ES书中的图
# ES核心原理 
> 从宏观到微观进行讲解 一个文档创建 -》 写入〉 -〉 存储 -〉生成索引
## ES读写原理及调优 0714
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
数据在由 node4 转发至 node5的时候，是通过以下公式来计算，指定的文档具体在那个分片的
`shard_num = hash(_routing) % num_primary_shards`
其中，\_routing 的默认值是文档的 id。
#### 写一致性策略 （怎么讲解？）
一致性策略由 `wait_for_active_shards` 参数控制：
即确定客户端返回数据之前必须处于active 的分片分片数（包括主分片和副本），默认为 wait_for_active_shards = 1，即只需要主分片写入成功，设置为 `all`或任何正整数，最大值为索引中的分片总数 ( `number_of_replicas + 1` )。如果当前 active 状态的副本没有达到设定阈值，写操作必须等待并且重试，默认等待时间30秒，直到 active 状态的副本数量超过设定的阈值或者超时返回失败为止。
执行索引操作时，分配给执行索引操作的主分片可能不可用。造成这种情况的原因可能是主分片当前正在从网关恢复或正在进行重定位。默认情况下，索引操作将在主分片上等待最多 1 分钟，然后才会失败并返回错误。
### ES写入原理

Memory Buffer是为了保证数据高性能写入
OS Cache是为了保证数据高性能检索
Translog是为了保证数据安全性，临时的
最终同步到磁盘保存数据

Translog

Refresh

Flush

Merge
### 写入性能调优

#### 关闭副本
当需要单次写入大量数据的时候，建议关闭副本，暂停搜索服务，或选择在检索请求量谷值区间时间段来完成。
> 这种场景一般都是对内的，比如每晚需要更新索引、重建索引，此时可以把副本先 关掉，等索引更新完毕之后再打开副本。 

#### 禁用swap
> swap，利用虚拟存储技术，当内存未满，直接启动磁盘IO，而内存已满时当发生缺页中断 优化！！！
Swap 是磁盘上划分或文件形式的**虚拟内存扩展区域**，当系统物理 RAM 不足时，暂存不常用内存页到这里，以释放 RAM，虽能避免 OOM，却会因 I/O 较慢而影响性能。

gpt 记录中详细整理下
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250716032702-bab087-20250716032701498.png)

### 查询性能调优

## 倒排索引底层原理 0715
### 压缩算法
FOR
稠密数据

RBM
稀疏数组

> 稀疏的数组不适合使用FOR压缩算法

所有计算机软件的本质都是状态机，软件的九成以上的逻辑都是储存和更改各种数据的状态
而ES就采用xxx状态机进行数据压缩
FSM
FSA
Finite State Transducers，FST

## 深度分页问题 (Deep Paging)
ES中正常的分页查询
```json
GET order_2290w/_search
{
  "from": 0,
  "size": 5
}
```
很好理解，即查询第一页的`5`条数据。图中数字2即返回的五条文档数据。但是如果我们查询的数据页数特别大，达到什么程度呢？当`from + size`大于`10000`的时候，就会出现问题，如下图报错信息所示：
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717164430-495380-20250717164429958.png)
分布式系统都面临着同一个问题，数据的排序不可能在同一个节点完成。


max_result_window是分页返回的最大数值，默认值为10000。max_result_window本身是对JVM的一种保护机制，通过设定一个合理的阈值，避免分页查询时由于单页数据过大而导致OOM。

在很多业务场景中经常需要查询10000条以后的数据，当遇到不能查询10000条以后的数据的问题之后，网上的很多答案会告诉你可以通过放开这个参数的限制，将其配置为100万，甚至1000万就行。但是如果仅仅放开这个参数就行，那么这个参数限制的意义有何在呢？如果你不知道这个参数的意义，很可能导致的后果就是频繁的发生OOM而且很难找到原因，设置一个合理的大小是需要通过你的各项指标参数来衡量确定的，比如你用户量、数据量、物理内存的大小、分片的数量等等。通过监控数据和分析各项指标从而确定一个最佳值，并非越大约好。
### 避免使用深度分页
解决深度分页的最有效手段就是避免使用深度分页（此处可以举例子 死锁的解决/ 避免死锁）
此处可以举例子：
全文搜索，Google / Baidu 等所有词条召回结果中均没有跳页功能；
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717014204-d148e4-20250717014202825.png)
垂直搜索，淘宝 / 京东 等检索商品结果中最多仅展示100页，本质和ES中的`max_result_window`作用是一样的。
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250717014111-4f14bb-20250717014109607.png)
### 滚动查询 Scroll Search
官方ES7之后已不再推荐使用滚动查询进行深度分页查询，因为无法保存索引状态。
#### 适合场景
单个滚动搜索请求中检索大量结果，即**非C端**应用场景
#### 使用
```json
GET <index>/_search?scroll=1m
{
  "size": 100
}
```
`scroll`表示Elasticsearch 应该保持“搜索上下文”多长时间

```json
POST /_search/scroll
{
	"scroll": "1m",
	"scroll _id": "DXF1ZХJ5QW5kRmV®Y2gBAАAAААABY08WN3Q4dDJjcVVRQ0NBb]1GMmFqN®ZVZw=="
}
```




## Search After

- 不支持向前搜索
    
- 每次只能向后搜索1页数据
    
- 适用于C端业务






活动预告
Q3 ES平台建设项目重新启动，各业务中心在日常使用ES搜索的CASE中的问题，在中台侧推动各业务中心接入并升级ES时，统一协助解决。重新规范化整合ES资源，并解决历史遗留问题。
