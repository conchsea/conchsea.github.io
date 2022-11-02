---
layout: post
title: containerd最新合入代码分析
category: 云原生 containerd
---
---



22年十月：

MR  https://github.com/containerd/containerd/pull/7588   主要对归档包名称做严格命名校验。 

---

22年十一月

11.2  

https://github.com/containerd/containerd/commit/8c5baf4ebb5a53911f6f8e54f931482794e01e12

修复ctr操作时crash问题：Fix ctr crash when pulling with --http-dump and --http-trace simultaneously

分类：可靠性

