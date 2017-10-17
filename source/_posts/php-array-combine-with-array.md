---
title: PHP数组的“合并相加”
date: 2016-06-08 10:31:24
tags: PHP
---

>情况大概是这样的，一个刚入门PHP程序狗，某天接到了一个需求，需要从两个表里获取数据，这两张表的字段是相同的，但记录的类型不同，现在需要将两个类型合并了，不建新表。大概就是把相同字段值的记录中其他某个字段值相加这个意思(可能说的不清楚看下面的例子就知道是什么意思了)。当时第一个反应就是写sql语句，拼啊拼，也不是不行，写了一个sql语句，等了n分钟重要出结果，要是这样就上线了，那不得被吊起来打。可能还是自己的原因，自己数据库学得特别烂，不会写sql语句，只能在原有的程序上，照葫芦画瓢儿，各自从每个表中去取数据，然后处理数组。也不知道这样是不是可行的。


### 情形一：先来一个简单的相加，很简单的 ###
数组1:

```php
Array
(
    '0' => 20,
    '1' => 10,
    '2' => 23,
    '3' => 12,
)
```
数组2：

```php
Array
(
    '0' => 10,
    '1' => 5,
    '2' => 12,
)
```

应该得到的结果是：

```php
Array
(
    '0' => 30,
    '1' => 15,
    '2' => 35,
    '3' => 12,
)
```
处理函数如下：

```php
    function array_add($arr1,$arr2)
    {
        //根据键名获取两个数组的交集
        $arr=array_intersect_key($arr1, $arr2);
        //遍历第二个数组，如果键名不存在与第一个数组，将数组元素增加到第一个数组
        foreach($arr2 as $key => $value){
            if(!array_key_exists($key, $arr1)){
                $arr1[$key] = $value;
            }
        }
        //计算键名相同的数组元素的和，并且替换原数组中相同键名所对应的元素值
        foreach($arr as $key => $value){
            $arr1[$key] = $arr1[$key] + $arr2[$key];
        }
        //返回相加后的数组
        return $arr1;
    }
```

### 情形二：比如有有两个数组 ###
数组1:

```php
array
(
    '0'=>array
    (
        'type'=>0,
        'hits'=>10
    ),
    '1'=>array
    (
        'type'=>1,
        'hits'=>20
    ),
    '2'=>array
    (
        'type'=>2,
        'hits'=>15
    )
)
```
数组2:

```php
array(
    '0'=>array
    (
        'type'=>0,
        'hits'=>8
    ),
    '1'=>array
    (
        'type'=>1,
        'hits'=>12
    ),
    '2'=>array
    (
        'type'=>2,
        'hits'=>14
    )，
    '3'=>array
    (
        'type'=>3,
        'hits'=>11
    )
)
```
按照要求后，需要得到结果是：

```php
array(
    '0'=>array(
        'type'=>0,
        'hits'=>18
    ),
    '1'=>array(
        'type'=>1,
        'hits'=>32
    ),
    '2'=>array(
        'type'=>2,
        'hits'=>29
    )，
    '3'=>array(
        'type'=>3,
        'hits'=>11
    )
)
```

处理函数：

```php
    function arrayAddArray($res1,$res2,$key1,$key2)
    {
        $re0 = array_merge($res1,$res2);
        $tmp = array();
        foreach($re0 as $k => $v)
        {
            isset($tmp[$v[$key1]]) ? $tmp[$v[$key1]] += $v[$key2] : $tmp[$v[$key1]] = $v[$key2];
        }
        $result = array();
        foreach($tmp as $k => $v){
            $result[] = array($key1=>$k,$key2=>$v);
        }
        return $result;
    }
```

### 情形三：处理两个数组 ####
数组1:

```php
array
(
    '0'=>array
    (
        'type'=>0,
        'hits'=>10,
        'ttt'=>0.1
    ),
    '1'=>array
    (
        'type'=>1,
        'hits'=>20,
        'ttt'=>0.2
    ),
    '2'=>array
    (
        'type'=>2,
        'hits'=>15,
        'ttt'=>0.12
    )
)
```
数组2:

```php
array
(
    '0'=>array
    (
        'type'=>0,
        'hits'=>8,
        'ttt'=>0.1
    ),
    '1'=>array
    (
        'type'=>1,
        'hits'=>12,
        'ttt'=>0.12
    ),
    '2'=>array
    (
        'type'=>2,
        'hits'=>14,
        'ttt'=>0.2
    ),
    '3'=>array
    (
        'type'=>3,
        'hits'=>11,
        'ttt'=>0.12
    )
)
```
按照要求后，需要得到结果是：

```php
array
(
    '0'=>array
    (
        'type'=>0,
        'hits'=>18,
        'ttt'=>0.2
    ),
    '1'=>array
    (
        'type'=>1,
        'hits'=>32,
        'ttt'=>0.32
    ),
    '2'=>array
    (
        'type'=>2,
        'hits'=>29,
        'ttt'=>0.32
    ),
    '3'=>array
    (
        'type'=>3,
        'hits'=>11,
        'ttt'=>0.12
    )
)
```

处理函数：

```php
    //里面的每个数组有三个键名，两个二维数组合并相加不再添加新的键名
    function arrayAdd3ArrayWithoutNewKey($re1,$re2,$key1,$key2,$key3)
    {
        $res1 = arrayAddArray($re1, $re2, $key1, $key2);//先将$key2键名对应的值相加,用到上面的函数。
        $res2 = arrayAddArray($re1, $re2, $key1, $key3); //将$key3键名对应的值相加
        $res = array_merge($res1, $res2);
        $tmp = array();

        foreach ($res as $v) {
            if (isset($tmp[$v[$key1]])) {
                empty($tmp[$v[$key1]][$key2]) && $tmp[$v[$key1]][$key2] = isset($v[$key2]) ? $v[$key2] : 0;
                empty($tmp[$v[$key1]][$key3]) && $tmp[$v[$key1]][$key3] = isset($v[$key3]) ? $v[$key3] : 0;
            } else {
                $tmp[$v[$key1]] = array(
                    $key1 => $v[$key1],
                    $key2 => isset($v[$key2]) ? $v[$key2] : 0,
                    $key3 => isset($v[$key3]) ? $v[$key3] : 0,
                );
            }
        }
        return $tmp;
    }
```

### 情形四:还是两个二维数组需要合并相加，可能现在是这样的,需要增加一个键计算新的值：###
数组1:

```php
array
(
    '0'=>array
    (
        'type'=>0,
        'hits1'=>10,
        'hits2'=>15
    ),
    '1'=>array
    (
        'type'=>1,
        'hits1'=>20,
        'hits2'=>12
    ),
    '2'=>array
    (
        'type'=>2,
        'hits1'=>15,
        'hits2'=>18
    )           
);
```
数组2:

```php
array
(
    '0'=>array
    (
        'type'=>0,
        'hits1'=>8,
        'hits2'=>12
    ),
    '1'=>array
    (
        'type'=>1,
        'hits1'=>12,
        'hits2'=>22
    ),
    '2'=>array
    (
        'type'=>2,
        'hits1'=>14,
        'hits2'=>12
    ),
    '3'=>array
    (
        'type'=>3,
        'hits1'=>11,
        'hits2'=>10
    )
)
```
按照要求后，需要得到结果是：(原先rate比率是在sql语句中就算好的，现在要得到正确的比率，就得先将'hits1'相加，'hits2'相加，相加完了再计算)

```php
array
(
    '0' => array
        (
            'type'=>0
            'hits1'=>18
            'hits2'=>27
            'rate'=>0.67
        )

    '1' => array
        (
            'type'=>1
            'hits1'=>32
            'hits2'=>34
            'rate'=>0.94
        )

    '2' => array
        (
            'type'=>2
            'hits1'=>29
            'hits2'=>30
            'rate'=>0.97
        )

    '3' => array
        (
            'type'=>3
            'hits1'=>5
            'hits2'=>10
            'rate'=>0.50
        )

)
```
处理函数：

```php
    function arrayAdd3Array($re1,$re2,$key1,$key2,$key3,$key4)
    {
        $res1 = arrayAddArray($re1, $re2, $key1, $key2);
        $res2 = arrayAddArray($re1, $re2, $key1, $key3);

        $res = array_merge($res1, $res2);
        $tmp = array();

        foreach ($res as $v) {
            if (isset($tmp[$v[$key1]])) {
                empty($tmp[$v[$key1]][$key2]) && $tmp[$v[$key1]][$key2] = isset($v[$key2]) ? $v[$key2] : 0;
                empty($tmp[$v[$key1]][$key3]) && $tmp[$v[$key1]][$key3] = isset($v[$key3]) ? $v[$key3] : 0;
                if(!empty($tmp[$v[$key1]][$key2]) && !empty($tmp[$v[$key1]][$key3])){
                    $tmp[$v[$key1]][$key4] = $tmp[$v[$key1]][$key2]/$tmp[$v[$key1]][$key3];  //计算新添加的键值
                }
            } else {
                $tmp[$v[$key1]] = array(
                    $key1 => $v[$key1],
                    $key2 => isset($v[$key2]) ? $v[$key2] : 0,
                    $key3 => isset($v[$key3]) ? $v[$key3] : 0,
                );
            }
        }
        return $tmp;
    }
```

### 情形五：变着花样玩，现在是根据两个键判断然后在合并 ###

数组一：

```php
array(
    '0'=>array(
        'type'=>0,
        'num'=>'500'，
        'hits'=>12
    ),
    '1'=>array(
        'type'=>1,
        'num'=>'502'，
        'hits'=>30
    ),
    '2'=>array(
        'type'=>2,
        'num'=>'404'，
        'hits'=>10
    )，
    '3'=>array(
        'type'=>3,
        'num'=>'400'，
        'hits'=>14
    )
)
```

数组二：

```php
array(
    '0'=>array(
        'type'=>0,
        'num'=>'500'，
        'hits'=>15
    ),
    '1'=>array(
        'type'=>1,
        'num'=>'502'，
        'hits'=>4
    ),
    '2'=>array(
        'type'=>2,
        'num'=>'404'，
        'hits'=>20
    )，
    '3'=>array(
        'type'=>3,
        'num'=>'400'，
        'hits'=>14
    )
)
```
应该得到的结果：

```php
array(
    '0'=>array(
        'type'=>0,
        'num'=>'500'，
        'hits'=>27
    ),
    '1'=>array(
        'type'=>1,
        'num'=>'502'，
        'hits'=>34
    ),
    '2'=>array(
        'type'=>2,
        'num'=>'404'，
        'hits'=>30
    )，
    '3'=>array(
        'type'=>3,
        'num'=>'400'，
        'hits'=>28
    )
)
```
处理函数：

```php
    function arrayAddArrayByTwoKeys($re1,$re2,$key1,$key2,$key3)
    {
        $tmp = array();
        foreach ($re1 as $v1) {
            $tmp[$v1[$key1]][$v1[$key2]] = $v1[$key3];
        }

        foreach($re2 as $v2) {
            if(isset($tmp[$v2[$key1]][$v2[$key2]])) {
                $tmp[$v2[$key1]][$v2[$key2]] += $v2[$key3];
            }
        }

        foreach($tmp as $kk => $vv){
            $n = count($vv);
            $k = array_keys($vv);
            $v = array_values($vv);
            for($i=0;$i<$n;$i++){
                $result[] = array(
                    $key1=>$kk,
                    $key2=>$k[$i],
                    $key3=>$v[$i],
                );
            }
        }
        return $result;
    }
```

差不多就是这几个情形比较多吧，第一次就这么处理了数组，给我留下深深地印象。主要自己把玩数组不是很溜，不管了，先记录一点吧。


