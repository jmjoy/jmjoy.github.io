文件： /ect/docker/daemon.json

```json
{
  "registry-mirrors": ["https://hub-mirror.c.163.com/"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {"max-size":"2G", "max-file": "3"},
  "graph": "/data/docker"
}
```
