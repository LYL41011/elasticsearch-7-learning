# 内置Analyze
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html
### Standard Analyzer
The standard analyzer divides text into terms on word boundaries, as defined by the Unicode Text Segmentation algorithm. 
去除标点、小写、支持去除停用词
The standard analyzer consists of:
- Tokenizer：Standard Tokenizer
- Token Filters：Lower Case Token Filter、Stop Token Filter (disabled by default)


### Simple Analyzer
The simple analyzer divides text into terms whenever it encounters a character which is not a letter. 
It lowercases all terms.
按照非字母切分（符号被过滤），小写处理

### Whitespace Analyzer
The whitespace analyzer divides text into terms whenever it encounters any whitespace character. 
It does not lowercase terms.
按照空格切分，不转小写

### Stop Analyzer
The stop analyzer is like the simple analyzer, but also supports removal of stop words.

### Keyword Analyzer
The keyword analyzer is a “noop” analyzer that accepts whatever text it is given and outputs the exact same text as a single term.
不分词，直接将输入当作输出

### Pattern Analyzer
The pattern analyzer uses a regular expression to split the text into terms. 
It supports lower-casing and stop words.
正则表达式，默认 \W+ (非字符分隔) 支持小写和停顿词

### Language Analyzers
Elasticsearch provides many language-specific analyzers like english or french.
提供了30多种常见语言的分词器

### Fingerprint Analyzer
The fingerprint analyzer is a specialist analyzer which creates a fingerprint which can be used for duplicate detection.

```
# Standard Analyzer 标准分词器 按词切分 小写处理
GET _analyze
{
  "analyzer": "standard",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

#Stop Analyzer 停用词被去掉了
GET _analyze
{
  "analyzer": "stop",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# Language Analyzers running变成了run
GET _analyze
{
  "analyzer": "english",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

```
