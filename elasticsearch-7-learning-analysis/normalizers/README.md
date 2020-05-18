# Normalizers
Normalizers和Analyzers类似，除了它只接受单一的token
也就是说 它不会有Tokenizer的存在，因为它的输入本来就是一个整体
可以解决这类问题："我不需要分词，但我需要过滤一些非法字符"
 

到目前为止，Elasticsearch还没有内置的Normalizers，所以获得一个Normalizers的唯一方法是自定义。
自定义Normalizers接受一个字符过滤器列表和一个token过滤器列表。
- Character Filter：字符过滤器接收原始文本作为字符流，并可以通过添加、删除或更改字符来转换流。比如将文本中html标签剔除掉。
- Token Filter：将切分的单词进行加工，小写，删除 stopwords(停顿词，a、an、the、is等),增加同义词

# 自定义
```
PUT index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "quote": {
          "type": "mapping",
          "mappings": [
            "« => \"",
            "» => \""
          ]
        }
      },
      "normalizer": {
        "my_normalizer": {
          "type": "custom",
          "char_filter": ["quote"],
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "foo": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      }
    }
  }
}
```