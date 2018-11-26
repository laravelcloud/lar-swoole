### 1. lar-swoole
laravel swoole集成解决方案,  目前推荐已有的库实现(`swooletw/laravel-swoole`)

### 2.3 laravel-swoole 扩展安装

以下是 swooletw/laravel-swoole 的主要特点：

* 在 Swoole 运行 Laravel/Lumen 应用
* 出色的性能提升至 30x
* 沙盒模式隔离应用程序容器
* 支持在 Laravel 应用中运行 WebSocket 服务器
* 支持 Socket.io 协议
* 支持 Swoole 表跨进程共享

使用 Composer 安装：

```bash
$ composer require swooletw/laravel-swoole
```

### 2.4 laravel/lumen配置

> 这个包支持包自动发现机制。如果你运行 Laravel 5.5 以上版本，你可以跳过这一步。

**laravel配置**: 在 config/app.php 服务提供者数组添加该服务提供者
```php
[
    'providers' => [
        SwooleTW\Http\LaravelServiceProvider::class,
    ],
]
```
 
**lumen配置**: 请将下面的代码添加到 bootstrap/app.php
```php
$app->register(SwooleTW\Http\LumenServiceProvider::class);
```


## 3. 基准测试数据

### 3.1 建立并运行起来

现在，你可以执行以下的命令来启动 Swoole HTTP 服务。
```bash
php artisan swoole:http start
```
然后你可以看到以下信息：


> Starting swoole http server...
> Swoole http server started: <http://127.0.0.1:1215>

现在可以通过访问 `http://127.0.0.1:1215` 来进入 Laravel 应用。

如果需要修改端口号或服务地址, 可配置相应的环境变量

```
// vendor/swooletw/laravel-swoole/config/swoole_http.php
SWOOLE_HTTP_HOST: '127.0.0.1'
SWOOLE_HTTP_PORT: '1215'
```
详细的文档参考: [https://wiki.swoole.com/wiki/page/14.html](https://wiki.swoole.com/wiki/page/14.html)

### 3.2 基于 FPM + Nginx 的测试结果
```bash
wrk -t4 -c100 http://your.domain.com/version

Running 10s test @ http://your.domain.com/version
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   341.64ms  108.70ms 801.47ms   69.08%
    Req/Sec    71.72     27.35   171.00     65.57%
  2864 requests in 10.03s, 2.84MB read
Requests/sec:    285.63
Transfer/sec:    289.79KB
```

```bash
wrk -t12 -c400 -d30s http://your.domain.com/version

Running 30s test @ http://your.domain.com/version
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   801.35ms  349.89ms   2.00s    68.56%
    Req/Sec    40.61     19.83   126.00     65.80%
  14390 requests in 30.10s, 14.24MB read
  Socket errors: connect 0, read 0, write 0, timeout 132
Requests/sec:    478.09
Transfer/sec:    484.34KB
```

### 3.3 Swoole HTTP 服务的测试结果

```bash
wrk -t4 -c100 http://your.domain.com/version

Running 10s test @ http://your.domain.com/version
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   198.64ms  324.54ms   1.96s    88.59%
    Req/Sec   225.62     91.30   430.00     62.72%
  9021 requests in 10.09s, 7.90MB read
  Socket errors: connect 0, read 0, write 0, timeout 25
Requests/sec:    893.71
Transfer/sec:    801.26KB
```

```bash
wrk -t12 -c400 -d30s http://your.domain.com/version

Running 30s test @ http://your.domain.com/version
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   319.84ms  398.59ms   2.00s    85.59%
    Req/Sec    66.94     35.58   240.00     66.09%
  23862 requests in 30.09s, 20.89MB read
  Socket errors: connect 0, read 0, write 0, timeout 619
Requests/sec:    793.04
Transfer/sec:    711.05KB
```

## 4. 问题&注意事项

* php7只能用swoole 4.0+ 版本

### 4.1 静态文件使用swoole性能是否受到影响?

使用Nginx来代理运行于Swoole上的Laravel

```nginx
server {
    listen 80;
    server_name your.domain.com;
    root /path/to/laravel/public;
    index index.php;
    location = /index.php {
        # Ensure that there is no such file named "not_exists"
        # in your "public" directory.
        try_files /not_exists @swoole;
    }
    location / {
        try_files $uri $uri/ @swoole;
    }
    location @swoole {
        set $suffix "";
        if ($uri = /index.php) {
            set $suffix "/";
        }
        proxy_set_header Host $host;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        # IF https
        # proxy_set_header HTTPS "on";
        proxy_pass http://127.0.0.1:1215$suffix;
    }
}
```


## 5. 参考
* [使用 Swoole 来加速你的 Laravel 应用](https://laravel-china.org/topics/10939/use-swoole-to-speed-up-your-laravel-application)
* [swoole 入门指引](https://wiki.swoole.com/wiki/page/1.html)
* [laravel-swoole](https://github.com/swooletw/laravel-swoole)



