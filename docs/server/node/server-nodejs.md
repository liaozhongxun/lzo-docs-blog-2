---
title: nodejs基础
---

> 终端直接输入 node 进入可以运行 node 代码的换叫 REPL 环境

nodejs
- javascript 运行时
- 不是语言、不是框架、是一个平台、一个基于v8引擎的js运行环境



## 基础操作

### 三大特点:

-   事件驱动
-   非阻塞 I(input)/O(output) 输入输出流
-   基于 chrome V8 runtime 轻量高效

### 运行环境
+ 浏览器
  - 基本语法 
  - DOM
  - BOM
  - Ajax
  - 系统文件数据库(处于安全性考虑 不能实现)
+ 服务器
  - 基本语法
  - 操作数据库
  - 操作本地文件
  - 进程管理
  - 网络通讯

### 模块化
+ 核心模块(node 软件提供的)
+ 第三方模块(用npm安装)
  + nodemailer
  
+ 自定义模块(一个js文件就是一个模块)

#### 模块操作
> node使用 CommonJS 规范，浏览器是不支持的

```javascript
//由于模块作用域,导入后模块间不会互相污染,必须通过导出才可以在外部调用
const os = require("os"); //导入模块

// 默认导出
module.exports = function () {
    console.log(os.freemem());
};
//导入
const req = require("./node");
req();
-----------------------------------------------
// 多个导出1
module.exports.export1 = function () {
    console.log("export1");
};
module.exports.export2 = function () {
    console.log("export2");
};
//多个导出2
module.exports = {
    export1,
    export2
}
//导入
const { export1, export2 } = require("./node");
export1();

--------------------------------------------------
//exports 是 module.exports 的引用
exports.export1 = export1
exports.export2 = export2

exports = {xxx,xxx} //错误引用改变了


```
### 爬虫
1. 获取目标网站 (http.get)
2. 分析网站内容 (cheerio 通过jq选择器获取想要的内容 或直接用正则匹配)
3. 获取有效信息 下载或其他操作

### express
[官网](https://www.expressjs.com.cn/)
#### api
+ ip
+ 端口号
+ pathname
+ method
+ 接收用户的参数

#### template(V层)
+ ejs
+ pug
+ jade
+ [art-template](http://www.sunxiaoning.com/language/474.html)
+ 将vue当做模板放到后端执行就是vue SSR

前后端分离:管理代码方便
服务端直接渲染返页面:速度效率快，不方便管理
#### 页面渲染 render
+ SSR (Server Side render 服务端渲染,返回整个页面)
+ CSR (Client Side render 客户端渲染,返回数据如JSON)



### 什么是服务器
+ 服务器
  + 服务器的本质就是一台电脑
  + 服务器软件(如:PHP的apach、Java的tomcat、微软的iis、ngnix、node等)
  + 得到服务器IP和端口号
+ 局域网：服务器通过网线或无线相连接,每个电脑都会有一个IP
+ 外网
+ IP:找到服务器主机的位置
+ 端口号:找到主机中某个程序的位置(0-65535)
  + 0-1023： BSD保留端口，也叫系统端口(0不使用)
  + 1024-5000： BSD临时端口,留给应用程序使用
  + 5001-65535：BSD服务器(非特权)端口，用来给用户自定义端
  + http服务 80,用户访问就不要输入端口号了

### 中间件middlewear
+ 内置中间件(静态资源目录 static)
+ 自定义中间件(全局 局部)
+ 第三方中间件 (如 body-parser)

注意next()
### 静态资源目录 static
> 指定一个目录,目录可以被访问 类似Apache 的www

### 图片上传
> 原理:将图片、音乐等要上传的东西转换文数据流,通过ajax或form表单传到服务器，服务器接收这些数据后再写入文件系统之中

1. 安装multer模块
```shell
npm install multer
```
2. 使用
```javascript
var multer = require('multer');
```
3. 设置路径与文件名
```javascript
var storage = multer.diskStorage({

})
```
### api接口
- url
- 方法
- 参数
- 结果
### node 跨域 cors中间件处理

### 身份验证
http请求 无状态 服务器、客户端相互不认识

基本思路:
1. 某个用户登录成功后,将用户相关信息加密,生成一个字符串给前端（通过cookie自动传）
2. 这个用户调用其他接口时，将加密字符串传给服务器，后端通过这些字符判断用户的身份（通过cookie自动传）
3. 再根据这个用户的权限进行验证，是否可以操作

#### 方案一、session + cookie
相关插件:
-  cookie-parser 解析cookie
-  express-session 
或
-  cookie-session

使用步骤:
1. 用户输入用户名、密码进行登录
2. 成功后，后端会存一个session，保存用户信息
3. session值当做cookie内容被种到客户端	
4. cookie会在请求下一个资源时携带
5. 后端将cookie内容与session值进行对比，相等通过，不相等不通过
6. 缺点:服务器需要存每个用户的session

#### 方案二、[jwt](http://jwt.io) （json web token）
使用步骤:
- 用户登录 服务器产生一个token(加密字符串)发送给前端
- 前端将token进行保存  
- 前端发起数据请求通过 headers 携带 token
- 服务器验证token是否合法,如果合法继续操作否则终止操作  
- token应用场景:无状态请求、保存用户登录状态、第三方登录...  

两种加密方式:
非对称加密:RS256...
    通过私钥产生token、通过公钥解密token
    指加密和解密使用不同密钥的加密算法,也称为公私钥加密。

[官网]([ https://www.openssl.org/source/](https://www.openssl.org/source/))    [下载地址](http://slproweb.com/products/Win32OpenSSL.html)

```shell
# openssl方式
# 生成私钥 
openssl 回车
OpenSSL> genrsa -out rsa_private_key.pem 2048  #(-out 文件名 字符数) 

# 根据私钥生成公钥
OpenSSL> rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem #(-in 私钥名 -pubout -out 公钥名)
```

对称加密：指加密和解密使用`相同密钥`的加密算法。对称加密算法用来对敏感数据等信息进行加密,常用的算法包括DES、3DES、AES、DESX、Blowfish、RC4、...


插件  
- jsonwebtoken
  
```javascript
const jwt = require("jsonwebtoken");
const scrict = "jfjdsajfdsajfdsa"; //私钥 （对称加密解密都是它）

let playload = {
    //传递的数据
    us: 123,
    ps: 456,
};

function creatToken(playload) {
    //产生token
    playload.ctime = Date.now();
    playload.exp = 1000 * 60 * 30; //30分钟过期
    // 签名 默认hs256加密方式
    return jwt.sign(playload, scrict);
}

function checkToken(token) {
    return new Promise((resovle, reject) => {
        //验证
        jwt.verify(token, scrict, (err, data) => {
            if (err) {
                reject("token 验证失败");
            }
            //ctime + exp < Data().now 说明超时了 
            resolve(data);
        });
    });
}

module.exports = {
    creatToken,
    checkToken,
};

```

#### 密码加密 bcrypt
> 每次加密得到的hash字符串是不一样的,验证时可以通过输入的密码与hash进行匹配

```shell
npm install bcrypt -S
```
### Express RMVP 模式
> MVC -> MVP|RMVP -> MVVM ,Web 设计模式

[参考](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)
#### MVC
+ M:Model,模型层，数据相关的操作
+ V:View,视图层，用户界面渲染逻辑(html、css...)
+ C:控制器，js
+ MVC之间三方都能相互通讯
#### MVP
+ M:Model,模型层，数据相关的操作
+ V:View,视图层，用户界面渲染逻辑(html、css...)
+ P:Presenter,响应视图指令，同时进行相关业务处理，必要时候获调用 Model 获取底层数据，返回指令结果到视图，驱动视图渲染
+ MVP中M和V无法通讯
  + Model 不再负责业务逻辑和视图变化，只负责底层数据处理
  + View 层只负责发起指令和根据数据渲染 UI，不再有主动监听数据变化等行为，所以也被称之为被动视图
+ R:请求,RMVP是通过请求与控制器通讯(如 app.use("/R",xxx)) 
#### MVVM
+ M:Model,模型层，数据相关的操作
+ V:View,视图层，用户界面渲染逻辑(html、css...)
+ VM:ViewModel,
+ 与MVP唯一的区别是，它采用双向绑定(data-binding),P虽然能与V相互通信,但是VM与V一端变化另一端直接变化


### node服务调用谷歌控制台
+ 通过 nodemon --inspect --inspect-brk? server.js 开启服务 
+ 配合debugger使用
+ 谷歌打开地址: chrome://inspect/#devices 
+ 点击 Open dedicated DevTools for Node -> Connection 添加域名端口号
+ 就能在 Open dedicated DevTools for Node -> console 中天使node项目了
  
### 开发中常用插件
+ log4js //日志操作

[参考资料](https://www.bilibili.com/video/BV1Ci4y1L7gk?p=7)


```
去年工作总结与2021年工作计划
    在去年一年时间里,逐渐完善了iot管理系统云版本，内网版，三套监控大屏系统，内网版的3D设备系统。移动端微信公众号也逐渐成熟，辉和、通佰、和猎狐三套样式都在维护，企业微信也有一套系统可以查看信息，也能方便维护人员在外面能做一些少量的管理，同时在休息时间学习了uni-app，nodejs，TypeScript，微信小程序以及云开发等新技术
    如果有时间并且公司需要的话在2021年的计划中可以通过uni-app技术为公司开发移动端app、微信小程序、百度小程序、支付宝小程序等各个平台系统，可以通过云服务配合nodejs开发一些简单的服务器功能,不需要额外购买分配服务器，此外新的一年里如果时间计划的话，还会将现有系统进行适当优化，有适合的ui的话也可以计划将界面进行改版

```