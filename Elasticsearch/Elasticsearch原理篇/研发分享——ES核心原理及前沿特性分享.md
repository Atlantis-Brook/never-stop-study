# ES9新特性
 25年04月15日 Elastic推出Elasticsearch9.0版本
- Full-Text Search
- RAG ,使用 Elasticsearch 的简单 RAG 系统
  ![Components of a simple RAG system using Elasticsearch](https://www.elastic.co/docs/solutions/images/elasticsearch-reference-rag-schema.svg)
ES8

# ES核心概念

# ES核心原理

## 深度分页问题 (Deep Paging)
ES中from+size分页

### 避免使用深度分页
解决深度分页的最有效手段就是避免使用深度分页（此处可以举例子 死锁的解决/ 避免死锁）
此处可以举例子，Google / Baidu 等所有词条召回结果中均没有页面跳转功能；垂直领域，淘宝 / 京东 等检索商品结果中最多仅展示100页
### 滚动查询 Scroll Search
官方ES7之后已不再推荐使用滚动查询进行深度分页查询，因为无法保存索引状态。
#### 适合场景
单个滚动搜索请求中检索大量结果，即非C端应用场景


## 倒排索引底层原理

## 高级检索

## 高级聚合

## 相关度评分算法

## ES读写原理及调优