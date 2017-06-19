---
title: HTTPS小解析
date: 2017-06-17 22:59:42
categories: 
	- iOS合集
---
####  一、HTTPS介绍

超文本传输安全协议（英语：Hypertext Transfer Protocol Secure，缩写：HTTPS，常称为HTTP over TLS，HTTP over SSL或HTTP Secure）是一种网络安全传输协议。在计算机网络上，HTTPS经由超文本传输协议进行通信，但利用SSL/TLS来加密数据包。HTTPS开发的主要目的，是提供对网络服务器的身份认证，保护交换数据的隐私与完整性。这个协议由网景公司（Netscape）在1994年首次提出，随后扩展到互联网上。

HTTPS连接经常用于万维网上的交易支付和企业信息系统中敏感信息的传输。

http协议直接放置在TCP协议之上，而HTTPS提出在http和TCP中间加上一层加密层。从发送端看，这一层负责把http的内容加密后送到下层的TCP，从接收方看，这一层负责将TCP送来的数据解密还原成http的内容。所以严格地讲，HTTPS并不是一个单独的协议，而是对工作在一加密连接（TLS或SSL）上的常规HTTP协议的称呼。

下面是一个简单的HTTPS协议栈的图：
![](http://oqepgj2jp.bkt.clouddn.com/https1.png)

---

####  二、HTTPS流程步骤

上面已经说过HTTPS主要是加了一层SSL/TLS加密，那么具体是如何进行加密，解密，验证的，且看下图：
![](http://oqepgj2jp.bkt.clouddn.com/https2.png)


###### 1. 客户端发起HTTPS请求
这个没什么好说的，就是用户在浏览器里输入一个https网址，然后连接到server的443端口。

###### 2. 服务端的配置
采用HTTPS协议的服务器必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面(startssl就是个不错的选择，有1年的免费服务)。这套证书其实就是一对公钥和私钥。如果对公钥和私钥不太理解，可以想象成一把钥匙和一个锁头，只是全世界只有你一个人有这把钥匙，你可以把锁头给别人，别人可以用这个锁把重要的东西锁起来，然后发给你，因为只有你一个人有这把钥匙，所以只有你才能看到被这把锁锁起来的东西。

###### 3. 传送证书
这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间等等。

###### 4. 客户端解析证书
这部分工作是有客户端的TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值。然后用证书对该随机值进行加密。就好像上面说的，把随机值用锁头锁起来，这样除非有钥匙，不然看不到被锁住的内容。

###### 5. 传送加密信息
这部分传送的是用证书加密后的随机值，目的就是让服务端得到这个随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密了。

###### 6. 服务端解密信息
服务端用私钥解密后，得到了客户端传过来的随机值(私钥)，然后把内容通过该值进行对称加密。所谓对称加密就是，将信息和私钥通过某种算法混合在一起，这样除非知道私钥，不然无法获取内容，而正好客户端和服务端都知道这个私钥，所以只要加密算法够彪悍，私钥够复杂，数据就够安全。

###### 7. 传输加密后的信息
这部分信息是服务段用私钥加密后的信息，可以在客户端被还原

###### 8. 客户端解密信息
客户端用之前生成的私钥解密服务段传过来的信息，于是获取了解密后的内容。整个过程第三方即使监听到了数据，也束手无策。

---

####  三、SSL/TLS概念

SSL/TLS是加密通信协议，SSL由NetScape在1994年设计，1999年互联网标准化组织ISOC接替NetScape公司，发布了SSL的升级版TLS 1.0版。现在主流的浏览器等都支持TLS1.2版本，如iOS9中新增App Transport Security（简称ATS）特性，强制http转向https，其中加密通信协议就需要TLS1.2及以上版本。

SSL/TLS协议的基本思路是采用公钥加密法，也就是说，客户端先向服务器端索要公钥，然后用公钥加密信息，服务器收到密文后，用自己的私钥解密。

SSL/TLS协议的基本过程是这样的：

```
（1） 客户端向服务器端索要并验证公钥。
（2） 双方协商生成"对话密钥"。
（3） 双方采用"对话密钥"进行加密通信。
```
所以说SSL/TLS协议主要是包含非对称加密（公钥加密）和对称加密，用非对称加密来得到对称加密的"对话秘钥"，然后用对称加密来进行加密通信。


####  四、两个问题

- 如何保证公钥不被篡改？

> 解决方法：将公钥放在数字证书中。只要证书是可信的，公钥就是可信的。那如何保证证书是可信的呢？证书由CA机构进行颁发，而游览器内置了这些CA机构的根证书，只要由这些CA机构办法的数字证书即是可信的。

- 为什么不直接使用公钥加密，还要加上个对称加密？

> 公钥加密是非对称加密，加密计算量大，而对称加密运算速度非常快。所以这里只有第一次握手时进行公钥加密来得到对称加密的"对话密钥"，之后的通信就使用对称加密来进行通信了。

####  五、加密算法

- 对称密码算法

> 是指加密和解密使用相同的密钥，典型的有DES、RC5、IDEA（分组加密），RC4（序列加密）；

- 非对称密码算法

> 又称为公钥加密算法，是指加密和解密使用不同的密钥（公开的公钥用于加密，私有的私钥用于解密）。比如A发送，B接收，A想确保消息只有B看到，需要B生成一对公私钥，并拿到B的公钥。于是A用这个公钥加密消息，B收到密文后用自己的与之匹配的私钥解密即可。反过来也可以用私钥加密公钥解密。也就是说对于给定的公钥有且只有与之匹配的私钥可以解密，对于给定的私钥，有且只有与之匹配的公钥可以解密。典型的算法有RSA，DSA，DH；

- 散列算法

> 散列变换是指把文件内容通过某种公开的算法，变成固定长度的值（散列值），这个过程可以使用密钥也可以不使用。这种散列变换是不可逆的，也就是说不能从散列值变成原文。因此，散列变换通常用于验证原文是否被篡改。典型的算法有：MD5，SHA，Base64，CRC等。

---

####  六、关于CA及数字证书

- 什么是CA

CA(Certificate Authority)是数字证书认证中心的简称，是指发放、管理、废除数字证书的机构。

CA的作用是检查证书持有者身份的合法性，并签发证书（在证书上签字），以防证书被伪造或篡改，以及对证书和密钥进行管理。

CA 也拥有一个证书（内含公钥）和私钥。网上的公众用户通过验证 CA 的签字从而信任 CA ，任何人都可以得到 CA 的证书（含公钥），用以验证它所签发的证书。
如果用户想得到一份属于自己的证书，他应先向 CA 提出申请。在 CA 判明申请者的身份后，便为他分配一个公钥，并且 CA 将该公钥与申请者的身份信息绑在一起，并为之签字后，便形成证书发给申请者。

如果一个用户想鉴别另一个证书的真伪，他就用 CA 的公钥对那个证书上的签字进行验证，一旦验证通过，该证书就被认为是有效的。

- 证书的内容

数字证书的格式遵循X.509标准，X.509是由国际电信联盟（ITU-T）制定的数字证书标准，规范了公开密钥认证、证书吊销列表、授权证书、证书路径验证算法等。

证书的内容包括：电子签证机关的信息、公钥用户信息、公钥、权威机构的签字和有效期等等。

下图就表示一个数字证书包含的内容：

![](http://oqepgj2jp.bkt.clouddn.com/https3.png)

> 我们这里能看到颁发机构签名是由申请者信息经过哈希算法得到hash值，然后再用机构的私钥进行加密。所以这个签名只有办法机构的公钥才能解密，而一般权威CA机构的根证书（含公钥）都内置在浏览器中，所以客户端接收到这个数字证书后，先把申请者信息用同样的哈希算法得到hash值h1，然后用公钥进行办法机构签名解密得到hash值h2，如果h1==h2，则表示证书是有效的。

下图就是Charles的根证书例子：

![](http://oqepgj2jp.bkt.clouddn.com/https4.png)

####  七、编码格式

同样的X.509证书,可能有不同的编码格式,目前有以下两种编码格式。

**PEM** - Privacy Enhanced Mail,打开看文本格式,以"-----BEGIN..."开头, "-----END..."结尾,内容是BASE64编码.
查看PEM格式证书的信息:openssl x509 -in certificate.pem -text -noout
Apache和*NIX服务器偏向于使用这种编码格式.

**DER** - Distinguished Encoding Rules,打开看是二进制格式,不可读.
查看DER格式证书的信息:openssl x509 -in certificate.der -inform der -text -noout
Java和Windows服务器偏向于使用这种编码格式.

####  八、相关的文件扩展名

这是比较误导人的地方,虽然我们已经知道有PEM和DER这两种编码格式,但文件扩展名并不一定就叫"PEM"或者"DER",常见的扩展名除了PEM和DER还有以下这些,它们除了编码格式可能不同之外,内容也有差别,但大多数都能相互转换编码格式。

**CRT** - CRT应该是certificate的三个字母,其实还是证书的意思,常见于*NIX系统,有可能是PEM编码,也有可能是DER编码,大多数应该是PEM编码,相信你已经知道怎么辨别.

**CER** - 还是certificate,还是证书,常见于Windows系统,同样的,可能是PEM编码,也可能是DER编码,大多数应该是DER编码.

**KEY** - 通常用来存放一个公钥或者私钥,并非X.509证书,编码同样的,可能是PEM,也可能是DER。
查看KEY的办法:`openssl rsa -in mykey.key -text -noout`
如果是DER格式的话:`openssl rsa -in mykey.key -text -noout -inform der`

**CSR** - Certificate Signing Request,即证书签名请求,这个并不是证书,而是向权威证书颁发机构获得签名证书的申请,其核心内容是一个公钥(当然还附带了一些别的信息),在生成这个申请的时候,同时也会生成一个私钥,私钥要自己保管好。
查看的办法:`openssl req -noout -text -in my.csr`
如果是DER格式的话:`openssl req -noout -text -in my.csr -inform der`

**PFX/P12** - predecessor of PKCS#12,对*nix服务器来说,一般CRT和KEY是分开存放在不同文件中的,但Windows的IIS则将它们存在一个PFX文件中,(因此这个文件包含了证书及私钥)这样会不会不安全？应该不会,PFX通常会有一个"提取密码",你想把里面的东西读取出来的话,它就要求你提供提取密码,PFX使用的时DER编码,如何把PFX转换为PEM编码？
`openssl pkcs12 -in for-iis.pfx -out for-iis.pem -nodes`
这个时候会提示你输入提取代码. for-iis.pem就是可读的文本。
生成pfx的命令类似这样:`openssl pkcs12 -export -in certificate.crt -inkey privateKey.key -out` `certificate.pfx -certfile CACert.crt`

其中CACert.crt是CA(权威证书颁发机构)的根证书,有的话也通过-certfile参数一起带进去.这么看来,PFX其实是个证书密钥库.

**JKS** - 即Java Key Storage,这是Java的专利,跟OpenSSL关系不大,利用Java的一个叫"keytool"的工具,可以将PFX转为JKS,当然了,keytool也能直接生成JKS,不过在此就不多表了。

####  九、证书编码的转换

- .crt转.der方法

`openssl x509 -in cert.crt -out cert.der -outform DER`

- .crt转.cer方法

`openssl x509 -in cert.crt -out cert.cer -outform DER`

- .crt转.pem方法

`openssl x509 -in cert.crt -out cert.pem -outform PEM`


####  十、生成自签名证书的步骤

###### 建立CA

- 在任意目录建立文件夹，文件夹名称任意

```
mkdir ca
```

- 进入到新建立的文件夹ca

```
d ca
```

- 生成CA私钥

```
openssl genrsa -out ca.key 2048
```

- 用CA私钥生成CA的证书

```
openssl req -new -x509 -days 36500 -key ca.key -out ca.crt -subj 
"/C=CN/ST=Hangzhou/L=Hangzhou/O=Teamsun/OU=Dasheng"
```

- 建立CA相应目录

```
mkdir demoCA
cd demoCA/

mkdir newcerts

touch index.txt

echo '01' > serial
```

###### 生成server端证书

- 进入ca文件夹

```
cd ca
```


- 生成server私钥

```
openssl genrsa -out server.key 2048
```

- 使用server私钥生成server端证书请求文件

```
openssl req -new -key server.key -out server.csr -subj "/C=CN/ST=Hangzhou/L=Hangzhou/O=Teamsun/OU=dasheng/CN=dasheng"
```

- 使用server证书请求文件通过CA生成自签名证书

```
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
```

- 验证server证书

```
openssl verify -CAfile ca.crt server.crt
```

###### 测试

- 使用server证书测试单向认证

- 打开窗口1启动server

```
openssl s_server -accept 10001 -key server.key -cert server.crt
```

- 打开窗口2启动客户端

```
openssl s_client -connect localhost:10001
```

- 连接成功后在任意一个窗口输入字符串会传输到另外一个窗口回显。

###### 脚本
[快速创建证书的脚本](http://ofcckdrlc.bkt.clouddn.com/generate_certificate.sh)
使用（host表示证书用于的域名，cerFile表示证书保存的目录）：

```
sh ./generate_certificate.sh host cerFile
```

####  十一、信任自签名证书

###### 查看证书链

Chrome57及后续版本Chrome浏览器用户如要查看SSL证书信息只能通过开发者工具（右键->检查），选择安全标签（Security）进行查看了。然后点击View certificate查看证书链，如下图为查看www.google.com的证书链：

![](http://oqepgj2jp.bkt.clouddn.com/https5.png)

![](http://oqepgj2jp.bkt.clouddn.com/https6.png)

###### 信任证书

在MAC上直接双击证书，然后在钥匙串里就能看到这个证书了，我们能看到证书上会显示此证书是由不被信任的签发者签发的或此根证书不被信任。然后我们再在钥匙串中双击证书->信任->使用此证书时：始终信任。

![](http://oqepgj2jp.bkt.clouddn.com/https7.png)

这里我们可以信任两种证书：CA根证书和CA签名过的数字证书。两种证书在钥匙串中显示的颜色是不一样的。

下图就是自己创建的两种证书：

![](http://oqepgj2jp.bkt.clouddn.com/https8.png)

![](http://oqepgj2jp.bkt.clouddn.com/https9.png)

我们信任两种证书都可以使请求变成安全的请求，在浏览器中输入的时候就不会有不安全的提示了。这里说一下为什么两种证书都可以。

首先说CA根证书，这种就是我们正常的流程，CA根证书用公钥解密数字证书的签名得到hash值，然后根据hash值相等判断证书有效。
而不信任根证书只信任数字证书，就很容易理解了，他们本来就是同一张证书，也不用通过加密解密什么的来判断了。


####  十二、Chrome信任根证书后提示链接不安全

这里我信任根证书之后还是提示链接不安全：`ERR_CERT_WEAK_SIGNATURE_ALGORITHM`。而信任数字证书则没问题。发生这种情况的原因是`Chrome 57`版本以后是不支持SHA-1算出的hash值的证书签名的，而我们上面生成的证书默认为SHA-1，这里只要改为SHA-256就可以了。

```
//用CA私钥生成CA的证书
openssl req -new -x509 -days 36500 -key ca.key -out ca.crt -subj "/C=CN/ST=Hangzhou/L=Hangzhou/O=Teamsun/OU=Dasheng" -sha256

//使用server证书请求文件通过CA生成自签名证书
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -sha256
```

####  十三、获取证书小技巧

有时候我们没有这个网站的证书，那要如何得到呢？

###### 1、使用openssl能直接得到这个证书:

```
openssl s_client -connect 172.16.10.244:8000 </dev/null 2>/dev/null | openssl x509 -outform DER > https.cer
```

###### 2、直接Safari输入网站，如果是不安全的，会显示下图，然后点击显示证书，勾选连接时始终信任，点击继续证书就添加到钥匙串中了。

![](http://oqepgj2jp.bkt.clouddn.com/https10.png)

---

####  十四、iOS中使用自签名证书

iOS9中新增App Transport Security（简称ATS）特性, 主要使到原来请求的时候用到的HTTP，都转向TLS1.2协议进行传输。这也意味着所有的HTTP协议都强制使用了HTTPS协议进行传输。一般如果我们HTTPS服务使用的证书是CA权威机构颁发的话，客户端不用修改任何代码，因为iOS系统已经内置了这些权威机构的根证书。但是如果是自签名的证书的话就需要修改代码来内部信任这部分证书了。

###### AFNetworking使用自签名证书

AFNetWorking封装了如何使用自签名证书，简单的使用方式如下。

```objectivec
//先导入证书，找到证书的路径
NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"cert" ofType:@"der"];
NSData *certData = [NSData dataWithContentsOfFile:cerPath];
NSSet * certSet = [[NSSet alloc] initWithObjects:certData, nil];

//AFSSLPinningModeNone 这个模式表示不做 SSL pinning，只跟浏览器一样在系统的信任机构列表里验证服务端返回的证书。若证书是信任机构签发的就会通过，若是自己服务器生成的证书，这里是不会通过的。

//AFSSLPinningModeCertificate 这个模式表示用证书绑定方式验证证书，需要客户端保存有服务端的证书拷贝，这里验证分两步，第一步验证证书的域名/有效期等信息，第二步是对比服务端返回的证书跟客户端返回的是否一致。

//AFSSLPinningModePublicKey 这个模式同样是用证书绑定方式验证，客户端要有服务端的证书拷贝，只是验证时只验证证书里的公钥，不验证证书的有效期等信息。只要公钥是正确的，就能保证通信不会被窃听，因为中间人没有私钥，无法解开通过公钥加密的数据。

AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeNone];
// 是否允许,NO-- 不允许无效的证书
[securityPolicy setAllowInvalidCertificates:YES];
// 设置证书
[securityPolicy setPinnedCertificates:certSet];
//是否验证域名信息
securityPolicy.validatesDomainName = NO;


AFHTTPSessionManager *manager = [[AFHTTPSessionManager manager] initWithBaseURL:[NSURL URLWithString:@"https://192.168.3.13:8000"]];
manager.securityPolicy = securityPolicy;
manager.responseSerializer = [AFHTTPResponseSerializer serializer];
[manager GET:@"/getInfo" parameters:@{@"t":@"1490927497.569"} progress:^(NSProgress * progress){
} success:^(NSURLSessionDataTask *task, id responseObject) {
    NSArray * array = [NSJSONSerialization JSONObjectWithData:responseObject options:NSJSONReadingMutableLeaves error:nil];
    NSLog(@"OK === %@",array);
} failure:^(NSURLSessionDataTask *task, NSError *error) {
    NSLog(@"error ==%@",error.description);
}];
```

###### 证书需要满足的条件

这里的证书使用CA根证书或CA签名的数字证书都可以。

密钥交换算法有`RSA`和`ECDHE`，RSA 历史悠久，支持度好，但不支持 PFS（Perfect Forward Secrecy）；而 ECDHE 是使用了 ECC（椭圆曲线）的 DH（Diffie-Hellman）算法，计算速度快，支持 PFS。

iOS支持的秘钥交换算法为：至少**2048位的 RSA 密钥或至少256位的 ECC 密钥**

服务器证书的哈希算法必须为 SHA-2，其摘要长度至少位256位。

证书格式为.der，很多网上的教程都写的是.cer，应该是使用的旧版AFNetWorking，最新版的不支持.cer，需要使用.der格式。

---

####  十五、中间人攻击

###### 概念

关于Https最常讲到的就是中间人攻击，即所谓的Man-in-the-middle attack(MITM)。也就是攻击者插入原先攻击的双方，让双方以为还在直接跟对方通讯，但实际上双方的通信对方已变成了中间人，信息已经是被中间人获取或篡改。

其实http的中间人攻击是最简单的，因为http都是通过明文传输，而且没有任何认证之类的东西。我们常常用的Charles抓包就是一个最简单的中间人攻击。

###### 对HTTPS进行中间人攻击

我们用Charles进行HTTPS的抓包的时候会发现抓到的包都是加过密的无法查看，那是不是就意味着无法抓取HTTPS的包了呢？其实也是可以的，通过伪造证书，并且客户端又安装了Charles根证书，就可以抓取到HTTPS的包并解密了。

具体的步骤是这样的，手机安装Charles根证书，手机使用Charles的代理，所有请求都经过Charles中间人。Charles劫持到请求，替换服务端的证书为自己的伪证书，然后发送给客户端，客户端使用Charles根证书来验证这个伪证书，验证通过得到公钥，然后用公钥加密对话秘钥发送回Charles中间人，Charles中间人私用私钥解密得到对话秘钥并保存，然后再把对话秘钥用服务端的公钥加密返回给服务端，这样就表示两端握手成功，可以进行通信了。而且中间人也获得了之后的对话秘钥，可以解密之后的对话信息。

###### Charles实现HTTPS抓包

这里基本的HTTP抓包的设置就不讲了，下面是基于实现基本的HTTP抓包的基础上来实现HTTPS的抓包解密。

- 安装Charles CA根证书
点击`Help->SSL Proxying->Install Charles Root Certification ...`，会弹出如下提示，链接代理，手机浏览器输入`chls.pro/ssl`，就可以安装根证书了。

![](http://oqepgj2jp.bkt.clouddn.com/https11.png)

- 设置SSL代理
点击`Proxy->SSL Proxying Setting`，勾选`Enable SSL Proxying`，然后点击Add输入要SSL代理的请求Host和Port，可以使用通配符来表示某一类请求。

![](http://oqepgj2jp.bkt.clouddn.com/https12.png)

或者在对应的请求上右键选择`Enable SSL Proxying`，就会把这一个请求加入到上面的SSL代理列表中（类似于点击Add的效果）。

![](http://oqepgj2jp.bkt.clouddn.com/https13.png)

做完上述步骤后重新请求就能得到解密后的信息了。抓取PC端的HTTPS包也类似，在`Help->SSL Proxying`中下载证书，双击安装证书，并选择始终信任即可。

---

#### 参考
[超文本传输安全协议](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)
[图解HTTPS](http://www.cnblogs.com/zhuqil/archive/2012/07/23/2604572.html)
[HTTPS从原理到应用](http://www.jianshu.com/p/e767a4e9252e)
[那些证书相关的玩意儿](http://www.cnblogs.com/guogangj/p/4118605.html)
[SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)
[SSL/TLS协议及Openssl工具的实现](http://www.jianshu.com/p/da65e5cd552e)
[openssl自签名证书生成与单双向验证](http://blog.csdn.net/gx_1983/article/details/47866537)
[iOS 10 适配 ATS](http://www.jianshu.com/p/36ddc5b009a7)
[iOS安全系列之一：HTTPS](http://oncenote.com/2014/10/21/Security-1-HTTPS/)
[iOS安全系列之二：HTTPS进阶](http://oncenote.com/2015/09/16/Security-2-HTTPS2/#mitm)



