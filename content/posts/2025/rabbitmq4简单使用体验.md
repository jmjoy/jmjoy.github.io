+++
date = '2025-03-14T13:32:48+08:00'
title = 'Rabbitmq4简单使用体验'
categories = ["基础架构"]
tags = ["rabbitmq"]
+++

偶然间发现`rabbitmq 4`更新了不少东西，最关键是分布式方案更现代化了，于是尝试搭建和试用了一下。

## 更新了啥？

ChatGPT回答：

> RabbitMQ 4.0 系列的正式生产版本起始于 2024 年 9 月（例如，4.0.1 于 2024 年 9 月 18 日发布），随后陆续推出了多个补丁版本（最新补丁版本 4.0.7 发布于 2025 年 2 月 26 日）。
> 
> 这次重大更新主要包括以下几点：
> 
> 1. **AMQP 1.0 成为核心协议**  
>    以前 AMQP 1.0 可能只是通过插件支持，但在 4.0 中，它已经被内置并始终启用，并且其实现效率得到了大幅提升（峰值吞吐量比 3.13.x 高出一倍以上）。
> 
> 2. **Khepri 完全受支持**  
>    Khepri 是 RabbitMQ 新引入的元数据存储后端，此前在 3.13.x 中作为实验性功能出现。现在它更加稳定、性能更优，问题处理也更顺畅，因此在 4.0 中已被标记为完全受支持，不再是未来升级的障碍。
> 
> 3. **经典队列镜像被移除**  
>    经过三年的弃用周期，经典队列镜像在 4.0 中被彻底移除。取而代之的是成熟的复制队列技术——仲裁队列和流，这有助于提高数据安全性并简化队列复制模型。
> 
> 这些更新旨在优化核心代理性能、降低维护旧代码的负担并推动 RabbitMQ 向更高的稳定性与效率发展。

我这边一直都在用旧版的`rabbitmq 3.7`，印象中rabbitmq的高可用性不如kafka那种天然的分布式方案，什么主从模式和镜像模式那都是很落后的东西了，`rabbitmq 4`确实让我眼前一亮（没去深究这些东西是哪个版本已经引入了）。

## 分布式集群部署方式

`Rabbitmq 4`集群支持挺多种服务发现机制，包括etcd和kubernetes等：

<https://www.rabbitmq.com/docs/clustering#cluster-formation-options>

部署文档看得人眼花缭乱，还不如直接问chatGPT清晰可见。

本地部署rabbitmq有些麻烦，还要另外安装个erlang的运行环境，我直接Docker跑起来。

假设啊，我这边有三台服务器，IP分别是`192.168.100.1`, `192.168.100.2`, `192.168.100.3`，最简单的手动使用rabbitmqctl搭建集群方式如下：

1. 首先三台机分别把容器跑起来：

   ```shell
   # 192.168.100.1
   docker run -d --hostname rabbit1 --name rabbitmq --network host -e RABBITMQ_ERLANG_COOKIE="mysecretcookie123" -v /data/rabbitmq:/var/lib/rabbitmq --add-host rabbit1:192.168.100.1 --add-host rabbit2:192.168.100.2 --add-host rabbit3:192.168.100.3 --security-opt seccomp=unconfined rabbitmq:4.0-management

   # 192.168.100.2
   docker run -d --hostname rabbit2 --name rabbitmq --network host -e RABBITMQ_ERLANG_COOKIE="mysecretcookie123" -v /data/rabbitmq:/var/lib/rabbitmq --add-host rabbit1:192.168.100.1 --add-host rabbit2:192.168.100.2 --add-host rabbit3:192.168.100.3 --security-opt seccomp=unconfined rabbitmq:4.0-management

   # 192.168.100.3
   docker run -d --hostname rabbit3 --name rabbitmq --network host -e RABBITMQ_ERLANG_COOKIE="mysecretcookie123" -v /data/rabbitmq:/var/lib/rabbitmq --add-host rabbit1:192.168.100.1 --add-host rabbit2:192.168.100.2 --add-host rabbit3:192.168.100.3 --security-opt seccomp=unconfined rabbitmq:4.0-management
   ```

   这里图省事给docker加了些host解析，生产环境最好在DNS上解析好宿主机的主机名，docker容器也使用宿主机的主机名。

2. 然后分别在后两个节点上运行以下命令：

   ```shell
   # 192.168.100.2 和 192.168.100.3 上运行
   docker exec -it rabbitmq rabbitmqctl stop_app
   docker exec -it rabbitmq rabbitmqctl reset
   docker exec -it rabbitmq rabbitmqctl join_cluster rabbit@rabbit1
   docker exec -it rabbitmq rabbitmqctl start_app
   ```

运行成功，打开management界面，查看到nodes信息：

![rabbitmq nodes](/images/posts/2025/rabbitmq-nodes.webp)

### 高可用队列

创建三种队列：

![rabbitmq queues](/images/posts/2025/rabbitmq-queues.webp)

可以发现，classic队列就是最原始的队列，不支持高可用，数据只存放在一个节点上。

而quorum队列和stream都是支持高可用的。

虽然quorum队列是高可用，但是高可用除了服务端高可用还有一个层面，那就是客户端的高可用，而无论是AMQP 0‑9‑1还是AMQP 1.0都没有规定客户端层面的高可用性，由客户端自由发挥。

> AMQP 0‑9‑1 和 AMQP 1.0 的规范本身主要关注消息格式、路由、确认等通信语义，并没有专门定义分布式高可用（HA）或者“服务器下线后通知客户端”等机制。
> 
> 例如，在 AMQP 0‑9‑1（如 RabbitMQ 的实现）中，高可用性通常是通过集群、镜像队列等机制在 Broker 层面实现的；而协议层只规定了基本的连接和通道管理。当服务器（Broker）下线时，客户端会因为 TCP 连接中断或心跳超时而发现连接异常，但并没有一个标准的“通知”消息专门告知这一事件。
> 
> 同样，AMQP 1.0 规范也没有内置针对分布式高可用的特殊通知机制。它定义了断开连接、Detach 帧等错误处理方式，但高可用、故障转移和重连的具体策略完全依赖于 Broker 的实现以及客户端库的处理逻辑。
> 
> 简单来说，分布式高可用、故障通知和自动重连等功能更多是由具体消息中间件（如 RabbitMQ、Apache Qpid、Azure Service Bus 等）的设计与实现来提供，而不是 AMQP 协议规范本身所规定的。
> 
> 如果需要在应用层实现类似“服务器下线通知客户端”的功能，通常需要借助 Broker 提供的扩展特性或者在客户端增加自定义的心跳和重连机制。

### 创建一个quorum队列并发送消息

声明队列时添加属性即可：`x-queue-type: quorum`。

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import java.util.HashMap;
import java.util.Map;

public class QuorumQueueSender {
    // 队列名称
    private final static String QUEUE_NAME = "hello-quorum";

    public static void main(String[] args) throws Exception {
        // 创建连接工厂并设置用户名和密码（根据实际情况调整）
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("guest");
        factory.setPassword("guest");

        // 设置多个节点的地址（这里的地址仅为示例，请替换为实际的节点地址）
        Address[] addresses = new Address[]{
            new Address("node1.example.com", 5672),
            new Address("node2.example.com", 5672),
            new Address("node3.example.com", 5672)
        };

        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            // 声明一个quorum类型的队列
            Map<String, Object> arguments = new HashMap<>();
            // 设置队列类型为 quorum
            arguments.put("x-queue-type", "quorum");

            // durable 设置为 true 以确保队列持久化
            channel.queueDeclare(QUEUE_NAME, true, false, false, arguments);

            // 待发送的消息
            String message = "Hello Quorum Queue!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

我看好像只有Java和C#对AMQP 1.0的支持比较成熟。


## Stream

需要开启插件支持，三个节点都要运行：

```shell
docker exec rabbitmq rabbitmq-plugins enable rabbitmq_stream rabbitmq_stream_management 
```

很高兴rabbitmq stream对Rust有官方支持，如果有对AMQP 1.0的官方支持那就更好了。

<https://www.rabbitmq.com/tutorials/tutorial-one-rust-stream>
