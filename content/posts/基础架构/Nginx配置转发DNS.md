Nginx配置转发DNS

配置如下：

```nginx
upstream dns_servers {
  server <UPSTREAM>  weight=2 max_fails=1  fail_timeout=5s;
}

server {
  listen 53  udp;
  listen 53; #tcp
  proxy_pass dns_servers;

  proxy_timeout 3s;
  proxy_responses 1;

  error_log <LOG_PATH> error;
}
```

比较重要的是必须设置`proxy_responses`和`proxy_timeout`，原因如下：

1. 由于UDP协议是无状态协议，UDP协议自身并没有会话保持机制，nginx于是定义了一个非常简单的维持机制：如果将proxy_responses设置成1，客户端每发出一个UDP报文，通常期待接收回一个报文响应，当然也有可能不响应或者需要多个报文响应一个请求，此时proxy_responses可配为其他值。而proxy_timeout则规定了在最长的等待时间内没有响应则断开会话。

2. 如果没有设置这两个参数，所以按默认的UDP连接会在10分钟后超时释放。在请求量较大的情况下，UDP的65535个端口被占用不释放，将会导致了频繁的服务不可用。

参考：
1.  [https://www.nginx.com/blog/load-balancing-dns-traffic-nginx-plus/](https://www.nginx.com/blog/load-balancing-dns-traffic-nginx-plus/)
2.  [http://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_responses](http://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_responses)
