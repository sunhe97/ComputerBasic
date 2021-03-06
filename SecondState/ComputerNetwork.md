# 第二章

## 2.2 Web 与 HTTP

### web

+ web页由对象组成，对象可以是HTML文件、JPEG图像、JAVA小程序、声音剪辑文件等

+ Web页含有一个基本的HTML文件，该文件包含若干文件对象的引用，图片也是链接下载，浏览器运用，对象指对象

+ 通过URL对每个对象进行引用，URL：协议://用户：口令@主机名/路径名：port（端口），用户口令可以匿名，port默认

### HTTP:超文本传输协议

+ web的应用层协议

+ 服务端客户端模式

+ 使用TCP协议

+ 80端口，无状态

### 服务器socket为守候socket，每有一个连接请求，就新生成一个socket为服务端与新客户端的通信socket

### 客户端获得html文件（数据体），每遇到一个链接（图片等），再建立链接，发送请求，服务端发送图片

### http1.1长连接

### 非流水线：一个时刻只生产一辆汽车，一个请求的相应回来，再发第二个，流水线：都是发送到一个URL，前一个请求未回来，就发第二个，一个流水现有多个汽车在制造

### HTTP请求报文（ASCII）,换行回车符，表示报文结束

+ 请求头，首部行：Host, User-agent,Connection Accept-language ![HTTP请求通用格式](Picture\HTTP请求通用格式.png)

+ 可用过URL请求行字段进行上载，html也分头部和body![请求报文方法类型](Picture\请求报文分类.png)

### HTTP响应报文，content-length用于客户端数据解析，因为使用TCP这些传输层协议需要应用层自己规定传输的界限

### Cookie：客户端与服务器维护状态

+ 第一次连接服务器会生成cookie并将cookie发给客户端同时保存下来，客户端收到cookie也保存下来，下次向这个网站发消息时会带上cookie
+ HTTP响应报文和请求报文都有cookie头![Cookie](Picture\Cookie.png)

### Web缓存（代理服务器），即是客户端也是服务端，搭配If modified since搭配使用，如果第二次发送条件GET方法 ，没改发送一个304头部即可

+ 缓存命中直接返回
+ 缓存不命中发送请求到origin服务器，再返回

#### 使用原因

+ 降低客户端请求响应时间
+ 可以减少一个机构内部网络与Internet接入链路上的流量
+ 大量缓存使较弱的ICP也能提供有效内容

#### 流量强度 = I实际 / I理论 排队延时 = I / 1 - I * L(数据长度) / R(Mbps)

## 2.3 FTP：向远程主机上传输文件或从远程主机接收文件

### ftp服务器在21号端口

### FTP控制连接（21）与数据连接（20）分开

### 客户端通过控制连接获得身份确认，发送命令浏览远程目录![FTP](Picture\FTP.png)

 ## 2.4Email

### 服务器守候在25号端口

### 用户代理：用于撰写邮件的客户端软件

### 用户代理SMTP发送邮件到邮件服务器，邮件服务器通过SMTP发送到目标邮件服务器的对应邮箱中，目标客户客户端通过pop3再拉取邮件

![Email](Picture\Email.png)

### SMTP是持久连接，HTTP只有一个对象，一个HTML对象其他是链接，邮件可以是多个对象（exe，文本），邮件报文主题只能有ASCII值

### 编码方式Base64：将不在ascii码的字符转化到更长的ascii值

![iEmail1](Picture\Email1.png)

## 2.5DNS:域名到IP地址的转换，字符串到IP地址的转换的必要性

###  DNS主要思路：本地Host记住常用IP，不然找路由的DNS服务器

+ 分层的、基于域的命名机制
+ 若干分布式的数据库完成名字到IP地址的转换
+ 运行在UDP上的端口号53的应用服务
+ 核心的Internet功能，单一应用层协议实现 

### 根、TLD、权威

![DNS记录](Picture\DNS记录.png)

### 上层域需要知道下层域的名字以及地址

### 每个主机联网需要子网掩码、local name server IP 、主机IP地址、gateway

### local name server， 本地DNS，在一个子网内部的服务器，在每个服务器还可以缓冲权威服务器的IP地址，之后直接查询返回，缓冲是为了性能，TTL超了删除，删除是为了一致性

### DNS报文有一个标识符ID号，使用ID号唯一标识，实现流水线方式，使用ID号可以同时并行查询IP

![DNS攻击](Picture\DNS攻击.png)

## 2.6 P2P应用

### 纯P2P架构：文件分发，流媒体

+ 没有或极少一直运行的服务器
+ 任意端系统都可以直接通信
+ 利用peer的服务能力
+ peer节点间歇上网，每次IP地址都有可能变化

### 结构化P2P peer与peer形成树形或者环形；非结构化peer与peer任意连接

### 非结构化P2P:

+ 集中式目录：单点问题，性能瓶颈，侵犯版权。服务器记录每个节点的内容与状态
+ 完全分布式：每个节点有8、10个节点，邻居向邻居，邻居再向邻居，BFS
+ 混合体：组内（集中式）组员连接组长，组外（分布式），组长与组长之间相连
+ 例子：BitTorrent，新加入的peer先随机访问4个节点，用Bitmap表示拥有哪些资源，之后请求最稀缺的资源，先看4个，找哪个节点对他服务带宽大，先服务他，两个周期后，随机选一个节点服务，后面周期再评估

### 结构化P2P：

+ 节点与节点之间有序拓扑，节点与内容之间是重叠的，内容在哪些节点上有约定

## CDN:Content Distribution Networks

### 全网部署缓存节点，存储服务内容，就近为用户提供服务，提高用户体验，通过域名解析的重定向（DNS）重定向找到最近、质量最好的的缓存节点，进行服务

### 在CDN节点存储内容的多个拷贝，用户从CDN中请求内容

### 视频块提供不同分辨率，每个视频分为8-10秒的块

### DNS重定向:最后KingCDN.com提供服务

![CDN1](Picture\CDN1.png)

![CDN2](Picture\CDN2.png)

# 第三章

## UDP绑定自身IP，端口标识一个socket， UDP不同来源发送IP，端口的一个socket再发给同一个应用；一个TCP用过源IP,源端口，目标IP，目标端口标识一个socket，可以通过源IP，源端口号不同发给不同的socket，发给不同应用进程

## 校验和（报文端的加法和）通过校验与差错控制编码的关系，如果在传输过程中出错，未通过校验和，将数据报扔掉，将数据部分且为16bits的块，然后相加，不足补0，进位回滚（将进位来回底部再加），校验和变为和的反码，接收方求的和和校验和相加为全1，就通过。

