# 技术学习 js杂七杂八3

<!-- toc -->

<!--
  - 渐进增强与优雅降级
  - XMLHttpRequest level 2
  - callee 和 caller
  - cookie 和 session
  - 常用 http 状态码
  - bind 实现
  - 为什么传统上利用多个域名来提供网站资源会更有效
  - 从输入 url 到页面展示到底发生了什么
  - TCP 三次握手
  - 重定向 301 和 302
  - 重排(reflow)与重绘(repaint)
  - XSS
  - prototype 与 __proto__
-->

## 渐进增强与优雅降级

- 渐进增强(progressive enhancement): 针对低版本浏览器进行构建页面, 保证最基本的功能, 然后再针对高级浏览器进行效果/交互等改进和追加功能达到更好的用户体验

- 优雅降级(graceful degradation):一开始就构建完整的功能, 然后再针对低版本浏览器进行兼容

- 降级意味着往回看; 而增强则意味着朝前看，同时保证其根基处于安全地带


## XMLHttpRequest level 2

旧版本缺点:

- 只支持文本数据的传送，无法用来读取和上传二进制文件
- 传送和接收数据时，没有进度信息，只能提示有没有完成
- 受到"同域限制"(Same Origin Policy)，只能向同一域名的服务器请求数据

新版本:

- 可以设置 HTTP 请求的时限
- 可以使用 FormData 对象管理表单数据
- 可以上传文件
- 可以请求不同域名下的数据(跨域请求 CORS)
- 可以获取服务器端的二进制数据
- 可以获得数据传输的进度信息

## callee 和 caller

arguments 的主要用途是保存函数参数, 这个对象有一个名叫 callee 的属性, 该属性是一个指针, 指向拥有这个 arguments 对象的函数

```js
function factorial(num){
    if (num <=1) {
        return 1;
    } else {
        return num * arguments.callee(num-1)
    }
}
```

caller 这个属性中保存着调用当前函数的函数的引用, 如果是在全局作用域中调用当前函数, 它的值为 null

```js
function outer(){
    inner();
}
function inner(){
    console.log(inner.caller);
    console.log(arguments.callee.caller); // 一样
}
outer(); // 显示 outer 的源码
```

## cookie 和session
cookie 和session 的区别：

- cookie 数据存放在客户的浏览器上, session 数据放在服务器上
- cookie 不是很安全，别人可以分析存放在本地的 cookie 并进行欺骗, 考虑到安全应当使用 session
- session 会在一定时间内保存在服务器上, 当访问增多，会比较占用你服务器的性能考虑到减轻服务器性能方面，应当使用cookie
- 单个 cookie 保存的数据不能超过 4K，很多浏览器都限制一个站点最多保存 20 个 cookie
- 建议将登陆信息等重要信息存放为 session, 其他信息如果需要保留可以放在 cookie 中

## 常用 http 状态码

- 2开头 (请求成功)表示成功处理了请求的状态代码。
    200   (成功)  服务器已成功处理了请求。 通常，这表示服务器提供了请求的网页。 

- 3开头 (请求被重定向)表示要完成请求，需要进一步操作。 通常，这些状态代码用来重定向。
    301   (永久移动)  请求的网页已永久移动到新位置。 服务器返回此响应(对 GET 或 HEAD 请求的响应)时，会自动将请求者转到新位置。
    302   (临时移动)  服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
    304   (未修改) 自从上次请求后，请求的网页未修改过。 服务器返回此响应时，不会返回网页内容。 

- 4开头 (请求错误)这些状态代码表示请求可能出错，妨碍了服务器的处理。
    401   (未授权) 请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。
    403   (禁止) 服务器已经理解请求，但是拒绝执行它。
    404   (未找到) 服务器找不到请求的网页。
    407   (需要代理授权) 此状态代码与 401(未授权)类似，但指定请求者应当授权使用代理。

- 5开头 (服务器错误)这些状态代码表示服务器在尝试处理请求时发生内部错误。 这些错误可能是服务器本身的错误，而不是请求出错。
    500   (服务器内部错误)  服务器遇到错误，无法完成请求。 
    501   (尚未实施) 服务器不具备完成请求的功能。 例如，服务器无法识别请求方法时可能会返回此代码。

## bind 实现

非常简单的实现:

```js
Function.prototype.bind = function (scope) {
    var fn = this;
    return function () {
        return fn.apply(scope);
    };
}
```

[MDN 实现](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function (oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5 internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs = Array.prototype.slice.call(arguments, 1),
      fToBind = this,
      fNOP = function () {},
      fBound = function () {
        return fToBind.apply(this instanceof fNOP ?
          this :
          oThis,
          aArgs.concat(Array.prototype.slice.call(arguments)));
      };

    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype;
    }
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

## 为什么传统上利用多个域名来提供网站资源会更有效

- 静态内容和动态内容分服务器存放, 使用不同的服务器处理请求。处理动态内容的只处理动态内容, 不处理别的, 提高效率, 这样使得CDN缓存更方便
- 突破浏览器并发限制
- 跨域不会传 cookie,节省宽带, 也保证安全

## 从输入 url 到页面展示到底发生了什么

[参考](http://www.cnblogs.com/xianyulaodi/p/6547807.html)

1. 输入地址

2. 浏览器查找域名的 IP 地址　　
    - 请求一旦发起，浏览器首先要做的事情就是解析这个域名，一般来说，浏览器会首先查看本地硬盘的 hosts 文件，看看其中有没有和这个域名对应的规则，如果有的话就直接使用 hosts 文件里面的 ip 地址。

    - 如果在本地的 hosts 文件没有能够找到对应的 ip 地址，浏览器会发出一个 DNS请求到本地DNS服务器 。本地DNS服务器一般都是你的网络接入服务器商提供，比如中国电信，中国移动。

    - 查询你输入的网址的DNS请求到达本地DNS服务器之后，本地DNS服务器会首先查询它的缓存记录，如果缓存中有此条记录，就可以直接返回结果，此过程是递归的方式进行查询。如果没有，本地DNS服务器还要向DNS根服务器进行查询。

    - 根DNS服务器没有记录具体的域名和IP地址的对应关系，而是告诉本地DNS服务器，你可以到域服务器上去继续查询，并给出域服务器的地址。这种过程是迭代的过程。

    - 本地DNS服务器继续向域服务器发出请求，在这个例子中，请求的对象是.com域服务器。.com域服务器收到请求之后，也不会直接返回域名和IP地址的对应关系，而是告诉本地DNS服务器，你的域名的解析服务器的地址。

    - 最后，本地DNS服务器向域名的解析服务器发出请求，这时就能收到一个域名和IP地址对应关系，本地DNS服务器不仅要把IP地址返回给用户电脑，还要把这个对应关系保存在缓存中，以备下次别的用户查询时，可以直接返回结果，加快网络访问。

3. 浏览器向 web 服务器发送一个 HTTP 请求
    - 建立了TCP/IP的连接

4. 服务器的永久重定向响应
    - 服务器给浏览器响应一个301永久重定向响应，这样浏览器就会访问 http://www.google.com/ 而非 http://google.com/

5. 浏览器跟踪重定向地址
    - 现在浏览器知道了 http://www.google.com/ 才是要访问的正确地址，所以它会发送另一个http请求

6. 服务器处理请求

7. 服务器返回一个 HTTP 响应　

8. 浏览器显示 HTML
    - 解析html以构建dom树 -> 构建render树 -> 布局render树 -> 绘制render树

9. 浏览器发送请求获取嵌入在 HTML 中的资源(如图片、音频、视频、CSS、JS 等等)

## TCP 三次握手与四次挥手

### 三次握手

- 第一次握手：客户端A将标志位SYN置为1,随机产生一个值为seq=J（J的取值范围为=1234567）的数据包到服务器，客户端A进入SYN_SENT状态，等待服务端B确认；

- 第二次握手：服务端B收到数据包后由标志位SYN=1知道客户端A请求建立连接，服务端B将标志位SYN和ACK都置为1，ack=J+1，随机产生一个值seq=K，并将该数据包发送给客户端A以确认连接请求，服务端B进入SYN_RCVD状态。

- 第三次握手：客户端A收到确认后，检查ack是否为J+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=K+1，并将该数据包发送给服务端B，服务端B检查ack是否为K+1，ACK是否为1，如果正确则连接建立成功，客户端A和服务端B进入ESTABLISHED状态，完成三次握手，随后客户端A与服务端B之间可以开始传输数据了。

"三次握手"的目的是: 防止已失效的连接请求报文段突然又传送到了服务端因而产生错误

### 四次挥手

- 第一次挥手：Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态。

- 第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。

- 第三次挥手：Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态。

- 第四次挥手：Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。

建立连接是三次握手, 而关闭连接却是四次挥手的原因:

这是因为服务端在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。而关闭连接时，当收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送。

## 重定向 301 和 302 

### 301 和 302 的区别
　　301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取（用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）——这是它们的共同点。

　　他们的不同在于。301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；

　　302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容而保存旧的网址。 SEO302好于301

### 重定向原因：

- 网站调整（如改变网页目录结构）；
- 网页被移到一个新地址；
- 网页扩展名改变(如应用需要把.php改成.Html或.shtml)。这种情况下，如果不做重定向，则用户收藏夹或搜索引擎数据库中旧地址只能让访问客户得到一个404页面错误信息，访问流量白白丧失；再者某些注册了多个域名的网站，也需要通过重定向让访问这些域名的用户自动跳转到主站点等。
 
## 什么时候进行 301 或者 302 跳转

当一个网站或者网页24—48小时内临时移动到一个新的位置，这时候就要进行302跳转，而使用301跳转的场景就是之前的网站因为某种原因需要移除掉，然后要到新的地址访问，是永久性的。

清晰明确而言：使用301跳转的大概场景如下：
- 域名到期不想续费（或者发现了更适合网站的域名），想换个域名
- 在搜索引擎的搜索结果中出现了不带www的域名，而带www的域名却没有收录，这个时候可以用301重定向来告诉搜索引擎我们目标的域名是哪一个
- 空间服务器不稳定，换空间的时候

## 重排与重绘

- 重排 (reflow)  浏览器构建渲染树完成时不包含位置和大小信息。计算元素位置和其他几何信息的过程称为重绘。
- 重绘 (repaint) 当布局结束后，浏览器遍历呈现树，调用呈现器的paint方法，将呈现器的内容显示在屏幕上。

如下改变能引起重排:

- 重新调整浏览器窗口大小
- 修改字体
- 添加/删除样式表
- 修改页面元素内容
- 激活CSS伪类, 如a:hover
- 修改class的属性
- 修改DOM
- 计算offsetWidth和offsetHeight
- 设置style的属性

优化方向:

- 修改元素的class属性，并且尽可能在DOM树中比较低的节点上
- 避免在内联样式中设置多重属性
- 将动画应用在absolute定位或者fixed的元素上
- 减少table布局
- 避免使用CSS表达式

## XSS

XSS (Cross-Site Script) 攻击又叫跨站脚本攻击, 本质是一种注入攻击. 其原理, 简单的说就是利用各种手段把恶意代码添加到网页中, 并让受害者执行这段脚本. XSS能做用户使用浏览器能做的一切事情. 同源策略也无法保证不受XSS攻击，因为此时攻击者就在同源之内.

防止xss攻击:

### 转义
无论是服务端型还是客户端型xss，攻击达成都需要两个条件:代码被注入和代码被执行, 其实只要做好无论任何情况下保证代码不被执行就能完全杜绝xss攻击.总之, 任何时候都不要把不受信任的数据直接插入到dom中的任何位置, 一定要做转义。

对于某些位置,不受信任的数据做转义就可以保证安全
- 一般的标签属性值
- div body 的内部html

对于某些位置，即使做了转义依然不安全
- script标签中
- 注释中
- 标签的属性名名
- 标签名
- css标签中

使用 JSON.parse 而不是 eval, request 的content-type要指定是Content-Type: application/json;
如果链接的URL中部分是动态生成的, 一定要做转义.

### 使用浏览器自带的xss-filter

通过http头控制是否打开 xss-filter, 默认为开启. 

X-XSS-Protection:1 (默认)

### Content Security Policy, 内容安全策略

CSP 管理网站允许加载的内容, 并且使用白名单的机制对网站加载或执行的资源起作用. 在网页中, 这样的策略通过 HTTP 头信息或者 meta 元素定义.

CSP 并不是用来防止 xss 攻击的, 而是最小化 xss 发生后所造成的伤害. 

- 通过response头: 只允许脚本从本源加载 Content-Security-Policy: script-src ‘self’
- 通过HTML的META标签: <meta http-equiv=”Content-Security-Policy” content=”script-src ‘self’”>

### X-Frame-Options

X-Frame-Options 响应头是用来给浏览器指示允许一个页面可否在 frame, iframe 或者 object 等标签中展现的标记. 

网站可以使用此功能, 来确保自己网站的内容没有被嵌到别人的网站中去, 也从而避免了点击劫持 (clickjacking) 的攻击. 

但以后可能被CSP的 frame-ancestors取代。目前支持的状态比起 CSP frame-ancestors要好.

### Http-Only

使用 http-only后, 可禁止js读写cookie, 可以保证即使发生了xss, 用户的cookie也是安全的.

### iframe 沙箱环境

HTML5为iframe提供了安全属性 sandbox, 进而限制iframe的能力. 如下:

<iframe src="untrusted.html" sandbox="allow-scripts allow-forms"></iframe>

### 其他安全相关的HTTP头

- X-Content-Type-Options
- HPKP(Public Key Pinning)
- HSTS (HTTP Strict-Transport-Security)


## prototype 与 __proto__

- prototype 是函数(function) 的一个属性, 它指向函数的原型.
- __proto__ 是对象的内部属性, 它指向构造器的原型, 对象依赖它进行原型链查询.

因此, prototype 只有函数才有, 其他(非函数)对象不具有该属性. 而 __proto__ 是对象的内部属性, 任何对象都拥有该属性.

<!-- todo -->
<!--promise-->
<!--canvas-->
<!--http && https-->
<!--安全-->