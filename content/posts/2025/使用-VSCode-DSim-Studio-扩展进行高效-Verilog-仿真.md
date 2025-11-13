+++
date = '2025-11-13T14:38:41+08:00'
title = '使用 VSCode DSim Studio 扩展进行高效 Verilog 仿真'
categories = ["硬件"]
tags = ["FPGA", "DSim"]
+++

DSim Studio 是一个强大的 VS Code 扩展，它集成了 DSim 仿真器，为 Verilog/SystemVerilog 硬件描述语言提供了一个便捷、高效的仿真和调试环境。本文将指导您如何使用 DSim Studio 对一个基于 Gowin GW2A 系列 FPGA 的 UART 示例项目进行仿真。

-----

## 🛠️ 第一步：安装 DSim Studio 扩展与 DSim 仿真器

### 1\. 安装 VS Code 扩展

首先，您需要在 VS Code 中安装 DSim Studio 扩展。

* **扩展名称：** `DSim Studio`
* **发布者：** Altair Engineering
* **安装链接：** [https://marketplace.visualstudio.com/items?itemName=AltairEngineering.dsim-studio](https://marketplace.visualstudio.com/items?itemName=AltairEngineering.dsim-studio)

### 2\. 下载与配置 DSim 仿真器

安装扩展后，请按照 Welcome 页面或扩展文档的指引，完成 DSim 仿真器的下载、注册和激活过程。

* **注册账号：** 访问 Altair 的网站进行注册。
* **下载 DSim：** 下载最新版本的 DSim（例如：2025.1 或更新版本）。
* **激活：** DSim Studio 扩展通常会引导您完成本地激活。请注意，**对于本地使用和学习，DSim 仿真器通常是免费的**。

确保您的系统路径中包含了 DSim 仿真器的可执行文件，或者在 DSim Studio 扩展设置中配置了正确的路径。

-----

## 📁 第二步：项目结构示例

我们以一个典型的 Gowin FPGA 项目 `uart_echo` 作为示例。项目结构如下所示：

```
.
├── src
│   ├── gowin_rpll
│   │   ├── gowin_rpll.ipc
│   │   ├── gowin_rpll.mod
│   │   ├── gowin_rpll_tmp.v
│   │   └── gowin_rpll.v 
│   ├── tb_uart_echo.v    // 测试平台 (Testbench)
│   ├── top_uart_echo.v   // 顶层模块
│   ├── uart_rx.v         // UART 接收模块
│   └── uart_tx.v         // UART 发送模块
├── uart_echo.gprj
├── uart_echo.gprj.user
```

其中，`tb_uart_echo.v` 是我们的测试平台，我们将从这里开始仿真。

-----

## ⚙️ 第三步：创建和配置 DSim 仿真项目

DSim Studio 通过生成一个名为 `uart_echo.dpf` 的配置文件来管理仿真设置。您可以通过扩展的命令面板（`Ctrl+Shift+P` 或 `Cmd+Shift+P`）来完成配置。

### 1\. 新建 DSim 项目

打开您的项目文件夹后，执行以下命令：

* **命令：** `DSim Studio: New Project`

这将在您的项目根目录下创建一个基本的 DSim 项目配置文件，通常名为 `project_name.dpf` (例如：`uart_echo.dpf`)。

### 2\. 配置仿真文件（`DSim Studio: Configure File`）

接下来，您需要告诉 DSim 哪些文件需要被编译：

* **命令：** `DSim Studio: Configure File`

通过图形界面或提示，将项目中的所有 Verilog 源文件添加进来：

* `src/tb_uart_echo.v` (Testbench)
* `src/top_uart_echo.v`
* `src/uart_rx.v`
* `src/uart_tx.v`
* `src/gowin_rpll/gowin_rpll.v`
* **注意：** 对于 Gowin FPGA，您还需要手动添加或确保仿真库文件被包含，例如 `prim_sim.v`。

### 3\. 配置仿真选项（`DSim Studio: Configure Simulation`）

配置仿真运行时所需的参数，例如顶层模块、仿真选项和波形文件等：

* **命令：** `DSim Studio: Configure Simulation`

设置的关键参数包括：

* **Simulation Name (仿真名称):** `tb_uart_echo`
* **Options (选项):** \* `-top work.tb_uart_echo`：指定顶层模块为 `tb_uart_echo`。
  * `+acc+b`：启用对所有信号的访问和断点支持。
  * `-waves waves.mxd`：指定生成的波形文件名和格式 (`.mxd` 是 DSim Studio 的波形文件格式)。

### 4\. 生成 `uart_echo.dpf` 配置文件

完成上述步骤后，系统将生成如下所示的 `uart_echo.dpf` 文件：

```yaml
---
# Note: The contents of this file are automatically generated.
# Any changes made by hand may be overwritten.
version: '0.2'
work_dir: .
design_root_dir: .
simulations:
  - name: tb_uart_echo
    options: '-top work.tb_uart_echo +acc+b -waves waves.mxd'
source_files:
  - language: verilog
    path: src/tb_uart_echo.v
  - language: verilog
    path: src/top_uart_echo.v
  - language: verilog
    path: /opt/Gowin/IDE/simlib/gw2a/prim_sim.v   # 示例：Gowin 仿真库路径
  - language: verilog
    path: src/gowin_rpll/gowin_rpll.v
library_search_paths:
  - $STD_LIBS/ieee93
  - /opt/Gowin/IDE/simlib/gw2a                  # 示例：库搜索路径
```

> **注意：** 其中的 `/opt/Gowin/IDE/simlib/gw2a` 路径需要根据您实际的 Gowin EDA 安装路径进行修改。

-----

## ▶️ 第四步：编译与运行仿真

### 1\. 编译项目

在 VS Code 侧边的 DSim Studio 面板（或使用命令面板），找到您的项目。

1. 展开 `Library Configuration` 部分。
2. 点击 **Compile project**（或类似的编译按钮）。

DSim 仿真器将读取 `.dpf` 文件，编译所有 `source_files` 中的 HDL 代码，并将编译后的结果存储在工作目录中。编译成功是进行仿真的前提。

### 2\. 运行仿真

编译成功后：

1. 在 DSim Studio 面板中，展开 `Simulations` 部分，找到您配置的仿真名称 `tb_uart_echo`。
2. 点击 `tb_uart_echo` 条目旁的 **Run** 按钮（通常是一个播放图标 ▶️）。

DSim 将使用 `.dpf` 文件中为 `tb_uart_echo` 配置的 `options` 来启动仿真。

### 3\. 查看波形文件

仿真运行结束后，DSim 会在项目工作目录下生成波形文件 `waves.mxd`。

* 在 DSim Studio 面板的相应位置（通常在 Run 结束后自动显示），点击生成的 **`waves.mxd`** 文件。
* DSim Studio 内嵌的波形查看器将打开，您可以分析信号的时序和逻辑，从而验证您的设计功能。

通过 DSim Studio，您可以直接在 VS Code 中完成从代码编辑、项目配置、编译、仿真运行到波形分析的全流程，极大地提高了 Verilog 开发的效率。
