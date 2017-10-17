---
title: 由于自己的疏忽，一个低级的错误
tags: 
    -PHP
date: 2015-09-19 19:20:17
---

PHP中continue后面的分号，疏忽掉就会出现不一样的情况。
看一个简单的例子：
```php
<?php
for ($i = 0; $i &lt; 5; ++$i) {
if ($i == 2)
continue;
print "$i\n";
}
?>
```

这种情况输出的结果：
```
0
1
3
4
```

去掉分号后：
```php
<?php
for ($i = 0; $i &lt; 5; ++$i) {
if ($i == 2)
continue
print "$i\n";
}
?>
```
在php5.4之前的版本输出的结果：
```2```
因为整个 continue print "$i\n"; 被当做单一的表达式而求值，所以 print 函数只有在 $i == 2 为真时才被调用（print 的值被当成了上述的可选数字参数而传递给了 continue）。

还有重要的，在php5.4版本中，这个情况是不能输出结果的。
`5.4.0 continue 0; 不再合法。这在之前的版本被解析为 continue 1;。`
`5.4.0 取消变量作为参数传递（例如 $num = 2; continue $num;）。`