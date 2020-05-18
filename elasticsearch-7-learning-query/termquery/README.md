# 结构化查询
通常用于结构化数据，如数字、日期和枚举，而不是全文字段。或者，在分析过程之前，它允许你绘制低级查询。

Term-level查询分为四大类
- Term\Terms查询
- 模糊查询：fuzzy prefix wildcard regexp
- Exist存在查询
- Range范围查询

注意：Term-level查询，对输入不做分词。会将输入作为一个整体，在倒排索引中查找准确的词项，并且使用相关度算分公式为每个包含该词项的文档进行相关度算分
可以通过 Constant Score 将查询转换换成一个 Filtering，避免算分，并利用缓存，提交性能

与之对应的，ElasticSearch还提供了基于全文的搜索，其含义是：对查询串进行分词，并汇总查询结果和打分返回给用户。详见full text query模块



## Term Query

### term query 精确值查询
Returns documents that contain an exact term in a provided field.
```
GET /_search
{
    "query": {
        "term": {
            "user": {
                "value": "Kimchy"
            }
        }
    }
}
```


### terms query
Returns documents that contain one or more exact terms in a provided field.
```
GET /_search
{
    "query" : {
        "terms" : {
            "user" : ["kimchy", "elasticsearch"],
            "boost" : 1.0
        }
    }
}
```

### terms_set query
The terms_set query is the same as the terms query, except you can define the number of matching terms required to return a document. For example:




## 模糊匹配  fuzzy prefix wildcard regexp

### wildcard query
Returns documents that contain terms matching a wildcard pattern.

### fuzzy query
启用模糊匹配来捕捉拼写错误. 基于与原始词的Levenshtein距离来指定模糊度。
fuzzy搜索技术 --> 自动将拼写错误的搜索文本，进行纠正，纠正以后去尝试匹配索引中的数据
关键参数"fuzziness",表示你的搜索文本最多可以纠正几个字母去跟你的数据进行匹配，默认如果不设置，就是2

```
POST /fuzzytest/_bulk
{ "index": { "_id": 1 }}
{ "text": "Surprise me!"}
{ "index": { "_id": 2 }}
{ "text": "That was surprising."}
{ "index": { "_id": 3 }}
{ "text": "I wasn't surprised."}


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
### prefix query
返回在提供的字段中包含特定前缀的文档。
```
POST /fuzzytest/_bulk
{ "index": { "_id": 1 }}
{ "text": "Surprise me!"}
{ "index": { "_id": 2 }}
{ "text": "That was surprising."}
{ "index": { "_id": 3 }}
{ "text": "I wasn't surprised."}
# 输入su会返回3个 输入th会返回一个
GET fuzzytest/_search
{
    "query": {
        "prefix": {
            "text": {
                "value": "th"
            }
        }
    }
}
```

### regexp query
Returns documents that contain terms matching a regular expression.


## Exist查询 exists query
返回包含字段索引值的文档
```
GET products/_search
{
    "query": {
        "exists": {
            "field": "user"
        }
    }
}

GET products/_search
{
    "query": {
        "bool": {
            "must": {
                "exists": {
                    "field": "user"
                }
            }
        }
    }
}
```


## 范围查询
range query
Returns documents that contain terms within a provided range.
```
#数字 Range 查询
GET products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "price" : {
                        "gte" : 20,
                        "lte"  : 30
                    }
                }
            }
        }
    }
}


# 日期 range
POST products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "range" : {
                    "date" : {
                      "gte" : "now-1y"
                    }
                }
            }
        }
    }
}

```

## 其他查询
ids query
根据文档的id返回文档。即匹配_id字段
```
GET /_search
{
    "query": {
        "ids" : {
            "values" : ["1", "4", "100"]
        }
    }
}
```
type query
Returns documents of the specified type.




