---
layout: post
title:  "Kotlin Anko 找不到方法的解决办法"
date:   2018-11-02 10:38:51 +0800
categories: Kotlin
tags: Kotlin Anko
---

# Kotlin Anko 找不到方法的解决办法

![开局一张图][before]

今天按照[anko](https://github.com/Kotlin/anko)的官方文档配置过程中，一直无法找到[Anko Commons – Intents](https://github.com/Kotlin/anko/wiki/Anko-Commons-%E2%80%93-Intents)中示例的`startActivity`、`browse`等方法，尝试了`0.10.7`、`0.10.6`、`0.10.4`和[官方example](https://github.com/kotlin/anko-example)的`0.10.0-beta-2`版皆无法正确使用anko的方法。

我的AS版本是`Android Studio 3.3 Canary 6`,最终发现是需要安装一个**Anko Support**的AS插件，重启Studio之后就能import anko的方法啦。

![插件安装之后][after]

[before]:/img/2018-11-02-anko-before.png
[after]:/img/2018-11-02-anko-after.png