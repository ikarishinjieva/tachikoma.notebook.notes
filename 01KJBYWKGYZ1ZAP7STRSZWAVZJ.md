---
title: 20221207 - OMS checker代码分析
confluence_page_id: 2130349
created_at: 2022-12-07T05:33:28+00:00
updated_at: 2022-12-07T05:33:28+00:00
---

# 使用

启动OMS校验步骤后, 启动脚本:

```
/bin/bash /home/ds/bin/checker_new.sh -t 10.186.62.73-9000:2:0000000002 -c /home/ds//run/10.186.62.73-9000:2:0000000002/conf/checker.conf -m checker -l -server -Xms8g -Xmx8g -Xmn4g -Xss512k start
``` 

脚本准备了java的运行环境, 调用脚本 verifier-start

脚本中提供了cygwin兼容, 用于启动java进程

产生新进程: 

```
/opt/alibaba/java/bin/java -server -Xms8g -Xmx8g -Xmn4g -Xss512k -XX:ErrorFile=/u01/ds/bin/..//run//10.186.62.73-9000:2:0000000002/logs/hs_err_pid%p.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/u01/ds/bin/..//run//10.186.62.73-9000:2:0000000002/logs -verbose:gc -Xloggc:/u01/ds/bin/..//run//10.186.62.73-9000:2:0000000002/logs/gc_%p.log -XX:+PrintGCDateStamps -XX:+PrintGCDetails -classpath lib/verification-core-1.2.54.jar:lib/verification-OB05-1.2.54.jar:lib/verification-TIDB-1.2.54.jar:lib/verification-MySQL-1.2.54.jar:lib/verification-OB-Oracle-Mode-1.2.54.jar:lib/verification-OB10-1.2.54.jar:lib/verification-Oracle-1.2.54.jar:lib/verification-DB2-1.2.54.jar:lib/verification-Sybase-1.2.54.jar:lib/verification-common-1.2.54.jar:lib/apache-log4j-extras-1.2.17.jar:lib/slf4j-log4j12-1.7.21.jar:lib/oms-conditions-3.2.3.12-SNAPSHOT.jar:lib/oms-common-3.2.3.12-SNAPSHOT.jar:lib/log4j-1.2.17.jar:lib/dws-rule-1.1.6.jar:lib/dws-schema-1.1.6.jar:lib/oms-record-3.2.3.12-SNAPSHOT.jar:lib/commons-io-2.6.jar:lib/metrics-core-4.0.2.jar:lib/connect-api-2.1.0.jar:lib/kafka-clients-2.1.0.jar:lib/slf4j-api-1.7.25.jar:lib/dss-transformer-1.0.10.jar:lib/calcite-core-1.19.0.jar:lib/dss-record-1.0.0.jar:lib/avatica-core-1.13.0.jar:lib/jackson-datatype-jsr310-2.11.1.jar:lib/jackson-databind-2.11.1.jar:lib/esri-geometry-api-2.2.0.jar:lib/jackson-dataformat-yaml-2.9.8.jar:lib/jackson-core-2.11.1.jar:lib/jackson-annotations-2.11.1.jar:lib/mysql-connector-java-5.1.47.jar:lib/oceanbase-1.2.1.jar:lib/druid-1.1.11.jar:lib/etransfer-0.0.65-SNAPSHOT.jar:lib/commons-lang3-3.9.jar:lib/aggdesigner-algorithm-6.0.jar:lib/commons-lang-2.6.jar:lib/fastjson-1.2.72_noneautotype.jar:lib/commons-beanutils-1.7.0.jar:lib/log4j-1.2.15.jar:lib/mapdb-3.0.8.jar:lib/kotlin-stdlib-1.2.71.jar:lib/annotations-16.0.3.jar:lib/calcite-linq4j-1.19.0.jar:lib/guava-29.0-jre.jar:lib/maven-project-2.2.1.jar:lib/maven-artifact-manager-2.2.1.jar:lib/maven-reporting-api-3.0.jar:lib/doxia-sink-api-1.1.2.jar:lib/doxia-logging-api-1.1.2.jar:lib/maven-settings-2.2.1.jar:lib/maven-profile-2.2.1.jar:lib/maven-plugin-registry-2.2.1.jar:lib/plexus-container-default-1.0-alpha-30.jar:lib/groovy-test-2.5.5.jar:lib/plexus-classworlds-1.2-alpha-9.jar:lib/junit-4.12.jar:lib/commons-dbcp2-2.5.0.jar:lib/httpclient-4.5.6.jar:lib/commons-logging-1.2.jar:lib/commons-collections4-4.1.jar:lib/oms-operator-3.2.3.12-SNAPSHOT.jar:lib/retrofit-2.9.0.jar:lib/jsr305-3.0.2.jar:lib/servlet-api-2.5.jar:lib/org.osgi.core-4.3.1.jar:lib/protobuf-java-3.11.0.jar:lib/maven-plugin-api-2.2.1.jar:lib/okhttp-3.14.9.jar:lib/maven-artifact-2.2.1.jar:lib/wagon-provider-api-1.0-beta-6.jar:lib/maven-repository-metadata-2.2.1.jar:lib/maven-model-2.2.1.jar:lib/plexus-utils-3.0.16.jar:lib/javax.annotation-api-1.3.2.jar:lib/javassist-3.20.0-GA.jar:lib/xml-apis-1.3.03.jar:lib/error_prone_annotations-2.3.4.jar:lib/easy-random-core-4.2.0.jar:lib/objenesis-3.1.jar:lib/commons-collections-3.2.2.jar:lib/lombok-1.18.16.jar:lib/antlr4-runtime-4.9.1.jar:lib/ojdbc8-19.7.0.0.jar:lib/orai18n-19.3.0.0.jar:lib/oceanbase-client-1.1.10.jar:lib/db2jcc-db2jcc4.jar:lib/jtds-1.3.1.jar:lib/javax.ws.rs-api-2.1.1.jar:lib/groovy-ant-2.5.5.jar:lib/groovy-cli-commons-2.5.5.jar:lib/groovy-groovysh-2.5.5.jar:lib/groovy-console-2.5.5.jar:lib/groovy-groovydoc-2.5.5.jar:lib/groovy-docgenerator-2.5.5.jar:lib/groovy-cli-picocli-2.5.5.jar:lib/groovy-datetime-2.5.5.jar:lib/groovy-jmx-2.5.5.jar:lib/groovy-json-2.5.5.jar:lib/groovy-jsr223-2.5.5.jar:lib/groovy-macro-2.5.5.jar:lib/groovy-nio-2.5.5.jar:lib/groovy-servlet-2.5.5.jar:lib/groovy-sql-2.5.5.jar:lib/groovy-swing-2.5.5.jar:lib/groovy-templates-2.5.5.jar:lib/groovy-test-junit5-2.5.5.jar:lib/groovy-testng-2.5.5.jar:lib/groovy-xml-2.5.5.jar:lib/groovy-2.5.5.jar:lib/sketches-core-0.9.0.jar:lib/json-path-2.4.0.jar:lib/janino-3.0.11.jar:lib/commons-compiler-3.0.11.jar:lib/hamcrest-core-1.3.jar:lib/gson-2.8.5.jar:lib/httpcore-4.4.13.jar:lib/commons-codec-1.15.jar:lib/ini4j-0.5.2.jar:lib/backport-util-concurrent-3.1.jar:lib/plexus-interpolation-1.11.jar:lib/failureaccess-1.0.1.jar:lib/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar:lib/checker-qual-2.11.1.jar:lib/j2objc-annotations-1.3.jar:lib/classgraph-4.8.65.jar:lib/zstd-jni-1.3.5-4.jar:lib/lz4-java-1.5.0.jar:lib/snappy-java-1.1.7.2.jar:lib/ant-junit-1.9.13.jar:lib/ant-1.9.13.jar:lib/ant-launcher-1.9.13.jar:lib/ant-antlr-1.9.13.jar:lib/commons-cli-1.4.jar:lib/picocli-3.7.0.jar:lib/qdox-1.12.1.jar:lib/jline-2.14.6.jar:lib/junit-platform-launcher-1.3.2.jar:lib/junit-jupiter-engine-5.3.2.jar:lib/testng-6.13.1.jar:lib/avatica-metrics-1.13.0.jar:lib/commons-pool2-2.6.0.jar:lib/snakeyaml-1.23.jar:lib/memory-0.9.0.jar:lib/eclipse-collections-forkjoin-11.0.0.jar:lib/eclipse-collections-11.0.0.jar:lib/eclipse-collections-api-11.0.0.jar:lib/lz4-1.3.0.jar:lib/elsa-3.0.0-M5.jar:lib/okio-1.17.2.jar:lib/junit-platform-engine-1.3.2.jar:lib/junit-jupiter-api-5.3.2.jar:lib/junit-platform-commons-1.3.2.jar:lib/apiguardian-api-1.0.0.jar:lib/jcommander-1.72.jar:lib/kotlin-stdlib-common-1.2.71.jar:lib/opentest4j-1.1.1.jar:conf com.alipay.light.VEngine -t 10.186.62.73-9000:2:0000000002 -c /home/ds//run/10.186.62.73-9000:2:0000000002/conf/checker.conf start
``` 

句柄列表: 

```
[root@ubuntu ~]# ls -alh /proc/24505/fd
total 0
dr-x------ 2 ds ds  0 Dec  7 10:24 .
dr-xr-xr-x 9 ds ds  0 Dec  7 10:24 ..
lr-x------ 1 ds ds 64 Dec  7 10:24 0 -> /dev/null
l-wx------ 1 ds ds 64 Dec  7 10:24 1 -> /u01/ds/run/10.186.62.73-9000:2:0000000002/stdout.out
lr-x------ 1 ds ds 64 Dec  7 10:24 10 -> /u01/ds/plugins/checker/lib/verification-OB-Oracle-Mode-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 100 -> /u01/ds/plugins/checker/lib/groovy-groovysh-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 101 -> /u01/ds/plugins/checker/lib/groovy-console-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 102 -> /u01/ds/plugins/checker/lib/groovy-groovydoc-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 103 -> /u01/ds/plugins/checker/lib/groovy-docgenerator-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 104 -> /u01/ds/plugins/checker/lib/groovy-cli-picocli-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 105 -> /u01/ds/plugins/checker/lib/groovy-datetime-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 106 -> /u01/ds/plugins/checker/lib/groovy-jmx-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 107 -> /u01/ds/plugins/checker/lib/groovy-json-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 108 -> /u01/ds/plugins/checker/lib/groovy-jsr223-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 109 -> /u01/ds/plugins/checker/lib/groovy-macro-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 11 -> /u01/ds/plugins/checker/lib/verification-OB10-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 110 -> /u01/ds/plugins/checker/lib/groovy-nio-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 111 -> /u01/ds/plugins/checker/lib/groovy-servlet-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 112 -> /u01/ds/plugins/checker/lib/groovy-sql-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 113 -> /u01/ds/plugins/checker/lib/groovy-swing-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 114 -> /u01/ds/plugins/checker/lib/groovy-templates-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 115 -> /u01/ds/plugins/checker/lib/groovy-test-junit5-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 116 -> /u01/ds/plugins/checker/lib/groovy-testng-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 117 -> /u01/ds/plugins/checker/lib/groovy-xml-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 118 -> /u01/ds/plugins/checker/lib/groovy-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 119 -> /u01/ds/plugins/checker/lib/sketches-core-0.9.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 12 -> /u01/ds/plugins/checker/lib/verification-Oracle-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 120 -> /u01/ds/plugins/checker/lib/json-path-2.4.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 121 -> /u01/ds/plugins/checker/lib/janino-3.0.11.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 122 -> /u01/ds/plugins/checker/lib/commons-compiler-3.0.11.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 123 -> /u01/ds/plugins/checker/lib/hamcrest-core-1.3.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 124 -> /u01/ds/plugins/checker/lib/gson-2.8.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 125 -> /u01/ds/plugins/checker/lib/httpcore-4.4.13.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 126 -> /u01/ds/plugins/checker/lib/commons-codec-1.15.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 127 -> /u01/ds/plugins/checker/lib/ini4j-0.5.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 128 -> /u01/ds/plugins/checker/lib/backport-util-concurrent-3.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 129 -> /u01/ds/plugins/checker/lib/plexus-interpolation-1.11.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 13 -> /u01/ds/plugins/checker/lib/verification-DB2-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 130 -> /u01/ds/plugins/checker/lib/failureaccess-1.0.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 131 -> /u01/ds/plugins/checker/lib/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 132 -> /u01/ds/plugins/checker/lib/checker-qual-2.11.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 133 -> /u01/ds/plugins/checker/lib/j2objc-annotations-1.3.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 134 -> /u01/ds/plugins/checker/lib/classgraph-4.8.65.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 135 -> /u01/ds/plugins/checker/lib/zstd-jni-1.3.5-4.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 136 -> /u01/ds/plugins/checker/lib/lz4-java-1.5.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 137 -> /u01/ds/plugins/checker/lib/snappy-java-1.1.7.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 138 -> /u01/ds/plugins/checker/lib/ant-junit-1.9.13.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 139 -> /u01/ds/plugins/checker/lib/ant-1.9.13.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 14 -> /u01/ds/plugins/checker/lib/verification-Sybase-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 140 -> /u01/ds/plugins/checker/lib/ant-launcher-1.9.13.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 141 -> /u01/ds/plugins/checker/lib/ant-antlr-1.9.13.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 142 -> /u01/ds/plugins/checker/lib/commons-cli-1.4.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 143 -> /u01/ds/plugins/checker/lib/picocli-3.7.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 144 -> /u01/ds/plugins/checker/lib/qdox-1.12.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 145 -> /u01/ds/plugins/checker/lib/jline-2.14.6.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 146 -> /u01/ds/plugins/checker/lib/junit-platform-launcher-1.3.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 147 -> /u01/ds/plugins/checker/lib/junit-jupiter-engine-5.3.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 148 -> /u01/ds/plugins/checker/lib/testng-6.13.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 149 -> /u01/ds/plugins/checker/lib/avatica-metrics-1.13.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 15 -> /u01/ds/plugins/checker/lib/verification-common-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 150 -> /u01/ds/plugins/checker/lib/commons-pool2-2.6.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 151 -> /u01/ds/plugins/checker/lib/snakeyaml-1.23.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 152 -> /u01/ds/plugins/checker/lib/memory-0.9.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 153 -> /u01/ds/plugins/checker/lib/eclipse-collections-forkjoin-11.0.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 154 -> /u01/ds/plugins/checker/lib/eclipse-collections-11.0.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 155 -> /u01/ds/plugins/checker/lib/eclipse-collections-api-11.0.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 156 -> /u01/ds/plugins/checker/lib/lz4-1.3.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 157 -> /u01/ds/plugins/checker/lib/elsa-3.0.0-M5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 158 -> /u01/ds/plugins/checker/lib/okio-1.17.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 159 -> /u01/ds/plugins/checker/lib/junit-platform-engine-1.3.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 16 -> /u01/ds/plugins/checker/lib/apache-log4j-extras-1.2.17.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 160 -> /u01/ds/plugins/checker/lib/junit-jupiter-api-5.3.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 161 -> /u01/ds/plugins/checker/lib/junit-platform-commons-1.3.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 162 -> /u01/ds/plugins/checker/lib/apiguardian-api-1.0.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 163 -> /u01/ds/plugins/checker/lib/jcommander-1.72.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 164 -> /u01/ds/plugins/checker/lib/kotlin-stdlib-common-1.2.71.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 165 -> /u01/ds/plugins/checker/lib/opentest4j-1.1.1.jar
l-wx------ 1 ds ds 64 Dec  7 10:24 166 -> /u01/ds/run/10.186.62.73-9000:2:0000000002/logs/error.log
l-wx------ 1 ds ds 64 Dec  7 10:24 167 -> /u01/ds/run/10.186.62.73-9000:2:0000000002/logs/task.log
l-wx------ 1 ds ds 64 Dec  7 10:24 168 -> /u01/ds/run/10.186.62.73-9000:2:0000000002/logs/metrics.log
lrwx------ 1 ds ds 64 Dec  7 10:24 169 -> /u01/ds/run/10.186.62.73-9000:2:0000000002/verifier.pid
lr-x------ 1 ds ds 64 Dec  7 10:24 17 -> /u01/ds/plugins/checker/lib/slf4j-log4j12-1.7.21.jar
lrwx------ 1 ds ds 64 Dec  7 10:24 170 -> socket:[1551136075]
lr-x------ 1 ds ds 64 Dec  7 10:24 171 -> /opt/alibaba/java-1.8.0-alibaba-dragonwell-8.8.8.302.b1-1.al7/jre/lib/charsets.jar
lrwx------ 1 ds ds 64 Dec  7 10:24 172 -> socket:[1551134578]
lrwx------ 1 ds ds 64 Dec  7 10:24 173 -> socket:[1551134580]
lr-x------ 1 ds ds 64 Dec  7 10:24 174 -> /opt/alibaba/java-1.8.0-alibaba-dragonwell-8.8.8.302.b1-1.al7/jre/lib/jsse.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 175 -> /dev/random
lr-x------ 1 ds ds 64 Dec  7 10:24 176 -> /dev/urandom
lr-x------ 1 ds ds 64 Dec  7 10:24 177 -> /dev/random
lr-x------ 1 ds ds 64 Dec  7 10:24 178 -> /dev/random
lr-x------ 1 ds ds 64 Dec  7 10:24 179 -> /dev/urandom
lr-x------ 1 ds ds 64 Dec  7 10:24 18 -> /u01/ds/plugins/checker/lib/oms-conditions-3.2.3.12-SNAPSHOT.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 180 -> /dev/urandom
lrwx------ 1 ds ds 64 Dec  7 10:24 181 -> socket:[1551156433]
lrwx------ 1 ds ds 64 Dec  7 10:24 182 -> socket:[1551151794]
lr-x------ 1 ds ds 64 Dec  7 10:24 19 -> /u01/ds/plugins/checker/lib/oms-common-3.2.3.12-SNAPSHOT.jar
l-wx------ 1 ds ds 64 Dec  7 10:24 2 -> /u01/ds/run/10.186.62.73-9000:2:0000000002/stdout.out
lr-x------ 1 ds ds 64 Dec  7 10:24 20 -> /u01/ds/plugins/checker/lib/log4j-1.2.17.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 21 -> /u01/ds/plugins/checker/lib/dws-rule-1.1.6.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 22 -> /u01/ds/plugins/checker/lib/dws-schema-1.1.6.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 23 -> /u01/ds/plugins/checker/lib/oms-record-3.2.3.12-SNAPSHOT.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 24 -> /u01/ds/plugins/checker/lib/commons-io-2.6.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 25 -> /u01/ds/plugins/checker/lib/metrics-core-4.0.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 26 -> /u01/ds/plugins/checker/lib/connect-api-2.1.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 27 -> /u01/ds/plugins/checker/lib/kafka-clients-2.1.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 28 -> /u01/ds/plugins/checker/lib/slf4j-api-1.7.25.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 29 -> /u01/ds/plugins/checker/lib/dss-transformer-1.0.10.jar
l-wx------ 1 ds ds 64 Dec  7 10:24 3 -> /u01/ds/run/10.186.62.73-9000:2:0000000002/logs/gc_pid24505.log
lr-x------ 1 ds ds 64 Dec  7 10:24 30 -> /u01/ds/plugins/checker/lib/calcite-core-1.19.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 31 -> /u01/ds/plugins/checker/lib/dss-record-1.0.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 32 -> /u01/ds/plugins/checker/lib/avatica-core-1.13.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 33 -> /u01/ds/plugins/checker/lib/jackson-datatype-jsr310-2.11.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 34 -> /u01/ds/plugins/checker/lib/jackson-databind-2.11.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 35 -> /u01/ds/plugins/checker/lib/esri-geometry-api-2.2.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 36 -> /u01/ds/plugins/checker/lib/jackson-dataformat-yaml-2.9.8.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 37 -> /u01/ds/plugins/checker/lib/jackson-core-2.11.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 38 -> /u01/ds/plugins/checker/lib/jackson-annotations-2.11.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 39 -> /u01/ds/plugins/checker/lib/mysql-connector-java-5.1.47.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 4 -> /opt/alibaba/java-1.8.0-alibaba-dragonwell-8.8.8.302.b1-1.al7/jre/lib/rt.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 40 -> /u01/ds/plugins/checker/lib/oceanbase-1.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 41 -> /u01/ds/plugins/checker/lib/druid-1.1.11.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 42 -> /u01/ds/plugins/checker/lib/etransfer-0.0.65-SNAPSHOT.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 43 -> /u01/ds/plugins/checker/lib/commons-lang3-3.9.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 44 -> /u01/ds/plugins/checker/lib/aggdesigner-algorithm-6.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 45 -> /u01/ds/plugins/checker/lib/commons-lang-2.6.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 46 -> /u01/ds/plugins/checker/lib/fastjson-1.2.72_noneautotype.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 47 -> /u01/ds/plugins/checker/lib/commons-beanutils-1.7.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 48 -> /u01/ds/plugins/checker/lib/log4j-1.2.15.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 49 -> /u01/ds/plugins/checker/lib/mapdb-3.0.8.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 5 -> /opt/alibaba/java-1.8.0-alibaba-dragonwell-8.8.8.302.b1-1.al7/jre/lib/jfr.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 50 -> /u01/ds/plugins/checker/lib/kotlin-stdlib-1.2.71.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 51 -> /u01/ds/plugins/checker/lib/annotations-16.0.3.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 52 -> /u01/ds/plugins/checker/lib/calcite-linq4j-1.19.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 53 -> /u01/ds/plugins/checker/lib/guava-29.0-jre.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 54 -> /u01/ds/plugins/checker/lib/maven-project-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 55 -> /u01/ds/plugins/checker/lib/maven-artifact-manager-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 56 -> /u01/ds/plugins/checker/lib/maven-reporting-api-3.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 57 -> /u01/ds/plugins/checker/lib/doxia-sink-api-1.1.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 58 -> /u01/ds/plugins/checker/lib/doxia-logging-api-1.1.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 59 -> /u01/ds/plugins/checker/lib/maven-settings-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 6 -> /u01/ds/plugins/checker/lib/verification-core-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 60 -> /u01/ds/plugins/checker/lib/maven-profile-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 61 -> /u01/ds/plugins/checker/lib/maven-plugin-registry-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 62 -> /u01/ds/plugins/checker/lib/plexus-container-default-1.0-alpha-30.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 63 -> /u01/ds/plugins/checker/lib/groovy-test-2.5.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 64 -> /u01/ds/plugins/checker/lib/plexus-classworlds-1.2-alpha-9.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 65 -> /u01/ds/plugins/checker/lib/junit-4.12.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 66 -> /u01/ds/plugins/checker/lib/commons-dbcp2-2.5.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 67 -> /u01/ds/plugins/checker/lib/httpclient-4.5.6.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 68 -> /u01/ds/plugins/checker/lib/commons-logging-1.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 69 -> /u01/ds/plugins/checker/lib/commons-collections4-4.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 7 -> /u01/ds/plugins/checker/lib/verification-OB05-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 70 -> /u01/ds/plugins/checker/lib/oms-operator-3.2.3.12-SNAPSHOT.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 71 -> /u01/ds/plugins/checker/lib/retrofit-2.9.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 72 -> /u01/ds/plugins/checker/lib/jsr305-3.0.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 73 -> /u01/ds/plugins/checker/lib/servlet-api-2.5.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 74 -> /u01/ds/plugins/checker/lib/org.osgi.core-4.3.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 75 -> /u01/ds/plugins/checker/lib/protobuf-java-3.11.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 76 -> /u01/ds/plugins/checker/lib/maven-plugin-api-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 77 -> /u01/ds/plugins/checker/lib/okhttp-3.14.9.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 78 -> /u01/ds/plugins/checker/lib/maven-artifact-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 79 -> /u01/ds/plugins/checker/lib/wagon-provider-api-1.0-beta-6.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 8 -> /u01/ds/plugins/checker/lib/verification-TIDB-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 80 -> /u01/ds/plugins/checker/lib/maven-repository-metadata-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 81 -> /u01/ds/plugins/checker/lib/maven-model-2.2.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 82 -> /u01/ds/plugins/checker/lib/plexus-utils-3.0.16.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 83 -> /u01/ds/plugins/checker/lib/javax.annotation-api-1.3.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 84 -> /u01/ds/plugins/checker/lib/javassist-3.20.0-GA.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 85 -> /u01/ds/plugins/checker/lib/xml-apis-1.3.03.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 86 -> /u01/ds/plugins/checker/lib/error_prone_annotations-2.3.4.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 87 -> /u01/ds/plugins/checker/lib/easy-random-core-4.2.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 88 -> /u01/ds/plugins/checker/lib/objenesis-3.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 89 -> /u01/ds/plugins/checker/lib/commons-collections-3.2.2.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 9 -> /u01/ds/plugins/checker/lib/verification-MySQL-1.2.54.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 90 -> /u01/ds/plugins/checker/lib/lombok-1.18.16.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 91 -> /u01/ds/plugins/checker/lib/antlr4-runtime-4.9.1.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 92 -> /u01/ds/plugins/checker/lib/ojdbc8-19.7.0.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 93 -> /u01/ds/plugins/checker/lib/orai18n-19.3.0.0.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 94 -> /u01/ds/plugins/checker/lib/oceanbase-client-1.1.10.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 95 -> /u01/ds/plugins/checker/lib/db2jcc-db2jcc4.jar
lr-x------ 1 ds ds 64 Dec  7 10:24 96 -> /u01/ds/plugins/checker/lib/jtds-1.3.1.jar
``` 

# 代码分析

启动类: com.alipay.light.VEngine (verification-core-1.2.54.jar)
