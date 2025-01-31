+++
date = '2025-01-31T22:55:00+08:00'
title = 'Y7000笔记本UOS安装NVIDIA驱动记录'
categories = ["装机"]
tags = ["UOS", "NVIDIA"]
+++

## 背景

2025年春节前夕，国内大模型DeepSeek横空出世，火爆全球，搞得我特想认真学AI。虽然手头只有台老古董联想拯救者Y7000，显卡还是`GTX 1050Ti`，跑大模型基本做梦，但好歹能折腾点基础的吧？

结果在UOS系统上被NVIDIA驱动卡住了。这系统使用内核自带的`nouveau`驱动根本不支持CUDA，必须装官方闭源驱动。我之前试了很多遍：用源里的`nvidia-driver`、试过`deepin-graphics-driver-manager`和`deepin-nvidia-installer`、直接跑官方的`.run`脚本……全跪了。

关键问题出在Y7000的HDMI口只连独显，而我上班必须双屏和开会投屏，之前就算装好驱动，HDMI用不了也算白搭。

趁着春节放假，我决定死磕到底。翻了一遍[deepin-graphics-driver-manager源码](https://github.com/linuxdeepin/deepin-graphics-driver-manager/blob/30b92866c6a295d70a067f81443ec56d2f4ce625/server/scripts/nvidia_intel/install_nvidia_intel_prime.sh)，又让DeepSeek和ChatGPT轮番上阵。最后是ChatGPT-o1给的方案最接近，但还得自己改几处参数。折腾一整天终于搞定了HDMI输出和CUDA，虽然到现在也没完全搞懂XOrg的底层原理……

## 步骤

虽然过程是曲折的，但是结果却是简单的，总结一下步骤：

1. [下载NVIDIA官方驱动](https://www.nvidia.com/en-in/drivers/)，直接使用最新的版本就行，我使用的版本是550。
2. 添加可执行权限后运行`./NVIDIA-Linux-*.run`脚本，DKMS和32位兼容等选项随意，XOrg那里选择不修改。
3. 添加`/etc/X11/xorg.conf.d/70-nvidia.conf`：

   ```text
   Section "ServerLayout"
       Identifier "layout"

       Screen 0 "nvidia"
       Inactive "intel"
   EndSection

   # Intel integrated graphics configuration
   Section "Device"
       Identifier "intel"
       Driver "modesetting"
       BusID "PCI:0:2:0"           # Modify according to the output of lspci
       Option "DRI" "3"
   EndSection

   Section "Screen"
       Identifier "intel"
       Device "intel"
   EndSection

   # NVIDIA dedicated graphics configuration
   Section "Device"
       Identifier "nvidia"
       Driver "nvidia"
       BusID "PCI:1:0:0"           # Modify according to the output of lspci
       Option "AllowEmptyInitialConfiguration"
       #Option "Coolbits" "4"      # Optional, used for certain NVIDIA GPU fan/OC adjustments
   EndSection

   Section "Screen"
       Identifier "nvidia"
       Device "nvidia"
   EndSection
   ```

然后就搞定了。

比较反直觉的是这两行：

```text
       Screen 0 "nvidia"
       Inactive "intel"
```

按道理来说，跟Intel集显相连的笔记本屏幕（存疑）应该作为`Screen 0`，而跟NVIDIA独显相连的HDMI应该是`Inactive`状态？

其实ChatGPT-o1生成的答案就是这样，但实际效果是HDMI连接的屏幕没有显示，`xrandr`也不输出HDMI相关的信息，非常恼人。我还调过只有HDMI连接的屏幕有显示而笔记本屏幕没有显示的情况。

然而在我胡乱折腾一通后，我突然发现对调一下居然行得通！还有应该在`lightdm`启动时设置的`xrandr`等命令在删除后也没影响。

## 效果

贴一些命令的输出：

- `xrandr`

  ```text
  Screen 0: minimum 8 x 8, current 3600 x 1080, maximum 32767 x 32767
  DP-0 disconnected (normal left inverted right x axis y axis)
  DP-1 disconnected (normal left inverted right x axis y axis)
  DP-2 disconnected (normal left inverted right x axis y axis)
  DP-3 disconnected (normal left inverted right x axis y axis)
  HDMI-0 connected 1680x1050+1920+0 (normal left inverted right x axis y axis) 474mm x 296mm
  ......
  eDP-1-1 connected primary 1920x1080+0+0 (normal left inverted right x axis y axis) 344mm x 194mm
  ......
  ```

- `xrandr --listmonitors`

   ```text
   Monitors: 2
    0: +*eDP-1-1 1920/344x1080/194+0+0  eDP-1-1
    1: +HDMI-0 1680/474x1050/296+1920+0  HDMI-0
   ```

- `xrandr --listproviders`
   
   ```text
   Providers: number : 2
   Provider 0: id: 0x248 cap: 0x1, Source Output crtcs: 4 outputs: 5 associated providers: 1 name:NVIDIA-0
   Provider 1: id: 0x43 cap: 0xf, Source Output, Sink Output, Source Offload, Sink Offload crtcs: 3 outputs: 1 associated providers: 1 name:modesetting
   ```

- `nvidia-smi`

   ```text
   +-----------------------------------------------------------------------------------------+
   | NVIDIA-SMI 550.144.03             Driver Version: 550.144.03     CUDA Version: 12.4     |
   |-----------------------------------------+------------------------+----------------------+
   | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
   |                                         |                        |               MIG M. |
   |=========================================+========================+======================|
   |   0  NVIDIA GeForce GTX 1050 Ti     Off |   00000000:01:00.0  On |                  N/A |
   | N/A   49C    P0             N/A / ERR!  |     377MiB /   4096MiB |      0%      Default |
   |                                         |                        |                  N/A |
   +-----------------------------------------+------------------------+----------------------+
   ```

## 总结

我让ChatGPT详细地解释`/etc/X11/xorg.conf.d/70-nvidia.conf`配置文件的含义：

> 下面是一份对该 Xorg 配置文件的详细解释与讲解，每个部分都对应相应的 Section 和配置信息。该配置主要目的在于让用户可以在同一台机器上同时使用或者切换 Intel 集显 (Integrated Graphics) 和 NVIDIA 独显 (Dedicated Graphics)，常用于双显卡笔记本或混合显卡台式机环境中。
> 
> --------------------------------------------------------------------------------
> 1. Section "ServerLayout"
> --------------------------------------------------------------------------------
> 
> Section "ServerLayout"
>     Identifier "layout"
>     Screen 0 "nvidia"
>     Inactive "intel"
> EndSection
> 
> • Identifier "layout": 标识整个布局的名称为 "layout"。  
> • Screen 0 "nvidia": 指定默认启用的屏幕设备（Screen）是 "nvidia"，也就意味着在启动 X Server 时，会使用名称为 "nvidia" 的 Screen 配置。  
> • Inactive "intel": 将 "intel" Screen 标记为 Inactive（非活动），表示在默认情况下不会使用它进行显示输出，但它的配置依然存在于 Xorg 中，方便之后如果需要切换或使用 Intel 时重新激活。  
> 
> 这一部分实际上是 Xorg 的整体布局设置，决定了系统在启动 X Server 的时候，以哪个显卡 (Screen) 作为主要输出设备，以及哪些显卡配置处于备用或非活跃状态。
> 
> --------------------------------------------------------------------------------
> 2. Section "Device"（Intel）
> --------------------------------------------------------------------------------
> 
> Section "Device"
>     Identifier "intel"
>     Driver "modesetting"
>     BusID "PCI:0:2:0"           # 此处根据 lspci 命令的输出进行修改
>     Option "DRI" "3"
> EndSection
> 
> • Identifier "intel": 标识这个 Device（设备）段的名称为 "intel"。  
> • Driver "modesetting": 指定驱动为 "modesetting"。在较新的 Linux 发行版中，modesetting 驱动可以管理 Intel 集显，非常通用。  
> • BusID "PCI:0:2:0": 指定了该显卡在 PCI 总线上的位置，需根据 lspci 命令的实际输出修改。例如，如果 lspci 命令显示你的 Intel 显卡设备为 00:02.0，则这里写成 "PCI:0:2:0"；数字要和 lspci 的输出保持一致。  
> • Option "DRI" "3": 开启 Direct Rendering Infrastructure (DRI) 3 模式，通常可以提升 3D 加速性能与兼容性。  
> 
> 这个 Section 用来加载 Intel 集显对应的 Xorg 驱动和选项。
> 
> --------------------------------------------------------------------------------
> 3. Section "Screen"（Intel）
> --------------------------------------------------------------------------------
> 
> Section "Screen"
>     Identifier "intel"
>     Device "intel"
> EndSection
> 
> • Identifier "intel": 标识该 Screen 的名称为 "intel"。  
> • Device "intel": 与上方的 Device 段名称对应，告诉 Xorg 这个 Screen 使用的具体 Device 配置是哪一个（即前面的 "intel"）。  
> 
> Screen 段中，除了引用对应的 Device 外，往往也可以配置分辨率、显示模式等参数。如果只写了设备引用，并未额外写显示模式配置，就会使用系统或驱动默认的分辨率设置。
> 
> --------------------------------------------------------------------------------
> 4. Section "Device"（NVIDIA）
> --------------------------------------------------------------------------------
> 
> Section "Device"
>     Identifier "nvidia"
>     Driver "nvidia"
>     BusID "PCI:1:0:0"           # 此处根据 lspci 命令的输出进行修改
>     Option "AllowEmptyInitialConfiguration"
>     #Option "Coolbits" "4"      # 可选，用于某些 NVIDIA 显卡风扇 / 超频调整
> EndSection
> 
> • Identifier "nvidia": 标识这个设备段是 "nvidia"。  
> • Driver "nvidia": 指定使用专有的 NVIDIA 驱动（NVIDIA 提供的闭源驱动），而非 nouveau 开源驱动。  
> • BusID "PCI:1:0:0": 指定显卡的 PCI 总线位置，同样需要根据 lspci 的输出进行修改。  
> • Option "AllowEmptyInitialConfiguration": 允许在 Xorg 启动时，即使没有显示器实际连接到该显卡上，也能让配置正常初始化。这在混合显卡环境中常见，可以防止 Xorg 因找不到物理显示连接而出错。  
> • #Option "Coolbits" "4": 这是一项可选功能，如果开启，可以通过 nvidia-settings 等工具进行风扇转速、超频等设置（具体值含义可参考 NVIDIA 文档）。这里注释掉了，如果需要可以取消注释并调整数字。
> 
> --------------------------------------------------------------------------------
> 5. Section "Screen"（NVIDIA）
> --------------------------------------------------------------------------------
> 
> Section "Screen"
>     Identifier "nvidia"
>     Device "nvidia"
> EndSection
> 
> • Identifier "nvidia": 标识该 Screen 的名称为 "nvidia"。  
> • Device "nvidia": 引用上方定义好的 Device "nvidia"。  
> 
> 与 Intel 的配置类似，此处也可以进一步配置分辨率和其他显示参数。如果未额外指定，Xorg 将使用驱动默认配置。
> 
> --------------------------------------------------------------------------------
> 6. 总体工作原理
> --------------------------------------------------------------------------------
> 
> 该配置文件在启动 Xorg 时，会根据 "ServerLayout" 段的指示，默认启用 "nvidia" 屏幕（Screen 0 "nvidia"）。同时，"intel" 屏幕被标记为 Inactive。如果你想要使用 Intel 集显进行渲染，可以在需要时修改 /etc/X11/xorg.conf 或者使用 Xorg 相关工具切换，从而使 "intel" 配置变为活跃，"nvidia" 则变为 Inactive（或使用其他机制，比如 PRIME、Bumblebee、nvidia-xrun 等更灵活地在双显卡间切换）。
> 
> 这种模式常见于：
> 1. 笔记本电脑：CPU 内置 Intel Graphics + 独立的 NVIDIA 显卡。  
> 2. 台式机：主板集显 + 独立显卡共存，需要手动进行多卡配置。  
> 
> 在 Linux 下，如果希望在节省能耗和性能之间做取舍，就可能需要按需启用或禁用某个显卡。以上配置文件正是一个基础示例，不同发行版以及双显卡切换方法（如 PRIME Offloading、Bumblebee 等）可能有更复杂的配置方式和附加选项，也可能需要涉及到 Prime Render Offload、xrandr 等工具。
> 
> --------------------------------------------------------------------------------
> 7. 修改与调试建议
> --------------------------------------------------------------------------------
> 
> 1. 确认 BusID：  
>    - 使用 lspci 命令来确认 Intel 和 NVIDIA 在 PCI 总线上的位置信息。  
>    - 将其填入 BusID "PCI:xx:xx:x" 中，必须和 lspci 输出相匹配。  
> 
> 2. 检查驱动是否正确安装：  
>    - Intel 集显通常使用内置 i915 模块或者 modesetting 驱动，只要系统安装的是通用的 Xorg 都较为容易识别。  
>    - NVIDIA 的专有驱动需要确保系统安装了与内核版本匹配的 nvidia-driver 包，如果安装不完整，可能会在加载时出错。  
> 
> 3. 日志输出：  
>    - 如果 Xorg 启动失败或工作异常，查看 /var/log/Xorg.0.log 或者在某些发行版中查看 ~/.local/share/xorg/ 目录下的日志文件，以获取具体的错误提示。  
> 
> 4. 额外优化和功能：  
>    - 如果需要更高级的功能（如自动切换、节能模式），可以研究 PRIME、Bumblebee 或者 nvidia-xrun 等方案。  
>    - 对于高端显卡的超频或风扇控制，可以在 "Device" 段启用 Coolbits，然后通过 nvidia-settings 工具进行调节。
> 
> --------------------------------------------------------------------------------
> 总结
> --------------------------------------------------------------------------------
> 
> 这个 Xorg 配置文件的核心思路，是通过在 "ServerLayout" 中指定一个默认活动的 "Screen"（此处为 "nvidia"）和一个 Inactive 的 "Screen"（此处为 "intel"），来确保同一台计算机能够根据需要使用不同的显卡进行显示和渲染。对双显卡平台而言，如果只是想在某些场景下用更高性能的独立显卡或集显进行输出，就必须在 Xorg 配置和驱动层面对这两块显卡进行正确识别和初始化。以上配置文件给出了一个典型示例，后续可根据实际硬件和需求进行相应调整与扩展。
