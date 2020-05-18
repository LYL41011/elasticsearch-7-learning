# Filter和Query的区别

默认情况下，Elasticsearch根据相关性评分对匹配的搜索结果进行排序，相关性评分衡量每个文档与查询的匹配程度
query关注点：此文档与此查询子句的匹配程度如何？“How well does this document match this query clause?”
1）是否包含？2）相关度得分多少？3）得分越高，相关度越高。
适合于全文检索——这种相关性的概念非常适合全文搜索，因为很少有完全“正确”的答案。

filter关注点：此文档和查询子句匹配吗？“Does this document match this query clause?” 
1）是否包含？2）不考虑相关度评分 
适用于完全精确匹配，范围检索。更快。为什么会更快？经常使用的过滤器将被Elasticsearch自动缓存，以提高性能。

因此：全文检索以及任何使用相关性评分的场景使用query检索，其他就用filter


```
# 测试filter和query的score评分
POST /newmovies/_bulk
{ "index": { "_id": 1 }}
{ "title" : "Father of the Bridge Part II","year":1995, "genre":"Comedy","genre_count":1 }
{ "index": { "_id": 2 }}
{ "title" : "Dave","year":1993,"genre":["Comedy","Romance"],"genre_count":2 }

# 返回"_score" : 1.2111092
POST /newmovies/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}

      ]
    }
  }
}
# 返回"_score" : 0.0
POST /newmovies/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}
        ]

    }
  }
}

```

#Filter实战

```

# 以下案例 两个match会用于评估每个文档的匹配程度 而filter中的term和range 它们将过滤掉不匹配的文档，但不会影响匹配文档的分数。
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}

POST _search
{
  "query": {
    "bool" : {

      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      }
    }
  }
}
```