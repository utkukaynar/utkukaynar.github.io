---
title: Using Devise For JSON APIs
layout: post
description: How to use Devise for JSON Api on Ruby on Rails?
author: utkukaynar
image: assets/utku.jpg
categories:
  - Development
---
[Devise](https://github.com/plataformatec/devise) is a highly popular authentication solution for Rails and its power is coming from its flexibility. Yet, there are additonal gems which enhance the flexibility of Devise and add useful behaviour to the gem.

The solution I'll mention today is useful for api authentication. As you may
imagine, there are no forms on Rails views for api and you should send all your
data using JSON payload.

We'll use `devise-token-auth` gem for this behaviour. This gem is a beautiful
add-on to Devise functionality and I highly recommend using it.

Start by adding the following gems to your `Gemfile`:

{% highlight ruby %}
gem 'devise-token-auth'
gem 'rack-cors'
{% endhighlight %}

If you're using Rails with _api option_, you should configure your application for
Cross Origin Requests, a.k.a [Cors](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing). The [rack-cors](https://github.com/cyu/rack-cors) gem that we're using enable us share the resources with requests coming from other origins.

Once you `bundle install` the gems you can now modify your
`application.rb` :

```ruby
module YourApp
  class Application < Rails::Application
    config.middleware.use Rack::Cors do
      allow do
        origins '*'
        resource '*',
          :headers => :any,
          :expose  => ['access-token', 'expiry', 'token-type', 'uid', 'client'],
          :methods => [:get, :post, :options, :delete, :put]
      end
    end
  end
end
```

From this point on, you can run the installer for the Devise Token Auth

`rails g devise_token_auth:install User auth`

This will install token auth fields for `User` model at the mount path of `/auth`. It will also modify the existing Devise initializer for the tokens.

### Authentication With Email

This article only focuses on email authentication but you can find other auth methods in devise-token-auth's [README](https://github.com/lynndylanhurley/devise_token_auth) as well.

For email, you have to provide stmp sender settings to your Rails app.

```ruby
# config/environments/development.rb
Rails.application.configure do
  config.action_mailer.default_url_options = { :host => 'localhost:3000' }
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = { :address => 'localhost:3000', :port => 1025 }
end
```

Alternatively, you can use awesome [letter_opener](https://github.com/ryanb/letter_opener) Gem, which enables you preview the actual sent emails in your browser.

Doing this, you can now register your users at `/auth` for email registration and `/auth/sign_in`, `/auth/sign_out` for signing_in and out, respectively. In all cases, Devise will now return a access_token in the header and you can use the same info in all requests which you want your users to be authenticated to perform.

```
"access-token": "wwwww",
"token-type":   "Bearer",
"client":       "xxxxx",
"expiry":       "yyyyy",
"uid":          "zzzzz"
```

Beware that with its default settings, devise-token-auth gem changes access token headers on every request. You have to keep track of the changed headers in your client side. To set this to false, you can change the following in `config/initializers/devise_token_auth.rb`:

`change_token_on_each_request: false`

Since you'll use this on the client side, it's possible that you woould be using AngularJS as the frontend framework. In this case, there's no need to set a complicated setup on the frontend, because [ng-token-auth](https://github.com/lynndylanhurley/ng-token-auth) gem belonging to the same author handles this task gracefully.

Now you have a working, JSON based authentication solution for your api.

Happy coding!
