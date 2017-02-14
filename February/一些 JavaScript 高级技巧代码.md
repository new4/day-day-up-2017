# 技术学习 一些 JavaScript 高级技巧代码

## 函数节流

实现：

```javascript
var throttle = function(fn, interval) {
  var timer,
      firstTime = true;
  return function() {
    var me = this,
        args = arguments;

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

使用：

```javascript
window.onresize = throttle(function(){
    console.log("onresize");
}, 1000);
```

## 柯里化 currying

实现：

```javascript
var currying = function(fn) {
  var args = Array.prototype.slice.call(arguments, 1);
  return function() {
    var innerArgs = Array.prototype.slice.call(arguments);
    var finalArgs = args.concat(innerArgs);
    return fn.apply(null, finalArgs);
  };
}
```

使用：
```javascript
function add(num1, num2) {
  return num1 + num2;
}

var curriedAdd1 = currying(add, 5);
console.log(curriedAdd1(3)); // 8

var curriedAdd2 = currying(add, 5, 12);
console.log(curriedAdd2()); // 17
```

## 反柯里化 uncurrying

将`obj.func(arg1, arg2, ...)`转化成`func(obj, arg1, arg2, ...)`

实现：

```javascript
var uncurrying = function(fn){
  return function(){
    var args = Array.prototype.slice.call(arguments, 1);
    return fn.apply(arguments[0], args);
  }
}
```

使用：

```javascript
var push = uncurrying(Array.prototype.push);
var arr = [];
push(arr, 1, 2, 3, 4);
console.log(arr); // [1, 2, 3, 4]
```

## 分时函数

作用：避免短时大量操作

当发现某个循环占用了大量时间，同时满足如下两个条件：

- 处理无需同步完成
- 数据无需按顺序完成

就可以使用定时器分割这个循环，小块小块地处理数组，通常每次一小块。

实现：

````javascript
var chunk = function(array, fn, count) {
  var obj,
      timer;

  var handler = function() {
    for (var i = 0; i < Math.min(count || 1, array.length); i++) {
      var obj = array.shift();
      fn(obj);
    }
  }

  return function() {
    timer = setInterval(function() {
      if (array.length === 0) {
        return clearInterval(timer);
      }
      handler();
    }, 200);
  }
}
```

使用：

```javascript
var array = [];
for (var i = 1; i < 1000; i++) {
  array.push(i);
}

var renderDom = chunk(array, function(n) {
  var p = document.createElement('p');
  p.innerHTML = n;
  document.body.appendChild(p);
}, 8);

renderDom();
````
