查看cluster健康状态（颜色、分片等）：
http://127.0.0.1:9200/_cluster/health?pretty

查看node健康状态：
http://127.0.0.1:9200/_cat/nodes?v&pretty

查看node thread_pool配置：
http://127.0.0.1:9200/_nodes/thread_pool?pretty

查看节点写入rejected：
http://127.0.0.1:9200/_cat/thread_pool/write?v

查看index设置：
http://127.0.0.1:9200/_settings?pretty

查看线程池：
http://127.0.0.1:9200/_nodes/stats?pretty
