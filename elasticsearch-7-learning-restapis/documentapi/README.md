# 文档API
参考文档:https://www.elastic.co/guide/en/elasticsearch/reference/7.6/cat-fielddata.html 

## Single document APIs
Index API:将JSON文档添加到指定的索引并使其可搜索。如果文档已经存在，则更新文档并增加其版本。
```
指定id:PUT /<index>/_doc/<_id>
自动生成id:POST /<index>/_doc/

PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
指定路由:
POST twitter/_doc/2?routing=kimchy
   {
       "user" : "kimchy",
       "post_date" : "2009-11-15T14:12:12",
       "message" : "trying out Elasticsearch"
   }
```
Get API
```
GET test/_doc/1
# 可以使用_source资源仅检索文档部分字段
GET test/_doc/1?_source=address,mobile
# 包含xxx 不包括xxx 支持正则匹配
GET test/_doc/1?_source_includes=*name&_source_excludes=mobile
# 如果POST的时候指定了routing 那么Get的时候也要指定 默认routing是_id
GET test/_doc/2?routing=user1
```

Delete API
```
DELETE /test/_doc/1
# 如果POST的时候指定了routing 那么Delete的时候也要指定
DELETE /test/_doc/2?routing=user1
# 指定timeout 在执行删除操作时，分配给执行删除操作的shard可能不可用。其中的一些原因可能是shard目前正在从存储中恢复或正在进行迁移。默认情况下，delete操作将等待主碎片可用1分钟，然后失败并返回一个错误。可以使用timeout参数显式地指定它的等待时间
DELETE /test/_doc/1?timeout=5m
```

Update API update API 允许基于提供的脚本更新文档。新增、删除、修改字段值
```
POST test/_update/1
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```       
Upsert
如果文档还不存在，upsert元素的内容将作为新文档插入。如果文档存在，则执行脚本:
```
POST test/_update/1
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
```

## Multi-document APIs

Multi get 在一个API调用中执行多个创建或删除操作。这减少了开销，并可以极大地提高索引速度。
```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_id" : "2",
            "_source" : ["address", "birthday"]
        },
        {
            "_index" : "test",
            "_id" : "3",
            "_source" : {
                "include": ["name"],
                "exclude": ["birthday"]
            }
        }
    ]
}
```

Bulk
```
# 该命令依次执行创建1、2、3三个document,然后删除2，update3中的mobile
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "name" : "张三" }
{ "index" : { "_index" : "test", "_id" : "2" } }
{ "name" : "李四", "address": "shenzhen" }
{ "index" : { "_index" : "test", "_id" : "3" } }
{ "name" : "王五", "mobile": "124124" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "update" : {"_id" : "3", "_index" : "test"} }
{ "doc" : {"mobile" : "777"} }
```

Update By Query API
```

```


Delete by query
```
POST /test/_delete_by_query
{
  "query": {
    "match": {
      "name": "张三"
    }
  }
}
```
Reindex 将文档从一个索引复制到另一个索引
```
POST _reindex
{
  "source": {
    "index": "test",
    "query": {
      "match": {
        "name": "tt"
      }
    }
  },
  "dest": {
    "index": "test_dest",
    "routing": "=shenzhen"
  }
}
```