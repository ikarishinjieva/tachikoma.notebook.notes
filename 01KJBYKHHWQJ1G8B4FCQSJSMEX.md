---
title: 20220811 - golang产生对MySQL的并发连接代码
confluence_page_id: 1933644
created_at: 2022-08-11T06:26:40+00:00
updated_at: 2022-08-17T10:13:44+00:00
---

```
package main

import (
    "database/sql"
    "fmt"
    "log"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

func main() {
    for i: = 0;
    i <= 10000;
    i++{
        time.Sleep(10 * time.Millisecond)

        go func() {
            db, err: = sql.Open("mysql", "msandbox:msandbox@tcp(127.0.0.1:5725)/test")
            db.QueryRow("SELECT VERSION()")
            fmt.Println(i);
            if err != nil {
                log.Fatal(err)
            }
        }()
    }
    fmt.Println("test");
    time.Sleep(1 * time.Hour)
}
``` 

源码包: 

[附件: mysql-stress-golang.tar.gz] 

Tips: 用golang 1.13, 最新版本golang强制使用module, 配置不方便
