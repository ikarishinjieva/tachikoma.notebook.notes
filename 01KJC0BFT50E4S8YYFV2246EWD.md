---
title: 20260201 - nice DCV license破解
confluence_page_id: 4620373
created_at: 2026-02-01T04:39:33+00:00
updated_at: 2026-02-01T04:39:33+00:00
---

在 <https://www.amazondcv.com/> 下载 ubuntu 22.04版本, 解压

卸载之前的版本: 

```
sudo apt remove nice-dcv-server nice-dcv-web-viewer nice-xdcv
``` 

用strace观察安装过程

```
sudo strace -f -o /tmp/test.strace dpkg -i nice-dcv-server_2025.0.20103-1_amd64.ubuntu2204.deb nice-dcv-web-viewer_2025.0.20103-1_amd64.ubuntu2204.deb nice-xdcv_2025.0.688-1_amd64.ubuntu2204.deb
``` 

观察到 /var/tmp 中的文件与DEMO license相关

```
1990382 openat(AT_FDCWD, "/var/tmp/.qlfhxlcedwjhjm", O_RDONLY) = 3
1990382 close(3)                        = 0
1990382 write(2, "Demo license for product 'dcv' i"..., 52) = 52
1990382 write(1, "Installing detached demo license"..., 53) = 53
1990382 openat(AT_FDCWD, "/var/tmp/.nicekocfrlm_demo", O_RDWR) = -1 ENOENT (No such file or directory)
1990382 openat(AT_FDCWD, "/var/tmp/.qlfhxlcedw.hmjhjm", O_RDONLY) = 3
1990382 close(3)                        = 0
1990382 write(2, "Demo license for product 'dcv-gl"..., 55) = 55
1990382 write(1, "Installing detached demo license"..., 53) = 53
1990382 openat(AT_FDCWD, "/var/tmp/.nicekocfrlm_demo", O_RDWR) = -1 ENOENT (No such file or directory)
1990382 openat(AT_FDCWD, "/var/tmp/.qlfhxlcedw.tnjhjm", O_RDONLY) = 3
1990382 close(3)                        = 0
1990382 write(2, "Demo license for product 'dcv-sm"..., 55) = 55
1990382 exit_group(0)                   = ?
``` 

删除相关文件: 

```
sudo rm -f /var/tmp/.qlfhxlcedw*
``` 

重新安装: 

```
sudo dpkg -i nice-dcv-server_2025.0.20103-1_amd64.ubuntu2204.deb nice-dcv-web-viewer_2025.0.20103-1_amd64.ubuntu2204.deb nice-xdcv_2025.0.688-1_amd64.ubuntu2204.deb
sudo systemctl start dcvsessionlauncher
sudo systemctl start dcvserver
```
