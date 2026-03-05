---
note: 01KJBYWAGNCMC09YKANA534EKM.md
title: 20221123 - 使用IntelliJ的jar进行 反编译
indexed_at: 2026-03-05T08:17:28.471526+00:00
---

## 标签
反编译，IntelliJ, FernFlower, Java, jar 包，OMS

## 摘要
记录使用 IntelliJ 的 FernFlower 反编译器处理 OMS 的 jar 包源码的方法。当 JD-GUI 无法反编译时，通过 Maven 获取 java-decompiler-engine，使用 ConsoleDecompiler 命令行工具进行反编译，并解决部分 class 解析卡住的问题。

## 关键概念
- FernFlower Decompiler: IntelliJ IDEA 内置的 Java 反编译器
- ConsoleDecompiler: FernFlower 的命令行反编译工具
- java-decompiler-engine: JetBrains 发布的独立反编译器引擎 Maven 包

## 关联笔记
- 01KJBYVT1YX1432FCSM17916P1.md: 同样涉及 OMS 的 jdbc_connector.jar 反编译以分析复制链路原理
- 01KJBZ4AQYF4DGS7F7000H9XZ8.md: 涉及 OMS 全量复制链路分析，使用相同的 jdbc_connector.jar
