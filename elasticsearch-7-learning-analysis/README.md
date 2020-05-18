# Analyze 对文本字符串执行分析并返回结果标记。
参考:https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html
# 基本概念
analysis：
  文本分析，是将全文本转换为一系列单词的过程，也叫分词。analysis是通过analyzer(分词器)来实现的，可以使用Elasticearch内置的分词器，也可以自己去定制一些分词器。
Analyzer(分词器)——无论是内置的还是自定义的，都包含如下三个方面：

由三部分组成：

- Character Filter：字符过滤器接收原始文本作为字符流，并可以通过添加、删除或更改字符来转换流。比如将文本中html标签剔除掉。
- Tokenizer：按照规则进行分词，比如whitespace分词器按照空格分词
- Token Filter：将切分的单词进行加工，小写，删除 stopwords(停顿词，a、an、the、is等),增加同义词

具体参考本模块的characterfilter目录 tokenizer目录 tokenfilter目录
我们可以用analyze api 组合三要素char_filter\tokenizer\filter 输出分析后的效果 便于我们做出合适选择

#测试Analyze
```
# 组合三要素
POST _analyze
{
  "tokenizer" : "keyword",
  "filter" : ["lowercase"],
  "char_filter" : ["html_strip"],
  "text" : "this is a <b>test</b>"
}
# 测试指定的Analyzer
POST _analyze
{
  "analyzer": "whitespace",
  "text":     "The quick brown fox."
}
# 也可以给索引创建一个analyzer 然后指定索引和filed
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_folded": { 
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_text": {
        "type": "text",
        "analyzer": "std_folded" 
      }
    }
  }
}

GET my_index/_analyze 
{
  "field": "my_text", 
  "text":  "Is this déjà vu?"
}
```

# 三个疑惑问题
1、什么字段类型会进行analysis？？
Elasticsearch在索引或搜索text字段时会执行analysis。如果索引不包含text字段，则可以跳过本节。
2、analyzer发生在什么时候？
- index time：当文档被索引时，任何文本字段值都会被分析。
- search time(query time)：在text字段上运行full text search时，将分析查询字符串。
注意full text search，可以移步querydsl模块详细看full text query和term query的区别
3、index和search analyzer是如何协调工作的
在大多数情况下，index和search时使用的是相同的分析器。
但有时在索引和搜索时使用不同的分析程序是有意义的。为此，Elasticsearch允许您指定一个单独的搜索分析器。
```
PUT my_index8/_mapping/_doc
{
  "properties": {
    "title": {
        "type": "text",
        "analyzer": "my_ik_analyzer",
        "search_analyzer": "other_analyzer" 
    }
  }
}
```

# 如何使用Analyzer
我们可以为每个查询、每个字段、每个索引指定分词器。
- 1、内置Analyze\自定义Analyze
- 2、可以为字段指定\为索引指定
- 3、允许为索引和查询使用不同的Analyzer

Analyzer的使用顺序
1、首先选用字段mapping定义中指定的analyzer 
2、字段定义中没有指定analyzer，则选用 index settings中定义的名字为default 的analyzer。
3、如index setting中没有定义default分词器，则使用 standard analyzer.

```
# 创建索引的时候为索引设置一个自定义analyzer
PUT my_index1
{
  "settings": {
    "analysis": {
        "char_filter": {
            "my_char_filter": {
               "type": "html_strip",
                "escaped_tags": ["br"]
            }
        },
        "filter": {
            "my_stopwords": {
                "type": "stop",
                "stopwords": ["the", "a"]
            }
        },
        "analyzer": {
            "my_analyzer": {    
                "char_filter": ["my_char_filter"], 
                "tokenizer": "standard",
                "filter": ["lowercase", "my_stopwords"] 
            }
        }
    }
  }
}

# 为指定字段设置analyzer 注意content2指定了search_analyzer
PUT my_index1/_mapping/
{
  "properties": {
    "chinese": {
        "type": "text",
        "analyzer": "ik_max_word"
    },
    "content1": {
        "type": "text",
        "analyzer": "my_analyzer"
    },
    "content2": {
        "type": "text",
        "analyzer": "my_analyzer",
        "search_analyzer": "whitespace" 
    }
  }
}


POST /my_index1/_bulk
{ "index": { "_id": 1 }}
{ "chinese" : "公众号胖滚猪学编程","content1":"<b> This is <br> a Pig","content2":"<b> This is <br> a Pig","default":"<b> This is <br> a Pig" }
{ "index": { "_id": 2 }}
{ "chinese" : "漫画趣味编程","content1":"<b> This is <br> a Pig","content2":"<b> This is <br> a Pig","default":"<b> This is <br> a Pig"}


# 测试chinese字段使用了中文分词器
POST /my_index1/_search
{
  "query":{
    "match": {
      "chinese": "编程"
    }
  }
}
# 测试content1使用了自定义的分词器my_analyzer
POST /my_index1/_search
{
  "query":{
    "match": {
      "content1": "Pig"
    }
  }
}
# 测试content2 使用了自定义的分词器my_analyzer 但是搜索使用whitespace 所以搜不到
POST /my_index1/_search
{
  "query":{
    "match": {
      "content2": "Pig"
    }
  }
}
# 测试默认的分词器是standard分词器
POST /my_index1/_search
{
  "query":{
    "match": {
      "default": "Pig"
    }
  }
}
```

# Analyze API
四种请求方式 案例参考elasticsearch-7-learning-restapis
- GET /_analyze
- POST /_analyze
- GET /<index>/_analyze
- POST /<index>/_analyze

