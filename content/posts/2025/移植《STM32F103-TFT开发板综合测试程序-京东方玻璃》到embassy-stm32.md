+++
date = '2025-05-04T11:43:57+08:00'
title = '移植《STM32F103 TFT开发板综合测试程序 京东方玻璃》到embassy-stm32'
categories = ["嵌入式"]
tags = ["stm32", "embassy"]
cover = "/images/posts/2025/stm32f103-tft.webp"
+++

五一假期闲来无事，决定做一件一直想尝试的事情：将`《STM32F103 TFT开发板综合测试程序 京东方玻璃》`这个示例程序移植到`embassy-stm32`平台。

整个过程大约花费了一天半的时间，期间充分利用了`GitHub Copilot`和`Claude 3.7 Sonnet`等AI工具，大大提升了开发效率。

原项目采用的是C语言，并基于`STM32`官方已不再维护的`SPL`（Standard Peripheral Library）库。

项目仓库地址： <https://github.com/jmjoy/stm32f103-tft-board-boe-suite>

### 移植效果展示

![stm32f103-tft](/images/posts/2025/stm32f103-tft.webp)

### 为什么选择`embassy`？

`embassy`是一个基于Rust语言的异步嵌入式开发框架，具有以下显著优势：

1. **全自动`wfe`（Wait For Event）机制，极致省电**  
   `embassy`自动管理低功耗等待，极大降低了能耗，非常适合对功耗敏感的嵌入式场景。
2. **基于Rust的`async/await`，原生支持多任务**  
   利用Rust的异步编程模型，可以轻松实现多任务并发，无需传统的RTOS（实时操作系统），简化了任务调度和资源管理。
3. **Rust所有权机制，杜绝外设配置错误**  
   Rust的类型系统和所有权机制能够在编译期捕获外设使用中的潜在错误，提升系统稳定性和安全性。
4. **丰富的`embassy`生态系统，简化嵌入式开发流程**  
   借助`embassy`及其配套库，可以方便地集成各种外设驱动和中间件，大幅提升开发效率。

---

强烈推荐嵌入式爱好者和从业者尝试使用`embassy`，体验Rust异步嵌入式开发的高效与安全！
