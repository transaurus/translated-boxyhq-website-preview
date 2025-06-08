---
title: Add SAML SSO to Laravel App with BoxyHQ
description: Add SAML SSO to Laravel App with BoxyHQ
sidebar_label: Laravel
---

# 为 Laravel 应用添加 SAML 单点登录

本指南假设您已有一个 Laravel 应用，并希望为企业客户启用 SAML 单点登录认证。完成本指南后，您的应用将能够通过 SAML 单点登录对用户进行身份验证。

访问 [GitHub 仓库](https://github.com/boxyhq/php-examples/tree/main/laravel-enterprise-sso) 查看 Laravel SAML 单点登录集成的源代码。

为应用集成 SAML 单点登录需要完成以下步骤。

- 配置 SAML 单点登录
- 通过 SAML 单点登录进行认证

## 配置 SAML 单点登录

此步骤允许您的租户为其用户配置 SAML 连接。阅读以下指南以了解更多信息。

- [配置 SAML 单点登录的 UI 最佳实践](/guides/jackson/configuring-saml-sso)
- [单点登录连接 API](/docs/jackson/sso-flow/)

## 通过 SAML 单点登录进行认证

添加 SAML 连接后，应用可使用该连接通过 SAML Jackson 发起单点登录认证流程。以下部分主要关注单点登录认证环节。

### 安装 SAML Jackson

第一步是部署 SAML Jackson 服务。请按照 [部署文档](/docs/jackson/deploy/service) 进行安装和配置。

### 设置 SAML Jackson

我们将使用 Laravel Socialite 进行集成。Socialite 为外部认证提供商的 OAuth 认证提供了简洁流畅的接口。

创建新的配置文件来保存 SAML Jackson 的配置值。

```php title="config/jackson.php"
<?php

return [
  'host' => 'http://localhost:5225', // SAML Jackson service URL
  'product' => 'your-app-name',
  'client_id' => 'dummy', // Keep this as `dummy`, we'll pass the tenant & product as dynamic params
  'client_secret' => 'dummy', // Keep this as `dummy`, we'll pass the tenant & product as dynamic params
  'redirect' => env('APP_URL') . '/sso/callback'
];
```

将 `host` 设置为运行中的 SAML Jackson 服务 URL。

让我们为 SAML Jackson 向 Laravel Socialite 添加一个自定义提供商。

```php title="app/BoxyHQ/JacksonProvider.php"
<?php

namespace App\BoxyHQ;

use Laravel\Socialite\Two\AbstractProvider;
use Laravel\Socialite\Two\ProviderInterface;
use Laravel\Socialite\Two\User;
use GuzzleHttp\RequestOptions;

class JacksonProvider extends AbstractProvider implements ProviderInterface
{
  /**
   * The SAML Jackson instance host.
   *
   * @var string
   */
  protected $host;

  /**
   * Set the SAML Jackson instance host.
   *
   * @param  string|null  $host
   * @return $this
   */
  public function setHost($host)
  {
    if (! empty($host)) {
      $this->host = rtrim($host, '/');
    }

    return $this;
  }

  protected function getAuthUrl($state)
  {
    return $this->buildAuthUrlFromBase($this->host . '/api/oauth/authorize', $state);
  }

  protected function getTokenUrl()
  {
    return $this->host . '/api/oauth/token';
  }

  protected function getUserByToken($token)
  {
    $response = $this->getHttpClient()->get($this->host . '/api/oauth/userinfo', [
      RequestOptions::QUERY => ['access_token' => $token],
    ]);

    return json_decode($response->getBody(), true);
  }

  protected function mapUserToObject(array $user)
  {
    return (new User)->setRaw($user)->map([
      'id' => $user['id'],
      'email' => $user['email'],
      'name' => $user['firstName'].' '.$user['lastName'],
      'first_name' => $user['firstName'],
      'last_name' => $user['lastName'],
      'requested' => $user['requested'],
      'nickname' => null,
      'avatar' => null,
    ]);
  }
}
```

在 `AppServiceProvider` 中初始化 `JacksonProvider`。

```php title="app/Providers/AppServiceProvider.php"
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\BoxyHQ\JacksonProvider;

class AppServiceProvider extends ServiceProvider
{
  /**
   * Register any application services.
   *
   * @return void
   */
  public function register()
  {
    //
  }

  /**
   * Bootstrap any application services.
   *
   * @return void
   */
  public function boot()
  {
    $socialite = $this->app->make('Laravel\Socialite\Contracts\Factory');

    $socialite->extend(
      'jackson',
      function ($app) use ($socialite) {
        $config = config('jackson');

        return $socialite->buildProvider(JacksonProvider::class, $config)
          ->setHost($config['host'] ?? null);
      }
    );
  }
}
```

### 发起认证请求

添加路由以开始认证流程；该路由通过将用户重定向至其配置的身份提供商来发起 SAML 单点登录流程。

```php title="routes/web.php"
<?php

use Illuminate\Support\Facades\Route;

Route::post('/sso', '\App\Http\Controllers\AuthController@store');
```

`AuthController` 的 `store` 方法负责将用户重定向至身份提供商。

```php title="app/Http/Controllers/AuthController.php"
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Laravel\Socialite\Facades\Socialite;

class AuthController extends Controller
{
  public function store(Request $request)
  {
    $jackson = config('jackson');

    $tenant = $request->input('tenant');

    return Socialite::driver('jackson')
      ->with(['tenant' => $tenant, 'product' => $jackson['product']])
      ->redirect();
  }
}
```

### 获取用户资料

添加另一个路由以接收认证后的回调。确保该路由与先前配置的 `redirect` 值匹配。

```php title="routes/web.php"
<?php

Route::get('/sso/callback', '\App\Http\Controllers\AuthController@callback');
```

若授权有效，`AuthController` 的 `callback` 方法负责获取用户资料。

```php title="app/Http/Controllers/AuthController.php"
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Laravel\Socialite\Facades\Socialite;

class AuthController extends Controller
{
  public function callback(Request $request)
  {
    $user = Socialite::driver('jackson')->user();

    // $user has all the properties you need. Do your business logic here.
  }
}
```

### 认证用户

从身份提供商获取用户信息后，您可判断该用户是否存在于您的应用中并进行认证。如果用户不存在，通常需要在数据库中创建新记录来代表该用户。