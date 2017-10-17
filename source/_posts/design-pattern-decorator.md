---
title: 设计模式之“装饰者模式”——手机与包装盒示例
date: 2017-03-29 22:27:22
tags: 设计模式
---

## 前言
在“装饰模式”中很好的提现了开放关闭原则，即类应该对扩展开放对修改关闭。装饰者模式可以让我们在不对原来代码的修改的情况下对类进行扩展。现在我们举一个例子，就好比给生产好的手机进行包装，我们在对手机进行包装的过程不会去对手机进行任何修改，只管用盒子将盒子包装好就行。

装饰者模式，用另一种表达方式就是“对原有的物体进行装饰，给原有的物体添加上新的装饰品”。这里的例子就是，手机是被装饰者，包装盒就是装饰品。这样的话，可以看出装饰者模式就是“动态地将责任附加到对象上。若要扩展功能，装饰着提供了比继承更有弹性的替代方案。”

在“装饰者模式”中所使用的装饰就是变化的部分，也就是Decorator是变化的部分对应着我们的包装盒，因为对手机进行包装的过程就是包装盒变化的过程，也就是为手机装饰的过程。而手机就可以看做是组件。需要注意的是，所谓的装饰者不仅仅是给组件添加的新的装饰品。一个装饰者对象就是添加该装饰后的组件，也就是说装饰者=旧组件 + 新装饰品。

## 手机+包装盒的具体实现

### 实现一个手机的接口`PhoneInterface.php`

```php
<?php
namespace decorator;

interface PhoneInterface
{
    // 生产手机
    public function product();
}
```

### 实现一个装饰器抽象的基类`Decorator.php`

```php
<?php
namespace decorator;

/**
 * 装饰器抽象类
 */
abstract class Decorator implements PhoneInterface
{
    /**
     * 产品生产线对象
     * @var Object
     */
    protected $phone;

    public function __construct(PhoneInterface $phone)
    {
        $this->phone = $phone;
    }

    /**
     * 生产
     */
    public function product()
    {
        $this->phone->product();
    }

    abstract public function decorate($value);

}

```

### 实现包装盒子的装饰器`DecoratorPack.php`

```php
<?php
namespace decorator;

/**
 * 包装盒子装饰器
 */
class DecoratorPack extends Decorator
{
    private $pack;

    public function __construct(PhoneInterface $phone)
    {
        $this->phone = $phone;
    }

    public function __set($name = '', $value = '')
    {
        $this->$name = $value;
    }

    /**
     * 生产
     */
    public function product()
    {
        $this->phone->product();
        $this->decorate($this->pack);
    }

    /**
     * 进行包装
     */
    public function decorate($value = '')
    {
        echo "使用{$value}进行了包装\n";
    }
}
```


### 实现各个装饰者(Decorator),这里是`ApplePhone.php`和`SmartisanPhone.php`

```php
<?php
namespace decorator;

/**
 * 苹果手机
 */
class ApplePhone implements PhoneInterface
{
    public function product()
    {
        echo "生产了一部苹果手机\n";
    }
}
```

```php
<?php
namespace decorator;

/**
 * 锤子手机
 */
class SmartisanPhone implements PhoneInterface
{
    public function product()
    {
        echo "生产了一部锤子手机\n";
    }
}
```

### 大差不差了，最后测试 `test.php`

```php
<?php
/**
 * 结构型模式
 * 装饰器模式
 * 对现有的对象增加功能，动态的将责任附加到对象上。
 * 和适配器的区别： 适配器是链接两个接口，装饰器是对现有的对象包装
 */
// 注册自加载
spl_autoload_register('autoload');

function autoload($class)
{
    require dirname($_SERVER['SCRIPT_FILENAME']) . '//..//' . str_replace('\\', '/', $class) . '.php';
}

use decorator\DecoratorPack;
use decorator\ApplePhone;
use decorator\SmartisanPhone;

try {
    echo "为加装饰器之前： \n";
    // 生产苹果手机
    $applePhone = new ApplePhone();
    $applePhone->product();

    // 生产锤子手机
    $smartisanPhone = new SmartisanPhone();
    $smartisanPhone->product();

    echo "\n------------\n";

    echo "进行包装装饰器: \n";
    // 初始化一个包装适配器
    $decoratorPack1 = new DecoratorPack(new ApplePhone);
    $decoratorPack1->pack = '白盒子';
    $decoratorPack1->product();

    $decoratorPack2 = new DecoratorPack(new SmartisanPhone);
    $decoratorPack1->pack = '黑盒子';
    $decoratorPack1->product();

} catch (\Exception $e) {
    echo $e->getMessage();
}
```

### 运行结果：通过结果更清晰这个过程了

```
为加装饰器之前：
生产了一部苹果手机
生产了一部锤子手机

------------
进行包装装饰器:
生产了一部苹果手机
使用白盒子进行了包装
生产了一部苹果手机
使用黑盒子进行了包装
```
