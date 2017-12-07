---
layout: post
title: VSCode 执行 Python 报 [Done] exited with code=null
categories: []
tags: [Bug]
comments: true
---

在VSCode中写了个Python循环，执行5000次，执行到1500多次就退出，并打印
```
[Done] exited with code=null
```

不明所以...

Google之，找到相同的[问题](https://github.com/formulahendry/vscode-code-runner/issues/171)

原来是code-runner插件的问题，修改一行配置即可
```
{
    "code-runner.runInTerminal": true
}
```
