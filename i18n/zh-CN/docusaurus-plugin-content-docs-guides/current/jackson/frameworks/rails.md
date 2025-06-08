---
title: Add SAML SSO to Ruby on Rails App with BoxyHQ
description: Add SAML SSO to Ruby on Rails App with BoxyHQ
sidebar_label: Ruby on Rails
toc_min_heading_level: 2
toc_max_heading_level: 5
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 为Ruby on Rails应用添加SAML单点登录

本指南假设您已有一个Ruby on Rails应用，并希望为企业客户启用SAML单点登录认证。通过本指南，您将获得一个能够使用SAML单点登录认证用户的应用。

访问[GitHub仓库](https://github.com/boxyhq/ruby-examples/tree/main/rails-enterprise-sso)查看Ruby on Rails SAML单点登录集成的源代码。

为应用集成SAML单点登录包含以下步骤：

- 配置SAML单点登录
- 通过SAML单点登录进行认证

## 在Rails上配置企业级单点登录

此步骤允许您的租户为其用户配置SAML连接。阅读以下指南以了解更多信息：

- [配置SAML单点登录的UI最佳实践](/guides/jackson/configuring-saml-sso)
- [单点登录连接API](/docs/jackson/sso-flow/)

## 通过SAML单点登录进行认证

添加SAML连接后，应用可使用此连接通过SAML Jackson发起单点登录认证流程。以下部分更侧重于单点登录认证方面。

### 部署SAML Jackson

第一步是部署SAML Jackson服务。按照[部署文档](/docs/jackson/deploy/service)安装并配置SAML Jackson。

### 设置SAML Jackson集成

我们将通过两种流行的认证库深入探讨Jackson集成：

- [使用Sorcery](#with-sorcery)
- [使用OmniAuth](#with-omniauth)

#### 使用Sorcery

首先，需要安装并配置[sorcery](https://github.com/Sorcery/sorcery)。

##### 安装依赖

使用以下命令安装`sorcery` gem：

```shell
bundle add sorcery
```

##### 配置数据库

```shell title="Generate migration scripts for sorcery"
    bin/rails g sorcery:install external --only-submodules
```

```shell title="Run migration scripts"
    bin/rake db:migrate
```

```shell title="Generate the Authentication model"
    bin/rails generate model Authentication --migration=false
```

```shell title="Modify the user schema"
    # remove the unused columns from the user table, we won't need the password field as the login is external
    bin/rails generate migration RemoveColumnsFromUsers crypted_password:string salt:string
    # add the new columns
    bin/rails generate migration AddColumnsToUsers firstName:string lastName:string uid:string
    # run the migrations
    bin/rake db:migrate
```

##### 为Jackson添加自定义提供者

为Jackson添加自定义sorcery提供者，命名为`Boxyhqsso`。

我们依赖sorcery包中的`Protocols::Oauth2`混入。简而言之，这里我们正在与Jackson建立OAuth 2.0流程的连接。Jackson将根据租户/产品重定向到配置的身份提供者连接。

:::info
通过将文件包含在`app/lib`文件夹中，Rails将自动加载提供者类。
:::

```ruby title="app/lib/sorcery/providers/boxyhqsso.rb"
    module Sorcery
      module Providers
       # This class adds support for OAuth2.0 SSO flow with Jackson service.
       #
       #   config.boxyhqsso.site = <http://localhost:5225>
       #   config.boxyhqsso.key = <key>
       #   config.boxyhqsso.secret = <secret>
       #   ...
       #
       class Boxyhqsso < Base
         include Protocols::Oauth2

         attr_reader :parse
         attr_accessor :auth_url, :token_url, :user_info_path

         def initialize
           super

           @site          = ENV['JACKSON_URL']
           @auth_url      = '/api/oauth/authorize'
           @token_url     = '/api/oauth/token'
           @user_info_path = '/api/oauth/userinfo'
           @parse = :json
           # @state = SecureRandom.hex(16)
         end

         def get_user_hash(access_token)
           response = access_token.get(user_info_path)
           body = JSON.parse(response.body)
           auth_hash(access_token).tap do |h|
             h[:user_info] = body
             h[:uid] = body['id']
           end
         end

         # calculates and returns the url to which the user should be redirected,
         # to get authenticated at the external provider's site.
         def login_url(params, _session)
           add_param(authorize_url(authorize_url: auth_url),
                     [
                       { name: 'tenant', value: params[:tenant] },
                       { name: 'product', value: params[:product] }
                     ])
         end

         # tries to login the user from access token
         def process_callback(params, _session)
           args = {}.tap do |a|
             a[:code] = params[:code] if params[:code]
           end
           get_access_token(args, token_url: token_url, token_method: :post, auth_scheme: :request_body)
         end

         def add_param(url, query_params)
           uri = URI(url)
           qp = URI.decode_www_form(uri.query || [])
           query_params.each do |param|
             qp << [param[:name], param[:value]]
           end
           uri.query = URI.encode_www_form(qp)
           uri.to_s
         end
       end
    end
    end

```

##### 配置自定义sorcery提供者

添加一个初始化文件来配置sorcery模块。这里我们告诉sorcery加载`:external`子模块，并将上一步中的`boxyhqsso`自定义提供者添加到`external_providers`列表中。同时，参见内联注释了解`boxyhqsso`提供者的设置。

```ruby title="config/initializers/sorcery.rb"
    Rails.application.config.sorcery.submodules = [:external]

    # Here you can configure each submodule's features.
    Rails.application.config.sorcery.configure do |config|

      config.external_providers = [:boxyhqsso]

      # URL of Jackson service
      config.boxyhqsso.site = ENV['JACKSON_URL']
      # This translates to client_id in OAuth 2.0. Setting it to dummy will allow us to use `tenant` and product` params instead
      config.boxyhqsso.key = 'dummy'
      # The url of the rails app to which Jackson sends back the authorization code
      config.boxyhqsso.callback_url = 'http://localhost:3366/oauth/callback'
      # This will be passed to Jackson token endpoint as part of credentials
      config.boxyhqsso.secret = ENV['CLIENT_SECRET_VERIFIER']
      # Takes care of converting the user info from the provider (Jackson) into the attributes of the User.
      config.boxyhqsso.user_info_mapping = { email: 'email', uid: 'id', firstName: 'firstName', lastName: 'lastName'}

      # --- user config ---
      config.user_config do |user|

       # -- external --
       user.authentications_class = Authentication

      end
      # This line must come after the 'user config' block.
      # Define which model authenticates with sorcery.
      config.user_class = User

    end
```

##### 路由和控制器

最后，需要添加路由和控制器文件以发起登录流程并处理来自Jackson服务的回调。

:::info
登录流程通过向`/sso`发起POST请求来启动。
:::

<Tabs>
     <TabItem value="routes" label="Routes" default>

```ruby title="config/routes.rb"
    Rails.application.routes.draw do
      ...
      # Renders the login page
      get 'sso', to: 'logins#index', as: :login
      # Initiates the OAuth 2.0 redirect to Jackson SSO service
      post 'sso', to: 'sorcery#oauth'
      # logout the user
      delete 'logout' => 'logins#destroy', as: :logout
      # handles the redirect back from Jackson SSO service, exchanges code with access_token and then fetches userprofile. Sorcery creates the user if not present in database, else return the one in the db.
      resource :oauth do
        get :callback, to: 'sorcery#callback', on: :collection
      end
      # Show profile data
      get 'profile', to: 'profiles#index', as: :profile

      ...
    end

```

  </TabItem>
  <TabItem value="controllers" label="Controllers">
     <Tabs>
     <TabItem value="SorceryController" label="SorceryController" default>

The `oauth` action initiates the OAuth 2.0 flow to Jackson SSO service. In the `callback` action, sorcery exchanges the code for access_token and user profile. If a user exists in the database, then the value of `@current_user` is loaded from the database. Else a new user is created in the database and returned.

```ruby title="app/controllers/sorcery_controller.rb"
     class SorceryController < ApplicationController
         skip_before_action :require_login, raise: false

         def oauth
           login_at('boxyhqsso', state: SecureRandom.hex(16))
         end

         # rubocop:disable Metrics/AbcSize, Metrics/MethodLength
         def callback
           provider = 'boxyhqsso'
           if @user = login_from(provider)
             redirect_to profile_path, notice: "Logged in from #{provider.titleize}!"
           else
             begin
               @user = create_from(provider)

               reset_session # protect from session fixation attack
               auto_login(@user)
               redirect_to profile_path, notice: "Logged in from #{provider.titleize}!"
             rescue
               redirect_to root_path, alert: "Failed to login from #{provider.titleize}!"
             end
         end
         rescue ::OAuth2::Error => e
           Rails.logger.error e
           Rails.logger.error e.code
           Rails.logger.error e.description
           Rails.logger.error e.message
           Rails.logger.error e.backtrace
         end

    end

```

  </TabItem>

  <TabItem value="LoginsController" label="LoginsController">

```ruby title="app/controllers/logins_controller.rb"
    class LoginsController < ApplicationController
          skip_before_action :require_login

          def index; # render login view
          end

          def destroy
              logout
              redirect_to(root_path, notice: 'Logged out!')
          end

    end

```

  </TabItem>

  <TabItem value="ProfilesController" label="ProfilesController">

```ruby title="app/controllers/profiles_controller.rb"
    class ProfilesController < ApplicationController
       def index; end # display profile information
    end

```

  </TabItem>
  </Tabs>
  </TabItem>
</Tabs>

#### 使用OmniAuth

:::info
与sorcery不同，omniauth不会自动关联用户模型，也不会将用户信息持久化到数据库。
:::

首先，我们需要安装并配置[omniauth](https://github.com/omniauth/omniauth)。

##### 安装依赖项

```shell
    bin/bundle add omniauth
    bin/bundle add omniauth-rails_csrf_protection # Used to protect against CSRF vulnerability
    bin/bundle add omniauth-oauth2 # generic OAuth2 strategy for OmniAuth that we will inherit from
```

##### 添加Jackson自定义策略

为Jackson添加自定义omniauth策略，我们将其命名为`Boxyhqsso`。
通过继承`OmniAuth::Strategies::OAuth2`，我们可以实现与Jackson的OAuth 2.0流程对接。Jackson将根据租户/产品重定向到配置的IdP连接。

```ruby title="app/lib/omniauth/strategies/boxyhqsso.rb"
      module OmniAuth
      module Strategies
          class Boxyhqsso < OmniAuth::Strategies::OAuth2
              # strategy name
              option :name, "boxyhqsso"

              args %i[
                client_id
                client_secret
                domain
              ]

              # Setup client URLs used during authentication
              def client
                options.client_options.site = domain_url
                options.client_options.authorize_url = '/api/oauth/authorize'
                options.client_options.token_url = '/api/oauth/token'
                options.client_options.userinfo_url = '/api/oauth/userinfo'
                options.client_options.auth_scheme = :request_body
                options.token_params = { :redirect_uri => full_host + '/auth/boxyhqsso/callback' }
                super
              end

              # These are called after authentication has succeeded. If
              # possible, you should try to set the UID without making
              # additional calls (if the user id is returned with the token
              # or as a URI parameter). This may not be possible with all
              # providers.
              uid{ raw_info['id'] }

              # Define the parameters used for the /authorize endpoint
              def authorize_params
                params = super
                %w[connection connection_scope prompt screen_hint login_hint organization invitation ui_locales tenant product].each do |key|
                  params[key] = request.params[key] if request.params.key?(key)
                end

                # Generate nonce
                params[:nonce] = SecureRandom.hex

                # Store authorize params in the session for token verification
                session['authorize_params'] = params.to_hash

                params
              end

              extra do
                {
                  'raw_info' => raw_info
                }
              end

              # Declarative override for the request phase of authentication
              def request_phase
                if no_client_id?
                  # Do we have a client_id for this Application?
                  fail!(:missing_client_id)
                elsif no_client_secret?
                  # Do we have a client_secret for this Application?
                  fail!(:missing_client_secret)
                elsif no_domain?
                  # Do we have a domain for this Application?
                  fail!(:missing_domain)
                else
                  # All checks pass, run the Oauth2 request_phase method.
                  super
                end
              end

              def raw_info
                userinfo_url = options.client_options.userinfo_url
                @raw_info ||= access_token.get(userinfo_url).parsed
              end

              # Check if the options include a client_id
              def no_client_id?
                ['', nil].include?(options.client_id)
              end

              # Check if the options include a client_secret
              def no_client_secret?
                ['', nil].include?(options.client_secret)
              end

              # Check if the options include a domain
              def no_domain?
                ['', nil].include?(options.domain)
              end

              # Normalize a domain to a URL.
              def domain_url
                domain_url = URI(options.domain)
                domain_url = URI("https://#{domain_url}") if domain_url.scheme.nil?
                domain_url.to_s
              end
          end
        end
    end

```

##### 配置自定义omniauth提供商

添加初始化文件将omniauth插入rack中间件管道。`OmniAuth::Builder`允许我们加载多个策略。

```ruby title="config/initializers/omniauth.rb"
    Rails.application.config.middleware.use OmniAuth::Builder do
          provider(
                :boxyhqsso,
                'dummy',
                ENV['CLIENT_SECRET_VERIFIER'],
                ENV['JACKSON_URL'],
                callback_path: '/auth/boxyhqsso/callback',
                authorize_params: {
                    scope: 'openid'
                }
            )

    end
```

##### 路由与控制器

最后需要添加路由和控制器文件来处理登录流程及Jackson服务的回调。我们使用控制器`concern`来保护诸如个人资料页等受保护路由的访问权限。

:::info
登录流程通过向`/auth/boxyhqsso`发起POST请求触发，该请求由rack中间件管道中的omniauth处理。
:::

<Tabs>

  <TabItem value="routes" label="Routes" default>

```ruby title="config/routes.rb"
    Rails.application.routes.draw do
      # Renders the login page
       get 'sso', to: 'logins#index', as: :login
       # handles the redirect back from Jackson SSO service, exchanges code with access_token and then fetches userprofile.
       get 'auth/boxyhqsso/callback', to: 'omniauth#callback'
       # Show profile data
       get 'omniauth/profile', to: 'omniauth_profiles#show', as: :omniauth_profile
       # logout the user
       delete 'omniauth/logout' => 'omniauth#logout', as: :omniauth_logout
    end
```

  </TabItem>

  <TabItem value="controllers" label="Controllers">

  <Tabs>

  <TabItem value="omniauth" label="OmniauthController" default>

After omniauth handles the callback from Jackson SSO service, it sets an authentication hash (`omniauth.auth`) on the rack environment of a request to `/auth/boxyhqsso/callback`. This contains the information about the logged-in user. We then set this value in the session which can then be displayed on the profile page.

```ruby title="app/controllers/omniauth_controller.rb"
    class OmniauthController < ApplicationController
      skip_before_action :require_login, raise: false

      def callback
        user_info = request.env['omniauth.auth']
        session[:userinfo] = user_info['extra']['raw_info']
        redirect_to omniauth_profile_path,  notice: "Logged in using omniauth!"
      end

      def logout
        reset_session
        redirect_to root_path,  notice: "Logged out from Omniauth!"
      end
    end
```

  </TabItem>

  <TabItem value="omniauth_profiles" label="OmniauthProfilesController">

Here we set the instance variable `@user` from the session. This can then be referenced in the profile view. Also by using the concern `OmniauthSecured`, we ensure that the profile view is rendered only if a user is logged in, else we redirect to the login page.

```ruby title="app/controllers/omniauth_profiles_controller.rb"
    class OmniauthProfilesController < ApplicationController
        skip_before_action :require_login, raise: false
        include OmniauthSecured

        def show
          @user = session[:userinfo]
        end
    end
```

  </TabItem>

  </Tabs>

  </TabItem>
  <TabItem value="concern" label="Controller concern">

```ruby title="app/controllers/concerns/omniauth_secured.rb"
    module OmniauthSecured
        extend ActiveSupport::Concern

        included do
          before_action :logged_in_using_omniauth?
        end

        def logged_in_using_omniauth?
          redirect_to login_path, notice: "⚠️ Please login using omniauth" unless session[:userinfo].present?
        end
    end
```

  </TabItem>

  </Tabs>