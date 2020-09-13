---
title: Security
date: 2020-07-27 11:14:50
tags:
    - 加解密
    - openssl
    - ssh
    - git
---

加解密以及ssh 登录原理备忘
<!-- more -->

# Security

## 加解密

### 对称加密

加密和解密使用同一套秘钥，加密强度高，难以被破解。随之而来的问题就是如何安全的保存秘钥，特别是客户端庞大的时候，很难保证秘钥不被泄露。

### 非对称加密

公钥和私钥

公钥和私钥都可以加密和解密，公钥加密私钥解密，私钥加密公钥解密。

#### 中间人攻击

中间人在客户端和服务端中间，让客户端误以为是真实的服务端，从而获取相关内容。

#### 数字签名和数字证书

数字签名：对data 进行hash，将hash 生成的摘要用私钥进行加密。并附在data 发给对方。对方使用公钥进行解密，获取了hash值和data，对data 进行hash 和机密的摘要进行对比。判断内容是否被篡改。

这有个问题就是公钥的分发可能被欺骗，比如我接收银行的公钥，对银行发送给我的内容进行验证。结果黑客将我保存的银行的公钥替换为黑客的公钥。这个时候我还以为我收到的是银行的公钥，这个时候黑客就可以让我误以为使用了银行的公钥进行通信。



数字证书，找一个第三方对银行的公钥进行公证，后续我使用都回去公证中心进行验证数据是否正确。

## SSH

ssh 主要用于计算机之间的加密登录，仅仅是一个标准的协议，具体的实现有很多种，目前使用范围最广泛的是OpenSSH。

```shell
ssh user@host -p port
```

ssh 不像https 有证书中心进行公钥的公证，那么就有可能在客户端和服务端中间存在"中间人攻击"。

### ssh 如何解决中间人攻击

#### 口令登录

通常在第一次登录的时候，系统会出现下面提示信息：

```shell
ssh user@host
The authenticity of host 'host (12.18.429.21)' can't be established.
RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.
Are you sure you want to continue connecting (yes/no)?
```

上面大概意思是，无法确定主机的真实性，不过知道他的公钥信息，询问你是否继续？

所谓公钥信息，就是指公钥的数字摘要(通过MD5生成的)。

很自然的一个问题就是如何知道远程主机的公钥指纹是多少？远程主机必须在自己的网站上贴出公钥指纹，以方便用户自己比较。

远程主机的公钥被接收后，会被保存在文件~/.ssh/known_hosts 中，下一次再连接该主机的时候，就会自动跳过警告部分，直接要求输入密码了。

#### 公钥登录

使用口令登录，每一次都需要输入密码，比较麻烦，ssh 提供了另外一种免去输入密码的登录方式：公钥登录

公钥登录：客户端自己生产公钥和私钥密钥对，将自己的**公钥**保存在远程主机上。

登录原理：远程主机向客户端发送一个随机字符串，用户用私钥加密后，再发回来。远程主机用公钥进行解密，并将机密后的字符串和随机字符串比对，如果成功，允许登录shell，不再要求密码。

```shell
#远程主机的authorized_keys 文件保存了用户的公钥
ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```



## openssl

```shell
#生成私钥
openssl genrsa > cert.key

#从私钥中获取公钥
openssl rsa -in cert.key -pubout -out pub.pem
openssl rsa -in cert.key -pubout -text -out pub.pem

#生成证书
openssl req -new -x509 -key cert.key > cert.pem

#生成证书的fingerprint
openssl x509 -fingerprint -sha256 -in cert.pem
```