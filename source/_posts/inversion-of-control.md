---
title: Laravel源码阅读笔记——服务容器
date: 2017-04-11 22:35:43
tags: Laravel
---

## 前言
> 使用laravel也有一段时间了，现在应该向深入理解laravel去看看一些源码了。laravel之所以新颖，使用了大量PHP新语法，包含命名空间（组件化开发的条件），匿名函数，反射机制，还有后期静态绑定，trait等等。
> 当你深入去挖掘源码的时候，laravel框架中使用的都是大家熟知的东西，并没有多么的高深。那就耐着性子一层一层地往下扒吧！

## 服务容器

服务容器是laravel框架中相当核心的东西，提供了整个框架运行需要的服务。服务是什么呢？通俗地讲就是系统运行中需要的比如对象，配置信息之类的东西。服务容器就是承载这些东西的，在程序运行过程中动态的给系统提供服务（资源）。

服务容器提供的东西比较多，在这些功能中，需要注意的问题就是解决依赖实现解耦。说到这里，我们可能都听说过控制反转（Inversion Of Control）——IOC容器。服务容器也可以是IOC容器。控制反转模式是解决系统组件之间的相互依赖关系的一种模式。那什么是依赖？怎么解决依赖？不妨看一个例子。

我们就拿出去春游儿来谈吧，出去玩有很多方式，比如开车出去，乘坐火车出去。

```php
<?php
interface Visit
{
    public function go();
}

class Car implements Visit
{
    public function go()
    {
        echo "Drive car";
    }
}

class Train implements Visit
{
    public function go()
    {
        echo "Take train";
    }
}

// 实现该类需要依赖交通工具实例
class Traveller
{
    protected $trafficTool;
    public function __construct($trafficTool)
    {
       // 依赖产生
        $this->trafficTool = new Car();
    }

    public function visitOut()
    {
        $this->trafficTool->go();
    }
}
$traveller = new Traveller();
$traveller->visitOut();
```

不难看出上面的代码，两个组件之间就产生的依赖，`Traveller`的初始化依赖与`Car`，如果需求改动了，实例化的交通工具不是car是其他的，就需要不断改变实例化的对象，你说这样的代码怎样？我们不应该在`Traveller`中固定交通工具的初始化，而是交给外部去实现，将这种依赖关系通过动态注入的方式实现，这就是IOC模式的思想。

于是乎，我们将交通工具的实例化提取出来外部管理，这样做，也体现出面向对象的设计的一个原则——将经常变化的部分提取出去，与固定不变的部分进行分离。这里我们使用工厂模式来实现。

### 工厂模式

```php
class TrafficToolFactory
{
    public function createTrafficTool($name)
    {
        switch ($name) {
            case 'Car' :
                return new Car();
                break;
            case 'Train':
                return new Train();
                break;
            default:
                exit("set traffic tool error");
                break;
        }
    }
}

class Traveller
{
    protected $trafficTool;
    public function __construct($trafficTool)
    {
        // 通过工厂生产依赖的交通工具实例
        $factory = new TrafficToolFactory();
        $this->trafficTool = $factory->createTrafficTool($trafficTool);
    }

    public function visitOut()
    {
        $this->trafficTool->go();
    }
}
$traveller = new Traveller('Car');
$traveller->visitOut(); 
```
现在看，`Traveller`跟`trafficTool`之间没有依赖关系了，但是却变成`Traveller`跟`TrafficToolFactory`之间的依赖了，如果我们再碰到需求，需要修改工厂模式，这样的代码还是不易于维护。

### IOC模式
控制反转模式也叫做依赖注入模式，控制反转是将组件之间的依赖关系从程序内部提到外部的容器中，而依赖注入是将组建的依赖通过外部以参数或其他形式注入。两种说法一个意思。

```php
class Traveller
{
    protected $trafficTool;
    public function __construct(Visit $trafficTool)
    {
        $this->trafficTool = $trafficTool;
    }

    public function visitOut()
    {
        $this->trafficTool->go();
    }
}
// 生成依赖的交通工具实例
$trafficTool = new Train();
// 依赖注入的方式解决依赖问题
$traveller = new Traveller($trafficTool);
$traveller->visitOut();
```

这儿就是一个依赖注入的过程，`Traveller`类的构造函数依赖一个外部具有`Visit`接口的实例，在实例化`Traveller`时，传入一个`$trafficTool`实例，即通过依赖注入的方式解决依赖问题。

但是还是有一个小问题，这儿是我们是通过手动的方式注入依赖，依赖注入需要通过接口来限制，而不能随便开放。你还有完没完？其实通过IOC容器就可以实现自动依赖注入。

```php
class Container
{
    // 用于提供实例的回调函数
    protected $bindings = [];

    // 绑定接口和生成相应实例的回调函数
    public function bind($abstract, $concrete = null, $shared = false)
    {
        if (! $concrete instanceof Closure) {
                // 如果提供的参数不是回调函数，则产生默认的回调函数
            $concrete = $this->getClosure($abstract, $concrete);
        }

        $this->bindings[$abstract] = compact('concrete', 'shared');
    }
    
    // 默认生成实例的回调函数
    protected function getClosure($abstract, $concrete)
    {
        // 生成实例的回调函数，$container 一般为IOC容器对象，在调用回调生成实例时提供
        // 即 build函数中的 $concrete($this)
        return function ($container, $parameters = []) use ($abstract, $concrete) {
            $method = ($abstract == $concrete) ? 'build' : 'make';
                // 调用的是容器的build或者make方法生成实例
            return $container->$method($concrete, $parameters);
        };
    }

    // 生成实例对象，首先解决接口和要实例化类之间的依赖关系
     public function make($abstract)
    {
        $concrete = $this->getConcrete($abstract);
        if ($this->isBuildable($concrete, $abstract)) {
            $object = $this->build($concrete);
        } else {
            $object = $this->make($concrete);
        }
        return $object;
    }

    protected function isBuildable($concrete, $abstract)
    {
        return $concrete === $abstract || $concrete instanceof Closure;
    }

    // 获取绑定的回调函数
    protected function getConcrete($abstract)
    {
        if (!isset($this->bindings[$abstract])) {
            return $abstract;
        }
        return $this->bindings[$abstract]['concrete'];
    }

    // 实例化对象
    public function build($concrete)
    {
        // 服务能否被服务提供者注册为实例
        if ($concrete instanceof Closure) {
            return $concrete($this);
        }
            // $concrete就是类名
        $reflector = new ReflectionClass($concrete);

        if (! $reflector->isInstantiable()) {
            echo $message = "Target [$concrete] is not instantiable.";
        }
            // 获取构造信息
        $constructor = $reflector->getConstructor();

        if (is_null($constructor)) {
            return new $concrete;
        }
            // 获取构造函数依赖的输入参数
        $dependencies = $constructor->getParameters();


        $instances = $this->resolveDependencies(
            $dependencies
        );

        return $reflector->newInstanceArgs($instances);
    }

    // 解决通过反射机制实例化对象时的依赖
    protected function resolveDependencies(array $dependencies)
    {
        $results = [];

        foreach ($dependencies as $dependency) {
            
            $results[] = is_null($class = $dependency->getClass())
                            ? NULL
                            : $this->resolveClass($dependency);
        }

        return $results;
    }

    protected function resolveClass(ReflectionParameter $parameter)
    {
        return $this->make($parameter->getClass()->name);
    }
}

$app = new Container();
// 容器的填充
$app->bind('Visit', "Car");
$app->bind("traveller", "Traveller");
// 通过容器实现依赖注入，完成类的实例化
$tra = $app->make("traveller");
$tra->visitOut();
```

在这个实现过程中，没有用new关键字来实例化对象，不需要关注对象的依赖关系，只需要在容器填充的过程中理顺接口与实现类之间的关系以及实现类与依赖接口之间的关系。这里的实例化是通过反射的机制完成的。


下篇继续以Laravel的源码进行分析。

在Laravel中，服务容器是由`Illuminate\Container\Container`类实现的，实现了服务容器的核心功能，而`Illuminate\Foundation\Application`类继承了该类，主要实现服务容器的初始配置和功能扩展。
在`bootstrap\app.php`中，$app就是服务容器的创建，然后在index.php中，通过require_once `bootstrap\app.php`就生成了服务容器。


服务容器生成了之后，里面除了最基本的好像什么也没有，那是不行的，首先需要向容器中填充服务，也即是服务绑定。那是怎么绑定的呢？可以简单地理解为键值对的概念，根据一个“key”就能找到对应的服务。难怪里面有一些数组属性。对于不同的绑定需要在服务容器中不同的绑定函数来实现，主要包括回调函数服务绑定和实例对象服务绑定。回调函数服务绑定就是一个回调函数，而实例对象服务绑定的就是一个实例对象。

回调函数服务绑定还分为普通绑定和单例绑定。普通绑定就是每次生成该服务的实例对象时还会生成一个新的实例对象，也就是说在生命周期中可以同时生成多个这种实例对象，而单例绑定在生成一个实例对象后，如果再次生成就会返回第一次生成的实例对象也就是在程序生命周期中只能生成一个这样的实例的对象，这个就是设计模式中的单例模式。

说了这么多，现在我们来一个简单的测试。

```php
<?php
namespace App\Test;

class GeneralService 
{
    public $serviceName;
}
```
```php
<?php
namespace App\Test;

class SingleService 
{
    public $serviceName;
}
```
```php
<?php
namespace App\Test;

class InstanceService 
{
    public $serviceName;
}
```

在`bootstrap\app.php`中，

```php
<?php
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);

$app->bind(App\Test\GenernalService::class, function ($app) {
    return new App\Test\GeneralService();
})
$app->singleton(App\Test\SingleService::class, function ($app) {
    return new App\Test\SingleService();
})
$instance = new App\Test\InstanceService();
$app->instance('instanceService', $instance);
return $app;
```
上面的服务容器通过三种不同的方式绑定服务，一种是普通模式绑定回调函数，另一种是单例模式绑定回调函数，还有一种是绑定实例对象。绑定后在服务容器增加内容：

```php
$bindings = array("App\Test\GeneralService" => array("concrete" => {Closure}, "shared" => false), "App\Test\SingleService" => array("concrete" => {Closure}, "share" => true));
$instances = array("instanceService" => {App\Test\InstanceService});
```

不难看出，回调函数服务绑定是在$bindings中记录的，其key为绑定的服务名称，value是回调函数和模式标识。如果是普通模式，则share为false，如果是单例模式则为true。实例对象服务绑定在$instances中记录，key为服务名称，value为实例对象。

说到这里其实还有一个形式的绑定，就是绑定具体类名称，本质上也是绑定回调函数的方式，只是回调函数是服务容器根据提供的参数自动生成的。绑定服务的时候可以通过类名或者接口名作为服务，而服务是类名。

















