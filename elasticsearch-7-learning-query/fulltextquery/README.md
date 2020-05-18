# Full Text Queries 全文检索
https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html
- 索引和搜索时会针对查询串进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
- 将分词后得到的每个词项在索引中进行搜索，并汇总查询结果和打分返回给用户。 例如查 “Martix reloaded”, 会查到包括 Matrix 或者 reload 的所有结果。
与之对应的，ElasticSearch还提供了基于词项term的搜索，其含义是：不对查询串进行分词，直接查询。详见term query模块

先看一组基于全文和基于词项的查询的差异，这也是初学者非常迷惑的地方:
```
# match正常查询出数据
POST /movies/_search
{
  "query":{
    "match": {
      "title": "Delicatessen"
    }
  }
}
# term查不到数据 因为原始数据在加入索引时，默认会进行分词处理，默认的分词器会将分词后所有词项转化为小写(可以用_analyze进行验证写时的分词) 
# 而term表示查询的时候不会分词，因此查询是Delicatessen但倒排索引中是delicatessen
# match之所以能查到，是因为match查询的时候会分词，所以查和存储都是delicatessen
# 改为小写则可以查询 或者使用title.keyword
POST /movies/_search
{
  "query":{
    "term": {
      "title": "Delicatessen"
    }
  }
}

# match查询会进行分词,相当于Saving OR Christmas
POST /movies/_search
{
  "query":{
    "match": {
      "title": "Saving Christmas"
    }
  }
}
# 如果希望查出的是同时含有Saving Christmas那么可以使用operator
POST /movies/_search
{
  "query":{
    "match": {
      "title": {      
         "query":    "Saving Christmas",
          "operator": "and"
      }
    }
  }
}


# term查不到数据，上面的案例说改成小写就可以，但是现在即使改成小写也查不到
POST /movies/_search
{
  "query":{
    "term": {
      "title": "Saving Christmas"
    }
  }
}
# 我们可以使用_analyze来分析一下为什么查不到数据 可以看到有两个tokens
# 问题不在 term 查询，而在于索引数据的方式
# Elasticsearch 用 2 个不同的 token 而不是单个 token 来表示,所有字母都是小写的,且丢失了连字符和哈希符（ # ）。
# 为了避免这种问题，我们需要告诉 Elasticsearch该字段具有精确值，要将其设置成 not_analyzed 无需分析的 因为不能修改索引，只能删除重建
# 或者还有一种方式 使用title.keyword查询
GET /movies/_analyze
{
  "field": "title",
  "text": "Saving Christmas"
}

# 总结:要明白ES的分词原理和倒排索引，学会使用_analyze进行分析，对于字符串类型，如果需要精确匹配(包括大小写也不能忽略)，那么使用keyword
```

现在开始Full Text Query的学习：
- match query
返回与提供的文本、数字、日期或布尔值匹配的文档。在匹配之前，对提供的文本进行分词。match查询是执行全文搜索的标准查询，包括用于模糊匹配。
```

```



- intervals query
intervals query 允许用户精确控制查询词在文档中出现的先后关系，实现了对terms顺序、terms之间的距离以及它们之间的包含关系的灵活控制
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html#intervals-match

```
POST /products/_bulk
{ "index": { "_id": 5 }}
{ "productID" : "XHDK-A-1293-#fJ3","desc":"my favourite food is cold porridge" }
{ "index": { "_id": 6 }}
{ "productID" : "KDKE-B-9947-#kL5","desc":"when it's cold my favourite food is porridge"}
{ "index": { "_id": 7 }}
{ "productID" : "JODL-X-1937-#pV7","desc":"my wife favourite food is cold porridge" }

POST _search
{
  "query": {
    "intervals" : {
      "desc" : {
        "all_of" : {
          "ordered" : true,
          "intervals" : [
            {
              "match" : {
                "query" : "my favourite food", #用户查询的字符串
                "max_gaps" : 0, #字符串中每个词在text field中出现的最大词间距，超过最大间距的将不会被检索到；默认值是-1，即不限制，设置为0的话，query中的字符串必须彼此相连不能拆分。可以试一下如果是0那么my wife不会被检索到。-1的话才可以。
                "ordered" : true #query中的字符串是否需要有序显示，默认值是false，即不考虑先后顺序
              }
            },
            {
              "any_of" : { #集合里面的所有match不需要同时在一个文档数据上同时满足 如果是all_of则需要
                "intervals" : [
                  { "match" : { "query" : "hot water" } },
                  { "match" : { "query" : "cold porridge" } }
                ]
              }
            }
          ]
        },
        "boost" : 2.0,
        "_name" : "products"
      }
    }
  }
}
```

- match_phrase query
Like the match query but used for matching exact phrases or word proximity matches.
假如文档中有1、"ElasticSearch Java Cliet" 2、ElasticSearch Analysis 3、Java Utils
我们搜索"ElasticSearch Java" 其实是希望只返回第一条数据 因为我们希望搜索出ES的Java客户端操作文档
但如果用match 结果是都会返回 
match_phrase有个参数叫做slop，代表单词之间的位置差，默认是0，即必须相邻。??
但是很多时候我们的要求并不那么苛刻，比如"ElasticSearch High Java Cliet"也是应该需要找到的,此时应该设置slop=1,所以可以根据条件设置。
总结：match_phrase可以作为精确匹配,不过不要当成term了,它还是分词后去搜的:
1、目标文档需要包含分词后的所有词
2、目标文档还要保持这些词的相对顺序和文档中的一致
3、可通过slop控制

- match_phrase_prefix query ??
Like the match_phrase query, but does a wildcard search on the final word.
原理跟match_phrase类似，唯一的区别，就是把最后一个单词作为前缀去搜索。
可以设置max_expansions，代表最多返回结果，比如一个文档有zo1,zo2,...,zo1000,你搜zo且设置max_expansions=10，那么很多结果都查不到了
另外注意一下官网这句话“It is important to understand that the max_expansions query limit works at the shard level, meaning that even if set to 1, multiple terms may match, all coming from different shards. This behavior can make it seem as if max_expansions is not in effect, so beware that counting unique terms that come are returned is not a valid way to determine if max_expansions is working” 大意是说max_expansions是作用在分片级别（shard level）的，这意味着即使设置为1，依然有可能匹配到多个词，这些词来自不同的分片（shards）。这种行为使得结果看起来跟max_expansions没生效一样，因此谨记计算返回搜索结果的关键词数量不能作为检验max_expansions是否生效的方法
https://www.elastic.co/cn/blog/found-fuzzy-search#


- multi_match query
multi_match查询建立在match query的基础上，支持多字段查询:
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}
# 可以使用通配符
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] 
    }
  }
}
# 有多种type可以设置 如best_fields返回最佳结果、还有most_fields等 具体参考文档
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "brown fox",
      "type":       "best_fields",
      "fields":     [ "subject", "message" ],
      "tie_breaker": 0.3
    }
  }
}
```

- query_string query
Supports the compact Lucene query string syntax, allowing you to specify AND|OR|NOT conditions and multi-field search within a single query string. For expert users only.
支持与或非的字符串检索
```
GET /_search
{
    "query": {
        "query_string" : {
            "query" : "(new york city) OR (big apple)",
            "default_field" : "content"
        }
    }
}
```
- simple_query_string query
A simpler, more robust version of the query_string syntax suitable for exposing directly to users.
simple_query_string查询是query_string查询的一个版本，更适合用于暴露给用户的单个搜索框， 
+代替AND、 |代替OR 、-代替NOT。默认的operator是OR
```
GET /_search
{
    "query": {
        "simple_query_string" : {
            "fields" : ["content"],
            "query" : "foo bar -baz"
        }
    }
}
```