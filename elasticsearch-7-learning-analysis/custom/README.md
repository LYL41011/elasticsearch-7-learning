# 自定义Analyzer 
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html
当内置的分析仪不能满足你的需要，你可以创建一个自定义Analyzer，需要满足
- zero or more character filters
- a tokenizer
- zero or more token filters.

自定义Analyzer首先应该了解三要素character filters、tokenizer、token filters分别有哪些，以及它们的参数
具体参考本模块的characterfilter目录 tokenizer目录 tokenfilter目录

举例，我给索引创建了一个自定义Analyzer，
1、char_filter是html_strip,其中我设置了escaped_tags参数
2、tokenizer是standard tokenizer
3、token filters有两个，一个是自带的lowercase filters，还有一个是自定义的my_stopwords，它是基于stop filters,只不过指定了stopwords参数
 
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
```