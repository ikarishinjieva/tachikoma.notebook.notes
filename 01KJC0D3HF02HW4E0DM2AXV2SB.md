---
title: 20260209 - arduino使用, 接入压敏传感器
confluence_page_id: 4620457
created_at: 2026-02-09T05:20:54+00:00
updated_at: 2026-02-09T05:20:54+00:00
---

y单片压敏传感器使用

压敏薄膜接入 压敏感应模块, 感应模块接入 arduino: 

  - A0接 arduino A0
  - GND 接 arduino GND
  - ACC 接 arduino 5V

配置 arduino: 

```
apt install arduino
``` 

在桌面打开arduino: 

![image2026-2-9 13:6:46.png](/assets/01KJC0D3HF02HW4E0DM2AXV2SB/image2026-2-9%2013%3A6%3A46.png)

选择型号和端口: 

![image2026-2-9 13:9:18.png](/assets/01KJC0D3HF02HW4E0DM2AXV2SB/image2026-2-9%2013%3A9%3A18.png)

粘贴代码: 

```
/*
 * 专为 Python 通信优化的压敏传感器代码
 * 接口: A0
 * 功能: 只通过串口发送纯数字 (0-1023)
 */

const int sensorPin = A0; // 传感器连接到 A0

void setup() {
  // 初始化串口，波特率 9600
  // 注意：Python 代码中的 baud_rate 必须也是 9600
  Serial.begin(9600);
}

void loop() {
  // 1. 读取模拟值
  int sensorValue = analogRead(sensorPin);
  
  // 2. 发送数据
  // Serial.println 会发送 "数字 + \r\n" (换行符)
  // Python 的 readline() 正好利用这个换行符来判断数据结束
  Serial.println(sensorValue);
  
  // 3. 延时控制
  // delay(50) 意味着每秒发送约 20 次数据
  // 这个频率对压敏传感器来说足够流畅，也不会让 Python 处理不过来
  delay(50); 
}
``` 

进行编译和上传: 

![image2026-2-9 13:12:40.png](/assets/01KJC0D3HF02HW4E0DM2AXV2SB/image2026-2-9%2013%3A12%3A40.png)

用右上角的放大镜查看结果: 

![image2026-2-9 13:13:17.png](/assets/01KJC0D3HF02HW4E0DM2AXV2SB/image2026-2-9%2013%3A13%3A17.png)

压敏结果正确

使用python脚本读取压力值: 

```
import serial
import time

# --- 配置区域 ---
# 请根据你的实际情况修改端口号
# Linux 上通常是 /dev/ttyACM0 或 /dev/ttyUSB0
SERIAL_PORT = '/dev/ttyUSB0'
BAUD_RATE = 9600

def main():
    try:
        # 初始化串口
        ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
        print(f"连接成功: {SERIAL_PORT}")
        print("等待数据中... (按 Ctrl+C 停止)")

        # 等待 Arduino 重启稳定
        time.sleep(2)

        # 清空一下缓冲区，防止读取到积压的旧数据
        ser.reset_input_buffer()

        while True:
            if ser.in_waiting > 0:
                # 1. 读取一行，解码，去除首尾空白符
                line = ser.readline().decode('utf-8', errors='ignore').strip()

                # 2. 校验数据是否有效 (必须是纯数字)
                if line.isdigit():
                    pressure = int(line)

                    # --- 在这里处理你的业务逻辑 ---
                    # 例如：打印进度条效果
                    bar_length = int(pressure / 20) # 缩放一下以便显示
                    bar = '#' * bar_length
                    print(f"压力值: {pressure:4d} |{bar}")

                    if pressure > 800:
                        print(">>> 警告：重压！ <<<")
                    # ---------------------------
                else:
                    # 如果收到空行或乱码，直接忽略
                    pass

    except serial.SerialException:
        print(f"错误: 无法打开端口 {SERIAL_PORT}")
        print("请检查：1. USB是否插好？ 2. Arduino IDE 串口监视器是否已关闭？")
    except KeyboardInterrupt:
        print("\n程序已停止。")
    finally:
        if 'ser' in locals() and ser.is_open:
            ser.close()

if __name__ == "__main__":
    main()
root@bot:/tmp/yamin#
``` 

最大压力值 663
