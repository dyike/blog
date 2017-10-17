---
title: Go语言学习笔记【第三目：内建函数】
tags: Golang
date: 2015-06-19 22:37:07
---

## **内建函数**

预定义了少数函数，这意味着无需引用任何包就可以使用它们。下面的表格列出了所有的内建函数。

Table Go中的预定义函数
<table>
<tbody>
<tr>
<td width="138">close</td>
<td width="138">new</td>
<td width="138">panic</td>
<td width="138">complex</td>
</tr>
<tr>
<td width="138">delete</td>
<td width="138">make</td>
<td width="138">recover</td>
<td width="138">real</td>
</tr>
<tr>
<td width="138">len</td>
<td width="138">append</td>
<td width="138">print</td>
<td width="138">imag</td>
</tr>
<tr>
<td width="138">cap copy</td>
<td width="138">println</td>
<td width="138"></td>
<td width="138"></td>
</tr>
</tbody>
</table>

##Close

         用于 channel 通讯。使用它来关闭 channel



##delete

        用于在 map 中删除实例



##len 和 cap

       可用于不同的类型，len 用于返回字符串、slice 和数组的长度



##new

用于各种类型的内存分配



##make

用于内建类型（map、slice 和 channel）的内存分配



##copy

用于复制 slice



##append

用于追加 slice



##panic 和 recover

用于异常处理机制



##print 和 println

是底层打印函数，可以在不引入  fmt 包的情况下使用。它们主要用于调试



##complex、real 和 imag

全部用于处理 复数。有了之前给的简单的例子，不用再进一步讨论复数了



##array、slices 和 map

可以利用 array 在列表中进行多个值的排序，或者使用更加灵活的：slice。字典或哈希 类型同样可以使用，在Go中叫做map



##array

array 由 [n]&lt;type&gt; 定义，n 标示 array 的长度，而 &lt;type&gt; 标示希望存储的内容的类 型。对 array 的元素赋值或索引是由方括号完成的：

```go
var arr [10] int

arr[0] = 42

arr[1] = 13

fmt.Printf("The first element is %d\n", arr[0])
```

像var arr = [10]int 这样的数组类型有固定的大小。大小是类型的一部分。由于不同的大小是不同的类型，因此不能改变大小。数组同样是值类型的：将一个数组赋值给另一个数组，会复制所有的元素。尤其是当向函数内传递一个数组的时候，它会获 得一个数组的副本，而不是数组的指针。

可以像这样声明一个数组：var a [3]int，如果不使用零来初始化它，则用复合声明： a := [3]int{1, 2, 3} 也可以简写为 a := [...]int{1, 2, 3}，Go会自动统计元素的个数。

注意，所有项目必须都指定。因此，如果你使用多维数组，有一些内容你必须录入：

```go
a := [3][2] int {  [2] int { 1,2 }, [2] int { 3,4 }, [2] int { 5,6 }  }
```

类似于：

```go
a := [3][2] int {  [...] int { 1,2 }, [...] int { 3,4 }, [...] int { 5,6 }  }
```

声明一个 array 时，你必须在方括号内输入些内容，数字或者三个点 (...)。

array、slice 和 map 的复合声明变得更加简单。使用复合声明的 array、slice和 map，元素复合声明的类型与外部一致，则可以省略。

这表示上面的例子可以修改为：

```go
a := [3][2] int {  { 1,2 }, { 3,4 }, { 5,6 }  }
```

* * *

##slice

slice 与 array 接近，但是在新的元素加入的时候可以增加长度。slice 总是指向底层的一个array。slice 是一个指向 array 的指针，这是其与array不同的地方；slice 是引用类型，这意味着当赋值某个 slice 到另外一个变量，两个引用会指向同一个 array。例如，如果一个函数需要一个slice 参数，在其内对slice元素的修改也会体现在函数调用者中，这和传递底层的array指针类似。通过：

```go
sl := make([] int , 10)
```

创建了一个保存有10个元素的slice。需要注意的是底层的 array 并无不同。slice 总 是与一个固定长度的 array 成对出现。其影响 slice 的容量和长度。

给定一个 array 或者其他 slice，一个新 slice 通过 a[I:J] 的方式创建。这会创建一个 新的 slice，指向变量a，从序号 I 开始，结束在序号 J 之前。长度为 J - I。

// array[n:m] 从 array 创建了一个  slice，具有元素  n 到 m-1

```go
a := [...] int { 1, 2, 3, 4, 5 }

s1 := a[2:4]

s2 := a[1:5]

s3 := a[:]

s4 := a[:4]

s5 := s2[:]
```

```
2  定义一个 5 个元素的 array，序号从 0 到 4；

3  从序号 2 至 3 创建 slice，它包含元素 3, 4；

4  从序号 1 至 4 创建，它包含元素 2, 3, 4, 5；

5  用 array 中的所有元素创建 slice，这是 a[0:len(a)] 的简化写法；

6  从序号 0 至 3 创建，这是 a[0:4] 的简化写法，得到 1, 2, 3, 4；

7  从 slice s2 创建 slice，注意 s5 仍然指向 array a。
```

函数 append 向 slice s 追加零值或其他 x 值，并且返回追加后的新的、与 s 有相同类型的 slice。如果 s 没有足够的容量存储追加的值，append 分配一 个足够大的、新的 slice 来存放原有 slice 的元素和追加的值。因此，返回 的 slice 可能指向不同的底层 array。

```go
s0 := [] int { 0, 0 }

s1 := append(s0, 2) //0

s2 := append(s1, 3, 5, 7) //1

s3 := append(s2, s0...) //2
```

* * *

##map

许多语言都内建了类似的类型， 例如 Perl 有哈希， Python 有字典， 而 C++ 同样也 有map（作为库）。在Go中有 map 类型。map 可以认为是一个用字符串做索引的数 组（在其最简单的形式下）。下面定义了 map 类型，用于将 string （月的缩写）转换 为 int – 那个月的天数。一般定义 map 的方法是：map[&lt;from type&gt;]&lt;to type&gt;

```go
monthdays := map[ string] int {

"Jan": 31, "Feb":  28, "Mar": 31,

"Apr":30, "May": 31, "Jun":  30,

"Jul":  31, "Aug": 31, "Sep":  30,

"Oct":       31, "Nov": 30, "Dec":  31,     //逗号是必须的

}
```

留意，当只需要声明一个 map 的时候，使用 make 的形式：monthdays := make(map[string]int)

当在map中索引（搜索）时，使用方括号。例如打印出 12 月的天数：fmt.Printf("%d\n", monthdays["Dec"])

当对 array、slice、string 或者 map 循环遍历的时候，range 会帮助你，每次调用，它都会返回一个键和对应的值。

```go
year := 0

for _ , days := range  monthdays {    //键没有使用，因此用 _, days

year += days

}

fmt.Printf("Numbers of days in a year: %d\n", year)
```

向 map 增加元素，可以这样做：

```go
monthdays["Undecim"] = 30    //添加一个月

monthdays["Feb"]       = 29            // ‰年时重写这个元素
```

检查元素是否存在，可以使用下面的方式 ：

```go
var  value int

var  present bool

value, present = monthdays["Jan"]    //如果存在，present 则有值 true

//或者更接近 Go 的方式

v, ok := monthdays["Jan"]           //“逗号 ok”形式  
```

也可以从 map 中移除元素：

```go
delete(monthdays, "Mar")      //删除”Mar”  吧，总是下雨的月份
```

通常来说语句 delete(m, x) 会删除 map 中由 m[x] 建立的实例。


----