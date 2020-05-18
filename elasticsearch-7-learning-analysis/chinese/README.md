# 中文分词
中文分词在所有搜索引擎中都是一个很大的难点，
中文的句子应该是切分成一个个的词，但是一句中文，在不同的上下文，其实是不同的理解，
例如: 这个苹果，不大好吃/这个苹果，不大，好吃。

## 相关资源
- Elasticsearch IK分词插件 https://github.com/medcl/elasticsearch-analysis-ik/releases
- Elasticsearch hanlp 分词插件 https://github.com/KennFalcon/elasticsearch-analysis-hanlp
- 分词算法综述 https://zhuanlan.zhihu.com/p/50444885

### 一些分词工具，供参考：
- 中科院计算所NLPIR http://ictclas.nlpir.org/nlpir/
- ansj分词器 https://github.com/NLPchina/ansj_seg
- 哈工大的LTP https://github.com/HIT-SCIR/ltp
- 清华大学THULAC https://github.com/thunlp/THULAC
- 斯坦福分词器 https://nlp.stanford.edu/software/segmenter.shtml
- Hanlp分词器 https://github.com/hankcs/HanLP
- 结巴分词 https://github.com/yanyiwu/cppjieba
- KCWS分词器(字嵌入+Bi-LSTM+CRF) https://github.com/koth/kcws
- ZPar https://github.com/frcchang/zpar/releases
- IKAnalyzer https://github.com/wks/ik-analyzer

### IK分词器案例
有一些比较不错的中文分词插件:IK、THULAC等。我们可以试试用IK进行中文分词。
```
#安装插件 集群里每一个实例都要安装ik插件。 否则，当我们更新包含指定分词的mapping的时候会报错。
https://github.com/medcl/elasticsearch-analysis-ik/releases
在plugins目录下创建analysis-ik目录 解压zip包到当前目录 重启ES
#查看插件
bin/elasticsearch-plugin list
#查看安装的插件
GET http://localhost:9200/_cat/plugins?v
```
IK分词器：支持自定义词库、支持热更新分词字典
- ik_max_word: 会将文本做最细粒度的拆分，比如会将“这个苹果不大好吃”拆分为"这个，苹果，不大好，不大，好吃"等，会穷尽各种可能的组合；
- ik_smart: 会做最粗粒度的拆分，比如会将“这个苹果不大好吃”拆分为"这个，苹果，不大，好吃"

```
curl -X GET "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
{
  "analyzer" : "ik_max_word",
  "text" : "这个苹果不大好吃"
}
'
```