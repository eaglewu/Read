# HTTP权威指南

# 第四部分 实体、编码和国际化

# 15. 实体和编码

## 15.6 传输编码和分块编码

内容编码和传输编码的对比

* 内容编码: 只是对报文的实体部分进行了可逆的编码.
* 传输编码: 作用在整个报文上,报文本身的结构发生变化.

### 15.6.1 可靠传输

传输报文主体可能会引发的问题

1. 未知尺寸: 假如服务器想在未知大小之前就无法开始传输,Content-length必须声明在数据前.
2. 安全性: 用传输编码将报文打乱,然后发送,但是SSL已经够安全了,所以这个不用考虑

### 15.6.2 Transfer-Encoding首部

最新的HTTP规范(2017-03-15)只定义了一种传输编码: 分块编码

请求头声明分块编码,告知服务器可以使用哪种传输编码拓展

TE: trailers, chunked(HTTP1.1强制默认)

在响应头声明分块编码,告知接收方使用了哪种传输编码

Transfer-Enconding: chunked

### 15.6.3 分块编码

分块传输响应DEMO

```bash
//HTTP响应
HTTP/1.1 200 OK<CR><LF>
Content-type: text/plain<CR><LF>
Transfer-encoding: chunked<CR><LF>
Trailer: Content-MD5<CR><LF>
<CR><LF>

//第一块
5<CR><LF>
01234<CR><LF>

//第二块
2<CR><LF>
01<CR><LF>

//最后一块
0<CR><LF>

//拖挂
Content-MD5: ****1112341***
```

> 1. 分块与持久连接

* 非持久连接: 本身就不需要知道长度,等着服务器关闭主体链接即可
* 持久连接: 静态内容,通过content-length获取长度,而动态内容,发送前无法知道主体的长度

分块编码规则

1. 每个分块包括两行,一行十六进制的长度值,第二行为数据
2. 两行都需要\r\n结尾
3. 最后一块的长度值必须为0,对应的分块数据没有内容

当你上传给服务器想使用分块传输时,因为服务器不会发送TE请求

所以要做好被服务器拒绝分块传输的准备(411 Length Required, 表名需要Content-length长度)

> 2. 分块报文的拖挂

拖挂用于在报文开始时无法确认的首部

除了Transfer-Encoding,Trailer以及Content-length首部外

其他HTTP首部都可以作为拖挂发送

## 15.8 验证码和新鲜度

### 15.8.1 新鲜度

服务器可以用这两个首部之一来提供这种信息

1. Expires(过期)
2. Cache-Control(缓存控制)

### 15.8.2 有条件的请求与验证码

请求类型             验证码          描述

If-Modified-Since   Last-Modified 如果前一条响应的Last-Modified首部中说明时间之后,资源的版本发生变化,就发送其副本

If-Unmodified-Since Last-Modified 仅在前一条响应的Last-Modified首部中说明的时间之后,资源的版本没有变化,才发送其副本

If-Match             Etag         如果实体的标记前一次响应收不中的Etag相同,就发送资源的副本

If-None-Match        Etag         如果实体的标记与前一次响应收不中的Etag不同,就发送该资源的副本

HTTP校验分为弱检验码(Last-Modified)和强验证码(Etag)

## 15.9 范围请求



