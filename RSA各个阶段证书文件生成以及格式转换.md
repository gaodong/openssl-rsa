RSA密钥生成

## 编码格式

**X.509** - 这是一种证书标准,主要定义了证书中应该包含哪些内容.其详情可以参考RFC5280,SSL使用的就是这种证书标准.

目前有以下两种编码格式.

**1、PEM** - Privacy Enhanced Mail,打开看文本格式,以"-----BEGIN..."开头, "-----END..."结尾,内容是BASE64编码.
查看PEM格式证书的信息:

```shell
openssl x509 -in certificate.pem -text -noout
```

Apache和NGINX服务器偏向于使用这种编码格式.

   **PEM** – Openssl使用 PEM(Privacy Enhanced Mail)格式来存放各种信息,它是 openssl 默认采用的信息存放方式。Openssl 中的 PEM 文件一般包含如下信息:

1. 内容类型:表明本文件存放的是什么信息内容,它的形式为“——-BEGIN XXXX ——”,与结尾的“——END XXXX——”对应。
2. 头信息:表明数据是如果被处理后存放,openssl 中用的最多的是加密信息,比如加密算法以及初始化向量 iv。
3. 信息体:为 BASE64 编码的数据。可以包括所有私钥（RSA 和 DSA）、公钥（RSA 和 DSA）和 (x509) 证书。它存储用 Base64 编码的 DER 格式数据，用 ascii 报头包围，因此适合系统之间的文本模式传输。

使用PEM格式存储的证书：

```shell
—–BEGIN CERTIFICATE—–
MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
………
1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
—–END CERTIFICATE—–
```

使用PEM格式存储的私钥：

```shell
—–BEGIN RSA PRIVATE KEY—–
MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
………
1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
—–END RSA PRIVATE KEY—–
```

使用PEM格式存储的证书请求文件：

```shell
—–BEGIN CERTIFICATE REQUEST—–
MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
………
1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
—–END CERTIFICATE REQUEST—–
```

**2、DER** – 辨别编码规则 (DER) 可包含所有私钥、公钥和证书。它是大多数浏览器的缺省格式，并按 ASN1 DER 格式存储。它是无报头的 － PEM 是用文本报头包围的 DER。

   **DER** - Distinguished Encoding Rules,打开看是二进制格式,不可读.
查看DER格式证书的信息

```shell
openssl x509 -in certificate.der -inform der -text -noout
```

Java和Windows服务器偏向于使用这种编码格式.

## 其他文件扩展名

这是比较误导人的地方,虽然我们已经知道有PEM和DER这两种编码格式,但文件扩展名并不一定就叫"PEM"或者"DER",常见的扩展名除了PEM和DER还有以下这些,它们除了编码格式可能不同之外,内容也有差别,但大多数都能相互转换编码格式.

**CRT** - CRT应该是certificate的三个字母,其实还是证书的意思,常见于*NIX系统,有可能是PEM编码,也有可能是DER编码,大多数应该是PEM编码,相信你已经知道怎么辨别.

**CER** - 还是certificate,还是证书,常见于Windows系统,同样的,可能是PEM编码,也可能是DER编码,大多数应该是DER编码.证书中没有私钥，DER 编码二进制格式的证书文件

**KEY** - 通常用来存放一个公钥或者私钥,并非X.509证书,编码同样的,可能是PEM,也可能是DER.
查看KEY的办法

```shell
openssl rsa -in mykey.key -text -noout
```

如果是DER格式的话,同理应该这样了

```shell
openssl rsa -in mykey.key -text -noout **-inform der**
```



**CSR** - Certificate Signing Request,即证书签名请求,这个并不是证书,而是向权威证书颁发机构获得签名证书的申请,其核心内容是一个公钥(当然还附带了一些别的信息),在生成这个申请的时候,同时也会生成一个私钥,私钥要自己保管好.做过iOS APP的朋友都应该知道是怎么向苹果申请开发者证书的吧.
查看的办法:

```shell
openssl req -noout -text -in my.csr
```

如果是DER格式的话照旧加上-inform der

```shell
openssl rsa -in my.csr -text -noout -inform der
```



**PFX/P12** - predecessor of PKCS#12,包含公钥和私钥的二进制格式证书

​    对nginx服务器来说,一般CRT和KEY是分开存放在不同文件中的,但Windows的IIS则将它们存在一个PFX文件中,(因此这个文件包含了证书及私钥)这样会不会不安全？应该不会,PFX通常会有一个"提取密码",你想把里面的东西读取出来的话,它就要求你提供提取密码,PFX使用的时DER编码,如何把PFX转换为PEM编码？

```shell
openssl pkcs12 -in for-iis.pfx -out for-iis.pem -nodes
```

这个时候会提示你输入提取代码. for-iis.pem就是可读的文本.
生成pfx的命令类似这样:

```shell
openssl pkcs12 -export -in certificate.crt -inkey privateKey.key -out certificate.pfx
```

其中CACert.crt是CA(权威证书颁发机构)的根证书,有的话也通过-certfile参数一起带进去.这么看来,PFX其实是个证书密钥库.

**p7b -** 以树状展示证书链(certificate chain)，同时也支持单个证书，不含私钥。

**JKS** - 即Java Key Storage,这是Java的专利,跟OpenSSL关系不大,利用Java的一个叫"keytool"的工具,可以将PFX转为JKS,当然了,keytool也能直接生成JKS,不过在此就不多表了.

证书格式详解：

1. der：.DER = DER扩展用于二进制DER编码证书。这些文件也可能承载CER或CRT扩展。 正确的说法是“我有一个DER编码的证书”不是“我有一个DER证书”。

2. pem： .PEM = PEM扩展用于不同类型的X.509v3文件，是以“ - BEGIN ...”前缀的ASCII（Base64）数据。

3. crt：.CRT = CRT扩展用于证书。 证书可以被编码为二进制DER或ASCII PEM。 CER和CRT扩展几乎是同义词。 最常见的于Unix 或类Unix系统。

4. cer：CER = .crt的替代形式（Microsoft Convention）您可以在微软系统环境下将.crt转换为.cer（.both DER编码的.cer，或base64 [PEM]编码的.cer）。

   .cer文件扩展名也被IE识别为 一个运行MS cryptoAPI命令的命令（特别是rundll32.exe cryptext.dll，CryptExtOpenCER），该命令显示用于导入和/或查看证书内容的对话框。 

5. key：     .KEY = KEY扩展名用于公钥和私钥PKCS＃8。 键可以被编码为二进制DER或ASCII PEM。

> PEM一般保存x509的明文密钥，一般是以base64格式存储，其他如pem，crt，cer，key都是以base64明文存储
>
> der一般是二进制证书文件，提取明文公钥需要转换为cer格式
>
> cer和crt其实是一个东西，存储内容一致
>

1. 生成RSA私钥，.pem格式或.key格式，两个格式是一样的，后缀名不一样，一般为pkcs1格式

```shell
openssl genrsa -out 1.rsa_private_key.pem 2048
```

2. 提取公钥

```shell
openssl rsa -in 1.rsa_private_key.pem -out 2.rsa_public_key.pem -pubout  
# 也可以通过以下命令提取der公钥
openssl rsa -in 1.rsa_private_key.pem -out 2.1.rsa_public_key.der -outform DER -pubout
# 可以通过以下命令查看der文件
openssl rsa -in 2.1.rsa_public_key.der -inform DER -pubin -text
```

2. 以上生成的私钥为PKCS#1格式，如果Java使用话，需要转换为PKCS#8

```shell
openssl pkcs8 -topk8 -in 1.rsa_private_key.pem -out 1.1pkcs8_rsa_private_key.pem -nocrypt
```

3. pkcs8转pkcs1

```shell
openssl rsa -in 1.1pkcs8_rsa_private_key.pem -out 1.1.2pkcs1_rsa_private_key.pem
```

4. 有些机构使用需要使用csr格式证书文件，如下转换命令

```shell
openssl req -nodes -newkey rsa:2048 -keyout rsa_private_key.pem -out rsa_private_key.csr -utf8
```

> csr证书说明：
>
> 比如我自己的网站，需要使用https 通信，那么我向“证书机构”申请数字证书的时候，就需要向他们提供相应的信息，这些信息以特定文件格式(.csr)提供的，这个文件就是“证书请求文件”；为了确保我提供的信息在互联网的传输过程中不会被有意或者无意的破坏掉，我们有如下的机制来对传输的内容进行保护：首先在本地生成一个私钥，利用这个私钥对“我们需要提供的信息“进行加密，从而生成 证书请求文件(.csr), 这个证书请求文件其实就是私钥对应的公钥证书的原始文件，里面含有对应的公钥信息；

5. 查看csr证书格式

```shell
openssl req -noout -text -in rsa_private_key.csr
```



1. 生成crt证书格式

```shell
openssl req -new -x509 -key 1.rsa_private_key.pem -days 3650 -out 3.rsa_private_key.crt
```

2. 也可以将crt文件生成二进制的der文件

```shell
openssl x509 -inform pem -in 3.rsa_private_key.crt -outform der -out 4.rsa_private_key.der

# 通过der文件提取crt文件
openssl x509 -in 4.rsa_private_key.der -inform der -outform pem -out 14.rsa_private_key.crt
# 这里提取出来的crt文件等同于 3.rsa_private_key.crt
```

3. 查看der证书文件格式

```shell
openssl x509 -in 4.rsa_private_key.der -inform der -text -noout
```



1. 依据crt生成pfx或p12证书

```shell
openssl pkcs12 -export -name ysmktaln -in 3.rsa_private_key.crt -inkey 1.rsa_private_key.pem -out 5.rsa_private_key.pfx
```

pfx证书一般是在windows的iis服务器中使用，一般为p12证书格式

2. 从pfx中导出crt和key

```shell
openssl pkcs12 -in 5.rsa_private_key.pfx -nocerts -nodes -out 6.rsa_private_key.key
openssl pkcs12 -in 5.rsa_private_key.pfx -clcerts -nokeys -out 7.rsa_private_key_pfx.crt
# 这里的 7.rsa_private_key_pfx.crt应该与上面的 3.rsa_private_key.crt一致
```

3. 也可以使用如下命令提取pfx中的crt以及key信息（也是server证书文件）

```shell
openssl pkcs12 -in 5.rsa_private_key.pfx -nodes -out 8.rsa_server.pem
# 可以通过8.rsa_server.pem转换为pfx
openssl pkcs12 -export -out 13.rsa_private_key.pfx -in 8.rsa_server.pem
# 这里的  13.rsa_private_key.pfx 与 5.rsa_private_key.pfx 一致
```

4. 从server证书文件中提取私钥

```shell
openssl rsa -in 8.rsa_server.pem -out 9.boc_private_key_pkcs1.pem
# 这里的 9.boc_private_key_pkcs1.pem 与 1.rsa_private_key.pem 一致
```

5. 可以通过server.pem证书文件提取cer证书

```shell
openssl x509 -in 8.rsa_server.pem -out 10.rsa_private_key.crt
# 这里获取到的 10.rsa_private_key.crt 文件与上述的 3.rsa_private_key.crt 一致
```

6. 将cer证书文件提取为公钥

```shell
openssl x509 -in 10.rsa_private_key.crt -pubkey  -noout > 11.rsa_public_key.pem
# 这里获取的公钥应该与2.rsa_public_key.pem保持一致
```



### 格式间转换

1. 如果证书文件为二进制形式，一般为der格式，需要做如下转换

```shell
openssl x509 -in 4.rsa_private_key.der -inform der -outform pem -out 12.rsa_private_key.crt
```

2. 其他操作

```shell
查看pem证书

openssl x509 -in cert.pem -text -noout

openssl x509 -in cert.cer -text -noout

openssl x509 -in cert.crt -text -noout

查看der证书
openssl x509 -in certificate.der -inform der -text -noout
```

15. pem和der格式的转换⚠️以下命令存在问题，后续补充

```shell
cer转der
openssl x509 -in cert.crt -outform der-out cert.der
cer转pem
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem

crt转der
openssl x509 -inform pem -in mycerts.crt -outform der -out mycerts.der

pem转der
openssl x509 -inform pem -in qingyidai.com.pem -outform der -out qingyidai.com.cer

der 到pem
openssl x509 -in name.cer -inform der -outform pem -out name.pem
pem到DER
openssl x509 -in name.pem -outform der -out name.der
```

## 后记

1. der和pem是文件编码方式，一个是二进制，一个是ascii码即明文可见
2. 至于cer和crt其实是一个证书文件，内部包含区域公司，联系方式、有效期等等，cer一般为der编码格式，主要用于windows下。
3. cer和crt文件也存在两种编码方式，即一个是pem一个是der，不能以后缀作为判断是哪个编码方式，主要看是否是可读的，如果不可读即为der格式
4. 


