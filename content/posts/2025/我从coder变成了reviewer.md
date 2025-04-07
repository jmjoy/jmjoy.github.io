+++
date = '2025-04-07T16:40:12+08:00'
title = '我从coder变成了reviewer'
categories = ["杂谈"]
tags = ["copilot"]
+++

现在我严重依赖`GitHub Copilot`，正如标题所言，我从`coder`变成了`reviewer`，只需要给AI提需求，AI就会帮我生成一大堆代码，效果比我手写的更好，我的工作只剩下review和提出修改意见，必要时再手动做一些微小的修改。

恰逢`GitHub Copilot`的`Agent Mode`在`VSCode`上正式可用，使用起来更加舒服了：

[Vibe coding with GitHub Copilot: Agent mode and MCP support rolling out to all VS Code users](https://github.blog/news-insights/product-news/github-copilot-agent-mode-activated/)

## 体验

我的Github账号很早就有免费的`Copilot Pro`授权，但我之前一直不怎么使用，原因我对AI补全代码不太感冒，即使现在我都是将这项功能给禁用了，因为我感觉他生成的代码并不太准确，或许是受限于上下文吧。

我常用的编辑器`VSCode`和IDE`JB全家桶`的Copilot插件不够强大的时候，我会偶尔使用ChatGPT或者Copilot的网页聊天界面来生成代码，提需求和贴上代码片段等，体验下来，`Claude 3.7`生成的代码质量比`ChatGPT-4o`更好。

后来Copilot插件有了`Edit Mode`，那就方便很多了，生成代码后会显示diff差异，并且能够保持会话一直提需求，直至满意，但缺点是还是要选择相应的代码文件作为上下文。

现在Copilot插件的`Agent Mode`更进一步，不需要选择代码文件了，可以自动根据需求分析该使用哪些项目文件，还能够在询问后执行一些命令，缺点是比较慢，而且需求要提得更明确一些，不然会找不对项目文件。

## 展望

目前AI应该没能取代程序员的，毕竟还是需要有懂的人来review代码，但是岗位需求可能会减少，毕竟AI生成代码让工作效率大增，以前需要两天时间的工作量可能一个下午就搞定了。

另外将代码上传到云端做推理分析，对于安全要求较高的行业不大可行，但可以使用支持MCP协议的私有部署的大模型服务。
