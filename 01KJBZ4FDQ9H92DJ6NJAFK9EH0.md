---
title: 20231215 - 配置minio, 将s3 API的加密套件配置为TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
confluence_page_id: 2589893
created_at: 2023-12-15T16:03:57+00:00
updated_at: 2023-12-16T02:35:56+00:00
---

容器启动minio: 

```
docker run  -p 9010:9010 -p 9019:9019 --name minio \
 -d \
 -e MINIO_ACCESS_KEY=minio \
 -e MINIO_SECRET_KEY=minio@123 \
  minio/minio server /data  --console-address ":9010" --address ":9019"
``` 

打开界面 9010端口, 增加Key: 

```
Access Key: ToofGsZnUQYLnE8cou66
Secret Key: 0KJOIl2sIqlgvw2UpfzT0BMOiePilxFhLcYW2Dfq

``` 

安装aws cli, 进行配置: (参考: <https://min.io/docs/minio/linux/integrations/aws-cli-with-minio.html>)

```
aws configure
aws configure set default.s3.signature_version s3v4

aws --endpoint-url http://10.186.16.126:9019/ s3 ls
``` 

生成秘钥: 

```
openssl ecparam -name prime256v1 -genkey -noout -out ecdh-key.pem
openssl genpkey -algorithm RSA -out rsa-key.pem -pkeyopt rsa_keygen_bits:2048
openssl req -new -key rsa-key.pem -out rsa-csr.pem
openssl x509 -req -days 365 -in rsa-csr.pem -signkey rsa-key.pem -out rsa-cert.pem
``` 

<https://min.io/docs/minio/container/operations/network-encryption.html>

使用nginx: 

```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    server {
        listen 443 ssl;
        server_name test_minio;

        ssl_certificate /etc/nginx/rsa-cert.pem;
        ssl_certificate_key /etc/nginx/rsa-key.pem;

        ssl_ciphers 'ECDHE-RSA-AES256-SHA';
        ssl_protocols TLSv1.2;
        
        # Modern security practices would recommend the following instead:
        # ssl_ciphers 'ECDHE-RSA-AES256-GCM-SHA384';
        # ssl_protocols TLSv1.2 TLSv1.3;

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout 10m;

        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_pass http://10.186.16.126:9090;
        }

        error_log /var/log/nginx/error.log debug;
    }
}
``` 

拉起nginx容器: 

```
docker run --name nginx-minio -v /tmp/nginx.conf:/etc/nginx/nginx.conf:ro -v /tmp/2/rsa-cert.pem:/etc/nginx/rsa-cert.pem:ro -v /tmp/2/rsa-key.pem:/etc/nginx/rsa-key.pem:ro -p 9443:443 -d nginx
``` 

测试连接: 

```
root@R740-26:/tmp/2# openssl s_client -connect 10.186.16.126:9443 -tls1_2 -cipher 'ECDHE-RSA-AES256-SHA'
CONNECTED(00000005)
depth=0 C = AU, ST = Some-State, O = Internet Widgits Pty Ltd
verify error:num=18:self signed certificate
verify return:1
depth=0 C = AU, ST = Some-State, O = Internet Widgits Pty Ltd
verify return:1
---
Certificate chain
 0 s:C = AU, ST = Some-State, O = Internet Widgits Pty Ltd
   i:C = AU, ST = Some-State, O = Internet Widgits Pty Ltd
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIDETCCAfkCFERGqS3cdctreHWmwF9L+dPiQwNkMA0GCSqGSIb3DQEBCwUAMEUx
CzAJBgNVBAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMSEwHwYDVQQKDBhJbnRl
cm5ldCBXaWRnaXRzIFB0eSBMdGQwHhcNMjMxMjE1MTYxNTU5WhcNMjQxMjE0MTYx
NTU5WjBFMQswCQYDVQQGEwJBVTETMBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UE
CgwYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMIIBIjANBgkqhkiG9w0BAQEFAAOC
AQ8AMIIBCgKCAQEA0/fLe/DPVi4IbCQeq7wHoH0QTGnbqvrSYTns6DZ8Ddkb3Kva
CM2CQdzH2uWwZdJz6UJAPM1qLqNIWmVHzGwhxN+jvKxYvxNZhmkLWUsVv/R3HOkG
Gs4GYqWeYVGjmCfl392plP7jRc1EJUkfKP2fN75CZ87pnhnGcIomMAHyH7Va+ttC
PFWzsfG4LSFkV7zRbyA3RPr1dnv/Y4pwm/uoOPEEUML5yynsi3dVidlBUZjLScx3
jvtPb+C82vxcu0Xg4xUcx5M/XraMZGUyhsYOqRnw5yJxOdy7baHS0v0MQ97QB/Wp
ko+rfXKWfX5TDx0Mo54bTxjV3C/Okrz0HVM9HwIDAQABMA0GCSqGSIb3DQEBCwUA
A4IBAQAnAFjYDUFotJsuIscUkuoEFyEmGfOkjuK3kgr2++gSTDWCK8pIOd3ZcSO4
X36NZO1g7ZWChsrmm9d74wnCQ44Vh/krYxYPojSXiXMjTWUaaLRTkY2qE8n8PNLV
JGwNFARQ3gcucRSiFa/60USzib67ywVJWiDyIxx6+ZxniAt5EznGjVRc1gEPfKpO
OQkAKp2AQddmNfxp+9NOxq2JY5PuKZBWasQey/S2JCmsR+iP8j4Jmcz10KDCHmoY
rQrF6pghJoBRKbpU0iz0sHaa/PoxoK7so1CrdYEDKsKYpvn33Wr5KCABBNOJsx1x
uCw+kC0tqPlSC0hW2SSX0JBIRQCv
-----END CERTIFICATE-----
subject=C = AU, ST = Some-State, O = Internet Widgits Pty Ltd

issuer=C = AU, ST = Some-State, O = Internet Widgits Pty Ltd

---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1498 bytes and written 285 bytes
Verification error: self signed certificate
---
New, TLSv1.0, Cipher is ECDHE-RSA-AES256-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-SHA
    Session-ID: 99CE37A4E69C28A2DB4CB5BF184F8B27F89936BAE99D140B92B7B2F164435C8D
    Session-ID-ctx:
    Master-Key: 1F7EB6A94E7533411F11DBCBCC657FF30424C5F591C1AE7B787FC051692347CED2298D0DE3CB7C7A1270F2E319FA13A6
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 600 (seconds)
    TLS session ticket:
    0000 - 63 1f 42 06 ea 9a 16 f6-5c d7 0f 6a 3d 50 df cf   c.B.....\..j=P..
    0010 - 04 5b fd bf bf 3e 58 f6-0d 83 84 32 94 81 e8 4e   .[...>X....2...N
    0020 - 45 cf 3c bf 47 e0 3f 66-9f 3d 86 b9 23 44 54 24   E.<.G.?f.=..#DT$
    0030 - e6 05 e4 6f 9f e3 97 57-a0 14 6a d2 d5 7d 03 f2   ...o...W..j..}..
    0040 - a5 30 02 6d 9f 5a 13 9a-cb b2 64 24 ee 46 0a 8f   .0.m.Z....d$.F..
    0050 - 99 a3 12 e2 46 55 fa 1f-cd 7a 9f 1f 8c 47 e5 3e   ....FU...z...G.>
    0060 - 20 3e 5e 39 c4 51 d8 3a-cf d1 f2 26 28 b3 15 13    >^9.Q.:...&(...
    0070 - fa e8 75 35 d7 97 61 b0-9f b4 b7 de 15 15 23 bb   ..u5..a.......#.
    0080 - 0c bd db 79 68 b8 e9 4c-f3 1f e1 81 3a bc 0b 79   ...yh..L....:..y
    0090 - b7 ff 46 0c aa 27 e7 54-1d a4 d0 da a7 df 63 5d   ..F..'.T......c]
    00a0 - db 06 a8 1a 78 e4 eb 52-b5 ed da e9 4f b3 cb ba   ....x..R....O...
    00b0 - c7 a1 53 76 3d 65 88 da-d5 c8 52 d2 5e 47 e4 70   ..Sv=e....R.^G.p
    00c0 - bd 02 ad cc 7c e1 3f a8-6f 37 da 7e 9d a6 d0 c3   ....|.?.o7.~....

    Start Time: 1702694119
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: yes
---

```
