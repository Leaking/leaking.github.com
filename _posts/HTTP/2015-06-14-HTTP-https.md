---
layout: post
title: 《HTTP权威指南》Secure HTTP
excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
categories: HTTP
tags: [HTTP]
image:
  feature: http_definitive_guide.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

## 一，Https基础介绍

https是http的安全形式，由网景公司发明，使用https时，所有http请求和响应数据发送到网络之前，都要进行加密，这部分工作是由SSL/TLS层完成（TLS是SSL的现代替代协议，我们用SSL表示SSL/TLS），SSL/TLS层是介于应用层和TCP层的传输安全层（Transport Layer Security,TLS）。

<figure>
  <img src="{{ site.url }}/images/ssl.jpg" alt="search screenshot">
  <figcaption> SSL betweens HTTP and TCP</figcaption>
</figure>


在使用https情况下，http报文未加密前不会发送给TCP层，而是通过安全层进行加密再发送给TCP层。在安全层，发送端和接收端会进行一次SSL握手，检查，交换一些关于证书，密码的信息。





## 二，数字加密

上面大致讲了https，接下来是和https息息相关的加密概念。

#### 1，密码

密码是一种加密算法（加密函数），往其中输入明文，则结果会输出密文。而密码机是能快速运行复杂密码的机器

#### 2，密钥

密钥是密码的参数（意义类似于函数的参数），密钥不同，密码的工作方式就不同。密钥越长，编码解码算法（密码）被破解的可能性越低。


## 三，对称加密，非对称加密，公开密钥

这里说的对称加密和非对称加密，区别在于编码和解码的密钥否相同。一般我们只需注意到密钥的安全及其他特点，而不是关注密码，因为密码（加密算法）都是公开的，而且加密解密算法一般都是反函数关系。

#### 1，对称加密

顾名思义，对称加密就是编码和解码的密钥匙一样的。常见的对称加密有：DES，3DES，RC2，RC4。密钥更长，更能抵抗得住枚举攻击，标准长度为56的DES已经不够安全。

使用对称加密，发送端和接收端在进行通信之前，需要先共享密钥才能进行通信，而这也是对称加密的缺点之一，而且，如果有n个节点，每个节点要和其他n-1个节点进行通信，总共会产生大概n＊（n－1）个密钥，这将难以管理。

#### 2，非对称加密，公开密钥

其实非对称加密和公开密钥可以算是同一个概念，平时说的非对称加密和公开密钥算法是一样的。因为非对称加密就是使用公开密钥的。确切地说是公开加密密钥，而解秘密钥匙保密的。这样一来，所有发送端都可以使用公开的加密密钥进行加密（而无需像对称加密一样，再通信之前先获取一个共享的密钥）。发送报文给接收端，接收端再使用保密的解密密钥获取明文。

非对称加密相比对称加密的一个缺点是，非对称加密算法比对称加密算法慢。

时下常用的非对称加密算法是RSA，还有用于对数字签名进行加密的DSA。

#### 3，混合加密

非对称加密算法（公开密钥加密算法）虽然好用，但是计算速度相比对称加密算法慢，所以我们可以混合这两种算法，先用非对称加密的方式，建立通信，然后在这个通信通道中产生并发送临时的随机对称密钥，然后使用更快对称加密方式处理其余数据。

四，数字签名

数字签名是一种加密的校验和，可以防止报文被篡改。数字签名也是通过加解密算法处理过的。对数字签名进行加密经常使用非对称算法DSA。

五，数字证书

数字证书用于代表某个权威机构，目前数字证书还没有一个全球标准，但是有一个大多数证书使用的标准———X.509.v3。


六，常用算法

对称算法：DES，3DES，RC2，RC4，AES 

非对称算法：RSA，DSA（Digital Signature Algorithm，数字签名算法），ECC

摘要算法（单向，不可逆）：MD5，SHA-1







