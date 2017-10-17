---
title: php二维数组去重
date: 2016-10-18 21:55:04
tags: php
---

没有啥，就是随便写点关于php数组的处理，数组一直是我的弱项，现在还是做一点记录吧。

对于二维数组咱们分两种情况讨论，一种是因为某一键名的值不能重复，删除重复项；另一种因为内部的一维数组不能完全相同，而删除重复项，下面举例说明：

## 某一键名的值不能重复，删除重复项

```php
<?php

function arrayUnique($arr, $key) {
    $tmp = [];
    foreach ($arr as $k => $v) {
        //搜索$v[$key]是否在$tmp_arr数组中存在，若存在返回true
        if (in_array($v[$key], $tmp)) {
            unset($arr[$k]);
        } else {
            $tmp[] = $v[$key];
        }
    } 
    sort($arr); //sort函数对数组进行排序
    return $arr;
}

$test = [
    ['id' => 123, 'name' => '张三'],
    ['id' => 123, 'name' => '李四'],
    ['id' => 124, 'name' => '王五'],
    ['id' => 125, 'name' => '赵六'],
    ['id' => 126, 'name' => '赵六'],
];
$key = 'id';
arrayUnique(&$test, $key);
print_r($test);
```
显示结果就知道了。。。

#### 内部的一维数组不能完全相同，而删除重复项

```php
<?php
function arrayUnique($array)
{
    foreach ($array as $v){
        $v = join(",", $v); //降维,也可以用implode,将一维数组转换为用逗号连接的字符串
        $temp[] = $v;
    }
    $temp = array_unique($temp);    //去掉重复的字符串,也就是重复的一维数组
                
    foreach ($temp as $k => $v){
        $temp[$k] = explode(",", $v);   //再将拆开的数组重新组装
    }
    return $temp;
}
$test = [
    ['id' => 123, 'name' => '张三'],
    ['id' => 123, 'name' => '李四'],
    ['id' => 124, 'name' => '王五'],
    ['id' => 125, 'name' => '赵六'],
    ['id' => 126, 'name' => '赵六'],
];
$data = arrayUnique($test);
print_r($data);
```
结果显示就知道了。。。。

就这样简单粗暴一点吧！