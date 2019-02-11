---
layout:     post
title:      tensorflow.js「译」
subtitle:   tensorflow的JS生态
date:       2019-02-11
author:     BY
header-img: img/post-bg-tensorflowJS.png
catalog: true
tags:
    - tensorflow
---

> 译自 [《TENSORFLOW.JS: MACHINE LEARNING FOR THE WEB AND BEYOND》](https://arxiv.org/pdf/1901.05350.pdf)

### 目录

- 摘要
TensorFlow.js是支持JavaScript构建和执行机器学习的库。TensorFlow.js库运行在浏览器中，运行在Node.js环境中。TensorFlow.js是TensorFlow生态系统的一部分，提供了兼容Python的API接口，允许这些模型在Python和JavaScript生态之间进行迁移。TensorFlow.js赋能了广泛的来自JavaScript社区的一大批新的开发者，允许他们构建和部署机器学习模型，并且支持多种新式类型的设备内置计算。本文详细描述了TensorFlow.js的设计原理、API、实现，并且突出一些非常有效的用例。
- 简介
- 背景分析
- 设计介绍
- 实现介绍
- 生态建设
- 应用举例
- 总结后续
