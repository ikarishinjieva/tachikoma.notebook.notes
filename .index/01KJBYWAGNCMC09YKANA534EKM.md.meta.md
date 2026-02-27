---
note: 01KJBYWAGNCMC09YKANA534EKM.md
title: 20221123 - 使用IntelliJ的jar进行 反编译
indexed_at: 2026-02-27T10:20:50.957240+00:00
---

用户希望我分析这篇关于使用 IntelliJ 的 FernFlower 反编译器进行 Java jar 包反编译的笔记，并生成索引元数据。

让我分析笔记内容：
1. 主题：使用 IntelliJ 的 FernFlower decompiler 反编译 OMS 的 jar 包
2. 问题：JD-GUI 无法反编译某些 jar 包，FernFlower 也有部分 class 无法解析
3. 解决方案：使用 Maven 获取反编译器，拆分命令处理无法解析的 class

标签应该包括：Java、反编译、IntelliJ、FernFlower、jar 包、OMS

摘要：记录使用 IntelliJ 的 FernFlower 反编译器解决 OMS jar 包无法反编译的问题，包括获取反编译器的方法和处理部分 class 无法解析的解决方案。

关键概念：
- FernFlower decompiler: IntelliJ 内置的 Java 反编译器
- JD-GUI: 另一种 Java 反编译工具，对某些 jar 包效果不佳
- ConsoleDecompiler: FernFlower 的命令行反编译工具

关联笔记：查看其他笔记列表，主要是关于 MySQL、Clickhouse、gdb 等主题，与这篇 Java 反编译的笔记没有明显关联，所以应该是「无」## 标签
Java, 反编译, IntelliJ, FernFlower, jar 包, OMS

## 摘要
记录使用 IntelliJ 的 FernFlower 反编译器解决 OMS jar 包无法用 JD-GUI 反编译的问题。包括通过 Maven 获取反编译器、命令行使用方法，以及处理部分 class 解析卡住的拆分方案。

## 关键概念
- FernFlower decompiler: IntelliJ 内置的 Java 反编译器，支持命令行使用
- JD-GUI: 图形化 Java 反编译工具，对某些 jar 包执行时间过长
- ConsoleDecompiler: FernFlower 的命令行反编译工具类

## 关联笔记
无
