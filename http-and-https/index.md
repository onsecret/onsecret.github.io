# HTTP与HTTPS协议


## 1. HTTP

### 1.1 HTTP的概念

超文本传输协议（HyperText Transfer Protocol，HTTP）定义了浏览器（客户端进程）如何向网络上的服务器请求网络文档，以及服务器如何将文档传送给浏览器。从层次角度看，HTTP 是一个应用层协议，使用网络层的 TCP 进行可靠传输。

一个大致的工作过程如下所述。

每个网络节点都有一个服务器进程，它在后台不间断地监听着 TCP 的 80 端口，以便发现是否有来自浏览器的连接建立请求。一旦监听到连接请求并遵照握手协议建立 TCP 连接后，浏览器就会向该服务器发出浏览某个页面的请求，服务器就返回对应的页面作为响应。最后，TCP 连接被释放掉。HTTP 就是浏览器和服务器之间请求和响应的交互需要遵循的格式与规则。

HTTP 规定浏览器与服务器的交互是一个 ASCII 码串，这段字符串的格式就是 HTTP 报文格式。因为 HTTP 是建立在 TCP 上的协议，因此这段字符串也是 TCP 报文的数据部分。

无论是用户主动在浏览器地址栏输入了某个 URL，还是在页面上点击了某个元素，在背后都会转化为一个链接，然后浏览器就会在网络上找到链接对应的页面。假设我输入的 URL 或点击的元素指向了「清华大学院系设置」页面，具体的链接为 http://www.tsinghua.edu.cn/chn/yxsz/index.htm，之后发生的事情如下所述：

1. 浏览器向 DNS 请求解析 www.tsinghua.edu.cn 的 IP 地址；
2. 域名系统返回清华大学服务器的地址 166.111.4.100；
3. 浏览器与服务器建立 TCP 连接；
4. 浏览器发出取文件命令：GET /chn/yxsz/index.htm；
5. 服务器给出响应，把文件 index.htm 发给浏览器；
6. 释放 TCP 连接；
7. 浏览器渲染并显示 index.htm 文件，显示的页面就是「清华大学院系设置」页面

数据的可靠性由底层的 TCP 保证，HTTP 本身是无连接的，因此从上面的过程可以看出，通信双方并不需要建立和释放 HTTP 连接。HTTP 也是无状态的，无状态的含义是，如果此时浏览器再次访问「清华大学院系设置」页面，服务器会执行一遍重复的过程，再返回一次 index.htm 页面，因为服务器不记得这个浏览器曾经访问过。这种无状态特性既有好处也有坏处，后面会介绍。

### 1.2 HTTP 报文结构

HTTP 有两类报文：

1. 请求报文—从客户向服务器发送请求报文；
2. 响应报文—服务器向客户返回响应；

上一小节已经提到过，HTTP 是面向文本的，本质上一串 ASCII 码字符串，报文的格式就是对字符串的各部分含义进行规定，各部分长度也是不固定的。

![HTTP请求与响应报文](/images/计算机网络-HTTP和HTTPS协议/epub_655484_323.jpg)

HTTP 请求与响应报文都是由三个部分组成：开始行、首部行和实体主体。

- **实体主体**就是数据部分，请求报文一般都不用，响应报文中也可能没有这个字段。
- **首部行**不一定是一行，可能有多行，也可能没有，每行都是「键：值」形式，其中键叫做首部字段名，每一行结束都要跟一个回车和换行。整个首部行部分结束，还要加一个空行和实体主体分开。首部行的作用是说明浏览器、服务器或报文主体的一些基本信息，首部字段名大都是规定好的。
- **开始行**是请求和响应报文唯一不同的地方。请求报文的开始行可以叫做请求行，响应报文的开始行叫做状态行，开始行的三个部分都以空格分开，末尾要添加 回车+换行 与首部行部分区分。

#### 请求报文

请求报文的第一行请求行包括三部分：方法、URL和版本。

**方法**是浏览器希望执行的操作，其实就是一些命令，比如提到的 GET，请求报文的类型一般根据方法的类型进行区分，下表列举了请求报文常用的一些方法：

| 方法（操作） | 意义                              |
| ------------ | --------------------------------- |
| OPTION       | 请求一些选项的信息                |
| GET          | 请求读取由 URL 所标志的信息       |
| HEAD         | 请求读取由 URL 所标志的信息的首部 |
| POST         | 给服务器添加信息                  |
| PUT          | 在指明的 URL 下存储一个文档       |
| DELETE       | 删除指明的 URL 所标志的资源       |
| TRACE        | 用来进行环回测试的请求报文        |
| CONNECT      | 用于代理服务器                    |

**URL** 则是请求资源的 URL，**版本**是指 HTTP 协议的版本，现在一般是 HTTP/1.1。前面 1.1 小节提到的例子，请求「清华大学院系设置」页面，其 HTTP 请求报文的开始行就应当是

```http
GET http://www.tsinghua.edu.cn/chn/yxsz/index.htm HTTP/1.1
```

这里给出一个完整的请求报文的例子，请求行使用相对 URL 是因为首部行给出了主机域名。

```http
GET /chn/yxsz/index.htm HTTP/1.1
Host: www.tsinghua.edu.cn  // 主机域名
Connection: close  // 告诉服务器发送完请求的文档后就可以释放连接
User-Agent: Mozilla/5.0  // 表明用户代理是使用 Netscape 浏览器
Accept-Language: cn  // 表示用户希望优先得到中文版本的文档
// 这里有一个空行，后面的实体主体部分为空
```

#### 响应报文

响应报文的状态行同样分三部分：版本，状态码和短语。

版本依然是 HTTP 协议版本，状态码用来说明不同的响应情况，短语是对状态码的简单说明。状态行的示例如下

```http
HTTP/1.1 202 Accepted
```

HTTP 状态码(tatus Code)都是三位数字，分为 5 大类共 33 种（见RFC 2616），大类的含义如下表

| 分类 | 分类描述                                       |
| :--- | :--------------------------------------------- |
| 1**  | 通知信息，请求收到了或正在处理                 |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

若请求的网页从 http://www.ee.xyz.edu/index.html 转移到了一个新的地址，则响应报文的状态行和一个首部行就是下面的形式：

```http
HTTP/1.1 301 Moved Permanently
Location: http://www.xyz.edu/ee/index.html
```

## 2. HTTPS

安全超文本传输协议（Secure Hypertext Transfer Protocol，HTTPS）比HTTP更加安全。

HTTPS 是基于 SSL/TLS 的 HTTP，HTTP 是应用层协议，TLS 是传输层协议，在应用层和传输层之间，增加了一个安全套接层 SSL。

服务器用 RSA 生成公钥和私钥，把公钥放在证书里发送给客户端，私钥自己保存。客户端首先向一个权威的服务器求证证书的合法性，如果证书合法，客户端产生一段随机数，这段随机数就作为通信的密钥，称为对称密钥。这段随机数以公钥加密，然后发送到服务器，服务器用密钥解密获取对称密钥，最后，双方以对称密钥进行加密解密通信。

HTTPS的作用首先是内容加密，建立一个信息安全通道，来保证数据传输的安全；其次是身份认证，确认网站的真实性；最后是保证数据完整性，防止内容被第三方替换或者篡改。

HTTPS和HTTP有一定的区别。HTTPS协议需要到CA申请证书。HTTP是超文本传输协议，信息是明文传输；HTTPS则是具有安全性的SSL加密传输协议。HTTP使用的是80端口，而HTTPS使用的是443端口。HTTP的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的、可进行加密传输和身份认证的网络协议，比HTTP协议安全。