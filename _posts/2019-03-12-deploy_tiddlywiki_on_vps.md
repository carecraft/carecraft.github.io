---
layout:     post
title:      "在VPS上运行TiddlyWiki"
category : language-instrument
date:       2019-03-12
author:     "Max"
header-img: "img/post-2018.jpg"
catalog:    true
tags:
    - tw
---


是搭建个人云端笔记服务的一次尝试。

## 1. TiddyWiki 安装

[官网](https://tiddlywiki.com/)提供了详细教程，单机使用只要下载一个空的网页副本就可以了。对于存储，官网同样提供了多种方案，本例选择通过 Node.js 部署 tw 在 VPS 上。

1. 安装 Node.js

    可以直接从 http://nodejs.org 下载安装，或通过包管理器安装，如在 Debian/Ubuntu 系统上执行 `apt-get install nodejs`、在 Mac 上执行`brew install node`。

2. 通过 npm 安装 tw

    ```
    npm install -g tiddlywiki
    ```

    `-g`参数表示全局安装，否则 tw 将只在执行安装命令所在目录可用。 

3. 通过查看版本号验证 tw 安装是否成功

    ```
    tiddlywiki --version
    ```

## 2. TiddyWiki 启动

### 2.1 直接启动

创建一个 tw 服务的文件夹：
```
tiddlywiki mynewwiki --init server 
```

启动服务：
```
tiddlywiki mynewwiki --listen
```

访问 http://127.0.0.1:8080/  即可使用 tw 进行编辑。变更会自动保存在 VPS 上，点击 “save changes” 按钮可以下载离线副本。

### 2.2 通过 pm2 启动

直接启动的话，如果会话关闭，服务也会随之关闭。想要随时随地都能访问到我们的 wiki，需要在后台启动 tw。因为是 node 服务，我们可以用 pm2 来启动和管理。

1. 通过 npm 安装 pm2

    ```
    $ npm install -g pm2
    ```

2. 后台启动

    ```
    $ cd mynewwiki
    $ pm2 start --name mynewwiki /usr/bin/tiddlywiki -- --listen
    ```

3. 查看后台进程

    ```
    pm2 list

    ┌──────────┬────┬─────────┬──────┬───────┬────────┬─────────┬────────┬──────┬───────────┬──────┬──────────┐
    │ App name │ id │ version │ mode │ pid   │ status │ restart │ uptime │ cpu  │ mem       │ user │ watching │
    ├──────────┼────┼─────────┼──────┼───────┼────────┼─────────┼────────┼──────┼───────────┼──────┼──────────┤
    │ mynewwiki │ 0  │ N/A     │ fork │ 20942 │ online │ 0       │ 16h    │ 0.3% │ 84.9 MB   │ root │ disabled │
    └──────────┴────┴─────────┴──────┴───────┴────────┴─────────┴────────┴──────┴───────────┴──────┴──────────┘
     Use `pm2 show <id|name>` to get more details about an app
    ```

## 3. 安全验证

因为使用的是云服务器，一般默认不会对外暴露 8080 端口，因此需要在云服务器控制台添加安全组规则，允许访问指定端口。

此外，为了避免任何人登录皆可修改，本例选择使用 TW 自带的 TLS 安全连接功能并添加访问权限进行验证。

### 3.1 自签发 SSL IP证书

使用 TLS 验证需要 SSL 证书。证书可以从类似 https://letsencrypt.org/ 的网站申请，也可以自行签发。本例选择自行签发单 IP 证书。

1. 安装密钥生成工具

    ```
    $ yum install openssl openssl-devel
    ```

2. 生成根密钥

    ```
    $ openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048
    ```

3. 生成根 CA 证书

    ```
    $ openssl req -new -x509 -days 3650 -key /etc/pki/CA/private/cakey.pem -out  /etc/pki/CA/cacert.pem

    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [CN]:
    State or Province Name (full name) []:Beijing
    Locality Name (eg, city) [Beijing]:
    Organization Name (eg, company) [MX]:
    Organizational Unit Name (eg, section) []:note
    Common Name (eg, your name or your server's hostname) []:Aliyun, Global Root CA
    ```

4. 添加信任

    由于证书是自建的，所以要把根证书添加到受信任的根证书颁发机构，后续利用此 CA 签发的证书才会受信任，否则仍然会提示不可信。

    ```
    $ cat /etc/pki/CA/cacert.pem >> /etc/pki/tls/certs/ca-bundle.crt
    ```

    Windows 需要添加根证书至“受信任的根证书颁发机构”，macOS 需要将其导入“钥匙串访问”并选择信任。

5. 创建请求

    ```
    $ openssl genrsa -out /etc/pki/tls/127.0.0.1.key 2048
    $ openssl req -new -key /etc/pki/tls/127.0.0.1.key -out /etc/pki/tls/127.0.0.1.csr

    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [CN]:
    State or Province Name (full name) []:Beijing
    Locality Name (eg, city) [Beijing]:
    Organization Name (eg, company) [MX]:
    Organizational Unit Name (eg, section) []:note
    Common Name (eg, your name or your server's hostname) []:Aliyun, Global Root CA

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:
    ```

6. 附加用途

    ```
    $ vim /etc/pki/tls/openssl.cnf 

    ...
    [ v3_req ]
    basicConstraints=CA:TRUE
    keyUsage = nonRepudiation, digitalSignature, keyEncipherment
    subjectAltName=IP:127.0.0.1
    extendedKeyUsage = serverAuth,clientAuth
    ...
    ```

    配置文件中的 IP 地址 127.0.0.1 需要替换为实际 VPS 的公网 IP 地址。

7. 签发证书

    ```
    $ openssl ca -in /etc/pki/tls/127.0.0.1.csr -extensions v3_req -days 365 -out /etc/pki/tls/127.0.0.1.crt

    Using configuration from /etc/pki/tls/openssl.cnf
    Check that the request matches the signature
    Signature ok
    Certificate Details:
            Serial Number: 1 (0x1)
            Validity
                Not Before: Mar 12 06:09:00 2019 GMT
                Not After : Mar 11 06:09:00 2020 GMT
            Subject:
                countryName               = CN
                stateOrProvinceName       = Beijing
                organizationName          = MX
                organizationalUnitName    = note
                commonName                = Aliyun, Global Root CA
            X509v3 extensions:
                X509v3 Basic Constraints: 
                    CA:TRUE
                X509v3 Key Usage: 
                    Digital Signature, Non Repudiation, Key Encipherment
                X509v3 Extended Key Usage: 
                    TLS Web Server Authentication, TLS Web Client Authentication
                X509v3 Subject Alternative Name: 
                    IP Address:127.0.0.1
    Certificate is to be certified until Mar 11 06:09:00 2020 GMT (365 days)
    Sign the certificate? [y/n]:y


    1 out of 1 certificate requests certified, commit? [y/n]y
    Write out database with 1 new entries
    Data Base Updated
    ```
### 3.2 添加用户权限

```
$ vim mynewwiki/myusers.csv

username,password
jane,do3
andy,sm1th
roger,m00re
```

注：首行的 username 和 password 必须存在。

### 3.3 设置启动参数

```
$ cd mynewwiki/
$ cp /etc/pki/tls/127.0.0.1.crt .
$ cp /etc/pki/tls/127.0.0.1.key .
$ pm2 start --name mynewwiki /usr/bin/tiddlywiki -- --listen host=172.24.1.12 port=443 credentials=myusers.csv "readers=(authenticated)" writers=roger tls-key=127.0.0.1.key tls-cert=127.0.0.1.crt
```

TW listen 参数详见[官网](https://tiddlywiki.com/#WebServer)。host 选项指定 VPS 的网口 IP 地址。

TW 的 WebServer 虽然提供了多用户验证登录功能，但其仅为少量用户的可信任网络设计。在 Internet 公开使用最好添加额外网关。

## 4. 参考

1. [哆啦辉梦《在 Node.js 上运行 tiddlywiki》](http://1q88.cn/2018/tiddlywiki-node-deploy/)
2. [《正确使用 OpenSSL 自签发代码/邮件/域名/IP 证书（附免费可信任 IP 证书申请）》](https://vircloud.net/operations/sign-ip-crt.html)
