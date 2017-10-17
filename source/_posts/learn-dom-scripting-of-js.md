---
title: 高性能JavaScript学习笔记——DOM编程
date: 2016-06-30 14:07:19
tags: JavaScript
---

DOM是文档对象模型，用于操作XML和HTML文档的程序接口。DOM是JavaScript编码中一个重要的部分。

### DOM的修改与访问 ###
访问DOM元素是具有代价的，修改DOM元素的代价更大，因为它会导致浏览器重新计算页面的几何变化。

最快的情况就是在循环中修改或者访问元素，尤其对HTML元素结合循环操作。先看一个简单的例子：

```javascript
function innerHTMLLoop() {
    for (var count = 0; count < 100 ; count++) {
        document.getElementById('here').innerHTML += 'a';
    }
}
```
这个函数就是修改页面元素的内容，那性能问题出现在哪儿？每次循环迭代，该元素都会被访问两次：第一次是读取innerHTML的属性值，第二次就是重写它。

那换一种局部变量的存储方式修改其中的内容，在循环体结束之后一次性写入，是不是效率要更高效呢？

```javascript
function innerHTMLLoop2() {
    var content = '';
    for (var count = 0; count < 100 ; count++) {
        content += 'a';
    }
    document.getElementById('here').innerHTML += content;
}
```
很明显修改后的代码运行速度更快了。访问DOM的次数越多，代码的运行速度越慢。因此，常用的经验法则：减少访问DOM的次数，把运行尽量留在ECMAScript处理。

### innerHTML 对比 DOM方法 ###
首先带着一个问题：修改页面区域的最佳方案是用非标准但支持良好的innerHTML属性？还是只用类似document.createElement()的原声DOM方法？如果不考虑web标准，他们的性能都差不多，但是在出开最新版的webkit内核之外的浏览器中，innerHTML会更快一点。
这儿省略一个例子：两种方式创建一个1000行的表格。
* 合并HTML字符，然后更新DOM的innerHTML属性。
* 只用标准的DOM方法，比如document.createElement()和document.createTextNode().

最终的结果怎么样？最终选择哪种方法取决你的用户使用的浏览器以及你的编码习惯。如果你的需求是在一个对性能有着苛刻的要求操作中更新一大段HTML，推荐使用innerHTML，在大部分的浏览器中，这个运行速度都要快！

### 节点克隆 ###
使用DOM方法是更新页面的内容的另外一个途径就是克隆已有元素。就是使用element.cloneNode()(element表示已有节点)替代document.createElement().
看个例子吧：用element.cloneNode()生成表格。

```javascript
function tableClonedDOM(){
    var i,table,thead,tbody,tr,th,td,a,ul,li,
        oth = document.createElement('th'),
        otd = document.createElement('td'),
        otr = document.createElement('tr'),
        oa = document.createElement('a'),
        oli = document.createElement('li'),
        oul = document.createElement('ul');
    tbody = document.createElement('tbody');

    for(i = 1; i <= 100; i++){
        tr = otr.cloneNode(false);
        td = otd.cloneNode(false);
        td.appendChild(document.createTextNode((i % 2) ? 'yes' : 'no'));
        tr.appendChild(td);
        td = otd.cloneNode(false);
        td.appendChild(document.createTextNode(i));
        tr.appendChile(td);
        td = otd.cloneNode(false);
        td.appendChild(document.createTextNode('my name is #' + i));
        tr.appendChild(td);
        // ....
    }
    // ....
}
```

### HTML集合 ###

html集合是包含了DOM节点引用的类数组对象。以下方法就是返回值就是一个集合：
* document.getElementsByName()
* document.getElementsByClassName()
* document.getElementsByTagName()
下面的属性同样是返回HTML集合：
* document.images 页面所有img元素
* document.links 页面所有a元素
* document.forms 页面所有表单元素
* document.forms[0].elements 页面中第一个表单所有的字段

#### 访问集合元素使用局部变量 ####
对于任何类型的DOM访问，需要多次访问同一个DOM属性或者方法需要多次访问，最好使用一个局部变量缓存此成员，前面文章开头也谈到了原因了。遍历一个集合是，第一个优化原则是把集合存储到局部变量中，并把length存储在循环外部，然后使用局部变量替代这些需要多次读取的元素。
看例子是最好的学习方法：

```javascript
//较慢
function collectionGlobal(){
    var coll = document.getElementsByTagName('div'),
        len = coll.length,
        name = '';
    for(var count = 0; count < len; count++){
        name = document.getElementsByTagName('div')[count].nodeName;
        name = document.getElementsByTagName('div')[count].nodeType;
        name = document.getElementsByTagName('div')[count].tagName;
    }
    return name;
}

//较快
function collectionLocal() {
    var coll = document.getElementsByTagName('div'),
        len = coll.length,
        name = '';
    for(var count = 0; count<len; count++){
        name = coll[count].nodeName;
        name = coll[count].nodeType;
        name = coll[count].tagName;
    }
    return name;
}

//最快
function collectionNodeLocal(){
    var coll = document.getElementsByTagName('div'),
        len = coll.length,
        name = '',
        el = null;
    for(var count = 0; count<len; count++){
        el = coll[count];
        name = el.nodeName;
        name = el.nodeType;
        name = el.tagName;
    }
    return name;
}
```

### 遍历DOM ###
#### 获取DOM元素 ####
有这样一个场景就是你需要从某一个DOM元素开始，操作周围的元素，或者递归查找所有子节点。可以使用childNodes得到元素集合，或者用nextSibling来获取每个相邻的元素。

两个等价的例子：

```javascript
function testNextSibling(){
    var el = document.getElementById('mydiv'),
        ch = el.firstChild,
        name = '';
    do {
        name = ch.nodeName;
    }while(ch = ch.nextSibling);
    return name;
};

function testChildNodes(){
    var el = document.getElementById('mydiv'),
        ch = el.childNodes,
        len = el.length,
        name = '';
    for(var count = 0; count<len; count++){
        name = ch[count].nodeName;
    }
    return name;
};
//需要注意的是childNodes是一个元素集合，在循环体中，缓存length属性以避免在每次迭代中更新。

```

#### 节点元素 ####
DOM元素属性诸如，childNodes,firstChild和nextSibling并不区分元素节点和其他类型节点，比如注释和文本节点。在某些情况下只需要访问元素节点，因此在循环中很可能需要检查返回节点类型并过滤掉非元素节点。






