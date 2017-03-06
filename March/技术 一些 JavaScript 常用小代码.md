# 技术学习 一些 JavaScript 常用小代码

- html escape

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
