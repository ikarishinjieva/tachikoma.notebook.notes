---
title: 20210526 - 农行 C# 程序 SSL超时
confluence_page_id: 1146886
created_at: 2021-05-26T13:19:21+00:00
updated_at: 2021-05-27T08:59:10+00:00
---

# 现象

C#程序报错: 

![image2021-5-26 21:15:32.png](/assets/01KJBYDCR25528Q2TPMH0SAR9P/image2021-5-26%2021%3A15%3A32.png)

抓包: 

[附件: 100.186.cap.zip] 

判断: C#程序在StartTLS中 hang, 客户端经过15s后超时 (driver的connect timeout 为15s), MySQL端经过10s超时断开连接 (MySQL的connect timeout为10s)

# 诊断

使用.Net framework 4.7, 使用样例程序, 开启tracing功能

注: .Net framework 5+版本 tracing功能无法调试成功

得到正常的tracing日志: 

[附件: normal_connection.txt] 

正常的抓包: 

[附件: p8021] 

得知: 故障时, mysql driver应当耗时在以下日志之间: 

```
System.Net Information: 0 : [25168] InitializeSecurityContext(credential = System.Net.SafeFreeCredential_SECURITY, context = 10a36c8:10b3fe0, targetName = 10.186.62.73, inFlags = ReplayDetect, SequenceDetect, Confidentiality, AllocateMemory, InitManualCredValidation)
System.Net Information: 0 : [25168] InitializeSecurityContext(In-Buffers 计数为 2，Out-Buffer 长度为 0，返回的代码为 OK)。
System.Net Information: 0 : [25168] 远程证书: [Version]
  V3

[Subject]
  CN=MySQL_Server_8.0.21_Auto_Generated_Server_Certificate
  Simple Name: MySQL_Server_8.0.21_Auto_Generated_Server_Certificate
  DNS Name: MySQL_Server_8.0.21_Auto_Generated_Server_Certificate

[Issuer]
  CN=MySQL_Server_8.0.21_Auto_Generated_CA_Certificate
  Simple Name: MySQL_Server_8.0.21_Auto_Generated_CA_Certificate
  DNS Name: MySQL_Server_8.0.21_Auto_Generated_CA_Certificate

[Serial Number]
  02

[Not Before]
  2021/5/21 13:07:50

[Not After]
  2031/5/19 13:07:50

[Thumbprint]
  4783FCDF2EBC1CB6950C7DA4B7031F5497108DFC

[Signature Algorithm]
  sha256RSA(1.2.840.113549.1.1.11)

[Public Key]
  Algorithm: RSA
  Length: 2048
  Key Blob: 30 82 01 0a 02 82 01 01 00 c9 14 d5 09 4f d9 7d 91 40 8a f2 87 19 26 1c 69 f1 9a 92 28 19 06 75 57 21 dc b0 01 e7 6d 08 4d 35 03 6c 00 a9 9e 99 3f 05 30 be ae a6 15 07 ba 3e 69 aa 9e 8e 86 82 16 88 f7 a2 47 c1 67 98 28 6a 7a 36 8d 53 0a c3 16 31 35 b3 92 e6 6d 3e c6 9c a2 b0 f9 be 4a 52 38 d8 e7 d2 7f 87 11 f0 e2...。
System.Net Information: 0 : [25168] SecureChannel#26753075 - 远程证书有错误:
System.Net Information: 0 : [25168] SecureChannel#26753075 - 	证书名不匹配。
System.Net Information: 0 : [25168] SecureChannel#26753075 - 	已处理证书链，但是在不受信任提供程序信任的根证书中终止。

System.Net Information: 0 : [25168] SecureChannel#26753075 - 远程证书经用户验证为有效。
System.Net Information: 0 : [25168] EndProcessAuthentication(协议=Tls12，密码=Aes256 256 位强度，哈希=Sha384 384 位强度，密钥交换=44550 255 位强度)。
“ConsoleApp1.exe”(CLR v4.0.30319: ConsoleApp1.exe): 已加载“C:\WINDOWS\Microsoft.Net\assembly\GAC_MSIL\System.Management\v4.0_4.0.0.0__b03f5f7f11d50a3a\System.Management.dll”。已跳过加载符号。模块进行了优化，并且调试器选项“仅我的代码”已启用。

``` 

怀疑在证书验证过程中, 有步骤卡住或等待超时, 导致整个操作变慢. 

参考: 

  - <https://crushonme.github.io/2018/05/30/Response-Delay-12-Seconds-With-HTTPS-Protocol/>
  - <https://blogs.iis.net/tomchris/web-site-stops-responding-for-15-25-seconds>

在故障的抓包中发现DNS包, 指向[ctldl.windowsupdate.com](<http://ctldl.windowsupdate.com>), 认为与微软的CTL自动更新机制有关, 参考: 

  - <https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn265983(v=ws.11)#registry-settings-modified>

给用户结论如下: 

```
Hi，我们目前的进展如下：
1. 今天进行了driver的调试重现，和SSL.ddl的调试重现，同时对比网络抓包的内容，可以看到故障时在等待SSL证书校验
2. 在故障抓包中，可以看到在等待SSL证书校验过程中，通过DNS查找了http://ctldl.windowsupdate.com域名
3. 目前认为问题出在windows的CTL自动刷新机制上， 参考如下：https://support.microsoft.com/en-us/topic/an-automatic-updater-of-untrusted-certificates-is-available-for-windows-vista-windows-server-2008-windows-7-and-windows-server-2008-r2-117bc163-d9e0-63ad-5a79-e61f38be8b77
4. 目前建议尝试参考中的方案，调整组策略关闭CTL自动刷新机制

以上仅是逻辑推断，不能确定是否跟windows的CTL其他机制相关，也请跟微软的support确认逻辑
``` 

# 关于样例程序

要点: 

  - linux上使用mono, SSL库包含的cipher不足够, 不包括我们需要的cipher: ECDHE-RSA-AES256-SHA384
  - .NET framework 5 / .NET core 3, 在无法使用App.config (没找到合适的方法将配置加载到system.diagnostics节)
  - 使用windows环境上的安装virtual studio, 配合.NET framework 4.6 (在一台机器上安装成功, 另外两台机器上安装都无法选择.NET framework 4.6, 只能使用跟linux相同的.NET版本)

主程序: 

```
#define TRACE
  
using System;
using MySql.Data.MySqlClient;
using System.Diagnostics;
using System.Configuration;

namespace ConsolApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            MySql.Data.MySqlClient.MySqlTrace.Switch.Level = System.Diagnostics.SourceLevels.Verbose;
            MySql.Data.MySqlClient.MySqlTrace.Listeners.Add(new ConsoleTraceListener());

            string cs = @"server=10.186.62.73;port=8021;userid=test;password=test;database=test;logging=true";

            MySqlConnection con = new MySqlConnection(cs);
            con.Open();

            Console.WriteLine($"MySQL version : {con.ServerVersion}");
            con.Close();
        }
    }
}
``` 

其中 以下部分, 可以打印 MySQL trace (参考: <https://dev.mysql.com/doc/connector-net/en/connector-net-programming-tracing.html>) :

  - #define TRACE
  - 为 MySqlTrace 配置Switch和Listener (也可以在App.config中配置)
  - 在连接字符串中加入logging=true  

App.config 中, 可以配置对网络库的trace, 配置文件如下: 

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    ...
	<system.diagnostics>
		<trace autoflush="true" />
		<sources>
			<source name="System.Net" maxdatasize="1024">
				<listeners>
					<add name="TraceFile" />
				</listeners>
			</source>
			<source name="System.Net.Sockets" maxdatasize="1024">
				<listeners>
					<add name="TraceFile" />
				</listeners>
			</source>
		</sources>
		<sharedListeners>
			<add name="TraceFile" type="System.Diagnostics.TextWriterTraceListener" initializeData="trace.log" />
		</sharedListeners>
		<switches>
			<add name="System.Net" value="Verbose" />
			<add name="System.Net.Sockets" value="Verbose" />
		</switches>
	</system.diagnostics>
</configuration>
```
