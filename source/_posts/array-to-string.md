---
title: 将数组转换成字符串
date: 2016-04-28 15:08:32
tags: 
      -php
---

---
##一个简单的方法：将一个二维的数组转换成字符串

常常出现这样一种情况，sql语句查询出来的结果是一个二维数组，但是查询的结果还是需要用到下一条查询语句，这个时候可能需要将一个二维数组转换成字符串用于sql语句的查询。
下面是一个简单的例子：

```php
function arr2str ($arr)
{
    foreach ($arr as $v)
    {
        $v = join(",",$v); //可以用implode将一维数组转换为用逗号连接的字符串
        $temp[] = $v;
    }
    $t="";
    foreach($temp as $v){
        $t.="'".$v."'".",";
    }
    $t=substr($t,0,-1);
    return $t;
}
```
