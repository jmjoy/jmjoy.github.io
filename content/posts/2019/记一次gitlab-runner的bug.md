---
title: "记一次gitlab runner的bug"
date: 2019-09-27T19:46:57+08:00
categories: ["基础架构"]
tags: []
---

今天给一个项目添加`gitlab ci`，发现`gitlab runner`死都`git clone`不到项目，检查了数遍配置（`.gitlab.yml`， `Deploy  SSH key`等等），但是不觉得有问题，非常恼火。后来平静下来google找答案，发现这TM居然是gitlab的BUG（还是特性，不太清楚），遂解决。

相关讨论：
[https://gitlab.com/gitlab-org/gitlab-foss/issues/39469](https://gitlab.com/gitlab-org/gitlab-foss/issues/39469)

解决方法，我摘抄了，就不翻译了，总结一下就是说即使你是gitlab的管理员，但是你依然不是某个项目的所有者，得在项目里Memebers上加上：
> YAY - it works for me. This problem seems to have multiple solutions.
> 
> The one that worked for me is [https://gitlab.com/gitlab-org/gitlab-ce/issues/44855](https://gitlab.com/gitlab-org/gitlab-ce/issues/44855)
> 
> To summarize. Being an Administrator on Gitlab does not mean you have the "access" to do whatever you want to do in Gitlab. "Unable to access" permissions applies to the person who is logged into Gitlab and running the job. To fix the problem - the person / account running the job must be a member (master) of the project.
> 
> This will apply to private projects. It is not necessary to make a private project Public even though that appears to fix the problem. GITLAB suggests you must have https for the project to work you can use http.
> 
> SOLUTION - add your account to the project even if you are the Administrator or it will give this error.
> 
> ```
> remote: You are not allowed to download code from this project.
> fatal: unable to access 'https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@xxxxxxxxxxxx/project.git/': The requested URL returned error: 403
> ```
