#es #elasticsearch
什么是ES？
为什么是ES？


Mapping映射
尽量不要使用自动映射
**生产环境中务必要使用显示映射**

映射参数
`index`
>ES调优：关闭不需要创建倒排索引的字段的index属性

`{python  }text`

	```{ 123 } 
	dafasfa 
	```
`keyword`


# Query DSL(Domain Specific Language)

相关度评分：\_score
元数据：\_source 

## Query String
- Search All: `GET /product/_search`
- Parameter: `GET /product/_search?q=name:xiaomi`
- Page: `GET /product/_search?from=0&size=2&sort=price:asc`
- Exact Value: `GET /product/_search?q=date:2021-06-01`
- All Search: `GET /product/_search?q=2021-06-01`

FullText query
- match: 匹配包含某个term的子句
```json
GET product/_search
{
	"query": {
		"match": {
			"name": "xiaomi nfc phone"
		}
	}
}
```
- match_all: 匹配所有子句的结果
```json
GET product/_search
{
	"query": {
	  "match_all": {}
	}
}
```
- multi_match: 多字段条件
```json
GET product/_search
{
	"query": {
		"multi_match": {
			"query": "xiaomi nfc",
			"field": ["name", "desc"]
		}
	}
}
```
- match_phrase: 短语查询，查询时会匹配字段中每个分词的顺序
```json
GET product/_search
{
	"query": {
		"match_phrase": {
			"name": "xiaomi nfc"
		}
	}
}
```

## Term query
- term: 匹配和搜索词项完全相等的结果
	- term和match_phrase区别：
	  match_phrase会将检索关键词分词，match_phrase的分词结果必须在被检索字段中的分词中都包含，而且顺序必须相同，而且默认必须都是连续的。term搜索**不会**将搜索词**分词**。
	- term和keyword区别
	  term是对于搜索词不分词，keyword是字段的类型（不分词），是对于source data中的字段值不分词。
- terms：匹配和搜索词项列表中任意项匹配的结果
- range：范围查找



  ```json
	GET product/_search
{
	"query": {
		"term": {
			"name": "xiaomi nfc"
		}
	}
}
  ```
 ```json
	GET product/_search
{
	"query": {
		"term": {
			"name.keyword": "xiaomi phone"
		}
	}
}
 ```
 ```json
	GET product/_search
{
	"query": {
		"terms": {
			"tags": ["lowbee", "gongjiaoka"]
		}
	}
}
 ```
 ```json
	GET product/_search
{
	"query": {
		"range": {
			"date": {
				"time_zone": "+8:00",
				"gte": "2021-04-15T08:00:00",
				"let": "2021-04-16T08:00:00"
			}
		}
	}
}
 ```

## Filter
-  filter：query和filter的主要区别在： filter是结果导向的而query是过程导向。query倾向于“当前文档和查询的语句的相关度”而filter倾向于“当前文档和查询的条件是不是相符”。即在查询过程中，query是要对查询的每个结果计算相关性得分的，而filter不会。另外filter有相应的[[Elasticsearch原理篇#缓存机制]]，可以提高查询效率。
```json
GET product/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "name": "phone"
        }
      },
      "boost": 1.2  
    }
  }
}

GET product/_search
{
  "query": {
    "bool": {
      "filter": [
	      {
		      "term": {
				"name": "phone"
			}
	      }
      ]
    }
  }
}
```

## Bool Query
bool：可以组合多个查询条件，bool查询也是采用more_matches_is_better的机制，因此满足must和should子句的文档将会合并起来计算分值
- **must** ：必须满足子句（查询）必须出现在匹配的文档中，并将有助于得分。   
- should

分词器

filter

tokenizer

重点：热更新