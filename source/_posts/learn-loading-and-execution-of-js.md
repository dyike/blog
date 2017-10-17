---
title: 高性能JavaScript的学习笔记——加载和执行
date: 2016-06-29 12:00:55
tags: JavaScript
---

作为一个web开发者来说，在开发的过程需要考虑性能的问题。JavaScript在浏览器中的性能，可能就是可用性的问题，这个问题因JavaScript的阻塞特性变得复杂，说白了就是当浏览器在执行JavaScript的时候，浏览器不能同时做其他的事情。js的执行时间越长，浏览器的响应时间就会越长。&lt;script&gt;标签出现的时候，就会让页面等待脚本的解析和执行，不管JavaScript的代码是内嵌还是包含的在外链的文件中，页面的下载和渲染都必须要停下来等待脚本的执行完毕。

### 脚本的位置 ###
在HTML4规范中指出&lt;script&gt;标签可以放在html文档中&lt;head&gt;或&lt;body&gt;的标签中。通常情况是这样的：&lt;script&gt;标签用来加载出现在&lt;head&gt;中外链的js文件，挨着&lt;link&gt;标签用来加载外部的css文件或其他页面元信息。

先看下面的一段代码：

```html
<html>
<head>
    <title>js学习例子</title>
    <script type="text/javascript" src="file1.js"></script>
    <script type="text/javascript" src="file2.js"></script>
    <script type="text/javascript" src="file3.js"></script>
    <link rel="stylesheet" type="text/css" href="styles.css">
</head>
<body>
    <p>你好，世界！</p>
</body>
</html>
```
上面的代码需要注意的是：在浏览器解析到&lt;body&gt;标签之前，不会渲染页面的其他的内容。所以说把脚本放在页面的顶部会导致明显的延迟，表现出页面空白，没有内容信息。执行过程差不多就是这样的：
![js执行过程](https://raw.githubusercontent.com/yf92/yf92.github.io/master/images/jslearning/jslearning1.png)

从上图可以清晰的看到，第一个js文件下载的同时阻塞了页面其他文件的下载，以此类推！

面对这样的原因，可以适当的调整js代码的位置是不是更合理呢？

```html
<html>
<head>
    <title>js学习例子</title>
    <link rel="stylesheet" type="text/css" href="styles.css">
</head>
<body>
    <p>你好，世界！</p>
    <script type="text/javascript" src="file1.js"></script>
    <script type="text/javascript" src="file2.js"></script>
    <script type="text/javascript" src="file3.js"></script>
</body>
</html>
```

推荐是将所有的&lt;script&gt;标签尽可能的放在&lt;body&gt;标签的底部，尽量减少对整个页面的影响。记住：优化JavaScript代码的首要规范：将代码放在底部！

上面谈到的原因也差不多明白，在组织脚本的时候，我们尽量减少&lt;script&gt;标签的个数，这不仅仅是针对外链，内嵌也是一样的。考虑HTTP请求会带来额外的性能开销，下载一个100kb的文件要比下载4个25kb的文件更快！这样我们就应该考虑文件的合并，可以通过离线打包工具等等，后面继续讲！

下面说明两种脚本：
* 无阻塞的脚本就是，在页面加载完成后才加载JavaScript代码，专业的就是说法就是在window对象的load事件触发后再下载脚本。
* 延迟的脚本就是，在&lt;script&gt;标签中定义了一个扩展属性：defer。致命本元素所含的脚本不会修改DOM，这样代码就会安全的延迟执行。

需要说明的是：目前主流的浏览器已经实现对defer属性的支持，W3C的html5规范，当且仅当src属性声明时生效。

还有一种无阻塞加载脚本的方法是使用XMLHttpRequest(XHR)对象获取脚本并注入页面中。先创建一个XHR对象，然后用它去下载JavaScript文件，最后通过创建动态的&lt;script&gt;元素将代码注入到页面中。代码示例：

```javascript
var xhr = new XMLHttpRequest();
xhr.open("get","file1.js",true);
xhr.onreadystatechange = function(){
    if(xhr.readyState == 1){
        if(xhr.status >=200 && xhr.status <300 || xhr.status == 304){
            var script = document.createElement("script");
            script.type = "text/javascript";
            script.text = xhr.responseText;
            document.body.appendChild(script);
        }
    }
};
xhr.send(null);
```
发送一个GET请求获取file1.js文件。事件处理函数onreadystatechange检查readyState的状态是否为1，同时还要校验http的状态码，如果满足条件，创建一个&lt;script&gt;元素。这样做的好处就是，可以先去下载一个JavaScript代码，但可以先不去执行。但也有局限性：JavaScript文件必须与请求的页面处与相同的域，也就是说不能从CDN上下载JavaScript文件，这种方法不用于大型web应用。










