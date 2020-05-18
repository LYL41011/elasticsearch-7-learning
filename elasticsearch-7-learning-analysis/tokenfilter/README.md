# Token Filter：将切分的单词进行加工，小写，删除 stopwords(停顿词，a、an、the、is等),增加同义词

### Character Filter
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html

- HTML Strip Char Filter
html过滤器 比如过滤掉<b> &amp;之类的html元素
- Mapping Char Filter
指定字符替换成指定字符
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "٠ => 0",
            "١ => 1",
            "٢ => 2",
            "٣ => 3",
            "٤ => 4",
            "٥ => 5",
            "٦ => 6",
            "٧ => 7",
            "٨ => 8",
            "٩ => 9"
          ]
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "My license plate is ٢٥٠١٥"
}
```
- Pattern Replace Char Filter
JAVA正则表达式替换
```
        "my_char_filter": {
          "type": "pattern_replace",
          "pattern": "(\\d+)-(?=\\d)",
          "replacement": "$1_"
        }
```

### Tokenizer
参考:https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html

Word Oriented Tokenizers 以下分词器通常用于将全文分词成单个单词:
- Standard Tokenizer 标准分词器，以单词边界作为切割，根据Unicode文本分割算法，适合于大部分语言
`"The 2 QUICK Brown-Foxes jumped over the lazy dog's bone." --> [ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]`
- Letter Tokenizer 将文本按非字符(non-letter)进行分词 适合大部分欧洲语言 对亚洲语言就糟糕了 因为无空格
`"The 2 QUICK Brown-Foxes jumped over the lazy dog's bone." --> [ The, QUICK, Brown, Foxes, jumped, over, the, lazy, dog, s, bone ]`
- Lowercase Tokenizer 先用Letter Tokenizer分词，然后再把分词结果全部换成小写格式
- Whitespace Tokenizer 将文本通过空格进行分词
- UAX URL Email Tokenizer 和standard 类型十分类似，但是会把email和url当作一个词
- Classic Tokenizer classic分词器提供基于语法的分词器，这对英语文档是一个很好的分词器。
`"The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."-->[ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]`

Partial Word Tokenizers 这些分词器将文本或单词分解成小片段，以进行部分单词匹配:
- N-gram tokenizer
- Edge n-gram tokenizer

Structured text Tokenizers 与结构化文本一起使用，如标识符、电子邮件地址、邮政编码和路径，而不是与全文一起使用:
- Keyword Tokenizer  将一整块的输入数据作为一个单独的分词 即不分词
```
POST _analyze
{
  "tokenizer": "keyword",
  "filter": [ "lowercase" ],
  "text": "john.SMITH@example.COM"
}
结果[ john.smith@example.com ]
```
- Pattern Tokenizer 正则表达式分词 比如指定以,分隔
```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "pattern",
          "pattern": ","
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "comma,separated,values"
}
```
- Simple Pattern Tokenizer | Simple Pattern Split Tokenizer 可以对比下结果
```
POST _analyze
 {
   "tokenizer": {
     "type": "simple_pattern",
     "pattern": "-"
   },
   "text": "fd-786-335-514-x"
 }
```

- Char Group Tokenizer 根据指定字符分隔 相比Pattern Tokenizer开销会小很多
```
POST _analyze
{
  "tokenizer": {
    "type": "char_group",
    "tokenize_on_chars": [
      "whitespace",
      "-",
      ","
    ]
  },
  "text": "The QUICK brown-fox,Good"
}
```
- Path Tokenizer 路径层次分词器 
`/foo/bar/baz → [/foo, /foo/bar, /foo/bar/baz ]`


### Token filters  
Tokenizer是将一句话分成一个个词 而Token filters是将这些词再进行一定规则的过滤
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html

- Classic 将英语中's从单词的末尾移除，并移除.号
```
GET /_analyze
{
  "tokenizer" : "classic",
  "filter" : ["classic"],
  "text" : "The 2 Q.U.I.C.K. Brown-Foxes jumped over the lazy dog's bone."
}
```
- Length length用于去掉过长或者过短的单词
```
GET _analyze
{
  "tokenizer" : "standard",
  "filter": [{"type": "length", "min":1, "max":3 }],  
  "text" : "this is a test"
}
```
- Lowercase 将所有字母转换为小写字母
- Uppercase 将所有字母转换为大写字母
- Stop 移除停用词, 比如"a"、"the"等
- Stemmer 词干提取器 foxes-->fox
https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html
https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis-stemmer-tokenfilter.html
https://www.elastic.co/guide/cn/elasticsearch/guide/current/choosing-a-stemmer.html
https://www.elastic.co/guide/cn/elasticsearch/guide/current/controlling-stemming.html

还有很多其他的可以参考文档去看
- Apostrophe
- ASCII folding
- CJK bigram
- CJK width
- Common grams
- Conditional
- Decimal digit
- Delimited payload
- Dictionary decompounder
- Edge n-gram
- Elision
- Fingerprint
- Flatten graph
- Hunspell
- Hyphenation decompounder
- Keep types
- Keep words
- Keyword marker
- Keyword repeat
- KStem
- Limit token count
- MinHash
- Multiplexer
- N-gram
- Normalization
- Pattern capture
- Pattern replace
- Phonetic
- Porter stem
- Predicate script
- Remove duplicates
- Reverse
- Shingle
- Snowball
- Stemmer override
- Synonym
- Synonym graph
- Trim
- Truncate
- Unique
- Word delimiter
- Word delimiter graph

我们可以用analyze api 组合三要素char_filter\tokenizer\filter 输出分析后的效果 便于我们做出合适选择
```
GET /_analyze
{
  "tokenizer" : "keyword",
  "filter" : ["lowercase"],
  "char_filter" : ["html_strip"],
  "text" : "this is a <b>test</b>"
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
1、内置Analyze\自定义Analyze
2、可以为字段指定\为索引指定
3、允许为索引和查询使用不同的Analyzer

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

