# HTTP 请求流程：为什么很多站点第二次打开速度会很快？

问题:

1. 为什么通常在第一次访问一个站点时，打开速度很慢，当再次访问这个站点时，速度就很快了？
2. 当登录过一个网站之后，下次再访问该站点，就已经处于登录状态了，这是怎么做到的呢？

## 浏览器端发起 HTTP 请求流程

前提: 输入www.baidu.com后,浏览器内部发生了什么?

### 1. 构建请求

首先,浏览器会构建请求行信息,如下所示

```
    GET /index.html HTTP1.1
```

### 2. 查找缓存

浏览器在缓存中查询是否有要请求的文件,如果发现请求的资源文件在缓存中存在且不过期,则拦截并结束请求,返回缓存中资源,否则进入网络请求过程;

浏览器缓存: 在本地保存资源副本,以供下次请求直接使用的技术

缓存好处:

1. 缓解服务器压力,提升性能
2. 加速资源的加载,提升页面响应速度

### 3. 准备 IP 地址和端口

这个时候要准备发起网络请求了,浏览器使用 HTTP 协议作为应用层协议,用来封装网络请求的信息,并使用 TCP/IP 协议作为传输层协议发起网络请求;  
所以,发起网络请求需要提供 IP 和端口;

准备 IP:

1. 已知请求的网址是 www.baidu.com,浏览器先从自己的DNS缓存中查找是否有域名为 www.baidu.com的IP映射,如果有 IP 映射,则返回 IP,结束查找,如果没有继续下一步;
2. 浏览器开始查找本机 DNS 缓存,查找到则返回 IP,结束查询,查找不到则继续下一步;
3. 向本地域名解析服务(本地区的域名服务器,由运营商提供服务)系统发起域名解析的请求,查找到则返回,查找不到,  
   本地域名解析向根域名服务器发起解析请求,根域名服务器返回查域的通用顶级域地址,本地域名解析服务器向 gTLD 服务器发起解析请求,  
   gTLD 服务器接收本地域名服务器发起的请求，并根据需要解析的域名，找到该域名对应的域名服务器,通常情况下，这个服务器就是你注册的域名服务器,  
   域名服务器查找域名对应的地址，将地址连同值返回给本地域名服务器。
4. DNS 服务器最终没有查找到域名的 IP 映射,则返回 DNS lookup failed
5. 浏览器没有得到 IP 地址,则显示 ERR_EMPTY_RESPONSE;
6. 查找到之后,本地域名服务器,用户系统和浏览器将缓存 DNS 查找结果;

准备端口:

1. 判断请求网址 www.baidu.com是否指定端口(指定端口: www.baidu.com:7788,指定端口则返回对应的端口7788),没有指定端口
2. HTTP 请求的默认端口为 80,返回 80

[移步域名系统](./../../topic/dns/README.md)

### 4. 等待 TCP 队列

HTTP1.x 中,Chrome 为同一个域名最多维护 6 个 TCP 连接,当前域名超过 6 个 TCP 连接时,其余的连接需要进入 TCP 队列,等待正在进行的 TCP 连接请求完成;

HTTP2.x 中,一个域名只使用一个长连接来传输数据,同域名资源下载只需要一次慢启动,同时解决了多个 TCP 连接竞争带宽的问题;

所以, HTTP2.x 中不存在等待 TCP 队列问题,HTTP2.x 采用多路复用并行请求资源,只会维护一个 TCP 长连接,只会有一次慢启动;

### 5. 建立 TCP 连接

[移步 TCP 连接](./../../topic/tcp/README.md)

### 6. 发送 HTTP 请求

一旦 TCP 连接建立完成,便进入通信环节,浏览器会向服务器发送请求行(包含请求方法,请求 URI,HTTP 版本协议);

发送请求行即告诉服务器浏览器需要什么资源,GET 请求即告诉服务器需要获取网站首页的资源,如果使用 POST 请求,浏览器还需要将请求体发送给服务器;

请求行发送完毕后,浏览器还要将浏览器自身的基础信息(操作系统,浏览器内核,请求域名,Cookie 等)发送给服务器,以便服务器根据不同的浏览器处理不同的逻辑

## 服务端处理 HTTP 请求流程

### 1.返回请求

服务器根据请求信息处理完内部逻辑后,将数据(响应行,响应头,响应体)返回给浏览器;

服务器返回响应行(状态码,协议版本),随后便返回响应头,发送完响应头之后,服务器便继续发送响应体的数据;

如果响应行中的状态码为 301,意味着浏览器需要重定向到新的地址,这个地址就是响应头中的`Location`字段,那么当前请求流程便结束,浏览器重新发起`Location`字段值的请求(重复第一步);

### 2.断开连接

通常情况下,服务器向客户端返回数据信息后,便准备关闭 TCP 连接;如果浏览器在请求头中加入 `Connection: keep-alive`,服务器不会关闭当前 TCP 连接,浏览器可以通过同一个 TCP 连接发送数据;

保持 TCP 连接有益于加快资源加载,减少 TCP 慢启动;

## 问题解答

![先看一张图](../images/01/cache.png)

当浏览器第一次加载页面资源时,会将部分页面资源缓存,当再次打开页面时,如果缓存有效,则直接返回缓存资源,并取消对服务器资源的请求,省去了网络请求的时间;

当用户登录某网址后,服务器响应头返回`Set-Cookie`,浏览器根据响应头设置 Cookie,下次打开相同网址后,如果`Cookie`没有过期,则保留用户登录状态,所以就会是登录状态了(还有 LocalStorage 的方案);
