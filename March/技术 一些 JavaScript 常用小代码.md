# 技术学习 一些 JavaScript 常用小代码

- html escape
- isArrayLike
- 生成闭区间`[min, max]`之内的随机数
- shuffle
- 单词边界匹配，单词首字母大写
- 动态脚本
- 动态样式
- 编码表单对象用于`HTTP`请求

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

## 单词边界匹配，单词首字母大写

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
    request.setReauestHeader("Content-Type", "x-www-form-urlencoded");
    // reuqest.send(serialize(form))
    request.send(encodeFormData(data));
}
```
