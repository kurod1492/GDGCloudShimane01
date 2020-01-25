# GDG Cloud Shimane #01 セッション用資料

Railsの環境構築の話と CloudRun へデプロイのデモ

（Windows10 Home + Vagrant + VirtualBox + Ubuntu + Docker で環境構築）

## 概要

Rails アプリを作成し Google Cloud Platform (GCP) の Cloud Run へデプロイする手順を書いたものです。
以下の環境で実行することを確認しています。

- ローカル環境
  - Windows 10 Home 64bit
  - VirtualBox 6.0.14 r133895
  - Vagrant 2.2.6
  - git for windows 2.24.0 64bit
- VM 環境
  - Ubuntu 18.04 64bit (ubuntu/bionic64)
  - rbenv
  - Ruby 2.6.5
  - Rails 6.0.1

## 手順

1. VirtualBox のインストール
2. Vagrant のインストール
3. Vagrantfile の作成
4. vagrant up の実行
5. GCP プロジェクト作成
6. vagrant ssh を実行
7. 名前解決の設定
8. Rails をインストール
9. Node.js のインストール
10. Yarn のインストール
11. libmysqld-dev パッケージのインストール
12. Rails プロジェクトの作成
13. Gemfile の修正
14. bundle install の実行
15. webpacker のインストール
16. Rails アプリケーションの作成
17. config/environments/production.rb の設定変更
18. ローカルで動作確認
19. Production 環境の DB 設定
20. Dockerfile の作成
21. Docker イメージの作成
22. タグ付け
23. Google Cloud SDK のインストール
24. プロジェクトの紐付け
25. Container Registry API の有効化
26. Container Registry を認証
27. Container Registry へプッシュ
28. Cloud SQL Admin API の有効化
29. Cloud SQL 上に MySQL インスタンスの作成
30. データベース作成
31. Cloud SQL Proxy のインストール
32. Cloud SQL Proxy を起動
33. rails db:migrate を実行
34. Cloud Run API の有効化
35. Docker イメージを Cloud Run へデプロイ
36. ブラウザでアクセス
37. 後始末
38. 参考文献

## 1. VirtualBox のインストール

[Downloads - Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads) から
自分の PC に合うインストーラをダウンロードして実行します。

ここでは「Windows hosts」のリンクから VirtualBox-6.0.14-133895-Win.exe をダウンロードします。

## 2. Vagrant のインストール

[Download - Vagrant by HashiCorp](https://www.vagrantup.com/downloads.html) から
自分の PC 用のインストーラをダウンロードして実行します。

ここでは Windows 64bit 用の vagrant_2.2.6_x86_64.msi をダウンロードします。

## 3. Vagrantfile の作成

作業用ディレクトリを作成し、Vagrantfile を作成します。
ここでは作業用ディレクトリを C:\GDGCloudShimane とします。

今回 Windows 上のコマンドは git for windows に含まれる Git Bash を使用しました。
「$」のプロンプトは Git Bash のコマンドです。

    $ mkdir /c/GDGCloudShimane
    $ vi Vagrantfile

Vagrantfile の内容はこちら→
[Vagrantfile](https://github.com/kurod1492/GDGCloudShimane01/blob/master/Vagrantfile)

vagrant up 時に

- rbenv インストール(v1) 
- Ruby インストール(v2) 
- Docker インストール(v3) 

を実行するようになっています。

(v3 については https://qiita.com/win-chanma/items/0ef2e68bff2a33cca0e6 を参考にしました）


## 4. vagrant up の実行

Vagrantfile を同じディレクトリで vagrant up を実行します。
VM のプロビジョニングが実行されます。
Ruby のインストールに少し時間がかかります。
この間に、5. GCP プロジェクト作成を行います。

```
$ vagrant up
```

## 5. GCP プロジェクト作成

GCP に新しいプロジェクトを作成します。
右上のメニュー→「プロジェクトの設定」をクリックします。
プロジェクト名をクリックします。
「プロジェクトの選択」ダイアログが表示されます。
右上の「新しいプロジェクト」をクリックします。
プロジェクト名を入力します。今回は rails-cloud-run-sample とします。
「作成」をクリックします。
作成に少し時間がかかります。作成中は右上のベルのまわりがぐるぐるまわります。
作成できたら、プロジェクト名をクリックして、rails-cloud-run-sample に切り替えます。

デプロイしたイメージや DB など全て削除したい場合は、このプロジェクトをシャットダウンしてください。

## 6. vagrant ssh を実行

「4. vagrant up の実行」が完了したら、vagrant ssh を実行して VM の中の Ubuntu にログインします。
ログインすると`vagrant@ubuntu-bionic:~$`というプロンプトが表示されます。
以後、`vagrant@ubuntu-bionic:~$`は VM の中で実行することを表します。

```
$ vagrant ssh
vagrant@ubuntu-bionic $
```

## 7. 名前解決の設定

Ubuntu 上で名前解決ができないことがあります。
/etc/systemd/resolved.conf に以下の DNS 設定を追加します。

    [Resolve]
    DNS=8.8.8.8

追記したら、以下のコマンドでresolved を再起動します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ sudo service systemd-resolved restart
```

## 8. Rails をインストール

Rails をインストールします。

```
vagrant@ubuntu-bionic:~$ gem install rails
vagrant@ubuntu-bionic:~$ rails -v
Rails 6.0.1
```

## 9. Node.js のインストール

Rails バージョン 6 から標準で webpack を利用する構成になりました。
そのため Node.js をインストールする必要があります。
Node.js をインストールします。

```
vagrant@ubuntu-bionic:~$ curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
vagrant@ubuntu-bionic:~$ sudo apt -y install nodejs
```

## 10. Yarn のインストール

Node.js の環境で利用するパッケージマネージャ Yarn をインストールします。
以下のコマンドを実行します。

```
vagrant@ubuntu-bionic:~$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
vagrant@ubuntu-bionic:~$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
vagrant@ubuntu-bionic:~$ sudo apt update
vagrant@ubuntu-bionic:~$ sudo apt -y install yarn
```

## 11. libmysqld-dev パッケージのインストール

デプロイ後のデータベースは Cloud SQL 上の MySQL を使います。
そのため mysql2 の gem を使います。
mysql2 の gem をインストールするために libmysqld-dev パッケージが必要なのでインストールします。

```
vagrant@ubuntu-bionic:~$ sudo apt -y install libmysqld-dev
```

## 12. Rails プロジェクトの作成

Rails のプロジェクトを作成します。
プロジェクト名を rails-cloud-run-sample とします。
--skip-bundle を付けて rails new コマンドを実行します。
「Could not find gem 'sqlite3 (~> 1.4)' in any of the gem sources listed in your Gemfile.」
というエラーが表示された場合も以降の手順で対処できますのでそのまま進めます。

```
vagrant@ubuntu-bionic:~$ rails new rails-cloud-run-sample --skip-bundle
```

## 13. Gemfile の修正

mysql2 の gem を追加するため、Gemfile の最終行に以下の1行を追記します。

```
gem "mysql2", "~> 0.5.2"
```

## 14. bundle install の実行

bundle install を --path vendor/bundle オプションを付けて実行します。
必要なファイルが vendor/bundle にインストールされます。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ bundle install --path vendor/bundle
```

## 15. webpacker のインストール

webpacker のインストールが必要です。
以下のコマンドを実行します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ rails webpacker:install
```

## 16. Rails アプリケーションの作成

Rails アプリケーションを作成します。
ここでは User モデルだけの簡単なアプリケーションを作成します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ bundle exec rails g scaffold user name:string
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ bundle exec rails db:migrate
```

## 17. config/environments/production.rb の設定変更

Production 環境でアセットパイプラインを自動で通るように
config/environments/production.rb の設定を以下のように変更します。

    config.assets.compile = false
    ↓
    config.assets.compile = true


## 18. ローカルで動作確認

ここで一度 Rails アプリケーションの動作を確認します。
ローカルで起動します。
Rails アプリケーションは、IP アドレスが 192.168.33.10 の VM で起動しているので、
ブラウザで http://192.168.33.10:3000/users にアクセスします。
動作を確認できたらアプリケーションを停止します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ bundle exec rails s -b 0.0.0.0
```

## 19. Production 環境の DB 設定

Rails アプリケーションを Production 環境で起動したとき、MySQL に接続するように、
config/database.yml を修正します。
データベース名は sample、接続するユーザー名は root とします。
パスワードとソケットは環境変数で渡すことにします。

    production:
      <<: *default
      database: db/production.sqlite3

↓

    production:
      adapter: mysql2
      encoding: utf8
      pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
      username: root
      password: <%= ENV['MYSQL_PASSWORD'] %>
      socket: <%= ENV['MYSQL_SOCKET'] %>
      database: sample


## 20. Dockerfile の作成

Rails アプリケーションを起動する Docker コンテナを作成するため、Dockerfile を作成します。
Dockerfile を作成する場所は ~/rails-cloud-run-sample にします。
Dockerコンテナ起動時に「bundle exec rails server」を実行するようにします。

    FROM ruby:2.6.5
    
    ADD Gemfile Gemfile.lock /
    RUN bundle install
    
    WORKDIR /app
    
    ADD . .
    
    CMD ["bundle", "exec", "rails", "server"]

## 21. Docker イメージの作成

docker build コマンドを実行します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ sudo docker build -t rails-cloud-run-sample .
```

## 22. タグ付け

作成した Docker イメージにタグを付けます。
コマンドは docker tag [SOURCE_IMAGE] [HOSTNAME]/[PROJECT-ID]/[IMAGE] です。
プロジェクトID を確認しておきます。ここでは「rails-cloud-run-sample」とします。
以下のコマンドを実行します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ sudo docker tag rails-cloud-run-sample gcr.io/rails-cloud-run-sample/rails-cloud-run-sample
```

## 23. Google Cloud SDK のインストール

作成した Docker イメージを Google Conntainer Registry に登録する作業を実行するため、Google Cloud SDK が必要なのでインストールします。
[Google Cloud SDK のドキュメント|Cloud SDK|Google Cloud](https://cloud.google.com/sdk/docs/#deb)
の記述に沿って、以下の作業を進めてください。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ sudo apt update && sudo apt -y install google-cloud-sdk
```

## 24. プロジェクトの紐付け

gcloud init を実行して、プロジェクト(ここでは rails-cloud-run-sample)と紐付けます。
途中で

    Go to the following link in your browser:

        https://accounts.google.com/o/oauth2/auth?....

という感じでアカウント確認の URL が出力されます。これをブラウザのアドレス欄ににコピペしてアクセスすると、
「アカウントの選択「Google Cloud SDK」という画面が表示されますので、GCP で使っているアカウントをクリックします。
次に「Google Cloud SDK が Google アカウントへのアクセスをリクエストしています」という画面が表示されますので「許可」をクリックします。
次に「ログイン このコードをコピーし、アプリケーションに切り替えて貼り付けてください。」という画面が表示されますので、コードをコピーします。
コピーしたコードを「Enter verification code:」の後ろに貼り付けてエンターを押します。
そのあと「Pick cloud project to use:」と表示されたら、利用するプロジェクトの番号を入力してエンターを押します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ gcloud init
Welcome! This command will take you through the configuration of gcloud.

Your current configuration has been set to: [default]

You can skip diagnostics next time by using the following flag:
  gcloud init --skip-diagnostics

Network diagnostic detects and fixes local network connection issues.
Checking network connection...done.
Reachability Check passed.
Network diagnostic passed (1/1 checks passed).

You must log in to continue. Would you like to log in (Y/n)?  y

Go to the following link in your browser:

    https://accounts.google.com/o/oauth2/auth?...(以下省略)

Enter verification code: <- コードをコピペしてエンターを押す

You are logged in as: [kurod1492@gmail.com].

Pick cloud project to use:
 [1] rails-cloud-run-sample
 [2] web-service-232808
 [3] Create a new project
Please enter numeric choice or text value (must exactly match list item): <- 番号を入力してエンターを押す
```

## 25. Container Registry API の有効化

Container Registry API を有効にしておきます。
メニューの 「APIとサービス」から探して有効にしてください。

## 26. Container Registry を認証

Container Registry を認証する必要があります。
gcloud auth コマンドを実行します。
「Do you want to continue (Y/n)?」と表示されたら y を入力しエンターを押します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ gcloud auth configure-docker
The following settings will be added to your Docker config file
located at [/home/vagrant/.docker/config.json]:
 {
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud"
  }
}

Do you want to continue (Y/n)?  <- y を入力しエンターを押す

Docker configuration file updated.
```

## 27. Container Registry へプッシュ

Docker イメージを Container Registry へプッシュします。
コマンドは docker push [HOSTNAME]/[PROJECT-ID]/[IMAGE] です。
docker push コマンドを実行します。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ sudo docker push gcr.io/rails-cloud-run-sample/rails-cloud-run-sample
The push refers to repository [gcr.io/rails-cloud-run-sample/rails-cloud-run-sample]
f96997b4421d: Pushed
493c73876cf4: Pushed
e7fcd3800173: Pushed
6cee9936dd8b: Pushed
a3549f44a2b1: Pushed
f2f48a68d40e: Pushed
c15e12dff0e9: Pushed
31f78d833a92: Layer already exists
2ea751c0f96c: Layer already exists
7a435d49206f: Layer already exists
9674e3075904: Layer already exists
831b66a484dc: Layer already exists
latest: digest: sha256:6b1bca6bda0fbc0f846d97ece09699071d2678ed314efb19830990820fa2392e size: 2842
```

## 28. Cloud SQL Admin API の有効化

Cloud SQL Admin API を有効にしておきます。
メニューの 「APIとサービス」から探して有効にしてください。

## 29. Cloud SQL 上に MySQL インスタンスの作成

Cloud SQL 上に MySQL サーバを用意します。
Cloud Run が現時点で us-central1 でしか使えないので、それにあわせて us-central1 に作成します。
インスタンス名は「rails-cloud-run-sample」とします。
以下のコマンドを実行します。
Cloud SQL には Cloud SQL Proxy 経由で接続するため、--assign-ip というオプションを付けます。
<PASSWORD>のところは自分で決めたパスワードに変えてください。

```
vagrant@ubuntu-bionic:~$ gcloud sql instances create rails-cloud-run-sample --region us-central1 --assign-ip --tier db-f1-micro --root-password <PASSWORD>
```

## 30. データベース作成

Cloud SQL 上の MySQL にデータベースを作成します。
データベース名は config/database.yml で設定した「sample」、
インスタンス名は MySQL のインスタンスを作成するときに設定した「rails-cloud-run-sample」です。

```
vagrant@ubuntu-bionic:~$ gcloud sql databases create sample --instance rails-cloud-run-sample
Creating Cloud SQL database...done.
Created database [sample].
instance: rails-cloud-run-sample
name: sample
project: rails-cloud-run-sample
```

## 31. Cloud SQL Proxy のインストール

Rails アプリから MySQL のインスタンスに接続するときに Cloud SQL Proxy を利用します。
公式ドキュメントを参考にしながら、Cloud SQL Proxy のバイナリファイルを VM にインストールします。
[Cloud SQL Proxy を使用して MySQL クライアントを接続する  |  Cloud SQL for MySQL  |  Google Cloud](https://cloud.google.com/sql/docs/mysql/connect-admin-proxy?hl=ja)
ダウンロードするバイナリファイルのファイル名を cloud_sql_proxy として VM の vagrant ユーザーのホームディレクトリに保存します。
保存した cloud_sql_proxy に実行権限を付与します。

```
vagrant@ubuntu-bionic:~$ wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
vagrant@ubuntu-bionic:~$ chmod +x cloud_sql_proxy
```

## 32. Cloud SQL Proxy を起動

Cloud SQL Proxy を起動するとき、UNIX ソケットを使用するようにします。
プロキシソケットを格納する /cloudsql というディレクトリを作成しアクセス権限を付与します。
Cloud SQL Proxy を起動します。
Cloud SQL Proxy を起動した Git Bash の画面はそのまま開いておきます。
```
vagrant@ubuntu-bionic:~$ sudo mkdir /cloudsql
vagrant@ubuntu-bionic:~$ sudo chmod 777 /cloudsql
vagrant@ubuntu-bionic:~$ ./cloud_sql_proxy -dir /cloudsql

```

## 33. rails db:migrate を実行

新しく Git Bash の画面を開き vagrant ssh を実行して VM にログインします。
環境変数 RAILS_ENV に production をセットします。
環境変数 MYSQL_SOCKET にプロキシソケットをセットします。
プロキシソケットは <プロキシソケットを格納するディレクトリ>/<PROJECT_ID>:us-central1:<Cloud SQL インスタンス名> という形式です。
環境変数 MYSQL_PASSWORD に MySQL インスタンスを作成するときに決めたパスワードを指定します。
Rails プロジェクトのディレクトr内に移動して bundle exec rails db:migrate を実行します。
これで Cloud SQL の MySQL インスタンスにテーブルが作成されます。

```
vagrant@ubuntu-bionic:~$ export RAILS_ENV=production
vagrant@ubuntu-bionic:~$ export MYSQL_SOCKET=/cloudsql/rails-cloud-run-sample:us-central1:rails-cloud-run-sample
vagrant@ubuntu-bionic:~$ export MYSQL_PASSWORD=<PASSWORD>
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ bundle exec rails db:migrate
== 20191120022332 CreateUsers: migrating ======================================
-- create_table(:users)
   -> 0.1686s
== 20191120022332 CreateUsers: migrated (0.1699s) =============================
```

## 34. Cloud Run API の有効化

Cloud Run API を有効にしておきます。
メニューの 「APIとサービス」から探して有効にしてください。

## 35. Docker イメージを Cloud Run へデプロイ

環境変数 SECRET_KEY_BASE に 「$(bundle exec rails g secret)」をセットします。
gcloud beta run deploy コマンドを実行して、GCR にプッシュした Docker イメージを Cloud Run にデプロイします。
<PASSWORD> は MySQL インスタンスを作成するときに決めたパスワードを指定します。
途中で「Please choose a target platform:」というプロンプトが表示されるので、1を入力してエンターを押します。
デプロイが成功したら、「32. Cloud SQL Proxy を起動」で起動した cloud_sql_proxy は停止しても大丈夫です。

```
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ export SECRET_KEY_BASE=$(bundle exec rails g secret)
vagrant@ubuntu-bionic:~/rails-cloud-run-sample$ gcloud beta run deploy rails-cloud-run-sample \
                                                   --image gcr.io/rails-cloud-run-sample/rails-cloud-run-sample \
                                                   --add-cloudsql-instances rails-cloud-run-sample \
                                                   --allow-unauthenticated \
                                                   --region us-central1 \
                                                   --memory 512Mi \
                                                   --set-env-vars "RAILS_ENV=production, \
                                                   RACK_ENV=production, \
                                                   MYSQL_SOCKET=/cloudsql/rails-cloud-run-sample:us-central1:rails-cloud-run-sample, \
                                                   MYSQL_PASSWORD=<PASSWORD>, \
                                                   SECRET_KEY_BASE=$SECRET_KEY_BASE"
Please choose a target platform:
 [1] Cloud Run (fully managed)
 [2] Cloud Run for Anthos deployed on Google Cloud
 [3] Cloud Run for Anthos deployed on VMware
 [4] cancel
Please enter your numeric choice:  1 <- 1 を入力してエンターを押す

To specify the platform yourself, pass `--platform managed`. Or, to make this the default target platform, run `gcloud config set run/platform managed`.

Deploying container to Cloud Run service [rails-cloud-run-sample] in project [rails-cloud-run-sample] region [us-central1]
✓ Deploying... Done.
  ✓ Creating Revision...
  ✓ Routing traffic...
  ✓ Setting IAM Policy...
Done.
Service [rails-cloud-run-sample] revision [rails-cloud-run-sample-00003-vas] has been deployed and is serving 100 percent of traffic at https://rails-cloud-run-sample-gn72izdkla-uc.a.run.app
```

## 36. ブラウザでアクセス

デプロイ完了後に画面に出力される URL に /users/ を付けた URL (ここでは https://rails-cloud-run-sample-gn72izdkla-uc.a.run.app/users/ ) にアクセスします。
アプリの画面が表示されたら成功です。
1回目は表示されるまでに時間がかかりますが、2回目以降はすぐに表示されます。

## 37. 後始末

GCP のプロジェクトをシャットダウンすると、デプロイしたアプリ、作成した MySQL インスタンス、プッシュしたコンテナイメージなど全て削除することができます。
ホーム画面でプロジェクトを選択し、「プロジェクト設定に移動」をクリックすると「IAM と管理」の「設定」という画面が表示されます。
表示されているプロジェクト名を確認し、間違いなければ「シャットダウン」をクリックします。
画面の指示に従ってシャットダウンします。

## 38. 参考文献

以下の記事を参考にしました。ありがとうございます。

[Ubuntu18.04 + VagrantでDocker環境お気楽構築 - Qiita](https://qiita.com/win-chanma/items/0ef2e68bff2a33cca0e6)

[Rails アプリケーションを Cloud Run にデプロイする - こうさくきろく](http://mukopikmin.hatenablog.com/entry/2019/06/27/000730)
