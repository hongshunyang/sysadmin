## Intro To PKI

> [Intro PKI](https://github.com/QueuingKoala/easyrsa3/blob/master/doc/Intro-To-PKI.md)


### Connect

* 0、初始化PKI

CA 创建人构建PKI， PKI相当与一个数据库，主要用来方便管理根证书，根密钥对，由客户端和服务端发送至CA的证书请求信息，
，此请求信息中包含了请求端的公有密钥（请求端持有私有密钥，同前述公有密钥共同构成请求端的密钥对，共有密钥和私有密钥必须配对（同时生成），同时包含物理机硬件特征信息）
和物理机特征信息（用来标识物理硬件），经过CA签名的数字证书（也就是颁发数字证书：CA利用自己的思域和证书信息对前述请求进行签名，生成新的证书）

```
easy-rsa init-pki
```

* 1、证书颁发机构CA／授权机构

CA的核心有ca.crt（根证书）和private/ca.key（私有密钥） 组成，当创建CA时自动生成这俩部分。
在生成ca.key时一般需要密码保护，主要在用来对请求授权的证书签名时用到，安全可信.
ca.crt中包含着颁发机构的信息，比如地理位置，组织名称等。

```
easy-rsa build-ca
```


* 2、密钥对，证书请求

终端主要分为服务端和客户端。
终端主要创建密钥对和证书请求，密钥对包含私有密钥和公有密钥，如0所述，公有密钥会被包含在请求之中，可以到处传播，不用担心其安全性
而私有密钥必须保存好，当你使用公共电脑时，最好在创建密钥对时设置密码，确保其他人未经允许无法使用你的私有密钥。
同时也要防止丢失，做好备份，一旦丢失，则意味着要重新向CA申请签名的证书。
当创建好密钥对时，就可以向CA发送证书请求了，可以通过email，web等方式进行，这个请求


```
easy-rsa gen-req serverReq1
>reqs/serverReq1.req
>private/serverReq1.key
```

其中 serverReq1.req 是证书请求文件（包含公有密钥和物理机特征信息，时间等）
serverReq1.key 是私有密钥必须保持好，当req经过ca签名之后，用来和授权颁发的证书相对应

* 3、颁发证书

导入证书请求

当CA 对证书进行授权／签名／颁发之前，要先导入req:

```
easy-rsa import-req path/to/serverReq1.req serverReq1
>pki/reqs/serverReq1.req
```

serverReq1 是导入ca pki 的证书请求文件名，保存在pki/reqs/serverReq1.req


签名／授权证书

指定授权为终端服务还是终端客户端

根据需要指定授权有效日期，默认3650天

```
easy-rsa sign-req server serverReq1(前述导入的文件名)
>pki/issued/serverReq1.crt
```
pki/issued/serverReq1.crt 就是经过签名的有效证书，默认3650天有效

* 4、openvpn服务器／客户端／颁发机构工作案例

颁发机构：

```
easyrsa init-pki
easyrsa build-ca

> pki/ca.crt
> pki/private/ca.key

```

openvpn服务器：

```
easyrsa gen-req openvpn_server1

> pki/private/openvpn_server1.key
> pki/reqs/openvpn_server1.req

#submit pki/reqs/openvpn_server1.req to ca

##密钥／ddos
easyrsa gen-dh
penvpn --genkey --secret pki/ta.key

>pki/dh.pem
>pki/ta.key

```

颁发机构：

```
easyrsa import-req path/to/openvpn_server1.req os1
easyrsa sign-req server os1
> pki/issued/os1.crt
````

openvpn服务端：

```
cp path/to/{ca.crt,openv_server1.key,os1.crt,dh.pem,ta.key} > /etc/openvpn/keys/
vim /etc/openvpn/server.conf
##
...
ca keys/ca.crt
cert keys/os1.crt
key keys/openvpn_server1.key
dh keys/dh.pem
...
tls-auth keys/ta.key 0
...
##

```
openvpn 客户端：

```
easyrsa gen-req client1

> pki/private/client1.key
> pki/reqs/client1.req

# submit pki/reqs/client1.req to ca

```

颁发机构：

```
easyrsa import-req path/to/client1.req ct1
easyrsa sign-req client ct1

>pki/issued/ct1.crt
```

openvpn客户端：

```
cp path/to/{ca.crt,ct1.crt,client1.key,ta.key} > config/

## vim client.ovpn
...
ca ca.crt
cert ct1.crt
key client1.key
tls-auth ta.key 1
...

```
