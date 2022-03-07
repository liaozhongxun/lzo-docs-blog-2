---
id: script-model-browser
title: 浏览器知识点
---
### 浏览器输入url后
#### 一、从浏览器接收 url 到开启网络请求线程

> 这一部分可以展开浏览器的机制以及进程与线程之间的关系

-   浏览器是多进程的，有一个主控进程，以及每一个 tab 页面都会新开一个进程
-   每一个 tab 页面可以看作是浏览器内核进程(一般包括 GUI 线程,JS 引擎线程,事件触发线程,定时器线程,网络请求线程等多个线程)
-   解析 URL(协议//主机:端口号/目录路径?查询参数#hash)
-   解析协议 : 浏览器会根据解析出的协议，开辟一个网络线程，前往请求资源

#### 二、开启网络线程到发出一个完整的 http 请求

> 这一部分涉及到 dns 查询，tcp/ip 请求，五层因特网协议栈等知识

##### (1) 解析主机 : 如果输入主机的是域名，需要进行 dns 解析成 IP

       1-1、如果浏览器有缓存，直接使用浏览器缓存，否则使用本机缓存，再没有的话就是用host
       1-2、如果本地没有，就向dns域名服务器查询，查询到对应的IP

##### (2) 连接主机 : tcp/ip 请求(三次握手建立连接发送请求,四次挥手断开连接)

       2-1、3次过程
            2-1-1、
            2-1-2、
            2-1-3、
       2-2、4次过程
            2-2-1、
            2-2-2、
            2-2-3、
            2-2-4、

#### 三、从服务器接收到请求后的处理

> 这一部分可能涉及到负载均衡，安全拦截以及后台内部的处理等等

(1) 负载均衡：对于大型的项目，由于并发访问量很大，所以往往一台服务器是吃不消的，所需要的处理方案称为负载均衡
(2) 一般需要的处理步骤
    2-1、一般有的后端是有统一的验证的，如安全拦截，跨域验证
    2-2、如果这一步不符合规则，就直接返回了相应的http报文（如拒绝请求等）
    2-3、然后当验证通过后，才会进入实际的后台代码，此时是程序接收到请求，然后执行（譬如查询数据库，大量计算等等）
    2-4、等程序执行完毕后，就会返回一个http响应包（一般这一步也会经过多层封装）
    2-5、然后就是将这个包从后端发送到前端，完成交互

#### 四、后台和前台的 http 交互

> 这一部分包括 http 头部、响应码、报文结构、cookie 等知识，可以提下静态资源的 cookie 优化，以及编码解码，如 gzip 压缩等

(1) http 报文(http 报文作为信息的载体)
> 响应报文报文分为响应行、响应头、和响应体;请求报文分为请求行、请求头、和请求体  
> get 请求无请求体

```javascript
1-1、通用头部(General)
    1-1-1、Request Url: //请求的web服务器地址
    1-1-1、Request Method: //请求方式（http:1.1 -> Get、POST、OPTIONS、PUT、HEAD、DELETE、CONNECT、TRACEd等）
    1-1-1、Status Code: //请求的返回状态码，如200代表成功
        1xx——指示信息，表示请求已接收，继续处理
        2xx——成功，表示请求已被成功接收、理解、接受
        3xx——重定向，要完成请求必须进行更进一步的操作
        4xx——客户端错误，请求有语法错误或请求无法实现
        5xx——服务器端错误，服务器未能实现合法的请求

        200——表明该请求被成功地完成，所请求的资源发送回客户端
        304——自从上次请求后，请求的网页未修改过，请客户端使用本地缓存
        400——客户端请求有错（譬如可以是安全模块拦截）
        401——请求未经授权
        403——禁止访问（譬如可以是未登录时禁止）
        404——资源未找到
        500——服务器内部错误
        503——服务不可用
        .....

    1-1-1、Remote Address: 请求的远程服务器地址

1-2、请求头(Request headers) [view source 请求行: POST /url HTTP/1.1]
    Accept: //接收类型，表示浏览器支持的MIME类型（对标服务端返回的Content-Type）
    Accept-Encoding：//浏览器支持的压缩类型,如gzip等,超出类型不能接收
    Content-Type：//客户端发送出去请求体内容的类型
        'application/x-www-form-urlencoded': 'URLSearchParams:name=username&age=20' 表单格式
        'application/json': "{name:'username',age:20}"
        'multipart/form-data': "FormData 对象"
        'Blob/BufferSource': "二进制数据"

        //服务器返回
        'text/plain;charset=UTF-8':'普通文本'
        'text/html;charset=UTF-8':'html标签'
        //对照表 https://tool.oschina.net/commons)
        
    Cache-Control: //指定请求和响应遵循的缓存机制，如no-cache
    If-Modified-Since：//对应服务端的Last-Modified，用来匹配看文件是否变动，只能精确到1s之内，http1.0中
    Expires：//缓存控制，在这个时间内不会请求，直接使用缓存，http1.0，而且是服务端时间
    Max-age：//代表资源在本地缓存多少秒，有效时间内不会请求，而是使用缓存，http1.1中
    If-None-Match：//对应服务端的ETag，用来匹配文件内容是否改变（非常精确），http1.1中
    Cookie: //有cookie并且同域访问时会自动带上
    Connection: //当浏览器与服务器通信时对于长连接如何进行处理,如keep-alive
    Host：//请求的服务器URL
    Origin：//最初的请求是从哪里发起的（只会精确到端口）,Origin比Referer更尊重隐私
    Referer：//该页面的来源URL(适用于所有类型的请求，会精确到详细页面地址，csrf拦截常用到这个字段)
    User-Agent：//用户客户端的一些必要信息，如UA头部等
1-3、响应头(Response headers) [view source 响应行:HTTP/1.1 200]
    Access-Control-Allow-Headers: //服务器端允许的请求Headers
    Access-Control-Allow-Methods: //服务器端允许的请求方法
    Access-Control-Allow-Origin: //服务器端允许的请求Origin头部（譬如为*）
    Content-Type：//服务端返回的响应体内容的类型
    Date：//数据从服务器发送的时间
    Cache-Control：//告诉浏览器或其他客户，什么环境可以安全的缓存文档
    Last-Modified：//请求资源的最后修改时间
    Expires：//应该在什么时候认为文档已经过期,从而不再缓存它
    Max-age：//客户端的本地资源应该缓存多少秒，开启了Cache-Control后有效
    ETag：//请求变量的实体标签的当前值
    Set-Cookie：//设置和页面关联的cookie，服务器通过这个头部把cookie传给客户端
    Keep-Alive：//如果客户端有keep-alive，服务端也会有响应（如timeout=38）
    Server：//服务器的一些相关信息

//譬如，请求头部的Accept要和响应头部的Content-Type匹配，否则会报错
//譬如，跨域请求时，请求头部的Origin要匹配响应头部的Access-Control-Allow-Origin，否则会报跨域错误。。。
1-4、请求/响应体
...
```

##### (2) cookie 服务器向浏览器写入某些信息

##### (3) gzip 压缩效率很好（高达 70%左右）

##### (4) 长连接与短连接

        4-1、长连接:一个tcp/ip连接上可以连续发送多个数据包，在tcp连接保持期间，如果没有数据包发送，需要双方发检测包以维持此连接，一般需要自己做在线维持（类似于心跳包）http1.1起，默认使用长连接，使用长连接会有这一行Connection: keep-alive

        4-2、短连接:通信双方有数据交互时，就建立一个tcp连接，数据发送完成后，则断开此tcp连接(http1.0默认短连接)

##### (5) https

-   https 就是安全版本的 http，譬如一些支付等操作基本都是基于 https 的，因为 http 请求的安全系数太低了。
-   简单来看，https 与 http 的区别就是： 在请求前，**会建立 ssl 链接，确保接下来的通信都是加密的**，无法被轻易截取分析
-   一般来说，如果要将网站升级成 https，需要后端支持（后端需要申请证书等），然后 https 的开销也比 http 要大（因为需要额外建立安全链接以及加密等），所以\*一般来说 http2.0 配合 https 的体验更佳（因为 http2.0 更快了）

##### (6) http 2.0

    6-1、http2.0不是https，它相当于是http的下一代规范
    6-2、然后简述下http2.0与http1.1的显著不同点：
            http1.1中，每请求一个资源，都是需要开启一个tcp/ip连接的，所以对应的结果是，每一个资源对应一个tcp/ip请求，由于tcp/ip本身有并发数限制，所以当资源一多，速度就显著慢下来
            http2.0中，一个tcp/ip请求可以请求多个资源，也就是说，只要一次tcp/ip请求，就可以请求若干个资源，分割成更小的帧请求，速度明显提升。
            所以，如果http2.0全面应用，很多http1.1中的优化方案就无需用到了（譬如打包成精灵图，静态资源多域名拆分等）

    6-3、然后简述下http2.0的一些特性：
        多路复用（即一个tcp/ip连接可以请求多个资源）
        首部压缩（http头部压缩，减少体积）
        二进制分帧（在应用层跟传送层之间增加了一个二进制分帧层，改进传输性能，实现低延迟和高吞吐量）
        服务器端推送（服务端可以对客户端的一个请求发出多个响应，可以主动通知客户端）
        请求优先级（如果流被赋予了优先级，它就会基于这个优先级来处理，由服务器决定需要多少资源来处理该请求。）

#### 五、缓存

5-1、强缓存(200 from cache)与弱缓存(304)

-   强缓存（200 from cache）时，浏览器如果判断本地缓存未过期，就直接使用，无需发起 http 请求
-   协商缓存（304）时，浏览器会向服务端发起 http 请求，然后服务端告诉浏览器文件未改变，让浏览器使用本地缓存
-   对于协商缓存，使用 Ctrl + F5 强制刷新可以使得缓存无效
-   但是对于强缓存，在未过期时，必须更新资源路径才能发起新的请求（更改了路径相当于是另一个资源了，这也是前端工程化中常用到的技巧）

5-2、相关头部

> If-None-Match/E-tag、If-Modified-Since/Last-Modified、Cache-Control/Max-Age、Pragma/Expires

http1.0 中的缓存控制：

-   Pragma
-   Expires 用的是服务器时间
-   请求头(浏览器端)If-Modified-Since 与 响应头(服务器端) Last-Modifie 相匹配,说明内容未变(协商缓存)

http1.1 中的缓存控制：

-   Cache-Control：缓存控制头部，有 no-cache、max-age 等多种取值
-   max-age='xxx'：服务端配置的，用来控制强缓存，在规定的时间之内，浏览器无需发出请求，直接使用本地缓存
-   请求头(浏览器端)If-None-Match 与 响应头(服务器端) E-tag 相匹配,说明内容未变(协商缓存)

E-tag 相比 Last-Modified？

    Last-Modified：
    表明服务端的文件最后何时改变的
    它有一个缺陷就是只能精确到1s，
    然后还有一个问题就是有的服务端的文件会周期性的改变，导致缓存失效

    E-tag：
    是一种指纹机制，代表文件相关指纹
    只有文件变才会变，也只要文件变就会变，
    也没有精确时间的限制，只要文件一遍，立马E-tag就不一样了

    如果同时带有E-tag和Last-Modified，服务端会优先检查E-tag

#### 六、 浏览器接收到 http 数据包后的解析流程

> 解析 html-词法分析然后解析成 dom 树、解析 css 生成 css 规则树、合并成 render 树，然后 layout、painting 渲染、复合图层的合成、GPU 绘制、外链资源的处理、loaded 和 domcontentloaded 等

##### (1)、 解析 HTML，构建 DOM 树
    -   生产DOM树之前如果与到js代码会先执行js，再继续渲染，否则如果js有操作dom的话会渲染频繁

##### (2)、 解析 CSS，生成 CSSOM 树

##### (3)、 合并 DOM 树和 CSSOM 树，生成 render 树

##### (4)、 布局 render 树（Layout/reflow），负责各元素尺寸、位置的计算

-   Layout 也称为 Reflow，即回流,一般意味着元素的内容、结构、位置或尺寸发生了变化，需要重新计算样式和渲染树
-   Repaint，即重绘。意味着元素发生的改变只是影响了元素的一些外观之类的时候（例如，背景色，边框颜色，文字颜色等），此时只需要应用新样式绘制这个元素就可以了

> 回流的成本开销要高于重绘，而且一个节点的回流往往回导致子节点以及同级节点的回流，
> 所以优化方案中一般都包括，尽量避免回流。

引起重绘的条件

```javascript
// 1.页面渲染初始化

// 2.DOM结构改变，比如删除了某个节点

// 3.render树变化，比如减少了padding

// 4.窗口resize

// 5.最复杂的一种：获取某些属性，引发回流，
// 很多浏览器会对回流做优化，会等到数量足够时做一次批处理回流，
// 但是除了render树的直接变化，当获取一些属性时，浏览器为了获得正确的值也会触发回流，这样使得浏览器优化无效，包括
//     （1）offset(Top/Left/Width/Height)
//      (2) scroll(Top/Left/Width/Height)
//      (3) cilent(Top/Left/Width/Height)
//      (4) width,height
//      (5) 调用了getComputedStyle()或者IE的currentStyle
// 6、改变字体大小会引发回流

//一般dom结构发生变化引发回流加重绘,样色变化只会发生重绘, 回流一定伴随着重绘，重绘却可以单独出现
```

##### (5)、 绘制 render 树（paint），绘制页面像素信息

普通图层和复合图层

##### (6)、 浏览器会将各层的信息发送给 GPU，GPU 会将各层合成（composite），显示在屏幕上

### 七、 CSS 的可视化格式模型

> 元素的渲染规则，如包含块，控制框，BFC，IFC 等概念

### 八、 JS 引擎
> 作用:将js转汇编转二进制最终变为cpu可以认识的数据
> JS 的解释阶段，预处理阶段，执行阶段生成执行上下文，VO，作用域链、回收机制等等

-   常见的JS引擎
    -   `SpiderMonkey`:第一款，也是作者`Brendan Eich`开发的
    -   `Chakra`:微软开发主要`IE`浏览器
    -   `JavaScriptCore`:浏览器引擎(WebKit)中的内置JS引擎，Apply公司开发(小程序的JsCore)
        -   WebKit = 渲染工作的(WebCore) 与 JS引擎(JavaScriptCore) 组成 
    -   `V8`:Google开发
        -   `在Chrome中，只有Html的渲染采用了WebKit的WebCore代码，而在JavaScript上，重新搭建了一个NB哄哄的V8引`
        -   `谷歌浏览器体验好的原因之一`
        -   `V8`
            -   C++编写，作为js引擎的话主要用于Google浏览器与NodeJs
            -   也有作为其他语言的引擎，如`WebAssembly`
            -   可以独立运行，也能嵌入C++应用程序中 
            -   工作过程：
                - js引擎中的`Parse模块`，将js中有用到的代码转换成`AST（抽象语法树）`，引擎中的`Ignition(解释器)`才能认识解析
                - `Ignition(解释器)`将`AST`转成`ByteCode(字节码)`，同时收集`TurboFan`优化需要的信息
                -  `TurboFan(编译器)`可以将字节码编译成CPU可以直接执行的机器码
                    -   如果函数被多次调用，会被标记为热点函数，`直接`通过`TurboFan`转为机器码
                    -   但是函数后续执行时，如类型发生变化等等，机器码不能正确处理运算，就会被逆向还原为`字节码`（所以类型不固定很耗性能）
                -   `Orinoco` V8内存回收模块

### 九所有浏览器 userAgent 都是 Mozilla?

> 最初浏览器 NCSA Mosaic，简称 Mosaic,
> 后面出现另外一款浏览器 Mozilla( Mosaic + Killer)，--> Mozilla 更名为 Netscape，也就是网景
> `网站管理员探测 user agent，对 Mozilla 浏览器发送含有框架的页面，对非 Mozilla 浏览器发送没有框架的页面。`
> 后面软开发了自己的浏览器，Internet Explorer --> (开始只有 Mozilla 支持框架（frame），为了快速收到含有框架的页面了)微软宣布 IE 是兼容 Mozilla，并且模仿 Netscape 称 IE 为“Mozilla/1.22“
> 后面微软与网景的浏览器大众，网景失败退出
> Netscape 居然以 Mozilla(后面更名 Firefox)的名义重生了，并且开发了 `Gecko渲染引擎`（Mozilla/5.0(Windows; U; Windows NT 5.0; en-US; rv:1.1) Gecko/20020826）
> Mozilla 后来变成了 Firefox，并自称“Mozilla/5.0 (Windows; U; Windows NT 5.1; sv-SE; rv:1.7.5) Gecko/20041108 Firefox/1.0”
> 很多浏览器使用了它的代码，每一个都将自己装作 Mozilla，而它们全都使用 Gecko。
> `Gecko 很出色,因此 user agent 探测规则变了，使用 Gecko 的浏览器被发送了更好的代码`
> linux Konqueror 浏览器`KHTML渲染引擎` 伪装 Gecko `like Gecko`(Mozilla/5.0 (compatible; Konqueror/3.2; FreeBSD) (KHTML, like Gecko))
> Opera
> 后来苹果开发了 Safari 浏览器，并使用 KHTML 作为渲染引擎
> 但苹果加入了许多新的特性，于是苹果从 KHTML 另辟分支称之为 `WebKit`
> 但它又不想抛弃那些为 KHTML 编写的页面，于是 Safari 自称为“Mozilla/5.0 (Macintosh; U; PPC Mac OS X; de-de) AppleWebKit/85.7 (KHTML, like Gecko) Safari/85.5”
> 再后来，谷歌开发了 Chrome 浏览器，Chrome 使用 Webkit 作为渲染引擎(后面渲染引擎改用 Blink(基于 Webkit 开发)，V8 是 js 引擎不是渲染引擎)
> 和 Safari 之前一样，它想要那些为 Safari 编写的页面，于是它伪装成了 Safari
> 于是 Chrome 使用 WebKit，并将自己伪装成 Safari，WebKit 伪装成 KHTML，KHTML 伪装成 Gecko，最后所有的浏览器都伪装成了 Mozilla
> `因为网站开发者可能会因为你是某浏览器（这里是 Mozilla），所以输出一些特殊功能的程序代码（这里指好的特殊功能），所以当其它浏览器也支持这种好功能时，就试图去模仿 Mozilla 浏览器让网站输出跟 Mozilla 一样的内容，而不是输出被阉割功能的程序代码。大家都为了让网站输出最好的内容，都试图假装自己是 Mozilla，一个已经不存在的浏览器……`

-   渲染引擎之间关系(内核也叫做排版引擎、渲染引擎、浏览器引擎等)
    -   Gecko(Firefox)
    -   KHTML(linux Konqueror)
    -   KHTML --> Webkit(Safari)
    -   KHTML --> Webkit --> Blink(chrome)
    -   Presto(欧鹏) -> 欧鹏 Presto 后期被 Blink 代替
    -   Trident(IE)
    -   EdgeHTML(Edge 浏览器) --> 后期被 Blink 代替
### 十、 其它

> 可以拓展不同的知识模块，如跨域，web 安全，hybrid 模式等等内容

## 总结

[from](https://segmentfault.com/a/1190000013662126#articleHeader10)
