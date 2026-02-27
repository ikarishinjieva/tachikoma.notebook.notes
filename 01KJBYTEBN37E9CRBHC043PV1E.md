---
title: 20221116 - MySQL在arm/x86上, 性能不同
confluence_page_id: 2130156
created_at: 2022-11-16T09:21:27+00:00
updated_at: 2022-11-16T09:26:37+00:00
---

```
create table test_point(
citycode int default null,
xy point not null srid 4326
)

DELIMITER $
CREATE PROCEDURE insertadd(IN args INT)
BEGIN
DECLARE i INT DEFAULT 1;
START TRANSACTION;
WHILE i<=args DO
insert into test_point values(10,st_srid(point(116+i/100000,40+i/100000),4326));
SET i=i+1;
END WHILE;
COMMIT;
END
$

CALL insertadd(10000);

alter table test_point add spatial index idx_xy(xy);
``` 

现象 (arm环境, 麒麟v10操作系统): 

  - 8.0.24-arm 很慢: 41s
  - 8.0.24-x64 正常: 4s
  - 8.0.31-arm 正常: 4s

火焰图:

8.0.24-arm

[附件: 24(1).svg] 

8.0.31-arm

[附件: 31.svg] 

![image2022-11-16 16:28:46.png](/assets/01KJBYTEBN37E9CRBHC043PV1E/image2022-11-16%2016%3A28%3A46.png)

额外消耗于libm

尝试gdb libm, 需要debuginfo包: <https://update.cs2c.com.cn/NS/V10/V10SP1.1/os/adv/lic/updates/aarch64/debug/glibc-debuginfo-2.28-36.1.p03.ky10.aarch64.rpm>

升级glibc到这个带debuginfo的版本

重做24的火焰图, 可以看到指令: 

![image2022-11-16 17:10:32.png](/assets/01KJBYTEBN37E9CRBHC043PV1E/image2022-11-16%2017%3A10%3A32.png)

检查mbr_join_area的代码变更: 

![image2022-11-16 17:24:25.png](/assets/01KJBYTEBN37E9CRBHC043PV1E/image2022-11-16%2017%3A24%3A25.png)

在8.0.29升级了BOOST, 从1.67.0升级到1.77.0
