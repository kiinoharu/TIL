# アプリケーション作成フロー

### １）コントローラー＋indexアクションの定義＋ビューファイルの作成

````ターミナル
% rails g controller コントローラー名 index
# 命名したコントローラー名＋indexアクション定義＋コントローラーに対応するビューファイルの作成ができるコマンド。
````

### ２）ルーティング設定

````Ruby .route.rb
root to: “コントローラー名#index"
# ルートパスへのアクセスがあった際に、コントローラー名のindexアクションが呼び出される。
````

`reils s`にて開発環境サーバーを立ち上げ、`localhost:3000`にアクセスし、動作確認。

### ３）device導入
1.`Gemfile`編集

````Gemfile
# 中略
gem ‘device’
````
2.Gemインストール

````ターミナル
# Gemをインストール
% bundle install
````

3.ローカルサーバー再起動

````ターミナル
# サーバーを起動
% rails s
````

### ４）device設定ファイル作成

````ターミナル
# deviseの設定ファイルを作成
% rails g devise:install
````
このコマンドは追加したdeviceの「設定関連に使用するファイル」を自動で生成するコマンド。

### deviceのUserモデル作成

````ターミナル
# deviseコマンドでUserモデルを作成
% rails g devise user
````
`rails g devise`コマンドは、deviceによるユーザー機能の対象を指定することで、モデルとマイグレーションの生成やルーティングの設定などをまとめて処理する。<br>
実行後、モデルが作成され、`reote.rb`にdeviceに関連するパス（`devise_for :users`）が追加される。

### ５）テーブル作成

1.テーブル設計追記

````db/migrate/20XXXXXXXXX_devise_create_users.rb
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""
      t.string :name,               null: false, default: ""
      t.text   :profile,            null: false
      t.text   :occupation,         null: false
      t.text　　　　 :position,           null: false
     # README.mdの記述参考にテーブル追加
     # Type：textはdefauｌｔの値は設定できないため、記述しない。
````

2.マイグレーション実行

````ターミナル
# マイグレーションを実行
% rails db:migrate

# もしテーブルの内容等間違えてマイグレーションした際は以下のコマンド実施→修正→マイグレーション実行の流れ
% rails db:rollback
````

3.DBでuserテーブル確認<br>

4.ローカルサーバー再起動
