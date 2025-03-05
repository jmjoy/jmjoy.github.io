+++
date = '2025-02-28T17:26:36+08:00'
title = 'Linux下如何打开core dump，并将core文件保存在临时目录下？'
categories = ["编程技术"]
tags = ["Linux"]
+++

> 内容由[ChatGPT](https://chatgpt.com/)生成。

## 1. 配置 ulimit 开启 core dump

默认情况下，shell 可能会限制 core dump 文件的大小。为了确保程序崩溃时能够生成 core 文件，需要取消这个限制。你可以在当前 shell 中执行：

```bash
ulimit -c unlimited
```

**永久生效方法：**

- **用户级别：** 将上述命令添加到用户的 `~/.bashrc` 或 `~/.profile` 文件中。
- **系统级别：** 编辑 `/etc/security/limits.conf` 文件，添加以下两行（针对所有用户）：

  ```plaintext
  * soft core unlimited
  * hard core unlimited
  ```

这样设置后，用户在启动新 shell 时会自动启用 core dump 的生成。

---

## 2. 使用 sysctl 设置 core dump 保存路径和命名格式

Linux 内核通过 `/proc/sys/kernel/core_pattern` 控制 core 文件的生成位置和文件名格式。为了将 core 文件保存到 `/tmp` 目录，并包含程序名、进程 ID 及时间戳，可以采用以下两种方式：

### (1) 临时修改（立即生效）

直接使用 sysctl 命令：

```bash
sudo sysctl -w kernel.core_pattern="/tmp/core.%e.%p.%t"
```

其中：

- **%e** 表示崩溃程序的可执行文件名，
- **%p** 表示进程ID，
- **%t** 表示崩溃时的时间戳。

### (2) 永久配置

为了在系统重启后依然生效，需要将设置写入 sysctl 配置文件：

1. 编辑 `/etc/sysctl.conf` 文件或在 `/etc/sysctl.d/` 目录下新建一个配置文件（例如 `99-core-pattern.conf`），添加以下行：

   ```plaintext
   kernel.core_pattern = /tmp/core.%e.%p.%t
   ```

2. 保存文件后，运行以下命令使配置生效：

   ```bash
   sudo sysctl -p
   ```

---

## 注意事项

- **systemd-coredump：** 某些采用 systemd 的发行版可能使用 systemd-coredump 来管理 core 文件，此时 core 文件可能不会直接生成在 `/tmp` 目录。你可以查看系统文档或调整 systemd 的相关配置来改变这一行为。
- **特殊权限程序：** 对于 setuid 程序或其他安全要求较高的程序，系统可能会禁用 core dump 以防敏感信息泄露。
- **验证配置：** 可通过以下命令验证当前 core_pattern 设置是否正确：

  ```bash
  cat /proc/sys/kernel/core_pattern
  ```
