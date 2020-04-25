# cat指令，方便用来查看系统状态和数据分布 
参考文档:https://www.elastic.co/guide/en/elasticsearch/reference/7.6/cat-fielddata.html 

- 该指令提供一个快照，反映当前节点有多少个分片（shard）以及用了多少磁盘空间（disk）。
```GET /_cat/allocation?v```
- 可返回ES目前集群的基本信息
```GET /_cat/indices```
```GET /_cat/indices/kibana*?v```
- 该指令可以获取当前集群中有多少个document
```GET /_cat/count/kibana_sample_data_ecommerce?v```
- 返回群集中每个数据节点上field当前使用的堆内存量
```GET /_cat/fielddata?v```
- 可返回ES目前集群的基本信息
```GET /_cat/health?v```
- 可返回ES目前集群master节点的基本信息
```GET /_cat/master?v```
- 可查看目前集群下所有节点信息
```GET /_cat/nodes?v```
- 返回在集群的每个节点上运行的插件列表
```GET /_cat/plugins?v```
- 返回集群中每个节点的线程池统计信息。返回的信息包括所有内置线程池和自定义线程池。
```GET /_cat/thread_pool?v```
- 节点shards的详细视图。它会告诉你它是主文档还是副本、文档的数量、它在磁盘上占用的字节以及它所在的节点。
```GET _cat/shards/kibana_sample_data_ecommerce?v```
- 该指令反应当前系统中，索引分片的恢复信息，包括正在进行的以及已经完成了的。恢复，指的是当节点添加或者减少时发生的数据移动造成的
```GET _cat/recovery?v```
- 查看正在挂起的任务接口
```GET /_cat/pending_tasks```
