+++
date = '2025-03-21T13:57:00+08:00'
title = 'PHP通过ProxySQL作为Sidecar获得连接池能力'
categories = ["编程技术"]
tags = ["PHP"]
+++

传统的 PHP 一直被诟病无法常驻内存，因此无法实现类似 MySQL 连接池这样对数据库性能有显著优化的功能。其实，要实现连接池功能并不困难，可以通过外挂连接池来解决。

> 以下内容由Claude生成。

## 概述

实现的架构：

1. ProxySQL 作为一个 sidecar（边车模式）部署
2. PHP 通过 Unix Domain Socket 连接到 ProxySQL
3. ProxySQL 再连接到 MySQL 数据库

这种架构有以下优势：

- 减少网络开销（通过使用 Unix Domain Socket）
- 提供连接池功能
- 实现查询缓存和路由
- 增强安全性和可靠性

下面我将详细介绍配置步骤。

## 1. 部署架构

我将提供两种部署方式：Docker 容器化部署和传统部署。

### Docker 容器化部署

```dockerfile name=Dockerfile
FROM php:8.1-fpm

# 安装必要的依赖
RUN apt-get update && apt-get install -y \
    wget \
    gnupg2 \
    lsb-release \
    curl \
    supervisor \
    && rm -rf /var/lib/apt/lists/*

# 添加 ProxySQL 仓库并安装
RUN wget -O - 'https://repo.proxysql.com/ProxySQL/repo_pub_key' | apt-key add - \
    && echo deb https://repo.proxysql.com/ProxySQL/proxysql-2.5.x/$(lsb_release -sc)/ ./ \
        > /etc/apt/sources.list.d/proxysql.list \
    && apt-get update \
    && apt-get install -y proxysql

# 安装 PHP MySQL 扩展
RUN docker-php-ext-install mysqli pdo_mysql

# 配置目录
RUN mkdir -p /var/lib/proxysql \
    && mkdir -p /var/run/proxysql

# 复制配置文件
COPY proxysql.cnf /etc/proxysql.cnf
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# 暴露端口 (可选，如果需要从外部访问)
EXPOSE 9000

# 使用 supervisor 启动服务
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

```ini name=supervisord.conf
[supervisord]
nodaemon=true
user=root
logfile=/var/log/supervisor/supervisord.log
pidfile=/var/run/supervisord.pid

[program:php-fpm]
command=php-fpm
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:proxysql]
command=/usr/bin/proxysql -c /etc/proxysql.cnf
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

```cnf name=proxysql.cnf
datadir="/var/lib/proxysql"
errorlog="/var/lib/proxysql/proxysql.log"

admin_variables=
{
    admin_credentials="admin:admin"
    mysql_ifaces="127.0.0.1:6032"
    refresh_interval=2000
}

mysql_variables=
{
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033;/var/run/proxysql/proxysql.sock"
    default_schema="information_schema"
    stacksize=1048576
    server_version="8.0.26"
    connect_timeout_server=3000
    monitor_username="monitor"
    monitor_password="monitor"
    monitor_history=600000
    monitor_connect_interval=60000
    monitor_ping_interval=10000
    monitor_read_only_interval=1500
    monitor_read_only_timeout=500
    ping_interval_server_msec=120000
    ping_timeout_server=500
    commands_stats=true
    sessions_sort=true
    connect_retries_on_failure=10
}

# 定义 MySQL 服务器
mysql_servers =
(
    {
        address="mysql_host" # 替换为实际的 MySQL 主机地址
        port=3306
        hostgroup=0
        status="ONLINE"
        weight=1
        compression=0
        max_connections=1000
        max_replication_lag=0
    }
)

# 定义 MySQL 用户
mysql_users =
(
    {
        username="mysql_user" # 替换为实际的 MySQL 用户
        password="mysql_password" # 替换为实际的 MySQL 密码
        default_hostgroup=0
        max_connections=1000
        default_schema="your_database" # 替换为实际的数据库名
        active=1
    }
)

# 定义查询规则
mysql_query_rules =
(
    {
        rule_id=1
        active=1
        match_digest="^SELECT .* FOR UPDATE"
        destination_hostgroup=0
        apply=1
    },
    {
        rule_id=2
        active=1
        match_digest="^SELECT .*"
        destination_hostgroup=0
        apply=1
    }
)
```

### Docker Compose 示例

```yaml name=docker-compose.yml
version: '3'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./php:/var/www/html
    ports:
      - "9000:9000"
    environment:
      - DB_CONNECTION=mysql
      - DB_HOST=/var/run/proxysql/proxysql.sock
      - DB_PORT=
      - DB_DATABASE=your_database
      - DB_USERNAME=mysql_user
      - DB_PASSWORD=mysql_password
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_DATABASE=your_database
      - MYSQL_USER=mysql_user
      - MYSQL_PASSWORD=mysql_password
      - MYSQL_ROOT_PASSWORD=root_password
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

## 2. 传统部署（非容器化）

### 2.1 安装 ProxySQL

```bash
# Debian/Ubuntu
wget -O - 'https://repo.proxysql.com/ProxySQL/repo_pub_key' | apt-key add -
echo deb https://repo.proxysql.com/ProxySQL/proxysql-2.5.x/$(lsb_release -sc)/ ./ > /etc/apt/sources.list.d/proxysql.list
apt-get update
apt-get install proxysql

# CentOS/RHEL
yum install https://github.com/sysown/proxysql/releases/download/v2.5.5/proxysql-2.5.5-1-centos7.x86_64.rpm
```

### 2.2 配置 ProxySQL

创建或修改配置文件 `/etc/proxysql.cnf`，内容与前面提供的 `proxysql.cnf` 相同。

### 2.3 启动 ProxySQL

```bash
systemctl start proxysql
systemctl enable proxysql
```

## 3. PHP 配置

### 3.1 修改 PHP 连接配置

```php name=db_connection.php
<?php
// 使用 PDO 连接 ProxySQL (通过 Unix Socket)
$dsn = 'mysql:unix_socket=/var/run/proxysql/proxysql.sock;dbname=your_database';
$username = 'mysql_user';
$password = 'mysql_password';

try {
    $pdo = new PDO($dsn, $username, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false,
    ]);
    echo "连接成功！";
} catch (PDOException $e) {
    die("连接失败: " . $e->getMessage());
}

// 或者使用 mysqli
$mysqli = new mysqli();
$mysqli->real_connect(
    null,           // host (null 表示使用 socket)
    'mysql_user',   // username
    'mysql_password', // password
    'your_database',  // database
    null,           // port (null 因为使用 socket)
    '/var/run/proxysql/proxysql.sock' // socket path
);

if ($mysqli->connect_error) {
    die('连接错误 (' . $mysqli->connect_errno . ') ' . $mysqli->connect_error);
}
```

### 3.2 Laravel 配置 (如果使用 Laravel)

在 `.env` 文件中：

```
DB_CONNECTION=mysql
DB_HOST=/var/run/proxysql/proxysql.sock
DB_PORT=
DB_DATABASE=your_database
DB_USERNAME=mysql_user
DB_PASSWORD=mysql_password
```

在 `config/database.php` 文件中：

```php name=config/database.php
'mysql' => [
    'driver' => 'mysql',
    'unix_socket' => env('DB_HOST', ''),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    // 其他配置...
],
```

## 4. ProxySQL 管理

### 4.1 连接到 ProxySQL 管理接口

```bash
mysql -u admin -padmin -h 127.0.0.1 -P 6032 --prompt='ProxySQL Admin> '
```

### 4.2 常用管理命令

```sql
-- 查看配置的 MySQL 服务器
SELECT * FROM mysql_servers;

-- 查看配置的 MySQL 用户
SELECT * FROM mysql_users;

-- 查看查询规则
SELECT * FROM mysql_query_rules;

-- 应用配置更改
LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;

LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;

LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK;

-- 查看连接统计
SELECT * FROM stats.stats_mysql_connection_pool;
```

## 5. 监控与性能调优

### 5.1 查看连接状态

```sql
-- 在 ProxySQL 管理界面执行
SELECT * FROM stats.stats_mysql_connection_pool;
```

### 5.2 查看查询统计

```sql
-- 查看查询统计
SELECT * FROM stats.stats_mysql_query_digest ORDER BY sum_time DESC LIMIT 10;

-- 查看慢查询
SELECT * FROM stats.stats_mysql_query_digest WHERE avg_time > 1000000 ORDER BY avg_time DESC;
```

## 6. 故障排查

1. **检查 Socket 文件是否存在**

   ```bash
   ls -la /var/run/proxysql/proxysql.sock
   ```

2. **检查 Socket 权限**

   ```bash
   chmod 777 /var/run/proxysql/proxysql.sock
   chown www-data:www-data /var/run/proxysql/proxysql.sock
   ```

3. **检查 ProxySQL 日志**

   ```bash
   tail -f /var/lib/proxysql/proxysql.log
   ```

4. **测试 ProxySQL 到 MySQL 的连接**

   ```bash
   mysql -u mysql_user -pmysql_password -h 127.0.0.1 -P 6033 -e "SELECT 1"
   ```

5. **检查 PHP 错误日志**

   ```bash
   tail -f /var/log/php/error.log
   ```
