# 起因
突然很好奇 Laravel/Lumen 是如何传递中间件的参数，而参数又是如何被解析的？

# 如何传递参数？
```php
    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        ...
    ];
```

以最简单的 auth 中间件为例子，我们可以看到 Authenticate 该类的 handle 方法 源代码, 是可以接收可变参数 $guards
```php
    public function handle($request, Closure $next, ...$guards)
    {
        $this->authenticate($guards);

        return $next($request);
    }
```

其实我们这么传递就可以了, auth 是中间件，以 : 分割中间件和参数，参数之间以逗号分割
```php
    Route::prefix('api/resource')
        //->middleware('jwt.auth')
        ->middleware(['auth:abc,def'])
```

# 源码解析，为什么这么传？
1. 在 Illuminate\Foundation\Http\Kernel::class 的 handle 方法为解析请求，返回响应
```php
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

        $this->app['events']->dispatch(
            new Events\RequestHandled($request, $response)
        );

        return $response;
    }
```

其中与中间件验证有关的都在 sendRequestThroughRouter 方法中

2. sendRequestThroughRouter方法中的中间件解析
```php
    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);

        Facade::clearResolvedInstance('request');
        // 首先为整个应用初始化，做一些环境变量加载，配置加载，错误处理，别名机制注册，服务注册，启动服务的工作
        $this->bootstrap();

        /*
        1. Pipeline 其实就是一个装饰者模式，提供链式调用，暂时不分析
        2. $this->app->shouldSkipMiddleware() 查看应用是否设置了可以忽略中间件检查
        3. 如果没有忽略，走 $this->middleware， 这是全局中间件，是所有的请求都要验证的中间件，无论 web/api 请求
        4. 最重要的来了，当我们通过全局中间件，那么接下来对具体的请求配置的中间件做检验 $this->dispatchToRouter()

        */
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
```

3. 路由器分发请求, 之后的源码就进入了 Illuminate\Routing\Router
```php
    protected function dispatchToRouter()
    {
        return function ($request) {
            $this->app->instance('request', $request);

            return $this->router->dispatch($request);
        };
    }
```

4. 解析 $request, 主要为了分析 runRouteWithinStack() 函数
```php
    public function dispatch(Request $request)
    {
        $this->currentRequest = $request;

        return $this->dispatchToRoute($request);
    }

    public function dispatchToRoute(Request $request)
    {
        return $this->runRoute($request, $this->findRoute($request));
    }

    protected function runRoute(Request $request, Route $route)
    {
        $request->setRouteResolver(function () use ($route) {
            return $route;
        });

        $this->events->dispatch(new Events\RouteMatched($route, $request));

        return $this->prepareResponse($request,
            $this->runRouteWithinStack($route, $request)
        );
    }

        protected function runRouteWithinStack(Route $route, Request $request)
    {
        $shouldSkipMiddleware = $this->container->bound('middleware.disable') && $this->container->make('middleware.disable') === true;

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

5. Illuminate\Pipeline\Pipeline 的 carry() 方法
```php
    protected function carry()
    {
        return function ($stack, $pipe) {
            return function ($passable) use ($stack, $pipe) {
                if (is_callable($pipe)) {
                    // If the pipe is an instance of a Closure, we will just call it directly but
                    // otherwise we'll resolve the pipes out of the container and call it with
                    // the appropriate method and arguments, returning the results back out.
                    return $pipe($passable, $stack);
                } elseif (! is_object($pipe)) {
                    [$name, $parameters] = $this->parsePipeString($pipe);

                    // If the pipe is a string we will parse the string and resolve the class out
                    // of the dependency injection container. We can then build a callable and
                    // execute the pipe function giving in the parameters that are required.
                    $pipe = $this->getContainer()->make($name);

                    $parameters = array_merge([$passable, $stack], $parameters);
                } else {
                    // If the pipe is already an object we'll just make a callable and pass it to
                    // the pipe as-is. There is no need to do any extra parsing and formatting
                    // since the object we're given was already a fully instantiated object.
                    $parameters = [$passable, $stack];
                }

                $response = method_exists($pipe, $this->method)
                                ? $pipe->{$this->method}(...$parameters)
                                : $pipe(...$parameters);

                return $response instanceof Responsable
                            ? $response->toResponse($this->container->make(Request::class))
                            : $response;
            };
        };
    }
```

parsePipeString() 解析中间件, 到了这里，我们终于看到如何解析中间件和参数了

```php
    protected function parsePipeString($pipe)
    {
        [$name, $parameters] = array_pad(explode(':', $pipe, 2), 2, []);

        if (is_string($parameters)) {
            $parameters = explode(',', $parameters);
        }

        return [$name, $parameters];
    }
```