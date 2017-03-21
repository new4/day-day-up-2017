# 技术 web安全相关知识

- 浏览器的同源策略

## 浏览器的同源策略

源自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

> 同源策略限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。

### 定义

如果**协议**，**端口**（如果指定了一个）和**主机**对于两个页面是相同的，则两个页面具有相同的源。

### 源的更改：

脚本可以将 `document.domain` 的值设置为其当前域或其当前域的超级域。

如果将其设置为其当前域的超级域，则较短的域将用于后续原始检查。

例如，假设文档中的一个脚本在`http://store.company.com/dir/other.html`执行以下语句：

```javascript
document.domain = "company.com";
```

这条语句执行之后，页面将会成功地通过对`http://company.com/dir/page.html`的同源检测。

而同理，`company.com` 不能设置 `document.domain` 为 `othercompany.com`.

浏览器单独保存端口号。

任何的赋值操作，包括 `document.domain = document.domain` 都会以 `null` 值覆盖掉原来的端口号。

因此 `company.com:8080` 页面的脚本不能仅通过设置 `document.domain = "company.com"` 就能与 `company.com` 通信。赋值时必须带上端口号，以确保端口号不会为 `null`。

### 跨源网络访问

同源策略控制了不同源之间的交互，交互分为三类：

- 通常允许进行跨域写操作（Cross-origin writes）。例如链接（links），重定向以及表单提交。
- 通常允许跨域资源嵌入（Cross-origin embedding）。
- 通常不允许跨域读操作（Cross-origin reads）。但常可以通过内嵌资源来巧妙的进行读取访问。例如可以读取嵌入图片的高度和宽度，调用内嵌脚本的方法，或availability of an embedded resource.

以下是可能嵌入跨源的资源的一些示例：

- `<script src="..."></script>` 标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
- `<link rel="stylesheet" href="...">` 标签嵌入CSS。由于CSS的松散的语法规则，CSS的跨域需要一个设置正确的Content-Type消息头。不同浏览器有不同的限制。
- `<img>`嵌入图片。
- `<video>` 和 `<audio>`嵌入多媒体资源。
- `<object>`, `<embed>` 和 `<applet>`的插件。
- `@font-face`引入的字体。一些浏览器允许跨域字体，一些需要同源字体。
- `<frame>` 和 `<iframe>`载入的任何资源。站点可以使用X-Frame-Options消息头来阻止这种形式的跨域交互。

使用 `CORS` 允许跨源访问。

### 跨源脚本API访问

Javascript 的 APIs 中，如 `iframe.contentWindow`, `window.parent`, `window.open` 和 `window.opener` 允许文档间直接相互引用。当两个文档的源不同时，这些引用方式将对 `Window` 和 `Location` 对象的访问添加限制。可以使用 `window.postMessage` 作为替代方案，提供跨域文档间的通讯。

### 跨源数据存储访问

存储在浏览器中的数据，如 `localStorage` 和 `IndexedDB`，以源进行分割。每个源都拥有自己单独的存储空间，一个源中的Javascript脚本不能对属于其它源的数据进行读写操作。

`window.name` 属性可以用来临时存储数据，可以跨域访问。

`Cookies` 使用不同的源定义方式。一个页面可以为本域和任何父域设置 `cookie`，只要是父域不是公共后缀即可。
