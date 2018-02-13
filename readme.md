## Laravel Debugbar
[![Packagist License](https://poser.pugx.org/barryvdh/laravel-debugbar/license.png)](http://choosealicense.com/licenses/mit/)
[![Latest Stable Version](https://poser.pugx.org/barryvdh/laravel-debugbar/version.png)](https://packagist.org/packages/barryvdh/laravel-debugbar)
[![Total Downloads](https://poser.pugx.org/barryvdh/laravel-debugbar/d/total.png)](https://packagist.org/packages/barryvdh/laravel-debugbar)

### Lưu ý cho v3: Debugbar bây giờ được kích hoạt bằng cách yêu cầu gói, nhưng vẫn cần APP_DEBUG = true như thường!

### Đối với Laravel < 5.5, vui lòng sử dụng [2.4 branch](https://github.com/barryvdh/laravel-debugbar/tree/2.4)!

Đây là một gói đẻ tích hợp [PHP Debug Bar](http://phpdebugbar.com/) với Laravel 5.
Nó bao gồm một ServiceProvider để đăng kí debugbar và đính kèm nó tới đầu ra. Bạn có thể công bố assets và cấu hình nó thông qua Laravel.
It bootstraps some Collectors to work with Laravel and implements a couple custom DataCollectors, specific for Laravel.
Nó được cấu hình để hiển thị các yêu cầu Redirects và (jQuery) Ajax. (Hiển thị trong một dropdown)
Đọc [the documentation](http://phpdebugbar.com/docs/) để biết thêm các tuỳ chọn cấu hình.

![Screenshot](https://cloud.githubusercontent.com/assets/973269/4270452/740c8c8c-3ccb-11e4-8d9a-5a9e64f19351.png)

Lưu ý: Chỉ sử dụng DebugBar trong khi phát triển. Nó có thể làm chậm application (bởi vì nó phải thu thập dữ liệu). Vì vậy, khi gặp tình trạng chậm, thử vô hiệu hoá một vài collectors.

Gói này bao gồm một vài collectors tuỳ chọn:
 - QueryCollector: Hiện tất cả các truy vấn, bao gồm cả binding + timing
 - RouteCollector: Hiện thông tin về Route hiện tại.
 - ViewCollector: Hiện các view hiện được tải. (Tuỳ chọn: hiện dữ liệu được chia sẻ)
 - EventsCollector: Hiện tất cả các sự kiện
 - LaravelCollector:Hiện phiên bản vả môi trường Laravel. (Mặc định là vô hiệu)
 - SymfonyRequestCollector: Thay thế cho RequestCollector với nhiều thông tin hơn về request/response
 - LogsCollector:Hiện mục nhật kí mới nhất từ các nhật kí lưu trữ. (Mặc định là vô hiệu)
 - FilesCollector: Hiện các tệp được included/required bởi PHP. (Mặc định là vô hiệu)
 - ConfigCollector: Hiện các giá trị từ các tệp config. (Mặc định là vô hiệu)
 - CacheCollector: Hiển thị toàn bộ những sự kiện bộ nhớ đệm. (Mặc định là vô hiệu)

Bootstraps the following collectors for Laravel:
 - LogCollector: Show all Log messages
 - SwiftMailCollector and SwiftLogCollector for Mail

And the default collectors:
 - PhpInfoCollector
 - MessagesCollector
 - TimeDataCollector (With Booting and Application timing)
 - MemoryCollector
 - ExceptionsCollector

It also provides a Facade interface for easy logging Messages, Exceptions and Time

## Installation

Require this package with composer. It is recommended to only require the package for development.

```shell
composer require barryvdh/laravel-debugbar --dev
```

Laravel 5.5 uses Package Auto-Discovery, so doesn't require you to manually add the ServiceProvider.

The Debugbar will be enabled when `APP_DEBUG` is `true`.

> If you use a catch-all/fallback route, make sure you load the Debugbar ServiceProvider before your own App ServiceProviders.

### Laravel 5.5+:

If you don't use auto-discovery, add the ServiceProvider to the providers array in config/app.php

```php
Barryvdh\Debugbar\ServiceProvider::class,
```

If you want to use the facade to log messages, add this to your facades in app.php:

```php
'Debugbar' => Barryvdh\Debugbar\Facade::class,
```

The profiler is enabled by default, if you have APP_DEBUG=true. You can override that in the config (`debugbar.enabled`) or by setting `DEBUGBAR_ENABLED` in your `.env`. See more options in `config/debugbar.php`
You can also set in your config if you want to include/exclude the vendor files also (FontAwesome, Highlight.js and jQuery). If you already use them in your site, set it to false.
You can also only display the js or css vendors, by setting it to 'js' or 'css'. (Highlight.js requires both css + js, so set to `true` for syntax highlighting)

Copy the package config to your local config with the publish command:

```shell
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

### Lumen:

For Lumen, register a different Provider in `bootstrap/app.php`:

```php
if (env('APP_DEBUG')) {
 $app->register(Barryvdh\Debugbar\LumenServiceProvider::class);
}
```

To change the configuration, copy the file to your config folder and enable it:

```php
$app->configure('debugbar');
```

## Usage

You can now add messages using the Facade (when added), using the PSR-3 levels (debug, info, notice, warning, error, critical, alert, emergency):

```php
Debugbar::info($object);
Debugbar::error('Error!');
Debugbar::warning('Watch out…');
Debugbar::addMessage('Another message', 'mylabel');
```

And start/stop timing:

```php
Debugbar::startMeasure('render','Time for rendering');
Debugbar::stopMeasure('render');
Debugbar::addMeasure('now', LARAVEL_START, microtime(true));
Debugbar::measure('My long operation', function() {
    // Do something…
});
```

Or log exceptions:

```php
try {
    throw new Exception('foobar');
} catch (Exception $e) {
    Debugbar::addThrowable($e);
}
```

There are also helper functions available for the most common calls:

```php
// All arguments will be dumped as a debug message
debug($var1, $someString, $intValue, $object);

start_measure('render','Time for rendering');
stop_measure('render');
add_measure('now', LARAVEL_START, microtime(true));
measure('My long operation', function() {
    // Do something…
});
```

If you want you can add your own DataCollectors, through the Container or the Facade:

```php
Debugbar::addCollector(new DebugBar\DataCollector\MessagesCollector('my_messages'));
//Or via the App container:
$debugbar = App::make('debugbar');
$debugbar->addCollector(new DebugBar\DataCollector\MessagesCollector('my_messages'));
```

By default, the Debugbar is injected just before `</body>`. If you want to inject the Debugbar yourself,
set the config option 'inject' to false and use the renderer yourself and follow http://phpdebugbar.com/docs/rendering.html

```php
$renderer = Debugbar::getJavascriptRenderer();
```

Note: Not using the auto-inject, will disable the Request information, because that is added After the response.
You can add the default_request datacollector in the config as alternative.

## Enabling/Disabling on run time
You can enable or disable the debugbar during run time.

```php
\Debugbar::enable();
\Debugbar::disable();
```

NB. Once enabled, the collectors are added (and could produce extra overhead), so if you want to use the debugbar in production, disable in the config and only enable when needed.


## Twig Integration

Laravel Debugbar comes with two Twig Extensions. These are tested with [rcrowe/TwigBridge](https://github.com/rcrowe/TwigBridge) 0.6.x

Add the following extensions to your TwigBridge config/extensions.php (or register the extensions manually)

```php
'Barryvdh\Debugbar\Twig\Extension\Debug',
'Barryvdh\Debugbar\Twig\Extension\Dump',
'Barryvdh\Debugbar\Twig\Extension\Stopwatch',
```

The Dump extension will replace the [dump function](http://twig.sensiolabs.org/doc/functions/dump.html) to output variables using the DataFormatter. The Debug extension adds a `debug()` function which passes variables to the Message Collector,
instead of showing it directly in the template. It dumps the arguments, or when empty; all context variables.

```twig
{{ debug() }}
{{ debug(user, categories) }}
```

The Stopwatch extension adds a [stopwatch tag](http://symfony.com/blog/new-in-symfony-2-4-a-stopwatch-tag-for-twig)  similar to the one in Symfony/Silex Twigbridge.

```twig
{% stopwatch "foo" %}
    …some things that gets timed
{% endstopwatch %}
```
