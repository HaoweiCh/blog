---
title: 执行 hexo 命令报错，找不到模块 DTraceProviderBindings
tags:
- hexo
categories:
- DevOps
date: 2017-09-19T14:22:03+08:00
year: 2017
week: 38
updated: 2017-12-19T14:25:54+08:00
---

情况是这样的，过了有一段时间没有更新 hexo-cli 后每次使用无论是 `hexo s` 还是 `hexo d` 都会报错

```text
"Cannot find module './build/Release/DTraceProviderBindings'"
```

 <!-- more --> 

解决 方案是执行

```
npm uninstall hexo-cli -g
npm install hexo-cli -g
```

或

```
yarn global remove hexo-cli
yarn global add hexo-cli
```