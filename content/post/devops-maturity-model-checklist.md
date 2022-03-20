---
title: "DevOps能力成熟度模型"
date: "2019-07-24T07:58:12+08:00"
toc: true
categories:
 - "编程"
tags:
 - devops
---

之前听说过AWS的软件工程师是需要自己写需求说明书,前后端代码,测试和上线.还有instagram的工程师可以做到python的代码提交如果合并到主分支后可以在[一个小时内](https://instagram-engineering.com/static-analysis-at-scale-an-instagram-story-8f498ab71a0c)自动部署到生产被用户使用到,感觉这个非常神奇.如果需要做到这个,对组织级与个人都有极高的devops能力成熟度要求.
上周代表CRM项目通过了信通院的DevOps三级认证.感觉提升的空间很大. 专门看了一下成熟度模型的标准.

<!--more-->

核心要点是要有统一的管理系统,系统之间需要联动,
例如记录故事的系统,如何和你提交记录关联?
测试的缺陷问题如何和你的故事关联?
生产正在运行的代码,如何和代码库的某个基线对应上?
测试报告/需求说明书是否统一管理并和你的迭代有关联?
是否可以做到事故后定会之后回滚部署?

[研发运营一体化（DevOps）能力成熟度模型 第1部分：总体架构](http://std.samr.gov.cn/hb/search/stdHBDetailed?id=C362B3DB6202A067E05397BE0A0A1ED8)