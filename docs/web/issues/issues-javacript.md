---
title: javascript
---

### JS

#### 跨域方案总结

#### 事件循环 EventLoop

浏览器的事件循环

-   浏览器`渲染进程`、网络进程、GPU 进程....
-   重点`渲染进程`包含`js引擎进程`、`HTTP请求进程`、`定时器触发进程`、`事件触发进程`、`GUI进程`等
-   浏览器与`NodeJS`异步任务执行原理，背后是通过`事件驱动`完成的
-   事件驱动包括`事件触发`、`任务选择`、`任务执行`
-   由特定的事件触发特定的任务（如用户 click 事件，自动的定时器事件）
-   事件循环就是在事件驱动模式中，管理和执行事件的流程
-   在事件驱动中，当有事件触发后吧，被触发的事件会按顺序，暂时存在一个队列中，等待 js 同步任务执行完成，再从队列中取出`要处理的事件处理` - 事件循环来控制什么时候取事件，或优先取的事件
    浏览器与 js
    > 浏览器多线程，只会给 js 一个线程,所有 js 是单线程
    > js 单线程，所以只能通过浏览器的多线程来执行`异步任务`
    > js 执行代码的`主线程`只有一个,`浏览器提供的JS引擎线程`
    > 此外还有`定时器线程`、`HTTP请求线程`、`Promise线程`等,来执行其他任务
    > 当开定时器或调用接口时，这些任务将分配给他们对应的线程来做，完成了再回到住线程
    > 执行步骤
-   运行首先执行主线程(将同步代码按顺序排在`执行栈`中)，当遇到 Promise 或定时器时会丢到任务列队(Event Quque)里,继续执行主线程的内容
-   主线程完了才会走任务列队(已完成的)的微任务(await，Promise 等)，再走宏任务(Ajax、定时器,事件绑定等),加入执行栈继续执行,宏任务完了就运行结束了
    -   微任务(micro task):先，包含:Promise、
        -   微任务不会有浏览器进线帮忙处理，类似将`.then`的代放到主进程尾部，当前循环执行
    -   宏任务(macro task):后，包含:定时器相关、Ajax、I/O 操作
        -   宏任务有浏览器线程帮忙异步处理，下次循环，如有结果再通过回调执行
    -   其他：requestAnimationFrame
        -   事件周期最后是页面渲染，二页面渲染之前调用 requestAnimationFrame 回调，当前周期执行的，如果说宏任务是下次 tick 执行的话，那么就不属于宏任务
        -   但是 window.requestAnimationFrame 即使在`Promise`前面调用，也是渲染页面前最后执行，微任务顺序不定
        -   window.requestAnimationFrame() 告诉浏览器——你希望执行一个动画，时间是下次重绘之前。
-   主线程执行完后去找一个个的微宏任务，找到一个拿到主线程继续执行执行完再`不断循环`去找。。。，就叫做 Event Loop 事件循环
-   每次循环就是一个事件周期(tick)
-   每个周期结束之后浏览器才会对页面进行渲染，所以有时操作 DOM 不一定会立马刷新视图
-   视图重汇之前会执行`requestAnimationFrame`回调
-   主线程(执行栈)每次执行方法，会生成一个独有的执行环境(上下文 context)，执行结束，销毁独有环境，并从栈弹出此方法，继续下面代码
-   宏任务误差延迟问题:如果将定时器设置 100ms 后执行，首先会挂掉任务列队，100ms 时间到了,如果主线程还在执行中，定时器只能等待，同步代码越长，误差将会越大

Node 的事件循环
[学习](https://www.bilibili.com/video/BV1gb4y1U7Un?spm_id_from=333.999.0.0)

#### call、apply、bind

-   共同点
    -   都能改变函数内部 this 指向
-   使用

```javascript
function fu(a, b) {
    console.log(this);
}
let obj = {
    c: 10,
};

//fu函数调用call,通过原型链机制，找到fu.prototype上的call函数进行调用
//fu.call(newThis,a,b);
// 非严格模式
fu.call(); //不传，指向window(默认指向)
fu.call(null); //传null,指向window
fu.call(undefined); //传undefined ，指向window
fu.call(1, 2); //传1，指向 Number(1)
fu.call(obj, 1, 2); //传obj,指向obj

//严格模式
// fu.call() //不传，指向undefined
// fu.call(null) //传null,指向null
// fu.call(undefined) //传undefined ，指向undefined
// fu.call(1,2) //传1，指向 1
// fu.call(obj,1,2) //传obj,指向obj

//apply,只是参数列表通过数组传递
fu.apply(obj, [1, 2]);

//bind 用法也与call类似，区别是是否立即执行
fu.bind(obj, 1, 2)();
```

-   实现 call

    > 三点：修改 this、执行函数、返回行数返回值

```javascript
Function.prototype.call = function (thisArg, args) {
    // this指向调用call的对象
    if (typeof this !== "function") {
        // 调用call的若不是函数则报错
        throw new TypeError("Error");
    }
    thisArg = thisArg || window;
    thisArg.fn = this; // 将调用call函数的对象添加到thisArg的属性中
    const result = thisArg.fn(...[...arguments].slice(1)); // 执行该属性
    delete thisArg.fn; // 删除该属性,执行的fn内部this中依然存在该函数，这个删除是函数执行后才做的
    return result;
    问题: 如何去除调用函数fn时内部this对象会出现fn自己的问题;
};
```

#### new 内部做了什么？

#### 作用域

> 全局作用域、函数作用域、块级作用域、词法作用
> js 用的`词法作用域`:这就意味着函数的执行依赖于函数`定义的时候`所产生（而不是函数调用的时候产生的）的`变量作用域`。

#### 闭包

> 闭包就是外层函数中 return 出新函数,使新函数通过外层函数在外面可以使用，新函数中可以使用外层函数中定义的变量

```javascript
function fun(n, o) {
    console.log(n, o);
    return {
        fun2: function (m) {
            return fun(m, n);
        },
    };
}

var a = fun(0); // ?0
a.fun2(1); // ?
a.fun2(2); // ?
a.fun2(3).fun2(4); // ?

var b = fun(0).fun2(1).fun2(2).fun2(3); // ?
var c = fun(0).fun2(1); // ?
c.fun2(2); // ?
c.fun2(3);
```

#### Memoization

> 算法技巧叫做`记忆华搜索`，目的:`为了减少重复计算`,如递归或其他重复计算多的场景适合使用

```javascript
// 获取函数运行时间
function getTime(fn, ...args) {
    const now = Date.now();
    const result = fn.apply(null, args);
    console.log(`用时：${Date.now() - now}毫秒，结果：${result}`);
}

// 斐波那契数列
function fibonacci(n) {
    if (n === 0) return 0;
    else if (n === 1) return 1;
    else return fibonacci(n - 1) + fibonacci(n - 2);
}
getTime(fibonacci, 40);

// memoization化斐波那契数列
const fibonacciWithCache = (function () {
    // 设置缓存队列
    const cacheList = [0, 1, 1, 2];
    return function _fibonacciWithCache(n) {
        // 如果缓存队列中没有该运算结果，则计算出结果并加入缓存队列中
        if (!cacheList[n])
            cacheList[n] =
                _fibonacciWithCache(n - 1) + _fibonacciWithCache(n - 2);
        return cacheList[n];
    };
})();

getTime(fibonacciWithCache, 40);
```

-   两者对比
    用时：2830 毫秒，结果：102334155
    用时：0 毫秒，结果：102334155

#### 柯里化、偏函数、Compose、Pipe

> 收集函数多次调用的参数了列表

-   作用
    -   参数复用
        -   就是利用闭包的原理，让我们前面传输过来的参数不要被释放掉
    -   提前确认
    -   延迟运行

```javascript
/**
 * 固定参数类型，函数柯里化
 * 关键利用闭包的特性保存每次调用参数个数
 * @param {*} func
 * @param  {...any} args
 * @returns
 */
const curry = (func, ...args) => {
    // 获取函数的参数个数,不调用
    const fnLen = func.length;

    //利用闭包与递归收集所有小括号参数个数,如果func参数列表用到的少，后面的小括号都是白写
    return function (...innerArgs) {
        // 每个小括号调用的参数...innerArgs加上前面调用过储存的参数，拿来与func的参数比较
        innerArgs = args.concat(innerArgs);
        // 参数未搜集足的话，继续递归搜集
        if (innerArgs.length < fnLen) {
            return curry.call(this, func, ...innerArgs); //...innerArgs至少从第二个小阔号调用开始
        } else {
            // 否则拿着搜集的参数调用func,这时开始调用func回调
            func.call(this, innerArgs);
        }
    };
};
// 测试
const add = curry((num1, num2, num3) => {
    console.log(num1);
    // console.log(num1, num2, num3, num1 + num2 + num3);
});

add(1)(2)(3); // 1 2 3 6
add(1, 2)(3); // 1 2 3 6
add(1, 2, 3); // 1 2 3 6
add(1)(2, 3); // 1 2 3 6

//============================================
function curry2() {
    // 第一次执行时，定义一个数组专门用来存储所有的参数
    var _args = Array.prototype.slice.call(arguments);

    //arguments类数组Array.prototype.slice.call(arguments)将有length的对象转数组
    console.log(arguments.length);

    // 在内部声明一个函数，利用闭包的特性保存_args并收集所有的参数值
    var _adder = function () {
        _args.push(...arguments);
        return _adder; //第二次调用执行_adder，返回_adder给第三次执行用
    };

    // 利用toString隐式转换的特性，当最后执行时隐式转换，并计算最终的值返回
    _adder.toString = function (func) {
        return func(_args);
    };

    //外部回调操作
    _adder.toString1 = function (callback) {
        callback(_args);
    };

    //将内部函数返回出去
    return _adder; //第一次调用curry2执行得到_adder函数
}

//获取操作列表自定义函数操作
function func(_args) {
    console.log(
        _args.reduce(function (a, b) {
            return a + b;
        })
    );
}
curry2(1, 2, 3)(4)(9).toString(func);

curry2(
    1,
    2,
    3
)(4)(9).toString1((list) => {
    console.log(list);
});
```

#### 函数式编程的纯函数与副作用

#### JS 的内存管理

> 本质上讲, 内存泄露就是不再被需要的内存, 由于某种原因, 无法被释放.
> JS 中, 没隐藏了内存管理功能,有专门的内存管理接口, 所有的内存管理都是"自动"的. JS 在创建变量时, 自动分配内存, 并在不使用的时候, 自动释放. 这种"自动"的内存回收, 造成了很多 JS 开发者并不关心内存回收
> 全局变量引用的空间需要程序结束才能是否，局部变量指向的空间，需要改作用域执行完后自动释放

> 对象是否不再需要(没有任何变量指向的对象)

```javascript
let arr = [1, 2, 3, 4];
arr = null; //手动赋值null， [1,2,3,4]这时没有被引用, 会被自动回收
```

#### JS 耗性能操作与时间复杂度

#### 事件对象鼠标位置

> 事件对象 clientX、screenX、pageX、offsetX、layerX、movementX 的差别

#### 所有浏览器 userAgent 都是 Mozilla?

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

-   渲染引擎直接关系
    -   Gecko(Firefox)
    -   KHTML(linux Konqueror)
    -   KHTML --> Webkit(Safari)
    -   KHTML --> Webkit --> Blink(chrome)
    -   Presto(欧鹏) -> 欧鹏 Presto 后期被 Blink 代替
    -   Trident(IE)
    -   EdgeHTML(Edge 浏览器) --> 后期被 Blink 代替

#### js 判断服务器图片是否存在

```javascript
var ImgObj = new Image(); //判断图片是否存在
ImgObj.src = "xxxx.png";
ImgObj.onload = function () {
    console.log("图片存在");
};
ImgObj.onerror = function () {
    console.log("图片不存在");
};
```

#### 进制

```javascript
/**
 * 十进制:100
 *
 * 二进制:01100100

 * 八进制:01 100 100 => 144 
 *  占三位 最大值1+2+4 = 7

 * 十六进制:0110 0100 => 64
 *  占四位 最大值1+2+4+8 =15 
 *  颜色值六个十六进制占三个字节24位 => #ffffff => rgb(0b11111111,0b11111111,0b11111111) 所以范围是0-255,所以三个字节的颜色拥有256*256*256种颜色
 */
```

#### js 插入样式

```javascript
var style = "<style>#print .linetow{text-align:center}</style>";
var ele = document.createElement("div");
ele.innerHTML = style;
document.getElementsByTagName("head")[0].appendChild(ele.firstElementChild);
```

#### 类数组转数组

```javascript
function getArray() {
    console.log(arguments);
    //1. 原理是数组的slice()方法可以从已有数组中返回一个新数组，它可以接受两个参数arr.slice(start,end),如果不传参将返回原数组的一个副本，但该方法不会修改原数组，而是返回截取的新数组
    console.log(Array.prototype.slice.call(arguments));
    //2. splice(start,count,item) 改变原数组
    console.log(Array.prototype.splice.call(arguments, 0));
    //3. Array.from(arguments)
    console.log(Array.from(arguments));
    //4. Array.apply(null, arguments)
    console.log(Array.apply(null, arguments));
    //5. [].concat.apply([], arguments)
    console.log([].concat.apply([], arguments));
    //6. 循环遍历类数组对象，push到新创建的数组对象里
}

getArray(1, 2, 3);
```

#### 数组扁平化

```javascript
// 数组扁平化
let arr = [1, 2, [3, 4], 5, [8, [9, 10]]];
// 1. arr.flat默认两层 -> Infinity不限制层数
console.log(arr.flat(Infinity));
// 2. toString
console.log(
    arr
        .toString()
        .split(",")
        .map((item) => Number(item))
);
// 3. reduce
const flatten = (array) =>
    array.reduce((acc, cur) => {
        // console.log(acc, cur);
        return Array.isArray(cur) ? [...acc, ...flatten(cur)] : [...acc, cur];
    }, []);
console.log(flatten(arr));
// 4. xxxx
```

#### Blob

>   Blob(二进制大对象)对象是一个用来包装二进制文件的容器，File继承于Blob
>
>   **IE9-浏览器不支持**

##### Blob 创建

```javascript
var myBlob = new Blob([1,2,3],{type:'text/plain'});
console.log(myBlob);
console.log(myBlob.size);
console.log(myBlob.type);
// blob.slice 大文件切割返回小 blob

// 通过input FileList获取文件Blob
```

##### Blob 下载

```javascript
<script type="text/javascript">
    //创建XMLHttpRequest对象
    var xhr = new XMLHttpRequest();
    //配置请求方式、请求地址以及是否同步
    xhr.open('POST', '/sfwanshengjie/mp4/shipin1.mp4', true);
    //设置请求结果类型为blob
    xhr.responseType = 'blob';
    //请求成功回调函数
    xhr.onload = function(e) {
        if (this.status == 200) {
            //获取blob对象
            var blob = this.response;
            console.log(blob);
            //获取blob对象地址，并把值赋给容器
            document.getElementById("video").src = URL.createObjectURL(blob);
        }
    };
    xhr.send();
</script>
```



##### Blob转地址

```javascript
// 把blob转化成当前页面的一个data:image/jpeg;base64内存地址
let fr = new FileReader();
fr.readAsDataURL(file|blob);
fr.onloadend = function (e) {
    let base64 = e.target.result;
    console.log(base64)
};

// 把blob转化成当前页面的一个blob:xxxx内存地址
let src = window.URL.createObjectURL(file|blob); // 这个方法也可以传一个file
console.log(src)
img.src = src;
```
-	URL.createObjectURL(file|blob) 可以获取当前文件的一个内存URL
	-   得到 blob:http://127.0.0.1:5500/xxxxxxxx 格式地址，不是base64的
	-	比base64地址小节约空间
	-	立即执行的同步生成，FileReader需要在onload下异步获取base64
	-	URL.revokeObjectURL释放该地址
	-	data://URL会对内容进行编码。blob://URL只是对浏览器存储在内存中或者磁盘上的Blob的一个简单引用



#### JS获取base64的方式

>   base64是二进制数据的一个编码格式

-   JS通过 FileReader 获取base64

```javascript
let fileReader = new FileReader();
fileReader.readAsDataURL(file|Blob) // base64形式 读取图片
fileReader.onload = (e) => { //图片读取完成
    console.log(e.target.result);
};
```



-   JS通过 canvas 获取 base64

```javascript
// 这个image就是输入
// 除了new，也可以直接取页面上的标签
var image = new Image();

image.onload = function () {
   var w = image.width;
   var h = image.height;
   var canvas = document.createElement('canvas');
   var ctx = canvas.getContext("2d");
   canvas.width = w;
   canvas.height = h;
   ctx.drawImage(image, 0, 0, w, h);
   // 可以在这里添加水印或者合并图片什么的    
   ...
   // 把画布的内容转成base64，这个就是输出
   var base64 = canvas.toDataURL('image/jpeg'，0.1);//参数2压缩比例
   console.log(base64)
}
// 这个src可以是本地路径，服务器图片地址，也可以是上面fileReader的base64
image.src = "xxx.jpg";
```

#### 图片上传

[xxxx](https://www.cnblogs.com/pengdt/p/12037986.html)

-   多选:multiple
-   指定类型:accept="image/\*"
-   类数组 input.flies 返回一个 FileList 选择的图片数据
-   capture 调用摄像头

```html
<input id="uploadFile" type="file" accept="image/*" />
<button id="submit" onclick="uploadFile()">上传文件</button>
```

##### FileReader

>   FileReader是用来读取内存中的文件的API，支持File和Blob两种格式。

-   FileReader.readyState
    -   0：EMPTY/还没有加载任何数据
    -   1：LOADING/数据正在被加载
    -   2：DONE/已完成全部的读取请求
-   FileReader.result
-   FileReader.error
```javascript
// readAsArrayBuffer(file) :按字节读取文件内容，结果用ArrayBuffer对象表示
// readAsBinaryString(file) :按字节读取文件内容，结果为文件的二进制串
// readAsDataURL(file) :读取文件内容，结果用data:url的字符串形式表示
// readAsText(file,encoding) :按字符读取文件内容，结果用字符串形式表示
// abort() :终止文件读取操作
// 事件
// onloadstart 当读取操作开始时调用
// onprogress 在读取数据过程中周期性调用
// onabort 当读取操作被中止时调用
// onerror 当读取操作发生错误时调用
// onload 当读取操作成功完成时调用
// onloadend 当读取操作完成时调用，无论成功，失败或取消

let fileReader = new FileReader();
fileReader.readAsDataURL(file) // base64形式 读取图片
fileReader.onload = (e) => { //图片读取完成
    console.log(e.target.result);
};
//或
let fileReader = new FileReader();
fileReader.readAsDataURL(file)
fileReader.addEventListener('load', function() {
  // 读取完成
  let res = fileReader.result
  // res是base64格式的图片
})
```

##### FormData
> 用一些键值对来模拟一系列表单控件：即把 form 中所有表单元素的 name 与 value 组装成 一个 queryString
```javascript
let formData = new FormData();
//各种方式给formData添加文件数据
formData.set(fieldName, file);

/*
    name:属性名
    value:属性值，在我们这里则指 file 数据
    filename:当第二个参数为 file 或 blob 时，告诉服务器的文件名。Blob 对象的默认文件名是“blob”。File 对象的默认文件名 是文件的文件名。
*/
formData.append(name, value, filename)
```

##### 上传

```javascript
let config = {
    headers: {
        "Content-Type": "multipart/form-data",
    },
};
axios
    .post("url/load", formDate, config)
    .then((res) => {
        console.log(res);
    })
    .catch((err) => {
        console.log(err);
    });
```

##### 大文件上传
	-	file继承Blob利用Blob的slice方法将文件切片
	-	确定每片大小
	-	获取总大小 (file.size)
	-	片段数量 (Math.ceil(总大小/每片大小))
	-	定义一个偏移量决定每次调接口传哪一段，每次调用便宜量++
#####  Canvas图片上传
	-	通过 canvas.toDataURL('image/jpeg') 上传base64上传   


#### 字符串编码
```javascript
/*
ASCII 
ASCII  使用一个字节表示一个字符(8bit)，不使用最高位
二进制  0000 0000    十进制   0         
        0111 1111            127

最多只能表示128个字符
数字与编码存在着对应关系chr 和 ord 可查看对应关系

-----------------------------------------------
latin1(ISO-8859-1)
向下兼容ASCII ,启用最高位
二进制曾 1000 0000    十进制  128
        1111 1111            255

最多能表示255个字符

-------------------------------------------------
Unicode(国际码) 表示更多字符,大多数国家的文字都有一个对应编码
(1) Unicode 与 latin1无关,前128兼容ASCLL
(2) 通过连个字节来表示更多的自己付 最多可达16位
(3) 为了让计算机知道后面的两个字节代表的是一个字符,还是两个字符,所以所有字符都有两个字节代替,不够就补0(比较浪费空间)
(4)如果将一个字符的16位当做两个8位处理就会造成乱码(如:x.encode('utf8')  得到的结果 decode('gbk‘)情况)

(5) 基于浪费空间问题于是产生很多Unicode 的编码方式如:  UTF-8 、 UTF-16、GBK、BIG5 或 UTF-32等了


Unicode 的编码方式不同编码方式以不同规则表示
GBK: 国标扩，规定汉子占两个字节,用于简体中文
BIG5:繁体中午
UTF-8:统一编码 汉字占三个字节
-------------------------------------------------
*/
```
#### js 代码整洁之道

[暂时参考](https://zhuanlan.zhihu.com/p/159458364)
[暂时参考](https://www.cnblogs.com/wenxinsj/p/14646550.html)
[暂时参考](https://www.jianshu.com/p/fb4409d8ace2)

#### 断点调试

> 在某一行打下断点,当浏览器执行到这一行时，程序暂停，可以观察限制，代码状态，变量值等，在通过下一步下一步查看代码走向以及值的变化
