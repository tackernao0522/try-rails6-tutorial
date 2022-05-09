# 1.3.3 Model-View-Controller (MVC)

- `app/controllers/application_controller.rb`を編集<br>

```rb:application_controller.rb
class ApplicationController < ActionController::Base
  def hello
    render html: 'hello, world!'
  end
end
```

- `config/routes.rb`を編集<br>

```rb:routes.rb
Rails.application.routes.draw do
  root 'application#hello'
end
```

+ `http://localhost:3000/`にアクセスしてみる<br>

## 1.5.2 Heroku デプロイ準備

+ `config/puma.rb`を編集<br>

```rb:puma.rb
# Doc: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#config

# Specifies the number of `workers` to boot in clustered mode.
# Workers are forked web server processes. If using threads and workers together
# the concurrency of the application would be max `threads` * `workers`.
# Workers do not work on JRuby or Windows (both of which do not support
# processes).
#
workers Integer(ENV.fetch('WEB_CONCURRENCY') { 2 })

# Puma can serve each request in a thread from an internal thread pool.
# The `threads` method setting takes two numbers: a minimum and maximum.
# Any libraries that use thread pools should be configured to match
# the maximum value specified for Puma. Default is set to 5 threads for minimum
# and maximum; this matches the default thread size of Active Record.
#
max_threads_count = Integer(ENV.fetch('RAILS_MAX_THREADS') { 5 })
min_threads_count =
  Integer(ENV.fetch('RAILS_MIN_THREADS') { max_threads_count })
threads min_threads_count, max_threads_count

# Use the `preload_app!` method when specifying a `workers` number.
# This directive tells Puma to first boot the application and load code
# before forking the application. This takes advantage of Copy On Write
# process behavior so workers use less memory.
#
preload_app!

rackup DefaultRackup

# Specifies the `port` that Puma will listen on to receive requests; default is 3000.
#
port ENV.fetch('PORT') { 3000 }

# Specifies the `environment` that Puma will run in.
#
environment ENV.fetch('RACK_ENV') { 'development' }

on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
```

+ `root $ touch tutorial/heroku.yml`を実行<br>

+ `tutorial/heroku.yml`を編集<br>

```yml:heroku.yml
# アプリ環境を定義する場所
setup:
  # アプリ作成時にアドオンを自動で追加する
  addons:
    - plan: heroku-postgresql
  # 環境変数を指定する
  config:
    # Rackへ現在の環境を示す
    RACK_ENV: production
    # Railsへ現在の環境を示す
    RAILS_ENV: production
    # log出力のフラグ(enabled => 出力する)
    RAILS_LOG_TO_STDOUT: enabled
    # publicディレクトリからの静的ファイルを提供してもらうかのフラグ(enabled => 提供してもらう)
    RAILS_SERVE_STATIC_FILES: enabled
# ビルドを定義する場所
build:
  # 参照するDockerfileの場所を定義(相対パス)
  docker:
    web: Dockerfile
  # Dockerfileに渡す環境変数を指定
  config:
    WORKDIR: app
# プロセスを定義
run:
  # Bundlerでインストールされたgemを使用してコマンドを実行
  web: bundle exec puma -C config/puma.rb
```