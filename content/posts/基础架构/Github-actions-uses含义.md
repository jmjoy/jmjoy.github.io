以前一直疑惑Github actions的[**`jobs.<job_id>.steps.uses`**](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepsuses)这个字段的含义，今天结合了之前的几次使用经验和文档，终于理解了它的含义。

比如`uses: actions/setup-node@v1`指的是使用[https://github.com/actions/checkout](https://github.com/actions/checkout)这个库的`v1`这个tag的脚本来执行`step`，而不是checkout本库的v1版本。
