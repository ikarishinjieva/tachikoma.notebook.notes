---
title: 20210923 - linux rc script 相关知识
confluence_page_id: 1343739
created_at: 2021-09-23T03:58:50+00:00
updated_at: 2021-09-23T03:59:51+00:00
---

# 参考

  - <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/installation_guide/s1-boot-init-shutdown-process>
  - <https://alibaba-cloud.medium.com/understanding-and-changing-runlevels-in-systemd-ccc30065c53d>
  - <https://www.freedesktop.org/software/systemd/man/bootup.html#System%20Manager%20Bootup>

# Linux 5.x 启动流程

  - <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/installation_guide/s1-boot-init-shutdown-process>
    - BIOS
    - Boot Loader
    - Kernel
    - /sbin/init 切换 runlevel
      - /etc/inittab
      - 调用 /etc/rc.d/rc **.d/
        - x 为runlevel
        - 脚本名样例: S80sendmail
          - 首字母 为 K/S: 
            - K调用 script stop
            - S调用 script start
          - 80为优先级, 从小往大调用

  

# Systemd 启动流程

  

systemctl list-dependencies 的输出

```
default.target
● ├─auditd.service
● ├─chronyd.service
● ├─crond.service
● ├─dbus.service
● ├─irqbalance.service
● ├─kdump.service
● ├─network.service
● ├─one-context-local.service
● ├─one-context.service
● ├─postfix.service
● ├─rhel-configure.service
● ├─rpcbind.service
● ├─rsyslog.service
● ├─sshd.service
● ├─systemd-ask-password-wall.path
● ├─systemd-logind.service
● ├─systemd-readahead-collect.service
● ├─systemd-readahead-replay.service
● ├─systemd-update-utmp-runlevel.service
● ├─systemd-user-sessions.service
● ├─tuned.service
● ├─vmtoolsd.service
● ├─basic.target
● │ ├─microcode.service
● │ ├─rhel-dmesg.service
● │ ├─selinux-policy-migrate-local-changes@targeted.service
● │ ├─paths.target
● │ ├─slices.target
● │ │ ├─-.slice
● │ │ └─system.slice
● │ ├─sockets.target
● │ │ ├─dbus.socket
● │ │ ├─rpcbind.socket
● │ │ ├─systemd-initctl.socket
● │ │ ├─systemd-journald.socket
● │ │ ├─systemd-shutdownd.socket
● │ │ ├─systemd-udevd-control.socket
● │ │ └─systemd-udevd-kernel.socket
● │ ├─sysinit.target
● │ │ ├─dev-hugepages.mount
● │ │ ├─dev-mqueue.mount
● │ │ ├─kmod-static-nodes.service
● │ │ ├─proc-sys-fs-binfmt_misc.automount
● │ │ ├─rhel-autorelabel.service
● │ │ ├─rhel-domainname.service
● │ │ ├─rhel-import-state.service
● │ │ ├─rhel-loadmodules.service
● │ │ ├─sys-fs-fuse-connections.mount
● │ │ ├─sys-kernel-config.mount
● │ │ ├─sys-kernel-debug.mount
● │ │ ├─systemd-ask-password-console.path
● │ │ ├─systemd-binfmt.service
● │ │ ├─systemd-firstboot.service
● │ │ ├─systemd-hwdb-update.service
● │ │ ├─systemd-journal-catalog-update.service
● │ │ ├─systemd-journal-flush.service
● │ │ ├─systemd-journald.service
● │ │ ├─systemd-machine-id-commit.service
● │ │ ├─systemd-modules-load.service
● │ │ ├─systemd-random-seed.service
● │ │ ├─systemd-sysctl.service
● │ │ ├─systemd-tmpfiles-setup-dev.service
● │ │ ├─systemd-tmpfiles-setup.service
● │ │ ├─systemd-udev-trigger.service
● │ │ ├─systemd-udevd.service
● │ │ ├─systemd-update-done.service
● │ │ ├─systemd-update-utmp.service
● │ │ ├─systemd-vconsole-setup.service
● │ │ ├─cryptsetup.target
● │ │ ├─local-fs.target
● │ │ │ ├─-.mount
● │ │ │ ├─rhel-readonly.service
● │ │ │ └─systemd-remount-fs.service
● │ │ └─swap.target
● │ └─timers.target
● │   └─systemd-tmpfiles-clean.timer
● ├─getty.target
● │ └─getty@tty1.service
● ├─nfs-client.target
● │ ├─auth-rpcgss-module.service
● │ ├─rpc-statd-notify.service
● │ └─remote-fs-pre.target
● └─remote-fs.target
●   └─nfs-client.target
●     ├─auth-rpcgss-module.service
●     ├─rpc-statd-notify.service
●     └─remote-fs-pre.target
``` 

  

systemd 与 runlevel 的对应: 

![image2021-9-23 11:58:6.png](/assets/01KJBYEXSH3QYZQX5PG6DMJ36S/image2021-9-23%2011%3A58%3A6.png)

  

改变systemd的runlevel: 

```
$ sudo systemctl set-default rescue.target
Created symlink /etc/systemd/system/default.target → /lib/systemd/system/rescue.target.
```
