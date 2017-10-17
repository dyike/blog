---
title: 设计模式之“策略模式”——出行outing示例
date: 2017-03-27 21:42:50
tags: 设计模式
---

策略模式通俗地说就是将不同的策略（算法）进行封装，让他们之间可以相互替换，此模式让策略的变化独立于使用策略的用户。

在设计模式中有不同的设计原则，其中一条就是”将可能会变化的代码独立出来，不要和不变得代码混在一起“。

现在举个例子就是，我们周末出去游玩，我们出去可以乘坐多种交通工具，这个时候就可以选择”策略模式“来实现。简单分析一下，这个场景中的乘坐的交通工具是多种多样的，可以骑自行车，可以坐地铁，可以自己开车。

根据上面的设计思路，，我们可以对”乘坐交通工具“的策略进行提取。使用`StrategyInterface`接口来规定策略，使用不同的出行方式对外都有一个统一的接口。在此就是乘坐交通工具，不同的出行方式有不同的”乘坐交通工具“的策略。

对于出行（outing）的定义就是不变的部分。这个类中定义了出行的方式，以及是否改变策略，其中依赖于乘坐交通工具的接口。

## `策略模式`的具体实现:

### 乘坐交通工具的策略接口`StrategyInterface.php`

```php
<?php
namespace strategy;

interface StrategyInterface {
    public function takeTraffic();
}
```

### 紧接着我们实现出行(Outing)`Outing.php` 

```php
<?php
namespace strategy;

/**
 * 实体类
 * 依赖外部不同策略的实体类
 */
class Outing
{
    /**
     * 策略实例
     * @var Object
     */
    private $traffic;
    /**
     * 是否改变策略
     * @var boolean
     */
    private $isChangeMind = false;

    public function __construct(StrategyInterface $traffic)
    {
        $this->traffic = $traffic;
    }

    /**
     * 改变策略
     */
    public function change(StrategyInterface $traffic)
    {
        $this->traffic = $traffic;
        $this->isChangeMind = true;
    }

    public function outing()
    {
        if ($this->isChangeMind) {
            echo "改变策略\n";
            $this->traffic->takeTraffic();
        } else {
            $this->traffic->takeTraffic();
        }
    }
}
```

### 然后,具体实现每一个策略`OutingByCar.php`和`OutingBySubway.php`

```php
<?php
namespace strategy;

/**
 * 观察者实体类示例
 */
class OutingByCar implements StrategyInterface
{
    /**
     * 行为
     * @return string
     */
    public function takeTraffic()
    {
        echo "选择自己开车出去玩 \n";
    }
}
```

```php
<?php
namespace strategy;

/**
 * 观察者实体类示例
 */
class OutingBySubway implements StrategyInterface
{
    /**
     * 行为
     * @return string
     */
    public function takeTraffic()
    {
        echo "选择自己坐地铁出去玩 \n";
    }
}
```


最后，我们来测试一下

```php
<?php

/**
 * 行为型模式
 *
 * 策略模式
 * 策略依照使用而定
 */

// 注册自加载
spl_autoload_register('autoload');

function autoload($class)
{
    require dirname($_SERVER['SCRIPT_FILENAME']) . '//..//' . str_replace('\\', '/', $class) . '.php';
}

use strategy\Outing;
use strategy\OutingByCar;
use strategy\OutingBySubway;

// 使用策略1
$substance = new Outing(new OutingByCar);
$substance->outing();
// 现在改变策略
$substance->change(new OutingBySubway);
// 使用策略2
$substance->outing();
```

运行的结果：

```
选择自己开车出去玩
改变策略
选择自己坐地铁出去玩
```

## 优缺点

### 优点：

* 策略模式避免了多重条件的转移语句，消除一些if else条件语句。
* 提供了可以替换继承关系的办法： 继承提供了另一种支持多种算法或行为的方法。
* 策略模式提供相同行为的不同实现，这样就可以根据不同的时间空间条件进行不同策略的选择。

### 缺点：

* 客户端必须知道所有的策略类，并自行决定使用哪一个策略类。
* 策略模式将造成产生很多策略类，可以通过使用享元模式在一定程度上减少对象的数量。


## 什么时候使用

* 多个类只区别在表现行为不同，可以使用策略模式，在运行时动态选择具体要执行的行为。
* 需要在不同情况下使用不同的策略(算法)，或者策略还可能在未来用其它方式来实现。
* 对客户隐藏具体策略(算法)的实现细节，彼此完全独立。
