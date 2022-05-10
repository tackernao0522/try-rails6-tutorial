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

# 第2章 Toyアプリケーション

+ `root $ docker compose run --rm tutorial rails g scaffold User name:string email:string`を実行<br>

+ `root $ docker compose run --rm tutorial rails db:migrate`を実行<br>

+ `http://localhost:3000/users`にアクセスしてみる<br>

+ `config/routes.rb`を編集<br>

```rb:routes.rb
Rails.application.routes.draw do
  resources :users
  root 'users#index'
end
```

## 2.3 Micropostsリソース

+ `root rails g scaffold Micropost content:text user_id:integer`を実行<br>

+ `root rails db:migrate`を実行<br>

## 2.14 マイクロポストの最大文字数を140文字に制限する

+ `app/models/micropost.rb`を編集<br>

```rb:micropost.rb
class Micropost < ApplicationRecord
  validates :content, length: { maximum: 140 }
end
```

## 2.3.3 ユーザーはたくさん マイクロポストを持っている

+ `app/models/user.rb`(1人のユーザーに複数のマイクロポストがある)を編集<br>

```rb:user.rb
class User < ApplicationRecord
  has_many :microposts
end
```

+ `app/models/micropost.rb`(1つのマイクロポストは1人のユーザーのみに属する)を編集<br>

```rb:micropost.rb
class Micropost < ApplicationRecord
  belongs_to :user
  validates :content, length: { maximum: 140 }
end
```

+ Railsコンソールでアプリケーションの状態を調べてみる<br>

+ `$ rails console`を実行<br>

+ `>> first_user = User.first`を実行<br>

```
  User Load (6.2ms)  SELECT "users".* FROM "users" ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> #<User id: 1, name: "user1", email: "user1@example.com", created_at: "2022-05-10 03:20:12.805596000 +0000", updated_at: "2022-05-10 03:20:12.805596000 +0000">
```

+ `>> first_user.microposts`を実行<br>

```
  Micropost Load (8.3ms)  SELECT "microposts".* FROM "microposts" WHERE "microposts"."user_id" = $1 /* loading for inspect */ LIMIT $2  [["user_id", 1], ["LIMIT", 11]]
=> #<ActiveRecord::Associations::CollectionProxy [#<Micropost id: 1, content: "micropost1", user_id: 1, created_at: "2022-05-10 03:20:56.413554000 +0000", updated_at: "2022-05-10 03:20:56.413554000 +0000">, #<Micropost id: 2, content: "micropost1-1", user_id: 1, created_at: "2022-05-10 03:21:17.271998000 +0000", updated_at: "2022-05-10 03:21:17.271998000 +0000">]>
```

+ `>> micropost = first_user.microposts.first`を実行<br>

```
  Micropost Load (1.7ms)  SELECT "microposts".* FROM "microposts" WHERE "microposts"."user_id" = $1 ORDER BY "microposts"."id" ASC LIMIT $2  [["user_id", 1], ["LIMIT", 1]]
=> #<Micropost id: 1, content: "micropost1", user_id: 1, created_at: "2022-05-10 03:20:56.413554000 +0000", updated_at: "2022-05-10 03:20:56.413554000 +0000">
```

+ `>> micropost.user`を実行<br>

```
=> #<User id: 1, name: "user1", email: "user1@example.com", created_at: "2022-05-10 03:20:12.805596000 +0000", updated_at: "2022-05-10 03:20:12.805596000 +0000">
```
## 2.18 マイクロポストのコンテンツが存在しているかどうかの確認

+ `app/models/micropost.rb`を編集<br>

```rb:micropost.rb
class Micropost < ApplicationRecord
  belongs_to :user
  validates :content, length: { maximum: 140 }, presence: true
end
```

+ `rails db:reset`ができない時の対処法` https://qiita.com/Kiyo_Karl2/items/f1732fb7348b2ed9471d <br>
