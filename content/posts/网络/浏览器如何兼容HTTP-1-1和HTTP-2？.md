首先，如何兼容HTTP/1.1和HTTP/2？

分两种情况：

1. 对于h2 (HTTP2 TLS模式)，使用[ALPN](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation)。
1. 对于h2c (HTTP2非TLS模式)，使用[HTTP/1.1 Upgrade header](https://en.wikipedia.org/wiki/HTTP/1.1_Upgrade_header#Use_with_HTTP/2)。

而一般浏览器只支持h2 (HTTP2 TLS模式)，所以使用的是`ALPN`。
