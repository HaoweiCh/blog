```yaml
title: 记录一次机器学习之 CNN 验证码识别的尝试
categories:
- MachineLearning
tags:
- CNN
- 神经网络
- 验证码识别
date: 2021-12-19T09:40:39+08:00
year: 2021
week: 50
updated: 2021-12-20T09:40:39+08:00
```

任职的时候验证码识别这里的工作是AI 工程师专职同事做的， 虽然当时要来了模型但是现在也改不懂 😮‍💨

已离职也不太好打扰，遂顺手学习一下 :)

<!-- more -->

虽然几年前跟着 tutorial 走了一遍 tensorflow 的 CNN，早就忘得差不多了

跟着专业同事的路子，学习及实践一下 pytorch

## mind map

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/F62A928C3E5938498B30A413C89AFB3D9CC9F849.webp)

## 细节

记得他们常说 GPU 训练 CPU 跑模型
也是，除了吹牛逼哪里会有应用场景需要 GPU 跑验证码识别

目前样本数据量还小，现在还是用 mac 本的 CPU 边跑边迭代基础工具链。
后续计划阿里云按小时买 GPU 服务器进行训练

## conclusion

* 第一天，重温基础知识 && 迭代基础工具链
  * pyside 6 写了一个验证码辅助打码工具
    * pyqt6 是第三方，pyside6是 QT 亲儿子
  * 尝试训练 =》有模型训练代码也卵用 :( 
* 第二天，35%（max 35/100）的识别成功率。
  * 打码量积累到 1000 级别的 train 量
* 第三天，迭代基础工具链
  * 调参后识别成功率 59% （max 59/100）
* 结束 96% 以上识别率

---

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/54983C65DFD9CE73DA79E70F3C81490FEE5CD0DF.webp)

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/001274338988B8A753812CD4EF24B9980D4D8704.webp)

准确率

## 工具链

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/0AC7659B3D14190FA04116A25B1039E1992701EC.webp)