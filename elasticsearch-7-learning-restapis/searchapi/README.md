# search API允许您执行搜索查询并返回与查询匹配的搜索结果。可以使用简单的查询字符串作为参数提供查询，也可以使用请求body。 
参考文档:https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html

可以分为两大类 URI Search和Request Body Search

- 简单的字符串查询 使用q=指定查询语句
```
# 查询指定索引 
GET /kibana_sample_data_ecommerce/_search?q=order_id:584677
# 正则
GET kibana*/_search?q=customer_first_name:Eddie
# 查询多索引
GET /kimchy,elasticsearch/_search?q=user:kimchy
# 查询集群中全部索引
GET /_search?q=user:kimchy

#带profile可以查看查询如何被执行 df指定字段
GET /movies/_search?q=2012&df=title
{
	"profile":"true"
}

#排序sort、from和size用于分页
GET /kibana_sample_data_ecommerce/_search?sort=order_date:desc&from=0&size=10&timeout=1s&_source=order_date

#_source返回指定字段
GET /kibana_sample_data_ecommerce/_search?q=order_id:584677&_source=user

# 泛查询 没有指定字段 所有字段符合2012都可 可以看一下profile输出的description 就知道会怎么查了
GET /movies/_search?q=2012
{
	"profile":"true"
}

# 或查询 含Beautiful或者含Mind都行 等效于Beautiful OR Mind
GET /movies/_search?q=title:Beautiful Mind
{
	"profile":"true"
}

#使用引号，Phrase查询 等效于Beautiful AND Mind
GET /movies/_search?q=title:"Beautiful Mind"
{
	"profile":"true"
}

#布尔操作符 查找美丽心灵
GET /movies/_search?q=title:(Beautiful AND Mind)
{
	"profile":"true"
}

GET /movies/_search?q=title:(Beautiful NOT Mind)
{
	"profile":"true"
}

#范围查询
GET /movies/_search?q=title:beautiful AND year:>2012
{
	"profile":"true"
}


#通配符查询 查询效率低 不建议使用 ？代表一个字符 *代表0或多个字符
GET /movies/_search?q=title:b*
{
	"profile":"true"
}

//模糊匹配&近似度匹配 其实beautif这个单词是不对的 差2个字母 但通过~2就可以近似匹配了
GET /movies/_search?q=title:beautif~2
{
	"profile":"true"
}

```

关于返回结果集合 重点关注
1、took 花费的时间
2、total 符合条件的总条数
3、hits 结果集 默认返回前10个 。包括_index 索引， _id 文档ID， _score 相关性评分， _source文档原始信息


- request body 支持POST和GET 使用查询DSL(详见queryDSL)定义搜索。




- search_shards 返回将对其执行搜索请求的索引和碎片
```
GET /twitter/_search_shards
```
