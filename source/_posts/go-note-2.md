---
title: Go语言学习笔记【第二目：运算符、控制结构】
tags:
  - Golang
date: 2015-06-19 22:06:46
---

## **运算符**

Go支持普通的数字运算符，下面的表格列出了当前支持的运算符，以及其优先级。它们全部是从左到右结合的。

Table运算优先级
<table>
<tbody>
<tr>
<td width="113">Precedence</td>
<td width="236">Operator(s)</td>
</tr>
<tr>
<td rowspan="6" width="113">Highest&nbsp;

&nbsp;

&nbsp;

&nbsp;

Lowest</td>
<td width="236">*     /     %    &lt;&lt;  &gt;&gt;  &amp;    &amp;^</td>
</tr>
<tr>
<td width="236">+     -      | ^</td>
</tr>
<tr>
<td width="236">==  !=   &lt;     &lt;=  &gt;     &gt;=</td>
</tr>
<tr>
<td width="236">&lt;-</td>
</tr>
<tr>
<td width="236">&amp;&amp;</td>
</tr>
<tr>
<td width="236">||</td>
</tr>
</tbody>
</table>
+ - * / 和 % 会像你期望的那样工作，&amp; | ^ 和 &amp;^ 分别表示位运算符按位与，按位或，按位异或和位清除。&amp;&amp;和||运算符是逻辑与和逻辑或。表格中没有列出的是逻辑非：!。

虽然Go不支持运算符重载（或者方法重载），而一些内建运算符却支持重载。例如 + 可以用于整数、浮点数、复数和字符串（字符串相加表示串联它们）。

&nbsp;

* * *

&nbsp;

## **Go 关键字**

Table  Go中的关键字
<table>
<tbody>
<tr>
<td width="111">break</td>
<td width="111">default</td>
<td width="111">func</td>
<td width="111">interface</td>
<td width="111">select</td>
</tr>
<tr>
<td width="111">case</td>
<td width="111">defer</td>
<td width="111">go</td>
<td width="111">map</td>
<td width="111">struct</td>
</tr>
<tr>
<td width="111">chan</td>
<td width="111">else</td>
<td width="111">goto</td>
<td width="111">package</td>
<td width="111">switch</td>
</tr>
<tr>
<td width="111">const</td>
<td width="111">fallthrough</td>
<td width="111">if</td>
<td width="111">range</td>
<td width="111">type</td>
</tr>
<tr>
<td width="111">continue</td>
<td width="111">for</td>
<td width="111">import</td>
<td width="111">return</td>
<td width="111">var</td>
</tr>
</tbody>
</table>

*   func 用于定义函数和方法；
*   return 用于从函数返回；
*   go用于并行；
*   select用于选择不同类型的通讯；
*   interface；
*   struct 用于抽象数据类型；
&nbsp;

* * *

&nbsp;

## **控制结构**

在Go中只有很少的几个控制结构。例如这里没有do或者while循环，只有for。有灵活的switch语句和if，而switch接受像for那样可选的初始化语句。还有叫做**类型选择和多路通讯转接器的**** select**。语法有所不同（同 C 相比）： **无需圆括号，而语句体必须总是包含在大括号内。**

if-eles

在 Go 中 if 看起来是这样的：

```go
i f   x &gt; 0 {                    //{ 是强制的

return  y

}   else  {

return  x

}
```

强制大括号鼓励将简单的 if 语句写在多行上。无论如何，这都是一个很好的形式，尤其是语句体中含有控制语句，例return或者break。

if 和 switch 接受初始化语句，通常用于设置一个（局部）变量。

```go
i f err := Chmod(0664) ; err != nil {        ← nil与C的NULL类似

fmt.Printf(err)        ← err 的作用域被限定在if内

return  err

}
```

可以像通常那样使用逻辑运算符：

```go
i f  true &amp;&amp; true   {

fmt.Println("true")

}

i f  ! false {

fmt.Println("true")

}
```

在Go库中，你会发现当一个if语句不会进入下一个语句流程–也就是说，语句体结束于 break，continue，goto或者return–不必要的 else会被省略。

```
f, err := os.Open(name, os.O_RDONLY, 0)

i f err != nil {

return  err

}

doSomething(f)
```

这个例子通常用于检测可能的错误序列。成功的流程一直执行到底部使代码很好读，当遇到错误的时候就排除它。这样错误的情况结束于 return 语句，这样就无须else语句。

```
f, err := os.Open(name, os.O_RDONLY, 0)

i f err != nil { return  err

}

d, err := f.Stat()

i f err != nil { return  err

}
```

doSomething(f, d)

下面的语法在 Go 中是非法的：

```go
i f   err != nil

{                          //**  ****←****必须同**** if ****在同一行**

return  err

}
```

* * *

* * *

### **goto**

> 用 goto 跳转到一定是当前函数内定义的标签。例如假设这样一个循环：

```go
func  myfunc() {

i := 0

Here:         ← 这行的第一个词，以分号结束作为标签

println(i) i++

goto  Here      ← 跳转

}
```

标签名是大小写敏感的。

### **for**

Go 的 for 循环有三种形式，只有其中的一种使用分号。

> <del>_for  init ;   condition ;   post {   }      ← 和 C 的 for 一样_</del>
>
> <del>_for  condition {    }              ← 和 while 一样_</del>
>
> <del>_for   {   }                 ← 死循环_</del>

短声明使得在循环中声明一个序号变量更加容易。

```go
sum := 0

for  i := 0 ;   i &lt; 10 ;   i++ {

sum += i              ← sum = sum + i 的简化写法

}             ← i 实例在循环结束会消失
```

最后，由于Go没有逗号表达式，而 ++ 和--是语句而不是表达式，如果你想在 for 中 执行多个变量，应当使用平行赋值。

```go
// Reverse a

for i, j := 0, len(a)-1 ; i &lt; j ; i, j = i+1, j-1 {

a[i], a[j] = a[j], a[i]      ← 平行赋值

}

### **break 和 continue**

利用 break 可以提前退出循环，break 终止当前的循环。

for  i := 0 ;   i &lt; 10 ;   i++ { i f   i &gt; 5 {

break   ← 终止这个循环，只打印 0 到 5

}

println(i)

}
```

循环嵌套循环时，可以在break后指定标签。用标签决定哪个循环被终止：

```go
J:    for j := 0 ;   j < 5 ;   j++ {

for i := 0 ;   i < 10 ;   i++ {

i f  i > 5 {

break  J    ← 现在终止的是 j 循环，而不是 i 的那个

}

println(i)

}

}
```

利用continue 让循环进入下一个迭代，而略过剩下的所有代码。下面循环打印了0到 5。

```go
for i := 0 ;  i < 10 ;   i++ {

i f  i > 5 {

continue      ← 跳过循环中所有的代码println(i)
```

### **range**

**关键字**** range ****可用于循环**。它可以在 slice、array、string、map 和 channel。range是个迭代器，当被调用的时候，从它循环的内容中返回一个键值对。基于不同的内容，range返回不同的东西。

当对slice 或者array做循环时，range 返回序号作为键，这个序号对应的内容作为值。考虑这个代码：

```go
list := [] string {"a", "b", "c", "d", "e", "f"}

for  k, v := range  list {

// 对  k 和  v 做想做的事情

}
```

1  创建一个字符串的 slice。

2 用range对其进行循环。每一个迭代，range 将返回 int 类型的序号，string类型的值，以0和“a”开始。

3  k 的值为 0…5，而v在循环从“a”…“f”。

也可以在字符串上直接使用range。这样字符串被打散成独立的 Unicode 字符并且 起始位按照 UTF-8 解析。

【在 UTF-8 世界的字符有时被称作 runes。通常，当人们讨论字符时，多数是指 8 位字符。UTF-8 字符可 能会有 32 位，称作 rune。在这个例子里，char 的类型是 rune。】

循环：

```go
for pos, char := range  "aΦx" {

fmt.Printf("character '%c' starts at byte position %d\n", char, pos)

}
```

打印

```
character 'a' starts at byte position

character 'Φ' starts at byte position

character 'x' starts at byte position      ← Φ took 2 bytes
```

### **switch**

Go的switch非常灵活。表达式不必是常量或整数，执行的过程从上至下，直到找到匹 配项，而如果switch没有表达式，它会匹配true 。这产生一种可能——使用switch编写if-else-if-else判断序列。

```go
func unhex(c byte) byte {

switch  {

case  '0' &lt;= c &amp;&amp; c &lt;= '9':

return  c - '0'

case  'a' &lt;= c &amp;&amp; c &lt;= 'f':

return  c - 'a' + 10

case  'A' &lt;= c &amp;&amp; c &lt;= 'F':

return  c - 'A' + 10

}

return  0

}
```

它不会匹配失败后自动向下尝试，但是可以使用fallthrough 使其这样做。 没有fallthrough：

```go
switch  i {

case  0:         // 空的  case 体

case  1:

f()       // 当  i == 0 时，f 不会被调用！

}

而这样：

switch  i {

case  0:     fallthrough

case  1:

f()     // 当  i == 0 时，f 会被调用！

}
```

&nbsp;

用default 可以指定当其他所有分支都不匹配的时候的行为。

```go
switch  i {

case  0:

case  1:

f()

default:

g()         // 当  i 不等于  0 或  1 时调用

}
```

分支可以使用逗号分隔的列表。

```
func shouldEscape(c byte) bool {

switch  c {

case  ' ', '?', '&amp;', '=', '#', '+':   ← , as ”or”

return  true

}

return  false

}
```

这里有一个使用两个 switch 对字节数组进行比较的例子：

```go
//0

func  Compare(a, b []byte) int   {

for i := 0 ; i &lt; len(a) &amp;&amp; i &lt; len(b) ; i++ {

switch  {

case  a[i] &gt; b[i]:

return  1

case  a[i] &lt; b[i]:

return  -1

}

}

switch   {           //1

case len(a) &lt; len(b):

return  -1

case len(a) &gt; len(b):

return  1

}

return  0       //2

}
```
---
0  比较返回两个字节数组字典数序先后的整数。

如果 a == b 返回 0，如果 a &lt; b 返回 -1，而如果 a &gt; b 返回+1；

1  长度不同，则不相等；

2  字符串相等。