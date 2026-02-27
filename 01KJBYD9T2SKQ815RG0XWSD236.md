---
title: 20210330 - 农行 JDK导致mysql connector报错
confluence_page_id: 753749
created_at: 2021-03-30T11:43:05+00:00
updated_at: 2021-03-30T14:43:12+00:00
---

# 现象

IBM JDK 版本: 

```
[root@10-186-63-146 mnt]# java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 8.0.6.26 - pxa6480sr6fp26-20210216_01(SR6 FP26))
IBM J9 VM (build 2.9, JRE 1.8.0 Linux amd64-64-Bit Compressed References 20210216_465732 (JIT enabled, AOT enabled)
OpenJ9   - e5f4f96
OMR      - 999051a
IBM      - 358762e)
JCL - 20210108_01 based on Oracle jdk8u281-b09
``` 

OpenJDK 版本: 

```
[root@db58 mnt]# java -version
openjdk version "1.8.0_275"
OpenJDK Runtime Environment (build 1.8.0_275-b01)
OpenJDK 64-Bit Server VM (build 25.275-b01, mixed mode)
``` 

MySQL connector 从 8.0.18 升级到 8.0.20, 在IBM JDK上报错, 在OpenJDK上执行正常

测试代码: 

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class test{
        public static void main(String[] args) throws Exception {
                Class.forName("com.mysql.cj.jdbc.Driver");
                Connection con = DriverManager.getConnection("jdbc:mysql://10.186.65.105:3306/dota", "root", "123456");
                PreparedStatement sql1 = con.prepareStatement("select a from t1 where b=? and c=?;");
                sql1.setInt(1,1);
                sql1.setInt(2,2);
                System.out.println("Executing query...");
                ResultSet rs = sql1.executeQuery();
                while (rs.next()) {
                        System.out.println(rs.getDouble(1));
                        }
                rs.close();
                sql1.close();
                con.close();
        }
}
``` 

报错日志: 

```
Caused by: javax.net.ssl.SSLHandshakeException: No appropriate protocol (protocol is disabled or cipher suites are inappropriate)
	at com.ibm.jsse2.Z.<init>(Z.java:287)
	at com.ibm.jsse2.aa.<init>(aa.java:33)
	at com.ibm.jsse2.ba.a(ba.java:52)
	at com.ibm.jsse2.bi.a(bi.java:229)
	at com.ibm.jsse2.bi.startHandshake(bi.java:212)
	at com.mysql.cj.protocol.ExportControlled.performTlsHandshake(ExportControlled.java:336)
	at com.mysql.cj.protocol.StandardSocketFactory.performTlsHandshake(StandardSocketFactory.java:188)
	at com.mysql.cj.protocol.a.NativeSocketConnection.performTlsHandshake(NativeSocketConnection.java:99)
	at com.mysql.cj.protocol.a.NativeProtocol.negotiateSSLConnection(NativeProtocol.java:325)
``` 

# 分析

使用java -verbose, 获取class loader顺序: 

```
java -cp $CLASSPATH:/opt/mysql-connector-java-8.0.20.jar -verbose test
``` 

使用 java -Djavax.net.debug=true 查看 TLS 的信息

[ibm_jdk_with_error.txt](/assets/01KJBYD9T2SKQ815RG0XWSD236/ibm_jdk_with_error.txt)

可以看到日志中TLS12中没有合适的cipher suite: 

```
javax.net.ssl|FINE|01|main|2021-03-30 11:47:33.493 UTC|Thread.java:1164|No available cipher suite for TLS13
javax.net.ssl|FINE|01|main|2021-03-30 11:47:33.493 UTC|Thread.java:1164|No available cipher suite for TLS12
javax.net.ssl|FINE|01|main|2021-03-30 11:47:33.493 UTC|Thread.java:1164|No available cipher suite for TLS11
javax.net.ssl|FINE|01|main|2021-03-30 11:47:33.494 UTC|Thread.java:1164|No available cipher suite for TLS10
``` 

由IBM的文档可知: (<https://www.ibm.com/support/knowledgecenter/SSYKE2_8.0.0/com.ibm.java.security.component.80.doc/security-component/jsse2Docs/ciphersuites.html>): 

![image2021-3-30 22:9:39.png](/assets/01KJBYD9T2SKQ815RG0XWSD236/image2021-3-30%2022%3A9%3A39.png)

IBM JDK 变更了 TLS_开头的cipher suite的名字 为 SSL_开头, 并仅在少数方法提供了向后兼容的特性. 文档提示可以通过 com.ibm.jsse2.overrideDefaultCSName变量使得IBM JDK中cipher suite的命名回归到之前的状态 (与Oracle JDK类似)

重新运行测试程序: 

```
[root@10-186-63-146 mnt]# java -cp $CLASSPATH:/opt/mysql-connector-java-8.0.20.jar -Dcom.ibm.jsse2.overrideDefaultCSName=true test
Executing query...
``` 

可正确执行.

添加 [javax.net](<http://javax.net>).debug=true 观察SSL的握手阶段: 

[ibm_jdk_with_overrideDefaultCSName.txt](/assets/01KJBYD9T2SKQ815RG0XWSD236/ibm_jdk_with_overrideDefaultCSName.txt)

```
javax.net.ssl|FINE|01|main|2021-03-30 14:13:29.736 UTC|Thread.java:1164|Consuming ServerHello handshake message (
"ServerHello": {
  "server version"      : "TLSv1.2",
  "random"              : "33 17 60 BC C2 2E 3B D3 AA 40 E3 AC 8E EB 6B B8 8B 69 44 4B 3C 1F 2C F0 44 4F 57 4E 47 52 44 01",
  "session id"          : "7C 9E C9 73 9A 3B F1 5E D4 0D D7 7F B5 55 5A 34 B3 08 F3 1F 30 37 84 2F 5E 9D B0 52 8B 62 DA 13",
  "cipher suite"        : "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384(0xC030)",
  "compression methods" : "00",
  "extensions"          : [
    "renegotiation_info (65,281)": {
      "renegotiated connection": [<no renegotiated connection>]
    },
    "ec_point_formats (11)": {
      "formats": [uncompressed, ansiX962_compressed_prime, ansiX962_compressed_char2]
    },
    "extended_master_secret (23)": {
      <empty>
    }
  ]
}
)
``` 

握手阶段选择了正确的cipher suite: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384

# 为什么 mysql-connector 8.0.18 没有问题

用[javax.net](<http://javax.net>).debug=true观察mysql-connector 8.0.18的表现: 

[ibm_jdk_with_mysql_connector_8018.txt](/assets/01KJBYD9T2SKQ815RG0XWSD236/ibm_jdk_with_mysql_connector_8018.txt)

```
Produced ClientHello handshake message (
"ClientHello": {
  "client version"      : "TLSv1.2",
  "random"              : "B6 0B CF 14 D5 3F 42 00 B5 5C 11 61 CA F7 E0 FD 8E F3 82 DC 96 F1 30 47 80 EC 7D A6 29 FC 44 9A",
  "session id"          : "",
  "cipher suites"       : "[SSL_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384(0xC02C), SSL_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256(0xC02B), SSL_ECDHE_RSA_WITH_AES_256_GCM_SHA384(0xC030), SSL_RSA_WITH_AES_256_GCM_SHA384(0x009D), SSL_ECDH_ECDSA_WITH_AES_256_GCM_SHA384(0xC02E), SSL_ECDH_RSA_WITH_AES_256_GCM_SHA384(0xC032), SSL_DHE_RSA_WITH_AES_256_GCM_SHA384(0x009F), SSL_DHE_DSS_WITH_AES_256_GCM_SHA384(0x00A3), SSL_ECDHE_RSA_WITH_AES_128_GCM_SHA256(0xC02F), SSL_RSA_WITH_AES_128_GCM_SHA256(0x009C), SSL_ECDH_ECDSA_WITH_AES_128_GCM_SHA256(0xC02D), SSL_ECDH_RSA_WITH_AES_128_GCM_SHA256(0xC031), SSL_DHE_RSA_WITH_AES_128_GCM_SHA256(0x009E), SSL_DHE_DSS_WITH_AES_128_GCM_SHA256(0x00A2), SSL_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384(0xC024), SSL_ECDHE_RSA_WITH_AES_256_CBC_SHA384(0xC028), SSL_RSA_WITH_AES_256_CBC_SHA256(0x003D), SSL_ECDH_ECDSA_WITH_AES_256_CBC_SHA384(0xC026), SSL_ECDH_RSA_WITH_AES_256_CBC_SHA384(0xC02A), SSL_DHE_RSA_WITH_AES_256_CBC_SHA256(0x006B), SSL_DHE_DSS_WITH_AES_256_CBC_SHA256(0x006A), SSL_ECDHE_ECDSA_WITH_AES_256_CBC_SHA(0xC00A), SSL_ECDHE_RSA_WITH_AES_256_CBC_SHA(0xC014), SSL_RSA_WITH_AES_256_CBC_SHA(0x0035), SSL_ECDH_ECDSA_WITH_AES_256_CBC_SHA(0xC005), SSL_ECDH_RSA_WITH_AES_256_CBC_SHA(0xC00F), SSL_DHE_RSA_WITH_AES_256_CBC_SHA(0x0039), SSL_DHE_DSS_WITH_AES_256_CBC_SHA(0x0038), SSL_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256(0xC023), SSL_ECDHE_RSA_WITH_AES_128_CBC_SHA256(0xC027), SSL_RSA_WITH_AES_128_CBC_SHA256(0x003C), SSL_ECDH_ECDSA_WITH_AES_128_CBC_SHA256(0xC025), SSL_ECDH_RSA_WITH_AES_128_CBC_SHA256(0xC029), SSL_DHE_RSA_WITH_AES_128_CBC_SHA256(0x0067), SSL_DHE_DSS_WITH_AES_128_CBC_SHA256(0x0040), SSL_ECDHE_ECDSA_WITH_AES_128_CBC_SHA(0xC009), SSL_ECDHE_RSA_WITH_AES_128_CBC_SHA(0xC013), SSL_RSA_WITH_AES_128_CBC_SHA(0x002F), SSL_ECDH_ECDSA_WITH_AES_128_CBC_SHA(0xC004), SSL_ECDH_RSA_WITH_AES_128_CBC_SHA(0xC00E), SSL_DHE_RSA_WITH_AES_128_CBC_SHA(0x0033), SSL_DHE_DSS_WITH_AES_128_CBC_SHA(0x0032), TLS_EMPTY_RENEGOTIATION_INFO_SCSV(0x00FF)]",
  "compression methods" : "00",
  "extensions"          : [
    "supported_groups (10)": {
      "versions": [secp256r1, secp384r1, secp521r1]
    },
    "ec_point_formats (11)": {
      "formats": [uncompressed]
    },
    "signature_algorithms (13)": {
      "signature schemes": [ecdsa_secp256r1_sha256, ecdsa_secp384r1_sha384, ecdsa_secp521r1_sha512, rsa_pss_rsae_sha256, rsa_pss_rsae_sha384, rsa_pss_rsae_sha512, rsa_pss_pss_sha256, rsa_pss_pss_sha384, rsa_pss_pss_sha512, rsa_pkcs1_sha256, rsa_pkcs1_sha384, rsa_pkcs1_sha512, dsa_sha256, ecdsa_sha224, rsa_sha224, dsa_sha224, ecdsa_sha1, rsa_pkcs1_sha1, dsa_sha1]
    },
    "signature_algorithms_cert (50)": {
      "signature schemes": [ecdsa_secp256r1_sha256, ecdsa_secp384r1_sha384, ecdsa_secp521r1_sha512, rsa_pss_rsae_sha256, rsa_pss_rsae_sha384, rsa_pss_rsae_sha512, rsa_pss_pss_sha256, rsa_pss_pss_sha384, rsa_pss_pss_sha512, rsa_pkcs1_sha256, rsa_pkcs1_sha384, rsa_pkcs1_sha512, dsa_sha256, ecdsa_sha224, rsa_sha224, dsa_sha224, ecdsa_sha1, rsa_pkcs1_sha1, dsa_sha1]
    },
    "extended_master_secret (23)": {
      <empty>
    },
    "supported_versions (43)": {
      "versions": [TLSv1.2, TLSv1.1, TLSv1]
    }
  ]
}
)
``` 

可以看到MySQL connector 8.0.18在建立TLS连接时, 就已经选择了正确的cipher suite的名字

通过MySQL release note 可知: 

![image2021-3-30 22:40:36.png](/assets/01KJBYD9T2SKQ815RG0XWSD236/image2021-3-30%2022%3A40%3A36.png)

MySQL 8.0.19 在TLS协商阶段指定了cipher suite的名字, 而之前的版本不指定cipher suite的名字

也就是说 8.0.19 明确指定的cipher suite的名字, 只与Oracle JDK定义兼容, 不与IBM JDK定义兼容

# 运维建议

在IBM JDK的启动参数中明确指定: com.ibm.jsse2.overrideDefaultCSName=true
