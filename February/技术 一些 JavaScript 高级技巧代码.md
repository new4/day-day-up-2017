# 一些 JavaScript 高级技巧代码

列表

- 函数节流
- 柯里化 currying
- 反柯里化 uncurrying
- 分时函数
- AOP 面向切面编程
- 单例模式
- 虚拟代理实现图片预加载
- 发布-订阅模式
- 组合模式

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
window.onresize = throttle(function() {
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
var uncurrying = function(fn) {
    return function() {
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

```javascript
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
```

## AOP 面向切面编程

可以使用 AOP 装饰函数以实现装饰模式

实现：

```javascript
Function.prototype.before = function(beforefn) {
    var me = this;
    return function() {
        beforefn.apply(this, arguments);  // (1)
        return me.apply(this, arguments); // (2)
    }
}

Function.prototype.after = function(afterfn) {
    var me = this;
    return function() {
        var result = me.apply(this, arguments);
        afterfn.apply(this, arguments);
        return result;
    }
}
```

使用：

```javascript
function g(type) {
    type = type || "";
    return function() {
        console.log(type + "invoke func");
    }
}

var func = g();
var funcAop = func.before(g("before")).after(g("after"))

funcAop();
// "before invoke func"
// "invoke func"
// "after invoke func"
```

应用：

1. 数据统计

  ```javascript
  func.after(function(){
  console.log("统计数据")；
  })
  ```

2. 动态改变函数的参数

  上面的`before`代码的实现中的`(1)`和`(2)`部分的参数是一致的，因此就可以在`before`中对函数的参数进行修正。

3. 表单提交前的校验

注意点：

某函数通过`before`和`after`装饰之后返回的是一个新的函数，因此原函数上绑定的属性就不复存在了

## 单例模式

实现：

```javascript
var getSingle = function(fn) {
    var instance;
    return function() {
        return instance || (instance = fn.apply(this, arguments));
    }
}
```

使用：

```javascript
// 创建一个唯一的 iframe
var createIframe = function() {
    var iframe = document.createElement('iframe');
    document.body.appendChild(iframe);
    // 返回实例以供缓存
    return iframe;
}

var createSingleIframe = getSingle(createIframe);

var iframe1 = createSingleIframe()
var iframe2 = createSingleIframe()

console.log(iframe1 === iframe2); // true
```

## 虚拟代理实现图片预加载

```javascript
var myImage = (function() {
    var imgNode = document.createElement('img');
    document.body.appendChild(imgNode);
    return {
        'setSrc': function(src) {
            imgNode.src = src;
        }
    }
})();

var proxyImage = (function() {
    var img = new Image();
    img.onload = function() {
        myImage.setSrc(this.src)
    }
    return {
        'setSrc': function(src) {
            myImage.setSrc("c:/loading.gif");
            img.src = src;
        }
    }
})();

proxyImage.setSrc("http://blabla/foo.jpg");
```

## 发布-订阅模式

使用一个全局的 `Event` 对象来实现，它作为一个类似**中介者**的角色将发布者和订阅者联系起来。

实现：

```javascript
var Event = (function() {
    var clientList = {};
    return {
        // 接受订阅
        listen: function(event, fn) {
            if (!clientList[event]) {
                clientList[event] = [];
            }
            clientList[event].push(fn);
        },
        // 发布事件
        trigger: function() {
            var event = Array.prototype.shift.call(arguments),
                fns = clientList[event];

            if (!fns || fns.length === 0) {
                return false;
            }

            for (var i = 0, fn; fn = fns[i++];) {
                fn.apply(this, arguments);
            }
        },
        // 取消订阅的事件
        remove: function(event, fn) {
            var fns = clientList[event];

            if (!fns) {
                return false;
            }

            if (!fn) {
                // 没有传入 fn, 则取消所有订阅
                delete clientList[event];
            } else {
                for (var len = fns.length - 1; len >= 0; len--) {
                    var _fn = fns[len];
                    if (_fn === fn) {
                        fns.splice(len, 1);
                    }
                }
            }
        }
    }
})();
```

使用：

```javascript
Event.listen("ev", function(data) {
    console.log("ev " + data);
})
Event.trigger("ev", 1); // ev 1
Event.trigger("ev", 2); // ev 2
Event.remove("ev");
Event.trigger("ev", 3); // false
```

稍高级一点，让 `Event` 对象拥有先发布后订阅的能力。

实现方式是建立一个存放离线事件的堆栈，当事件发布的时候如果此时还没有订阅者订阅，就暂时将发布事件的动作包裹在一个函数里，当有对象来订阅这个事件之后，再遍历堆栈并依次执行这些函数（重新发布事件）。

## 组合模式

适用范围：

1. 表示对象的部分-整体层次结构
2. 可贺希望统一对待树中的所有对象

一种简单的宏命令：

```javascript
var MacroCommand = function() {
    return {
        commandList: [],
        add: function(command) {
            this.commandList.push(command);
        },
        execute: function() {
            for (var i = 0, len = this.commandList.length; i < len; i++) {
                this.commandList[i].execute();
            }
        }
    }
}
```

一般使用方式，只有一个层级：

```javascript
function gCommand(id) {
    return {
        execute: function() {
            console.log("execute command" + id);
        }
    }
}

var macroCommand = MacroCommand();
var command1 = gCommand(1);
var command2 = gCommand(2);
var command3 = gCommand(3);
macroCommand.add(command1);
macroCommand.add(command2);
macroCommand.add(command3);
macroCommand.execute();
// execute command1
// execute command2
// execute command3
```

组合使用方式，有多个层级的树形结构时：

```javascript
var macroCommand1 = MacroCommand();
macroCommand1.add(gCommand("1-1"));

var macroCommand2 = MacroCommand();
macroCommand2.add(gCommand("2-1"));
macroCommand2.add(gCommand("2-2"));

var macroCommand3 = MacroCommand();
macroCommand3.add(gCommand("3-1"));
macroCommand3.add(gCommand("3-2"));
macroCommand3.add(gCommand("3-3"));

var macroCommand = MacroCommand();
macroCommand.add(macroCommand1);
macroCommand.add(macroCommand2);
macroCommand.add(macroCommand3);

macroCommand.execute();
// execute command1-1
// execute command2-1
// execute command2-2
// execute command3-1
// execute command3-2
// execute command3-3
```

应用：扫描文件夹
