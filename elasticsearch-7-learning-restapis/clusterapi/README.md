# 集群API 查看集群相关信息
参考文档:https://www.elastic.co/guide/en/elasticsearch/reference/7.6/cluster.html

- 返回集群的健康状态
```GET _cluster/health```
```GET _cluster/health?level=shards```

- 返回关于集群状态的元数据
```GET /_cluster/state/metadata,routing_table/kibana_sample_data_ecommerce```

- 返回集群统计数据
```GET /_cluster/stats/```

- 返回集群的设置信息
```GET /_cluster/settings```
```GET /_cluster/settings?include_defaults=true```

- 返回集群节点信息
```GET /_nodes```

- 返回节点统计信息
```GET /_nodes/stats/```
```GET /_nodes/stats/indices/indexing```

- 返回集群中每个选定节点上的热线程。
```GET /_nodes/hot_threads```

- 查看集群挂起的任务接口
```GET /_cluster/pending_tasks```


