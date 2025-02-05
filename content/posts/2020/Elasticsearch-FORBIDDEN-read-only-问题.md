---
title: "Elasticsearch FORBIDDEN read-only 问题"
date: 2020-12-15T16:26:37+08:00
categories: ["数据库"]
tags: []
---

类似报错：

{"error":{"root_cause":[{"type":"cluster_block_exception","reason":"blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"}],"type":"cluster_block_exception","reason":"blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"},"status":403}


临时解决：

curl -XPUT -H "Content-Type: application/json" http://127.0.0.1:9200/_all/_settings -d '{"index.blocks.read_only_allow_delete": null}'


实际解决还是要升级硬件。
