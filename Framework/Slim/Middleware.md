
## Slim 中间件

你可以在你的 Slim 应用之前（before） 和 之后（after） 运行代码来处理你认为合适的请求和响应对象。这就叫做中间件。为什么要这么做呢？比如你想保护你的应用不遭受跨站请求伪造。也许你想在应用程序运行前验证请求。中间件对这些场景的处理简直完美。

什么是中间件？
从技术上来讲，中间件是一个接收三个参数的可回调（callable）对象：

\Psr\Http\Message\ServerRequestInterface - PSR7 请求对象
\Psr\Http\Message\ResponseInterface - PSR7 响应对象
callable - 下一层中间件的回调对象
它可以做任何与这些对象相应的事情。唯一硬性要求就是中间件必须返回一个 \Psr\Http\Message\ResponseInterface 的实例。 每个中间件都 应当调用下一层中间件，并讲请求和响应对象作为参数传递给它（下一层中间件）。

中间件是如何工作的？
不同的框架使用中间件的方式不同。在 Slim 框架中，中间件层以同心圆的方式包裹着核心应用。由于新增的中间件层总会包裹所有已经存在的中间件层。当添加更多的中间件层时同心圆结构会不断的向外扩展。

当 Slim 应用运行时，请求对象和响应对象从外到内穿过中间件结构。它们首先进入最外层的中间件，然后然后进入下一层，（以此类推）。直到最后它们到达了 Slim 应用程序自身。在 Slim 应用分派了对应的路由后，作为结果的响应对象离开 Slim 应用，然后从内到外穿过中间件结构。最终，最后出来的响应对象离开了最外层的中间件，被序列化为一个原始的 HTTP 响应消息，并返回给 HTTP 客户端。下图清楚的说明了中间件的工作流程：

<img src="https://github.com/emanci/Bookmark/blob/master/middleware.png" width = "450" alt="material-theme" align=center />

我们可以使用下面这段简单的代码来理解 Slim 中间件的执行流程：

```php

require __DIR__.'/vendor/autoload.php';

$app = new \Slim\App();

$app->add(function ($request, $response, $next) {
    $response->getBody()->write('a1->');
    $response = $next($request, $response);
    $response->getBody()->write('a2->');

    return $response;
});

$app->get('/', function ($req, $res, $args) {
    echo '[App]->';
});

$app->add(function ($request, $response, $next) {
    $response->getBody()->write('b1->');
    $response = $next($request, $response);
    $response->getBody()->write('b2->');

    return $response;
});

$app->add(function ($request, $response, $next) {
    $response->getBody()->write('c1->');
    $response = $next($request, $response);
    $response->getBody()->write('c2');

    return $response;
});

$app->run();

执行流程：
// c1->b1->a1->[App]->a2->b2->c2

```

参考：
http://slimphp.net/docs/concepts/middleware.html
