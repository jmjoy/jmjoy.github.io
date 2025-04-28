+++
date = '2025-04-28T21:07:20+08:00'
title = '如何禁用VSCode的C/C++扩展'
categories = ["编程技术"]
tags = ["VSCode"]
+++

`VSCode`的[`微软官方C/C++扩展`](https://marketplace.visualstudio.com/items/?itemName=ms-vscode.cpptools)并不太好用，用过[`clangd`](https://marketplace.visualstudio.com/items/?itemName=llvm-vs-code-extensions.vscode-clangd)的都想赶紧把它给禁了，但是，有一些鸡贼扩展又依赖它，比如[`PlatformIO IDE`](https://marketplace.visualstudio.com/items/?itemName=platformio.platformio-ide)。

其实不用把整个扩展禁用，只需要在个人`settings.json`这样配置就行了：

```json
{
  "C_Cpp.autocomplete": "disabled",
  "C_Cpp.intelliSenseEngine": "disabled"
}
```
