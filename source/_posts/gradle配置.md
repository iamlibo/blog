---
title: gradle构建项目 之 多项目依赖与独立
date: 2016-05-25 13:57:44
tags: [gradle]
categories: [技术,gradle]
---
本文主要内容是在应用gradle进行项目构建时多项目、多模块依赖构建的一些应用经验。

先说一下场景：
有项目A，依赖于B、C模块。
