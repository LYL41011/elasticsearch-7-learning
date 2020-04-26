# 索引API 用于管理单个索引、索引设置、别名、映射和索引模板
参考文档:https://www.elastic.co/guide/en/elasticsearch/reference/7.6/indices.html

## Index management
Create index
```
1、显示指定mapping
注意keyword和text的区别 text类型会分词 而keyword不会 一个字段是可以同时设置这两种
"ignore_above":10 长度超过ignore_above设置的字符串将不被索引或存储
date:format属性 通过format设置日期格式，常见的可以设置成年月日时分秒、年月日及毫秒值三种格式。
date:ignore_malformed属性 默认值false。如果为true，则忽略格式错误的数字。如果为false则格式错误的数字将引发异常并拒绝整个文档。
"index":false表示不会被索引

PUT /test
{
  "settings":{
    "number_of_shards" : 3,
    "number_of_replicas" : 2
  },
  "mappings":{
    "properties":{
      "name":{
        "type":"keyword"
      },
      "address":{
        "type":"text",
        "fields":{
          "keyword":{
            "type":"keyword",
            "ignore_above":10
          }
        }
      },
      "birthday": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
        "ignore_malformed": true
      },
      "mobile":{
        "type":"text",
        "index":false
      }
    }
  }
}

2、动态mapping 即直接插入一条数据 es会自动创建索引并且自动匹配字段的映射
PUT test1/_doc/1
{
  "user": "GB",
  "uid": 1,
  "city": "Beijing",
  "province": "Beijing",
  "country": "China"
}
GET test1/_mapping
```

Delete index
`DELETE /twitter`

Get index 返回索引的信息
`GET /twitter`

Index exists
`HEAD /twitter`

Close index 关闭的索引不允许读写，也不允许open index具有的一切操作
`POST /twitter/_close`

Open index
`POST /twitter/_open`

Shrink index 缩小主分片个数 需要满足一定条件才可 参考https://www.elastic.co/guide/en/elasticsearch/reference/7.6/indices-shrink-index.html#shrink-index-api-prereqs
主要用途：人们会期望在多分片快速写入数据以后，把老数据合并存储，节约过多的 cluster state 容量
`POST /twitter/_shrink/shrunk-twitter-index`

Split index 扩大主分片个数
```
POST /twitter/_split/split-twitter-index
{
  "settings": {
    "index.number_of_shards": 2
  }
}
```

Clone index
`POST /twitter/_clone/cloned-twitter-index`

Rollover index
Freeze index
Unfreeze index

## Mapping management
Put mapping
```
新增字段
PUT /twitter/_mapping
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}
对于已经存在的字段，不能更改数据类型，比如从keyword更改为text，会报错mapper [email] of different type, current_type [keyword], merged_type [text]
只能更改以下参数：https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html
例如更改ignore_above的值：
PUT /test/_mapping
{
  "properties": {
    "email": {
      "type": "keyword",
      "ignore_above":50
    }
  }
}
```

Get mapping
`GET /test/_mapping`
Get field mapping
`GET /test/_mapping/field/email`


## Alias management 
索引别名是用于引用一个或多个索引的逻辑名称，大多数Elasticsearch api接受索引别名来代替索引名。
Elasticsearch的别名，就类似数据库的视图。别名不仅仅可以关联一个索引，它能聚合多个索引
参考文档:https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-aliases.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-add-alias.html#alias-adding

Add index alias：创建或更新索引别名,可以再创建index的时候就指定，也可以后续再指定
```
PUT /test/_alias/test_alias
GET /test_alias/_mapping/
# 聚合作用：我们为索引test1 和test 创建同一个别名test_alias，这样对test_alias的操作(仅限读操作)，会操作test和test1，类似于聚合了test和test1
PUT /test1/_alias/test_alias
GET /test_alias/_search

# 过滤作用：创建filtered的别名 例如对于同一个index，我们给不同人看到不同的数据，my_index有个字段是team，team字段记录了该数据是那个team的。team之间的数据是不可见的。
#在创建索引的时候即指定别名
PUT /my_index
{
    "mappings" : {
        "properties" : {
            "team" : {"type" : "keyword"}
        }
    },
    "aliases" : {
        "my_index__teamA_alias" : {
            "filter" : {
                "term" : {"team":"teamA"}
            }
        },
        "my_index__teamB_alias" : {
            "filter" : {
                "term" : {"team":"teamB"}
            }
        }
    }
}

PUT my_index/_doc/1
{"team":"teamA"}
PUT my_index/_doc/2
{"team":"teamB"}

GET /my_index__teamA_alias/_search
GET /my_index__teamB_alias/_search

#路由作用:Routing 可以将路由值与别名相关联。此功能可以与过滤器别名一起使用，以避免不必要的分片操作。               
PUT /users/_alias/user_12
{
    "routing" : "12",
    "filter" : {
        "term" : {
            "user_id" : 12
        }
    }
}
```
Delete index alias
`DELETE /test/_alias/test_alias`
Get index alias
```
GET /_alias/
GET /_alias/test_alias
```
Index alias exists
`HEAD /_alias/test_alias`
Update index alias
```
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "test_alias" } },
        { "add" : { "index" : "test1", "alias" : "test_alias1" } }
    ]
}
```

## Index settings
index配置详细参考:https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-modules-settings
Update index settings 实时生效
```
PUT /twitter/_settings
{
    "index" : {
        "number_of_replicas" : 2
    }
}
```
Get index settings
`GET /twitter/_settings`
Analyze
```
GET /_analyze
{
  "analyzer" : "standard",
  "text" : "Quick Brown Foxes!"
}
```


## Index templates
当新建一个 Elasticsearch 索引时，自动匹配模板，完成索引的基础部分搭建。
Put index template
```
#创建两个索引模板
PUT _template/template_default
{
  "index_patterns": ["*"],
  "order" : 0,
  "version": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas":1
  }
}

#template_test模板会忽略日期检测、打开数字自动检测
PUT /_template/template_test
{
    "index_patterns" : ["test*"],
    "order" : 1,
    "settings" : {
    	"number_of_shards": 1,
        "number_of_replicas" : 2
    },
    "mappings" : {
    	"date_detection": false,
    	"numeric_detection": true
    }
}

# PUT插入数据 自动创建索引 会匹配到上述索引模板
PUT testtemplate/_doc/1
{
	"someNumber":"1",
	"someDate":"2019/01/01"
}


PUT lyltemplate/_doc/1
{
	"someNumber":"1",
	"someDate":"2019/01/01"
}
# 检验下是否生效 注意观察someNumber和someDate的type
GET testtemplate/_mapping
GET lyltemplate/_mapping
```

Delete index template
`DELETE /_template/template_1`

Get index template
`GET /_template/template_default`

Index template exists
`HEAD /_template/template_1`

## Monitoring
Index stats 返回索引的统计信息
`GET /test/_stats`

Index segments 返回关于索引分片中的Lucene索引段信息
`GET /test/_segments`

Index recovery 获取正在进行和完成的分片恢复信息
`GET /test/_recovery`

Index shard stores 返回索引中replica shard的存储信息
`GET /test/_shard_stores`

## Status management
Clear cache
Refresh
Flush
Synced flush
Force merge
