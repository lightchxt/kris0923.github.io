---
title: "Laravel中间件实现分析"
date: 2019-09-26T21:05:04+08:00
draft: false
categories: ["Laravel"]
tags: ["Laravel"]
---

Laravel的中间件提供了一种方便的机制过滤进入应用程序的 HTTP 请求，并且Laravel自带了一些中间件包括身份验证、CSRF保护、COOKIE加密解密等，使用非常方便。那么Laravel中“中间件”是怎么实现的，为什么只需要简单修改几个配置就能试中间件生效，中间件的执行顺序又是怎样的呢。出于对这些问题的好奇，我阅读了一下Laravel中间件的实现相关源码。

> **\Illuminate\Pipeline\Pipeline** - 实现中间件的核心类

在介绍Pipeline这个类之前，我们先简单浏览一下Laravel应用程序的[生命周期](https://learnku.com/docs/laravel/5.8/lifecycle/3885#lifecycle-overview)。接下里我一步一步的说明如何实例化Pipeline方法的。

1. 在入口文件index.php只有几行代码
```php
$app = require_once __DIR__.'/../bootstrap/app.php';

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

$response->send();
$kernel->terminate($request, $response);
```
首先实例化了一个Illuminate\Contracts\Http\Kernel，在bootstrap/app.php文件中，
Illuminate\Contracts\Http\Kernel接口已经被绑定了App\Http\Kernel类，所以这里的$kernel是一个App\Http\Kernel的对象

bootstrap/app.php
```
// 此处对Illuminate\Contracts\Http\Kernel进行类绑定
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

return $app;
```

然后接着往下看，调用了Kernel的handel方法，并传入了一个Request对象。

```
class Kernel implements KernelContract
...
/**
     * Handle an incoming HTTP request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();

            $response = $this->sendRequestThroughRouter($request);  // sendRequestThroughRouter 方里面实例化了Pipeline
        } catch (Exception $e) {
            $this->reportException($e);

            $response = $this->renderException($request, $e);
        } catch (Throwable $e) {
            $this->reportException($e = new FatalThrowableError($e));

            $response = $this->renderException($request, $e);
        }

        $this->app['events']->dispatch(
            new Events\RequestHandled($request, $response)
        );

        return $response;
    }
    
    /**
     * Send the given request through the middleware / router.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);

        Facade::clearResolvedInstance('request');

        $this->bootstrap();
    ## 这里实例化了Pipeline
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
...
```
接下来就到中间件的核心了- Pipeline->then()方法
```
public function then(Closure $destination)
    {
        $pipeline = array_reduce(
            array_reverse($this->pipes), $this->carry(), $this->prepareDestination($destination)
        );

        return $pipeline($this->passable);
    }
```
最关键的就是array_reduce()，函数原型：array_reduce ( array $array , callable $callback [, mixed $initial = NULL ] ) : mixed
>  array_reduce() 将回调函数 callback 迭代地作用到 array 数组中的每一个单元中，从而将数组简化为单一的值。
一般情况下，我们第二个参数返回的是一个基本的数据类型，比如：
```
$arr = [1, 2, 3, 4, 5];
$sum = array_reduce($arr, function($carry, $item) {
    return $carry + $item;
}, 0);
echo $sum;
// 结果为 15
```

那么如果我们第二个参数放回一个闭包函数会是什么样呢？

```
$arr = ["item1", "item2", "item3"];
$closure = array_reduce($arr, function ($carry, $item) {
    return function () use ($carry, $item) {
        dump($item);
        if (is_null($carry)) {
            return "carry is  null " . $item;
        }
        if($carry instanceof Closure) {
            dump($carry());
            return $item . '++';
        }
    };
});
dump("r:");
dump($closure());

```

结果为：

"r:"

"item3"

"item2"

"item1"

"carry is  null item1"

"item2++"

"item3++"

为什么是这个结果呢？array_reduce返回的是一个闭包，只有执行这个闭包函数，里面的逻辑才会执行，所以相当于是把函数暂存起来了，类似于存到了一个栈中。

理解了array_reduce,然后我们看一下Laravel中carry()函数每次都返回了什么，打印一下每次调用的的 pipe 和 method:

![](https://user-gold-cdn.xitu.io/2019/9/17/16d3ebcf37b2440b?w=945&h=242&f=png&s=25701)

输出：

![](https://user-gold-cdn.xitu.io/2019/9/17/16d3ebd35f39a95f?w=675&h=436&f=png&s=25385)

所有carry方法每次都返回了我们配置了的中间件的handle方法，而$pipeline 就是一个由众多handle方法组成的闭包函数，然后执行 
```
return $pipeline($this->passable);
```
对当前的Request进行过滤。因为后从carry()中返回的方法会被先执行，所以 array_reverse($this->pipes) 对我们的中间件顺序进行了翻转，这样执行的顺序就和我们配置中间件的顺序一致了

> 以上就是 Laravel 中间件的实现了

