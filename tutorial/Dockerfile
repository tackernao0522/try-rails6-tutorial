# ベースイメージを指定する
# FROM ベースイメージ:タグ(タグはなくてもよいが最新のものが指定されることになる)
FROM ruby:2.7.2-alpine

# Dockerfile内で使用する変数を定義
# appという値が入る
ARG WORKDIR

ARG RUNTIME_PACKAGES="nodejs tzdata postgresql-dev yarn postgresql git"

ARG DEV_PACKAGES="build-base curl-dev"

# 環境変数を定義(Dockerfile, コンテナ参照可能)
# Rails ENV["TZ"] => Asia/Tokyoが出力される
ENV HOME=/${WORKDIR} \
    LANG=C.UTF-8 \
    TZ=Asia/Tokyo

# Dockerfime内で指定した命令を実行する・・・RUN, COPY, ADD, ENTORYPOINT, CMD
# 作業ディレクトリを定義
# コンテナ/app/Railsアプリ
WORKDIR ${HOME}

# ホスト側の(PC)のファイルをコンテナにコピー
# COPY コピー元(ホスト) コピー先(コンテナ)
# Gemfile* ... Gemfileから始まるファイルを全指定(Gemfile, Gemfile, Gemfile.lock)
# コピー元(ホスト) ... Dockerfileがあるディレクトリ以下を指定(api) ../ NG
# コピー先(コンテナ) ... 絶対パス or 相対パス(./ ... 今いる(カレント)ディレクトリ)
COPY Gemfile* ./

# apk ... Alpine Linuxのコマンド
# apk update = パッケージの最新リストを取得
RUN apk update && \
  gem install bundler --no-document bundler:2.2.17 && \
  # apk upgrade = インストールパッケージを最新のものに
  apk upgrade && \
  # apk add = パッケージのインストールを実行
  # --no-cache = パッケージをキャッシュしない(Dokcerイメージを軽量化)
  apk add --no-cache ${RUNTIME_PACKAGES} && \
  # --virtual 名前(任意) = 仮想パッケージ
  apk add --virtual build-dependencies --no-cache ${DEV_PACKAGES} && \
  # Gemのインストールコマンド
  # -j4(jobs=4) = Gemインストールの高速化
  bundle install -j4 && \
  # パーケージを削除(Dokcerイメージを軽量化)
  apk del build-dependencies

# . ... Dockerfileがあるディレクトリ全てのファイル(サブディレクトリも含む)
COPY . ./

# ホスト(PC)       | コンテナ
# ブラウザ(外部)    | Rails
