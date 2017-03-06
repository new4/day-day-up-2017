# 技术学习 一些 JavaScript 常用小代码

- html escape
- isArrayLike
- 生成闭区间`[min, max]`之内的随机数
- shuffle

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
    return typeof value == 'number' && value > -1 && value % 1 == 0 && value <= MAX_SAFE_INTEGER
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
