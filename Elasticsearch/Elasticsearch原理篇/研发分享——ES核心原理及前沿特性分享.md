# ES9新特性 717
 25年04月15日 Elastic推出[Elasticsearch9.0]()版本
- Full-Text Search
- RAG ,使用 Elasticsearch 的简单 RAG 系统
  ![Components of a simple RAG system using Elasticsearch](https://www.elastic.co/docs/solutions/images/elasticsearch-reference-rag-schema.svg)
- semantic_text

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
> 从宏观到微观进行讲解 一个文档 -》 写入 -〉 存储 -〉生成索引
## ES读写原理及调优 0714
### ES文档的写入过程
#### ES支持的写入操作
- create
- delete，懒删除机制
- index
- update
#### ES数据写入流程
ES中数据写入均发生在Primary Shard，当数据在Primary写入完成之后，会同步到相应的Replica Shard，ES的数据写入流程：
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250715013247-762e07-20250715013246598.png)
#### 写一致性策略

### ES写入原理

Memory Buffer是为了保证数据高性能写入
OS Cache是为了保证数据高性能检索
Translog是为了保证数据安全性，临时的
最终同步到磁盘保存数据

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
ES中from+size分页

### 避免使用深度分页
解决深度分页的最有效手段就是避免使用深度分页（此处可以举例子 死锁的解决/ 避免死锁）
此处可以举例子，全文搜索，Google / Baidu 等所有词条召回结果中均没有页面跳转功能；垂直搜索，淘宝 / 京东 等检索商品结果中最多仅展示100页
### 滚动查询 Scroll Search
官方ES7之后已不再推荐使用滚动查询进行深度分页查询，因为无法保存索引状态。
#### 适合场景
单个滚动搜索请求中检索大量结果，即非C端应用场景
