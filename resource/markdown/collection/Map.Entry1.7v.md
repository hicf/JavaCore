> 本文是基于 `jdk1.7.0_71` 分析

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">一、Map接口</h3>

> **Map接口**是整个 **Key-Value** 存储容器的核心。它 **抽**(qú)(xíng)**象** 定义了 **Key-Value** 结构的容器框架。**Map**不会像 **数组** 中元素那样可以单独存放，到像似 `LinkedList` 中的节点，那么特殊的节点一定需要特定的来描述。在 `jdk1.7` 及其之前的版本，Map的作者将其其中的节点描述为 **Entry** ，请不要翻译成 ~~入口~~、~~大门~~，它有另一个名字 **项目** 或 **条目** ，下文简称为 **项**。

<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">二、Map.Entry接口</h3>

2.1、