# Analyze 对文本字符串执行分析并返回结果标记。
关于Analyze的概念和分词器等内容 详见elasticsearch-7-learning-analysis模块

API四种请求方式
- GET /_analyze
- POST /_analyze
- GET /<index>/_analyze
- POST /<index>/_analyze

```
# GET和POST均可
# 指定analyzer进行分析，例如standard、stop、english等
GET _analyze
{
  "analyzer": "standard",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# 指定索引和字段
GET /movies/_analyze
{
  "field": "title",
  "text": "Saving Christmas"
}

# 自定义Analyzer
GET /_analyze
{
  "tokenizer" : "keyword",
  "filter" : ["lowercase"],
  "char_filter" : ["html_strip"],
  "text" : "this is a <b>test</b>"
}
GET /_analyze
{
  "tokenizer" : "whitespace",
  "filter" : ["lowercase", {"type": "stop", "stopwords": ["a", "is", "this"]}],
  "text" : "this is a test"
}

#希望详细输出信息 用explain 这样会输出tokens、tokenfilters等信息
GET /_analyze
{
  "tokenizer" : "standard",
  "filter" : ["lowercase","stop"],
  "text" : "Hello,My name is LiuYanLing",
  "explain" : true,
  "attributes" : ["keyword"] 
}
```
