---
title: 20250626 - 将Mac OSX和Linux的音频双向转发
confluence_page_id: 4161570
created_at: 2025-06-26T08:48:15+00:00
updated_at: 2025-06-26T11:36:36+00:00
---

# ~~从 Linux 往 Mac 输出音频~~

~~在Mac OSX上:~~

  - ~~用MacPorts 安装 pulseaudio : sudo port install pulseaudio~~
  - ~~增加文件: ~/.config/pulse/[default.pa](<http://default.pa>) , 内容增加: ~~
    - ~~load-module module-native-protocol-tcp auth-anonymous=1~~
    - ~~load-module module-coreaudio-detect~~
  - ~~增加文件: ~/.config/pulse/daemon.conf, 内容增加: exit-idle-time = -1~~
  - ~~安装和启动dbus服务~~
    - ~~sudo port install dbus~~
    - ~~sudo port load dbus~~
    - ~~sudo launchctl start org.freedesktop.dbus-system~~
  - ~~重启pulseaudio服务:~~
    - ~~pulseaudio --kill~~

    - ~~pulseaudio --start~~

  - ~~检查状态:~~
    - ~~export PULSE_SERVER=tcp:127.0.0.1:4713~~
    - ~~pulseaudio --check~~
    - ~~pactl info -s tcp:127.0.0.1:4713~~
    - ~~pactl list modules~~
    - ~~pactl list sinks short : 列出音频设备~~

~~在Linux上:~~

  - ~~从Mac OSX上转发端口: ssh -R 4714:127.0.0.1:4713[root@10.186.58.23](<mailto:root@10.186.58.23>)~~
  - ~~安装pulseaudio: apt install pulseaudio~~
  - ~~增加文件: ~/.config/pulse/[default.pa](<http://default.pa>), 内容增加: load-module module-tunnel-sink server=127.0.0.1:4714 sink_name=macos_output~~
  - ~~增加文件: ~/.config/pulse/daemon.conf, 内容增加: exit-idle-time = -1~~
  - ~~pulseaudio -vvv, 检查连接~~
  - ~~pulseaudio --start~~
  - ~~检查音频设备:~~
    - ~~export PULSE_SERVER=tcp:127.0.0.1:4714~~
    - ~~pactl list sinks short 应当输出音频设备~~
  - ~~输出音频:~~
    - ~~PULSE_SINK=Channel_1__Channel_2.4 paplay /usr/share/sounds/alsa/Front_Center.wav~~

# 重新梳理步骤, 建立双向连接

Mac端: 

  - 用MacPorts 安装 pulseaudio : sudo port install pulseaudio
  - 增加文件: ~/.config/pulse/daemon.conf, 内容增加: exit-idle-time = -1
  - 安装和启动dbus服务
    - sudo port install dbus
    - sudo port load dbus
    - sudo launchctl start org.freedesktop.dbus-system
  - 配置文件: ~/.config/pulse/[default.pa](<http://default.pa/>)

```
load-module module-native-protocol-unix
load-module module-native-protocol-tcp auth-anonymous=1 port=4715

load-module module-coreaudio-detect
load-module module-tunnel-sink server=tcp:127.0.0.1:4713 sink_name=to_linux
```

  - 启动服务
    - pulseaudio -vvv

  - (在Linux端启动后)
  - 加载sink:  

    - pactl load-module module-tunnel-sink server=tcp:127.0.0.1:4713 sink_name=to_linux
  - 连接麦克风: 
    - pactl load-module module-loopback source=Channel_1 sink=to_linux
    - 其中: Channel_1 通过以下命令查询: 

```
pactl -s unix:/Users/tachikoma/.config/pulse/2a2e10f3be6384495e0f97cd665c50bd-runtime/native list short sources
 
(unix socket路径在 pulseaudio的启动日志中)
```

  - 向Linux端建立隧道: ssh -L 4713:localhost:4714 -R 4716:localhost:4715 root[@10.186.58.23](<mailto:user@10.186.58.23>)

Linux端: 

  - 安装pulseaudio: apt install pulseaudio
  - 增加文件: ~/.config/pulse/daemon.conf, 内容增加: exit-idle-time = -1
  - 启动服务: 
    - pulseaudio -vvv
  - 检查状态: 
    - export PULSE_SERVER=tcp:127.0.0.1:4714
    - pactl info
  - 测试播放音频: 
    - apt install alsa-utils (安装wav样例)
    - paplay /usr/share/sounds/alsa/Front_Center.wav
  - 录制音频:
    - parec --format=s16le --rate=44100 --channels=2 --file-format=wav test.wav
    - paplay test.wav
