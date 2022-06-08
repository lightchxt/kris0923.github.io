---
title: Laravel Facades 门面模式的实现
date: 2019-07-17 23:08:42
draft: true
categories: ["php", "laravel"]
tags: ["php", "laravel"]
---
> 以下是Laravel官方文档的介绍

Facades 为应用程序的 服务容器 中可用的类提供了一个「静态」接口。Laravel 本身附带许多的 facades，甚至你可能在不知情的状况下已经在使用他们！Laravel 「facades」作为在服务容器内基类的「静态代理」，拥有简洁、易表达的语法优点，同时维持着比传统静态方法更高的可测试性和灵活性。

从介绍中可以看出，Facades 好处就是让代码更加简介，优雅，这也是Laravel追求的特性，如何使用Facades这里就不介绍了，可以参考[Laravel文档(中文)](https://learnku.com/docs/laravel/5.2/facades/1111) ，本文介绍一下Facades是如何知道和创建你需要的类实例。

以 log Facade 为例，我们看下是如何通过log这个字符串找到 \Illuminate\Log\Writer 这个类的
先看 \Illuminate\Support\Facades\Log 门面
```
class Log extends Facade
{
    /**
     * Get the registered name of the component.
     *
     * @return string
     */
    protected static function getFacadeAccessor()
    {
        return 'log';
    }
}
```
这个类非常简单，只有一个静态方法 getFacadeAccessor(), 返回了一个log字符串。
然后看Log的父类 Facade, Facede中有很多方法，这里关注其中两个：
```
abstract class Facade
{
    /**
     * The application instance being facaded.
     *
     * @var \Illuminate\Contracts\Foundation\Application
     */
    protected static $app;

    /**
     * The resolved object instances.
     *
     * @var array
     */
    protected static $resolvedInstance;
	
	/**
     * Resolve the facade root instance from the container.
     *
     * @param  string|object  $name
     * @return mixed
     */
    protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {				// 如果$name已经是一个对象，则直接返回该对象
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {				// 如果是已经解析过的对象，直接从$resolvedInstance中返回该对象
            return static::$resolvedInstance[$name];
        }

        return static::$resolvedInstance[$name] = static::$app[$name];  	// 从容器中寻找$name对象，并放入$resolvedInstance 中以便下次使用
    }
	/**
     * Handle dynamic, static calls to the object.
     *
     * @param  string  $method
     * @param  array   $args
     * @return mixed
     *
     * @throws \RuntimeException
     */
    public static function __callStatic($method, $args)			// 魔术方法，当使用Log::error($msg) 的时候会调用该方法
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        switch (count($args)) {
            case 0:
                return $instance->$method();
            case 1:
                return $instance->$method($args[0]);
            case 2:
                return $instance->$method($args[0], $args[1]);
            case 3:
                return $instance->$method($args[0], $args[1], $args[2]);
            case 4:
                return $instance->$method($args[0], $args[1], $args[2], $args[3]);
            default:
                return call_user_func_array([$instance, $method], $args);
        }
    }
}
```
通过以上分析知道最终Facede找的是容器中绑定的实例，所以接下来我们找一下log是在什么时候被注册的，

这时候需要关注 \Illuminate\Foundation\Http\Kernel 类,Kernel类中包括以下几个方法：
```
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

		$response = $this->sendRequestThroughRouter($request);
	} catch (Exception $e) {
		$this->reportException($e);

		$response = $this->renderException($request, $e);
	} catch (Throwable $e) {
		$this->reportException($e = new FatalThrowableError($e));

		$response = $this->renderException($request, $e);
	}

	$this->app['events']->fire('kernel.handled', [$request, $response]);

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

	return (new Pipeline($this->app))
		->send($request)
		->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
		->then($this->dispatchToRouter());
}

/**
* Bootstrap the application for HTTP requests.
*
* @return void
*/
public function bootstrap()
{
	if (! $this->app->hasBeenBootstrapped()) {
		$this->app->bootstrapWith($this->bootstrappers());
	}
}
```
handle方法接收一个Request请求，并返回一个$response，$response->sendRequestThroughRouter()的时候调用了bootstrap()方法，继续看bootstrap方法里面加载了已经定义好的几个类：(这些定义都在Kernel类中)
```
/**
* The bootstrap classes for the application.
*
* @var array
*/
protected $bootstrappers = [
	'Illuminate\Foundation\Bootstrap\DetectEnvironment',
	'Illuminate\Foundation\Bootstrap\LoadConfiguration',
	'Illuminate\Foundation\Bootstrap\ConfigureLogging',
	'Illuminate\Foundation\Bootstrap\HandleExceptions',
	'Illuminate\Foundation\Bootstrap\RegisterFacades',
	'Illuminate\Foundation\Bootstrap\RegisterProviders',
	'Illuminate\Foundation\Bootstrap\BootProviders',
];
```

然后我们看ConfigureLogging中的registerLogger()方法
```
/**
* Register the logger instance in the container.
*
* @param  \Illuminate\Contracts\Foundation\Application  $app
* @return \Illuminate\Log\Writer
*/
protected function registerLogger(Application $app)
{
	$app->instance('log', $log = new Writer(			 // 这里把Writer注册到了容器中
		new Monolog($app->environment()), $app['events'])
	);

	return $log;
}
```
到此为止，我们已经知道Facede是如何找到想要使用的类了。
Facedes看起来挺高大上，但实现起来的原理挺简单的，实际上也是一种单例模式，只不过在调用处包装了一下