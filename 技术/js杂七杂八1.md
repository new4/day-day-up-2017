# 技术学习 一些 JavaScript 常用小代码

<!-- toc -->

<!--
  - html escape
  - 判断是否是空对象 isEmptyObject
  - isArrayLike
  - 生成闭区间`[min, max]`之内的随机数
  - shuffle
  - 单词边界匹配/首字母大写
  - 动态脚本
  - 动态样式
  - 编码表单对象用于`HTTP`请求
  - 使用`script`元素发送`JSONP`请求
  - 跨浏览器的事件处理程序
  - 获取两个数组之间差异
  - 清除浮动
  - isObject
  - isPlainObject
  - 深复制
  - div 实现 textbox
-->

## html escape

```javascript
function htmlEscape(text) {
    return text.replace(/[<>"&]/g, function(match, pos, originalText) {
        switch (match) {
            case "<":
                return "&lt;";
            case ">":
                return "&gt;";
            case "&":
                return "&amp;";
            case "\"":
                return "&quot;";
        }
    });
}
```

使用：

```javascript
console.log(htmlEscape("<p class=\"greeting\">Hello world!</p>"));
// &lt;p class=&quot;greeting&quot;&gt;Hello world!&lt;/p&gt;
```

## 判断是否是空对象

```javascript
function isEmptyObject( obj ) {
    for ( var name in obj ) {
        return false;
    }
    return true;
}
```

## isArrayLike

```javascript
var MAX_SAFE_INTEGER = Math.pow(2, 53) - 1;

/** isLength(3) // => true
  * isLength(Number.MIN_VALUE) // => false
  * isLength(Infinity) // => false
  * isLength('3') // => false
  */
function isLength(value) {
    return typeof value == 'number'
            && value > -1
            && value % 1 == 0 // Number.MIN_VALUE
            && value <= MAX_SAFE_INTEGER // Infinity
}

function isArrayLike(value) {
    return value != null && typeof value != 'function' && isLength(value.length)
}
```

## 生成闭区间`[min, max]`之内的随机数

```javascript
function myRandom(min, max) {
    if (max == null) {
        max = min;
        min = 0;
    }
    return min + Math.floor(Math.random() * (max - min + 1));
};
```

## shuffle

```javascript
function shuffle(arr) {
    var length = arr.length;
    var shuffled = Array(length);

    for (var index = 0, rand; index < length; index++) {
        rand = myRandom(0, index);
        if (rand !== index) {
            shuffled[index] = shuffled[rand];
        }
        shuffled[rand] = arr[index];
    }
    return shuffled;
};
```

## 单词边界匹配/首字母大写

```javascript
var str = "  kim a then read bbk   js tian na nenen  a";
str = str.replace(/\b\w+\b/g, function(word) {
    console.log(arguments);
    return word.slice(0, 1).toUpperCase() + word.slice(1);
});
console.log(str);
// Kim A Then Read Bbk   Js Tian Na Nenen  A
```

## 动态脚本

使用`<script>`元素可以向页面中插入`JavaScript`代码，一种方式是通过其`src`特性包含外部文件，另一种方式就是用这个元素本身来包含代码。

插入外部文件：

```javascript
function loadScript(url) {
    var script = document.createElement("script");
    script.type = "text/javascript";
    script.src = url;
    document.body.appendChild(script);
}
```

使用：

```javascript
loadScript("client.js");
```

直接插入JavaScript 代码:

```javascript
function loadScriptString(code) {
    var script = document.createElement("script");
    script.type = "text/javascript";
    try {
        script.appendChild(document.createTextNode(code));
    } catch (ex) {
        // for IE
        script.text = code;
    }
    document.body.appendChild(script);
}
```

使用：

```javascript
loadScriptString("function sayHi(){alert('hi');}");
```

## 动态样式

两种方式：利用`<link>`元素来包含来自外部的文件以及利用`<style>`元素指定嵌入的样式。

`<link>`方式：

```javascript
function loadStyles(url) {
    var link = document.createElement("link");
    link.rel = "stylesheet";
    link.type = "text/css";
    link.href = url;
    var head = document.getElementsByTagName("head")[0];
    head.appendChild(link);

}
```

使用

```javascript
loadStyles("styles.css");
```

`<style>`方式：

```javascript
function loadStyleString(css) {
    var style = document.createElement("style");
    style.type = "text/css";
    try {
        style.appendChild(document.createTextNode(css));
    } catch (ex) {
        style.styleSheet.cssText = css;
    }
    var head = document.getElementsByTagName("head")[0];
    head.appendChild(style);
}
```

使用：

```javascript
loadStyleString("body{background-color:red}");
```

## 编码表单对象用于`HTTP`请求

```javascript
// 若data是来自表单的名值对，使用 application/x-www-form-urlencoded 格式
function encodeFormData(data) {
    if (!data) {
        return "";
    }

    var pairs = [];
    for (var name in data) {
        if (!data.hasOwnProperty(name) || typeof data[name] === "function") {
            continue;
        }

        var value = data[name].toString();
        name = encodeURIComponent(name.replace("20%", "+"));
        value = encodeURIComponent(value.replace("20%", "+"));
        pairs.push(name + "=" + value);
    }
    return pairs.join("&");
}
```

发起一个 HTTP GET 请求

```javascript
function getData(url, data, callback) {
    var request = new XMLHttpRequest();
    request.open("GET", url + "?" + encodeFormData(data));
    request.onreadystatechange = function() {
        if (request.readyState === 4 && typeof callback === "function") {
            callback(request);
        }
    }
    request.send(null);
}
```

发起一个 HTTP POST 请求

```javascript
function postData(url, data, callback) {
    var request = new XMLHttpRequest();
    request.open("POST", url);
    request.onreadystatechange = function() {
        if (request.readyState === 4 && typeof callback === "function") {
            callback(request);
        }
    }
    request.setRequestHeader("Content-Type", "x-www-form-urlencoded");
    request.send(encodeFormData(data));
    // 使用 JSON 编码主体来发送 POST 请求
    // request.setRequestHeader("Content-Type", "application/json");
    // request.send(JSON.stringify(data));
}
```

## 使用`script`元素发送`JSONP`请求

```javascript
function getJSONP(url, callback) {
    // 为本次请求创建一个唯一的回调函数名称
    var cbnum = "cb" + getJSONP.counter++;
    var cbname = "getJSONP." + cbnum;

    // 将回调函数名称以表单编码的形式添加到URL的查询部分中
    // 此处为'jsonp'，也有可能是'callback'
    if (url.indexOf('?') === -1) {
        url += "?jsonp=" + cbname;
    } else {
        url += "&jsonp=" + cbname;
    }

    // 创建 script 元素用于发送请求
    var script = document.createElement("script");

    // 定义将被脚本执行的回调函数
    getJSONP[cbnum] = function(response) {
        try() {
            callback(response);
        } finally {
            delete getJSONP[cbnum];
            script.parentNode.removeChild(script);
        }
    }

    script.src = url;
    document.body.appendChild(script);
}
getJSONP.counter = 0;
```

## 跨浏览器的事件处理程序

```javascript
var EventUtil = {
    addHandler: function(element, type, handler) {
        if (element.addEventListener) {
            element.addEventListener(type, handler, false);
        } else if (element.attachEvent) {
            element.attachEvent("on" + type, handler);
        } else {
            element["on" + type] = handler;
        }
    },
    removeHandler: function(element, type, handler) {
        if (element.removeEventListener) {
            element.removeEventListener(type, handler, false);
        } else if (element.detachEvent) {
            element.detachEvent("on" + type, handler);
        } else {
            element["on" + type] = null;
        }
    }
};
```

## 获取两个数组之间差异

获取新数组 `newArr` 相对于旧数组 `oldArr` 新增和移除的值

```javascript
function compare(value1, value2) {
    return value1 - value2;
}

function getChange(newArr, oldArr) {
    var add = [],
        del = [];

    newArr.sort(compare);
    oldArr.sort(compare);

    while (newArr.length !== 0 && oldArr.length !== 0) {
        if (newArr[0] < oldArr[0]) {
            add.push(newArr.shift());
        } else if (newArr[0] > oldArr[0]) {
            del.push(oldArr.shift());
        } else {
            newArr.shift();
            oldArr.shift();
        }
    }

    if (newArr.length === 0) {
        del = del.concat(oldArr);
    }

    if (oldArr.length === 0) {
        add = add.concat(newArr);
    }

    return {add: add, del: del}
}
```

## 清除浮动

```css
.container::before,
.container::after {
    content:"";
    display:table;
}
.container::after {
    clear:both;
}
.container {
    zoom:1; /* For IE 6/7 (trigger hasLayout) */
}
```

## isObject

```javascript
// 验证对象是否是一个复合数据类型的对象(即非基本数据类型String, Boolean, Number, null, undefined)
// 如果基本数据类型通过new进行创建, 则也属于对象类型
_.isObject = function(obj) {
    return obj === Object(obj);
};
```

## isPlainObject

简单对象：某对象通过 `Object` 构造函数创建的或者某对象的原型 `[[Prototype]]` 值为 `null`

判断 `value` 是简单对象的方法：

1. 排除非类对象的值，排除标签不是 '[object Object]' 的值
2. 调用 'Object.getPrototypeOf' 来获取对象 `Object(value)` 的原型
3. 原型为 `null` 的话，`value` 就是简单对象
4. 若原型上有 `constructor` 属性，取得该属性为 `Ctor`
5. 若该属性 `Ctor` 是函数，且是它自身的实例，且 `Function.prototype.toString.call(Object) == Function.prototype.toString.call(Ctor)`, 那就是简单对象

```js
function isObjectLike(value) {
    return value != null && typeof value == 'object';
}

function isPlainObject(value) {
    if (!isObjectLike(value) || Object.prototype.toString.call(value) != '[object Object]') {
        return false;
    }

    var proto = Object.getPrototypeOf(value);

    if (proto === null) {
        return true;
    }

    var Ctor = Object.prototype.hasOwnProperty.call(proto, 'constructor') && proto.constructor;
    return typeof Ctor == 'function'
          && Ctor instanceof Ctor
          && Function.prototype.toString.call(Ctor) == Function.prototype.toString.call(Object);
}
```

例子：

```js
var obj = Object.create({a: 1});
var Ctor = Object.getPrototypeOf(obj).constructor;
console.log(Function.prototype.toString.call(Object) === Function.prototype.toString.call(Ctor)); // true

console.log(Object instanceof Object); // true
```

## 深复制

```javascript
var isArray = Array.isArray || function(arr) {
    return (arr != null) && (Object.prototype.toString.call(obj) == '[object Array]');
}

var isObject = function(obj) {
    return (obj != null) && (typeof obj == 'object');
}

var getListOp = (function() {
    var list = [];
    return {
        add: function(item) {
            return list.push(item);
        },
        find: function(item) {
            for (var i = 0, len = list.length; i < len; i++) {
                if (item === list[i]) {
                    return true;
                }
            }
            return false;
        }
    }
})();

function deepCopy(obj, listOp) {
    var result = false;
    listOp = listOp || getListOp;
    if (isArray(obj)) {
        result = [];
        for (var i = 0, len = obj.length; i < len; i++) {
            var item = obj[i];
            if (listOp.find(item)) {
                result[i] = item;
            } else {
                listOp.add(item);
                result[i] = deepCopy(item, listOp);
            }
        }
    } else if (isObject(obj)) {
        result = {};
        for (var key in obj) {
            var item = obj[key];
            if (listOp.find(item)) {
                result[key] = item;
            } else {
                listOp.add(item);
                result[key] = deepCopy(item, listOp);
            }
        }
    } else {
        result = obj;
    }
    return result;
}

var a = {
        "a": "a",
        c: ["a", "b"]
    },
    b = {
        "b": "b"
    };

a.b = b;
b.a = a;
a.c.push(b);
c = deepCopy(a);
```

## div 实现 textbox
HTML代码：
```html
<div class="test_box" contenteditable="true"><br /></div>
```

CSS代码：
```css
.test_box {
    width: 400px;
    min-height: 120px;
    max-height: 300px;
    _height: 120px; ` n
    margin-left: auto;
    margin-right: auto;
    padding: 3px;
    outline: 0;
    border: 1px solid #a0b3d6;
    font-size: 12px;
    word-wrap: break-word;
    overflow-x: hidden;
    overflow-y: auto;
    -webkit-user-modify: read-write-plaintext-only;
}
```

JS代码：
```javascript
if (typeof document.webkitHidden == "undefined") {
    // 非chrome浏览器阻止粘贴
    box.onpaste = function() {
        return false;
    }
}
```
