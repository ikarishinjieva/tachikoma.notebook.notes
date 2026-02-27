---
title: 20221123 - 使用IntelliJ的jar进行 反编译
confluence_page_id: 2130231
created_at: 2022-11-23T08:20:29+00:00
updated_at: 2022-11-23T10:09:45+00:00
---

# 问题

对于 OMS 的一些jar包, 使用JD-GUI 无法反编译出源码 (执行时间很长, 无法预期完成时间)

# 解决

使用IntelliJ的反编译器 FernFlower decompiler

获取反编译器: (用maven获取jar)

```
'/Applications/IntelliJ IDEA CE.app/Contents/plugins/maven/lib/maven3/bin/mvn' dependency:get \
  -DrepoUrl=https://www.jetbrains.com/intellij-repository/releases/ \
  -Dartifact=com.jetbrains.intellij.java:java-decompiler-engine:LATEST \
  -Ddest=.
``` 

将 OMS的jar包 (jdbc_connector.jar) 解压后, 进行反编译

```
java -cp /Users/tachikoma/Downloads/java-decompiler-engine-222.4345.14.jar org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler  -dgs=true $(find /Users/tachikoma/Code/oms-3.2.3-src/jdbc_connector.jar.untar -type d -name alipay -o -name alibaba -o -name oceanbase -o -name oms | xargs) /Users/tachikoma/Code/oms-3.2.3-src/decompiled
``` 

# 问题

有一些class会无法解析 (解析时间过长, 进程会卡住)

```
com/oceanbase/oms/operator/expr/predict/InPredictExpr
com/oceanbase/oms/operator/RecordUtil
``` 

解决:

拆分命令:

```
find . -type d -name alipay -o -name alibaba -o -name oceanbase -o -name oms -maxdepth 2 | xargs -n 1 -I {} mkdir -p /Users/tachikoma/Code/oms-3.2.3-src/decompiled/{} 

find . -type d -name alipay -o -name alibaba -o -name oceanbase -o -name oms -maxdepth 2 | xargs -n 1 -I {} java -cp /Users/tachikoma/Downloads/java-decompiler-engine-222.4345.14.jar org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler  -dgs=true {} /Users/tachikoma/Code/oms-3.2.3-src/decompiled/{}
```
