---
title: Laravel请求生命周期
date: 2017-04-22 20:45:40
tags: Laravel 
---

要想了解Laravel的整个运行流程，那么就需要从入口文件下手，再一步步的往下探索，剥开神秘的面纱。在Laravel框架中入口文件就是public文件夹下的index.php文件。Laravel的生命周期也就是从这里开始的，从这里结束。如此优雅的代码却清晰的展示请求到相应的整个生命周期。不妨先看看index.php的源码吧：

```php
<?php
// 第一块
require __DIR__.'/../bootstrap/autoload.php';
// 第二块
$app = require_once __DIR__.'/../bootstrap/app.php';
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
// 第三块
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
$response->send();
// 第四块
$kernel->terminate($request, $response);
```
为了分析上面的代码，将代码分为四块：

* 第一块
> 主要用来实现类的自动加载，注册加载composer自动生成的class loader。

* 第二块
> 主要来实例化服务容器，Laravel的一些基本服务的注册，核心组件注册等等，当然了也包括容器本身的注册。在注册的过程中服务容器会在对应的属性中记录注册的内容，方便于在程序运行期间提供对应的服务。这部分内容可以称为程序启动准备阶段。

* 第三块
> 处理请求，用户发送的请求入口文件是index.php，从生成`Illuminate\Http\Request`实例，交给handle()进行处理。将该$request实例绑定到第二步生成的$app容器上。并发送相应

* 第四块
> 请求的后期清理处理工作，请求结束并进行回调。


# 服务容器的实例化（详谈第二块代码）
先看看`bootstrap\app.php`源码：里面有注释

```php
<?php
// 服务容器实例化的过程
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);
// 向容器绑定了三个核心类服务
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);
$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);
$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);
// 返回服务容器实例
return $app;
```

关于`Illuminate\Foundation\Application.php`文件查看源码

#### 应用的基础路径`setBasePath()`
设置注册应用的基础路径，并在容器中绑定这些基础基础路径。

#### 注册基础绑定`registerBaseBindings()`

```php
   // 向容器注册基础绑定
    protected function registerBaseBindings()
    {
        static::setInstance($this);
        // 向服务共享实例数组中注册量单例服务，
        // 服务名称分别为`app`和`Illuminate\Container\Container`
        // 对应的实例对象即为服务容器本身
        $this->instance('app', $this);
        $this->instance(Container::class, $this);
    }
```

主要绑定容器实例本身，服务容器中设置了一个静态变量`$instance`，该变量是在`Illuminate\Container\Container.php`中定义,`Application`类继承了`Container`类，在`Container`类中可以通过`public static function getInstance()`获取服务容器实例。服务容器实例还绑定不同的服务容器别名，记录在`$instances`共享实例数组（在`Container`类中）

```php
    // 在容器中注册一个已经存在的实例作为共享实例
    public function instance($abstract, $instance)
    {
        $this->removeAbstractAlias($abstract);

        unset($this->aliases[$abstract]);
        // 检查该类有没有本绑定
        // 如果已经绑定，将触发向容器注册的反弹回调
        $this->instances[$abstract] = $instance;

        if ($this->bound($abstract)) {
            $this->rebound($abstract);
        }
    }
```

#### 注册基础服务提供者`registerBaseServiceProviders()`

服务提供者的注册是非常重要的，因为它给服务容器添加各种服务。这里只是注册了最基本的三个服务：

```php
    protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));
        $this->register(new LogServiceProvider($this));
        $this->register(new RoutingServiceProvider($this));
    }
```

在容器里注册一个服务提供者的方法：
`public function register($provider, $options = [], $force = false)`

源码阅读分析：
* 首先`getProvider($provider)`进行判断，如果服务提供者存在则获取这个实例对象。
* 第二，如果给定$provider是一个string，则通过类名`resolveProvider($provider)`进行实例对象。
* 然后，对实例化完成的$provider进行标记为已经注册，`markAsRegistered($provider)`。
* 最后，启动规定的服务提供者`bootProvider($provider)`。


#### 注册核心类别名`registerCoreContainerAliases()`
`$aliases`数组变量中定义了整个框架的核心服务别名，在解析的过程中需要根据实例化的类或者接口名查找服务别名，然后通过服务别名获取具体的服务。

这里放一张图进行小小的总结：
 ![程序启动准备阶段](https://raw.githubusercontent.com/yf92/yf92.github.io/master/images/laravel/程序启动准备阶段.png)


# 核心类实例化(Kernel类)
为什么要服务容器？服务容器实例化后，就可以通过服务容器自动实例化对象了，可以参考上一篇的[服务容器](https://www.dyike.com/2017/04/11/inversion-of-control/)。
`index.php`中Kernel类就是通过服务容器自动创建完成的。

在`bootstrap\app.php`文件中就注册了三个服务，其中包括了这个核心类接口，在注册服务时，服务名一般是接口。注册的服务是具体的类名，这一般是通过反射基础来实例化的，并通过反射机制解决构造函数的依赖关系，参考上篇的服务容器有讲解。

这里说的核心类是指`App\Http\Kernel`类，这个类只是定义了`$middleware` ，`$middlewareGroups` 和 `$routeMiddleware`是哪个数组属性。这个类是继承`Illuminate\Foundation\Http\Kernel`类的，不妨看看这个类中的构造函数，不难看出这个构造函数是存在依赖关系，一个是`Illuminate\Contracts\Foundation\Application`，还有一个是`Illuminate\Routing\Router`。他们在服务容器初始化的时候都进行了实例化。

# 请求实例化`capture()`

在程序启动准备工作完成了之后，就开始请求的实例化。对于请求就是客户端的发送的一个请求报文。这个对应着`Illuminate\Http\Request`类的实例对象。请求实例的创建是通过`Illuminate\Http\Request`类中的`capture()`函数完成的。

```php
    // 创建HTTP请求实例
    public static function capture()
    {
        static::enableHttpMethodParameterOverride();
        // 通过Symfony实例创建一个请求实例
        // 而Symfony请求实例是通过createFromGlobals()静态函数实现的
        return static::createFromBase(SymfonyRequest::createFromGlobals());
    }
    
    public static function createFromBase(SymfonyRequest $request)
    {
        if ($request instanceof static) {
            return $request;
        }

        $content = $request->content;

        $request = (new static)->duplicate(
            $request->query->all(), $request->request->all(), $request->attributes->all(),
            $request->cookies->all(), $request->files->all(), $request->server->all()
        );

        $request->content = $content;

        $request->request = $request->getInputSource();

        return $request;
    }
```
```php
    // 通过PHP全局变量创建一个新的请求实例
    public static function createFromGlobals()
    {
        $server = $_SERVER;
        if ('cli-server' === PHP_SAPI) {
            if (array_key_exists('HTTP_CONTENT_LENGTH', $_SERVER)) {
                $server['CONTENT_LENGTH'] = $_SERVER['HTTP_CONTENT_LENGTH'];
            }
            if (array_key_exists('HTTP_CONTENT_TYPE', $_SERVER)) {
                $server['CONTENT_TYPE'] = $_SERVER['HTTP_CONTENT_TYPE'];
            }
        }

        $request = self::createRequestFromFactory($_GET, $_POST, array(), $_COOKIE, $_FILES, $server);

        if (0 === strpos($request->headers->get('CONTENT_TYPE'), 'application/x-www-form-urlencoded')
            && in_array(strtoupper($request->server->get('REQUEST_METHOD', 'GET')), array('PUT', 'DELETE', 'PATCH'))
        ) {
            parse_str($request->getContent(), $data);
            $request->request = new ParameterBag($data);
        }
        return $request;
    }

    private static function createRequestFromFactory(array $query = array(), array $request = array(), array $attributes = array(), array $cookies = array(), array $files = array(), array $server = array(), $content = null)
    {
        // 如果定义了请求工厂方法，则可以将自定义的工厂方法赋值给属性$requestFactory
        // 否则通过new static来完成请求的实例。
        // new static语法是后期静态绑定
        // 参照http://php.net/manual/zh/language.oop5.late-static-bindings.php
        if (self::$requestFactory) {
            $request = call_user_func(self::$requestFactory, $query, $request, $attributes, $cookies, $files, $server, $content);

            if (!$request instanceof self) {
                throw new \LogicException('The Request factory must return an instance of Symfony\Component\HttpFoundation\Request.');
            }

            return $request;
        }

        return new static($query, $request, $attributes, $cookies, $files, $server, $content);
    }
```


# 处理请求`handle()`

完成了请求实例化自然需要对请求实例进行处理，最终返回响应。请求处理是通过`Illuminate\Foundation\Http\Kernel.php`中`handle()`进行的，处理是handle中的`sendRequestThroughRouter()`方法实现的，通过路由请求实例。而`enableHttpMethodParameterOverride()`方法会是使能请求拒绝，被使能后在请求过程中添加CSRF保护，服务端发送一个CSRF令牌给客户端，也就是一个cookie,在客户端发送POST请求需要将该令牌发送给服务端，否则拒绝处理该请求。

#### 请求前是不是也要准备一下？
直接看源码：
有6个步骤：环境检测、配置加载、异常处理、Facade注册、服务提供者注册、启动服务，通过`bootstrap()`方法完成准备工作，会调用服务容器$app实例中`bootstrapWith()`函数,进而通过`$this->make($bootstrapper)->bootstrap($this)`make方法完成每个准备类的初始化工作，然后调用准备类的`bootstrap`方法实现准备工作。

```php
    protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];
    // 请求通过中间件和路由转发
    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);
        Facade::clearResolvedInstance('request');

        $this->bootstrap();

        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
    /**
     * Bootstrap the application for HTTP requests.
     */
    public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }

    // Illuminate\Foundation\Application.php
    // 执行bootstrap类的数组
    public function bootstrapWith(array $bootstrappers)
    {
        $this->hasBeenBootstrapped = true;

        foreach ($bootstrappers as $bootstrapper) {
            $this['events']->fire('bootstrapping: '.$bootstrapper, [$this]);

            $this->make($bootstrapper)->bootstrap($this);

            $this['events']->fire('bootstrapped: '.$bootstrapper, [$this]);
        }
    }
```

* 环境检测和配置加载（这部分略过了）
* Facade注册
这个怎么理解呢，就是为了美观方便而起的别名，通过别名调用对应实例的属性和方法。这个在很多地方都用到了，比如路由，`Route::get()`以为是一个Route类，其实不然，只是通过别名实现的。
看源码是最好的老师：

```php
    // Illuminate\Foundation\Bootstrap\RegisterFacades.php
    public function bootstrap(Application $app)
    {
        Facade::clearResolvedInstances();

        Facade::setFacadeApplication($app);
        // Illuminate\Foundation\AliasLoader.php（外观自动加载类）
        // 注意与composer的自动加载类不同
        AliasLoader::getInstance($app->make('config')->get('app.aliases', []))->register();
    }
    // Illuminate\Foundation\AliasLoader.php
    // 创建别名加载的实例对象
    public static function getInstance(array $aliases = [])
    {
        if (is_null(static::$instance)) {
            return static::$instance = new static($aliases);
        }

        $aliases = array_merge(static::$instance->getAliases(), $aliases);

        static::$instance->setAliases($aliases);

        return static::$instance;
    }
    // 在自动加载栈中注册一个自动加载函数
    public function register()
    {
        if (! $this->registered) {
            $this->prependToLoaderStack();

            $this->registered = true;
        }
    }
```
主要分要两个步骤：完成外观自动加载类的实例化并将外观别名数组添加到实例中，然后完成外观自动加载类中自动加载函数的添加。

在laravel中有两个别名，一个是容器核心别名，定义在Application类中，而存储在中Application实例的$aliases属性中，另一个是外观别名，定义在app.php配置文件中，程序运行后存储在AliasLoader类实例中的$aliases属性中。

那像`Route::get()`是怎么调用的呢？程序首先需要加载类Route，注册了外观别名，那么自动加载栈的第一个函数是AliasLoader类的load()函数，此函数会查找外观别名对应的类名，也就是`Illuminate\Support\Facades\Route`类，加载这个类，执行get()方法，但是这个类中并没有此静态方法。这个类继承`Illuminate\Support\Facades\Facade`，也没有get()方法，但是有一个`__callStatic()`魔术方法，然后调用一个`getFacadeAccessor()`静态方法，每一个具体的外观类都需要这个静态方法，该方法就是返回别名类所对应的在服务容器中的名称，对于`Illuminate\Support\Facades\Route`返回的就是“router”，接着通过服务容器获取对应的实例对象，这里对应的`Illuminate\Routing\Router`类的实例，通过`static::$app[$name]`实现，最终调用这个类中的get()方法。

* 服务提供者注册
服务提供注册这位应用程序提供服务支持，在启动的准备阶段进行了基础服务提供者的加载，但这些服务职能应对前期的启动阶段，对于后期请求处理需要用到的数据库服务、认证服务、session服务还不够。

```php
    // 服务提供者注册
    public function bootstrap(Application $app)
    {
        $app->registerConfiguredProviders();
    }
    // Illuminate\Foundation\Application.php
    public function registerConfiguredProviders()
    {
        (new ProviderRepository($this, new Filesystem, $this->getCachedServicesPath()))
                    ->load($this->config['app.providers']);
    }
```

* 启动服务
服务提供者必须要实现`register()`函数，还有一个`boot()`函数根据需求实现，主要用于启动服务，不是必须的，不实现会在父类中统一处理。对于实现boot()函数的服务提供者，会通过BootProviders类进行统一管理调用。要实现boot()也比较简短，只需要调用服务容器中的boot()函数即可。

```php
    // Illuminate\Foundation\Bootstrap.php
    public function bootstrap(Application $app)
    {
        $app->boot();
    }
```


#### 中间件
在Laravel程序中有中间件的概念，就是对请求的处理，首先是经过中间的处理，然后经过路由的处理，最终到控制器生成响应。这个过程中基本是以装饰者模式的思想进行的。
看看app/Http/Kernel.php源码：

```php
    protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ];

    // Illuminate\Foundation\Http\Kernel.php
    // 将请求通过中间件和路由处理
    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);
        Facade::clearResolvedInstance('request');
        $this->bootstrap();
        // 主要看这里
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
    // 设置路由分发回调函数
    protected function dispatchToRouter()
    {
        return function ($request) {
            $this->app->instance('request', $request);
            return $this->router->dispatch($request);
        };
    }

    // Illuminate\Pipeline\Pipeline.php
    public function __construct(Container $container = null)
    {
        $this->container = $container;
    }
    // 设置被送入管道的对象
    public function send($passable)
    {
        $this->passable = $passable;
        return $this;
    }
    // 设置导管数组
    public function through($pipes)
    {
        $this->pipes = is_array($pipes) ? $pipes : func_get_args();
        return $this;
    }
    // 以一个回调函数为重点执行管道处理
    public function then(Closure $destination)
    {
        $pipeline = array_reduce(
            // 可以看看carry()
            array_reverse($this->pipes), $this->carry(), $this->prepareDestination($destination)
        );
        return $pipeline($this->passable);
    }
```

#### 路由处理生成响应

回到刚才看到的路由分发回调函数

```php
    protected function dispatchToRouter()
    {
        return function ($request) {
            $this->app->instance('request', $request);
            // 将请求信息传递给路由信息存储实例
            return $this->router->dispatch($request);
        };
    }

    // Illuminate\Routing\Router.php
    public function dispatch(Request $request)
    {
        $this->currentRequest = $request;

        return $this->dispatchToRoute($request);
    }
    // 分发请求到路由上，并返回响应
    public function dispatchToRoute(Request $request)
    {
        // 查找对应的路由实例
        $route = $this->findRoute($request);
        $request->setRouteResolver(function () use ($route) {
            return $route;
        });
        $this->events->dispatch(new Events\RouteMatched($route, $request));

        $response = $this->runRouteWithinStack($route, $request);

        return $this->prepareResponse($request, $response);
    }
    // 通过一个实例栈运行给定的路由
    protected function runRouteWithinStack(Route $route, Request $request)
    {
        $shouldSkipMiddleware = $this->container->bound('middleware.disable') &&
                                $this->container->make('middleware.disable') === true;

        $middleware = $shouldSkipMiddleware ? [] : $this->gatherRouteMiddleware($route);

        return (new Pipeline($this->container))
                        ->send($request)
                        ->through($middleware)
                        ->then(function ($request) use ($route) {
                            return $this->prepareResponse(
                                $request, $route->run()
                            );
                        });
    }
```
路由的信息都会保存在一个`Illuminate\Routing\Router`类实例中，而这个类实例存储在kernel类中。

查到请求对应的路由后，请求传递给对应的路由去处理

```php
    // Illuminate\Routing\Route.php
    // 执行路由动作并返回相应
    public function run()
    {
        $this->container = $this->container ?: new Container;
        try {
            if ($this->isControllerAction()) {
                return $this->runController();
            }
            return $this->runCallable();
        } catch (HttpResponseException $e) {
            return $e->getResponse();
        }
    }
```
进而交给相应的controller去处理。这部分，首先根据控制器类名通过服务容器进行实例化，再通过调用控制的实例对应的方法来生成响应的主体部分（并非最终的响应）。经过一系列的处理之后生成响应。响应是封装在`Illuminate\Http\Response`实例中的。

# 响应的发送和请求生命周期的终止

#### 响应发送`$response->send()`

```php
    // Symfony\Component\HttpFoundation\Response.php
    // 发送 HTTP响应头和内容
    public function send()
    {
        // 发送HTTP头部
        $this->sendHeaders();
        // 发送Web响应的内容
        $this->sendContent();

        if (function_exists('fastcgi_finish_request')) {
            fastcgi_finish_request();
        } elseif ('cli' !== PHP_SAPI) {
            static::closeOutputBuffers(0, true);
        }
        return $this;
    }
```

#### 程序终止——生命周期的最后阶段`$kernel->terminate($request, $response)`
程序终止，完成终止中间件的调用

```php
    // Illuminate\Foundation\Http\Kernel.php
    public function terminate($request, $response)
    {
        $this->terminateMiddleware($request, $response);

        $this->app->terminate();
    }
    // 终止中间件
    protected function terminateMiddleware($request, $response)
    {
        $middlewares = $this->app->shouldSkipMiddleware() ? [] : array_merge(
            $this->gatherRouteMiddleware($request),
            $this->middleware
        );

        foreach ($middlewares as $middleware) {
            if (! is_string($middleware)) {
                continue;
            }
            list($name, $parameters) = $this->parseMiddleware($middleware);

            $instance = $this->app->make($name);

            if (method_exists($instance, 'terminate')) {
                $instance->terminate($request, $response);
            }
        }
    }
```
什么东西都是有始有终的，至此请求生命周期结束。


# 总结
> 请求到响应整个执行过程，可以分为四个阶段：程序启动准备阶段，请求实例化阶段，请求处理阶段，响应发送和终止程序。
> 准备阶段：完成一些文件自动加载，服务容器实例化，基础服务提供者注册和Kernel类的实例化。
> 请求实例化阶段：将请求信息以对象的实行进行存储。
> 请求处理阶段：准备请求处理环境，完成环境和配置加载等6大东西。通过中间件处理通过路由和控制器处理，生成相应。
> 请求终止阶段：将响应发送给客户端并终止程序。



