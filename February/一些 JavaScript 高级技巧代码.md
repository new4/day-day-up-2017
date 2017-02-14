# 技术学习 一些 JavaScript 高级技巧代码

## 函数节流

```javascript
var throttle = function(fn, interval) {
    var timer;
    var firstTime = true;
    return function() {
        var me = this;
        var args = arguments;

        // 若第一次调用，无需延迟
        if (firstTime) {
            fn.apply(me, args);
            return firstTime = false;
        }

        // 定时器仍在，说明上一次执行未完成，则等待
        if (timer) {
            return false;
        }

        // 延迟一段时间执行
        timer = setTimeout(function() {
            clearTimeout(timer);
            timer = null;
            fn.apply(me, args);
        }, interval || 500);
    }
}
```

使用方式

```javascript
window.onresize = throttle(function(){
    console.log("onresize");
}, 1000);
```
