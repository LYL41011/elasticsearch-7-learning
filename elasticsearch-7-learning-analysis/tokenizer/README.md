
# Tokenizer 分词器
参考:https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html

Word Oriented Tokenizers 以下分词器通常用于将全文分词成单个单词:
### Standard Tokenizer 
标准分词器，以单词边界作为切割，根据Unicode文本分割算法，适合于大部分语言
`"The 2 QUICK Brown-Foxes jumped over the lazy dog's bone." --> [ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]`
### Letter Tokenizer 
将文本按非字符(non-letter)进行分词 适合大部分欧洲语言 对亚洲语言就糟糕了 因为无空格
`"The 2 QUICK Brown-Foxes jumped over the lazy dog's bone." --> [ The, QUICK, Brown, Foxes, jumped, over, the, lazy, dog, s, bone ]`
### Lowercase Tokenizer 
先用Letter Tokenizer分词，然后再把分词结果全部换成小写格式
### Whitespace Tokenizer 
将文本通过空格进行分词
### UAX URL Email Tokenizer 
和standard 类型十分类似，但是会把email和url当作一个词
### Classic Tokenizer 
classic分词器提供基于语法的分词器，这对英语文档是一个很好的分词器。
`"The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."-->[ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]`

Partial Word Tokenizers 这些分词器将文本或单词分解成小片段，以进行部分单词匹配:
### N-gram tokenizer
### Edge n-gram tokenizer

Structured text Tokenizers 与结构化文本一起使用，如标识符、电子邮件地址、邮政编码和路径，而不是与全文一起使用:
### Keyword Tokenizer  
将一整块的输入数据作为一个单独的分词 即不分词
```
POST _analyze
{
  "tokenizer": "keyword",
  "filter": [ "lowercase" ],
  "text": "john.SMITH@example.COM"
}
结果[ john.smith@example.com ]
```
### Pattern Tokenizer 
正则表达式分词 比如指定以,分隔
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
### Simple Pattern Tokenizer | Simple Pattern Split Tokenizer 
可以自行对比下结果
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

### Char Group Tokenizer 
根据指定字符分隔 相比Pattern Tokenizer开销会小很多
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
### Path Tokenizer 
路径层次分词器 
`/foo/bar/baz → [/foo, /foo/bar, /foo/bar/baz ]`

