# ES检索
根据以下几大块进行学习
### 检索和过滤的区别 见filter目录
### Search API 见restapis模块
### 结构化查询 见termquery目录
### 全文检索 见fulltextquery目录
### Join 查询 见join目录
### 复合查询 见compound目录
### 地理信息查询 见geo

### 处理搜索结果 
https://www.elastic.co/guide/en/elasticsearch/reference/7.0/search-uri-request.html
包括source filter、分页、排序、搜索结果高亮
```
# ignore_unavailable=true，可以忽略尝试访问不存在的索引“404_idx”导致的报错
# 测试分页
POST /movies,404_idx/_search?ignore_unavailable=true
{
  "profile": true,
	"query": {
		"match_all": {}
	}
}

POST /movies/_search
{
  "from":10,
  "size":20,
  "query":{
    "match_all": {}

# 测试排序 对日期排序
POST kibana_sample_data_ecommerce/_search
{
  "sort":[{"order_date":"desc"}],
  "query":{
    "match_all": {}
  }

}

# source filtering结果返回指定字段
POST kibana_sample_data_ecommerce/_search
{
  "_source":["order_date"],
  "query":{
    "match_all": {}
  }
}

#脚本字段
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_field": {
      "script": {
        "lang": "painless",
        "source": "doc['order_date'].value+'hello'"
      }
    }
  },
  "query": {
    "match_all": {}
  }
} 
```


### 控制相关度
### 一些通用参数
如果对dsl拆解不理解，那就再加上profile:true或者explain:true拆解结果一目了然。

- "profile": "true"
通过这个功能，可以看到一个搜索聚合请求，是如何拆分成底层的 Lucene 请求，并且显示每部分的耗时情况。
- "explain": true
查看打分情况 会输出_explanation
```
GET fuzzytest/_search
{
    "query": {
        "prefix": {
            "text": {
                "value": "th"
            }
        }
    },
    "profile": "true",
    "explain": true
}
```
- rewrite
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html
可以对任何多词项查询使用rewrite参数来控制查询改写。我们可以将rewrite参数存放在代表查询的JSON对象中
 rewrite参数的选项配置：
 --scoring_boolean :  该选项将每个生成的词项转化为布尔查询中的一个或从句。这种处理方法比较耗CPU（因为要计算和保存每个词项的得分），而且有些查询生成的词项太多从而超出了布尔查询的限制，默认为1024个从句。改写后的查询会保存计算出来的得分。默认的布尔查询限制可以通过设置elasticsearch.yml文件的Index.query.bool/max_clause_count属性来修改。但须谨记改写后的布尔查询从句越多，查询性能越低
 --constant_score:该选项与前面提到的scoring_boolean类似，但是CPU消耗较少，这是因为该过程并不计算每个从句的得分，而是每个从句得到一个与查询权重相同的常数得分，默认情况下等于1，当然我们也可以通过设置查询权重来改变这个默认值，与scoring_boolean类似，该选项也有布尔从句数的限制
 --constant_score_filter: 正如Lucene的javadoc描述的那样，该选项按如下方式改写原始查询：通过顺序遍历每个词项来创建一个私有的过滤器，标记跟每个词项相关的所有文档。命中的文档被赋予一个跟查询权重相关的常量得分。当命中词项数或文档数较大时，该方法比前两种执行速度更快
 --top_terms_N : 该选项将每个生成的词项转化为布尔查询中的一个从句，并保存计算出来的得分。与scoring_boolean不同之处在于，该方法只保留了最佳的前N个词项，从而避免超出布尔从句数的限制。
 --top_termd_boost_N: 该选项与top_terms_N类似，不同之处在于该选项产生的从句类型为常量得分查询，得分为从句权重。
```
GET /fuzzytest/_search 
{
  "query": {
    "fuzzy": {
      "text": {
        "value": "surprize",
        "fuzziness": 2,
        "rewrite": "constant_score"
      }
    }
  }
}
```