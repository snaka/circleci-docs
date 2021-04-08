---
layout: classic-docs
title: "CircleCI CLI 入門"
short-title: "CircleCI CLI 入門"
description: "コマンド ラインから CircleCI を操作する方法の基礎"
categories:
  - getting-started
order: 50
version:
  - Cloud
  - Server v2.x
---

# 概要

開発作業の大部分をターミナルで行いたい場合は、[CircleCI CLI](https://github.com/CircleCI-Public/circleci-cli) をインストールして CircleCI 上のプロジェクトを操作するとよいでしょう。 このドキュメントでは、CircleCI プロジェクトの初期化や操作を主にターミナルから行うための手順を説明します。 Please note that our server offering only supports a legacy version of the CLI. You can find more information on how to install that here: https://circleci.com/docs/2.0/local-cli/#using-the-cli-on-circleci-server.

# 前提条件

- Unix マシン (Mac または Linux) を使用している。Windows にも CircleCI CLI ツールのインストールは_可能_ですが、現在はベータ版であり、Unix 版ほどの機能は完備されていません。
- CI/CD、CircleCI サービスの機能とコンセプトについての基礎知識がある。
- GitHub アカウントを持っている。
- CircleCI アカウントを持っている。
- ターミナルを開いており、使用可能である。
- オプション: Github の [`Hub`](https://hub.github.com/) コマンドライン ツールがインストールされている (Web UI ではなくコマンド ラインから Github を使用できます)。 Hub のインストール方法については、[こちら](https://github.com/github/hub#installation)を参照してください。

上記の前提条件に不明点がある方や CircleCI プラットフォームの初心者は、先に[入門ガイド]({{site.baseurl}}/2.0/getting-started/)または[コンセプトに関するドキュメント](https://circleci.com/ja/docs/2.0/concepts/#section=getting-started)をお読みになることをお勧めします。

# 手順

## Initialize a git repo

基本中の基本から始めましょう。プロジェクトを作成し、Git リポジトリを初期化します。 各ステップについては、以下のコード ブロックを参照してください。

```sh
cd ~ # ホーム ディレクトリに移動します
mkdir foo_ci # "foo_ci" という名前のフォルダーにプロジェクトを作成します
cd foo_ci # 新しい foo_ci フォルダーにディレクトリを変更します
git init # create a git repository
touch README.md # Create a file to put in your repository
echo 'Hello World!' >> README.md
git add . # コミットするすべてのファイルをステージングします
git commit -m "Initial commit" # 最初のコミットを実行します
```

## Connect your git repo to a VCS

前述の手順で Git リポジトリがセットアップされ、「Hello World!」と記述された 1 つのファイルが格納されました。 We need to connect our local git repository to a Version Control System - either GitHub or BitBucket. やってみましょう。

Hub CLI のインストールとセットアップが完了している場合は、以下のコマンドを実行するだけです。

```sh
hub create
```

次に、ログインと Hub CLI の承認に関するプロンプトに従います。

Hub CLI を使用していない場合は、GitHub にアクセスしてログインし、[新しいリポジトリを作成](https://github.com/new)します。 指示に従ってコミットし、リモートにプッシュします。 この操作は通常、以下のようなコマンドになります。

```sh
git remote add origin git@github.com:<YOUR_USERNAME>/foo_ci.git
git push --set-upstream origin master
```

これで、Git リポジトリが VCS に接続され、 VCS 上のリモート ("origin") がローカルでの作業内容と一致するようになります。

## Download and set up the CircleCI CLI

次に、CircleCI CLI をインストールし、いくつかの機能を試してみます。 CLI を Unix マシンにインストールするには、ターミナルで以下のコマンドを実行します。

```sh
curl -fLSs https://circle.ci/cli | bash
```

CLI のインストール方法はいくつかあります。別の方法を使用する必要がある場合は、[こちら]({{site.baseurl}}/2.0/local-cli)を参照してください。

次に、インストール後のセットアップ手順を実行します。

```sh
circleci setup
```

ここで API トークンを要求されます。 [アカウントの設定ページ](https://circleci.com/account/api)に移動し、`[Create New Token (新しいトークンを作成する)]` をクリックします。 トークンに名前を付け、生成されたトークン文字列をコピーして、安全な場所に保存します。

CLI に戻って API トークンを貼り付ければセットアップは完了です。

## Setup and validate our first config

ここからは、プロジェクト ディレクトリに設定ファイルを作成します。

```sh
cd ~/foo_ci # カレント ディレクトリが foo_ci フォルダーであることを確認します
mkdir .circleci # ".circleci" という名前のディレクトリを作成します
cd .circleci # カレント ディレクトリを新しいディレクトリに変更します
touch config.yml # "config.yml" という名前の YAML ファイルを作成します
```

上記のコマンドにより、`.circleci` フォルダーが作成され、そこに設定ファイルが格納されます。

新しく作成した `config.yml` ファイルを開き、以下の内容を貼り付けます。

```yaml
version: 2.0
jobs:
  build:
    docker:
      - image: circleci/ruby:2.4.2-jessie-node
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD  # context / project UI env-var reference
    steps:
      - checkout
      - run: echo "Hello World"
```

ここで、この構成が有効であるかどうかをバリデーションします。 プロジェクトのルートで、以下のコマンドを実行します。

```sh
circleci config validate
```

**メモ:** 使用しているコマンドの詳細を知りたい場合は、`--help` を追加すると、いつでもコマンドに関する補足情報がターミナルに表示されます。

```sh
circleci config validate --help
```

## Testing a job before pushing to a VCS

CircleCI CLI では、コマンド ラインからジョブをローカルでテストできます。VCS にプッシュする必要はありません。 設定ファイル内のジョブに問題があることがわかっている場合は、プラットフォームでクレジットや時間を消費するよりも、ローカルでテストやデバッグを行う方が賢明です。

"build" ジョブをローカルで実行してみます。

```sh
circleci local execute
```

これで、指定した Docker イメージ (この場合は `circleci/ruby::2.4.2-jessie-node`) がプル ダウンされ、ジョブが実行されます。 使用している Docker イメージのサイズによっては、多少の時間がかかります。

ターミナルには大量のテキストが表示されるはずです。 出力の最後の数行は以下のようになります。

```sh
====>> Checkout code
  #!/bin/bash -eo pipefail
mkdir -p /home/circleci/project && cp -r /tmp/_circleci_local_build_repo/. /home/circleci/project
====>> echo "Hello World"
  #!/bin/bash -eo pipefail
echo "Hello World"
Hello World
Success!
```

## Connect your repo to CircleCI

このステップでは、ターミナルを離れる必要があります。 Head over to [the "Add Projects page"](https://app.circleci.com/projects/project-dashboard/github/circleci/). コードをプッシュするたびに CI が実行されるようにプロジェクトをセットアップします。

プロジェクトのリストから目的のプロジェクト ("foo_ci" または GitHub で付けた名前) を見つけ、[Set Up Project (プロジェクトのセットアップ)] をクリックします。 次に、ターミナルに戻り、最新の変更を GitHub にプッシュします (`config.yml` ファイルの追加分)。

```sh
git add .
git commit -m "add config.yml file"
git push
```

ブラウザーで CircleCI に戻ると、[Start building (ビルドの開始)] をクリックしてビルドを実行できます。

# Next steps

このドキュメントでは、CircleCI CLI ツールの使用を開始するための手順を簡単に説明してきました。 CircleCI CLI は、さらに複雑な機能も提供しています。

- [Orbs](https://circleci.com/ja/orbs/) の作成、表示、バリデーション、パブリッシュ
- CircleCI GraphQL API のクエリ
- 複雑な設定ファイルのパッケージ化と処理

詳細については、[CircleCI のローカル CLI に関するドキュメント]({{site.baseurl}}/2.0/local-cli)を参照してください。