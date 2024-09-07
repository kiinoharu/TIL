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

### ６）バリデーション設定

#### user.rb
````
class User < ApplicationRecord
  validates :name, presence: true
  validates :profile, presence: true
  validates :occupation, presence: true
  validates :position, presence: true
  # 追記する必要があるカラムに対してバリデーションの設定を上記記述のように行う。上記記述は指定したカラムが空では保存されない設定になっている。
end
````

### ７）device用のビューファイル生成・編集

#### ターミナル
````
% rails g device:views
````

編集については一部注意点解説。
#### app/views/devise/該当ディレクトリ/new.html.erb
````
# 例
<%= form_with model: @user, url: user_○○_path, id: 'new_user', class: 'new_user', local: true do |f| %>
# “model:”：今回であれば、@user
# ”url:”：device導入後に`rails routes`を実行し、devise/該当ファイル名#createへのパスを確認し、記載

 <div class="field">
          <%= f.label :email, "メールアドレス" %><br />
          <%= f.email_field :email, id:"user_email", autofocus: true, autocomplete: "email" %>
</div>
# 新規投稿時やログイン時に必要な項目を記述。
````

### ８）application_controllerに、emailとpassword以外の値も保存できるように追記
deviceのコントローラーが編集できないため、`application_controller.rb`に記述する。<br>

#### app/controllers/application_controller.rb
````
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  private
  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name,:profile,:occupation,:position])

  # 今回の記述だと、`:sign_up`時、許可するキーに`:name,:profile,:occupation,:position`を記述し、許可するパラメーターを追加している。
  end
end
````
### ９）ログインしているかどうかの判定を行い、ログイン状態とログアウト状態の表示を変更
ヘッダーの「ログイン」「新規登録」と、「ログアウト」「New Proto」を、ログインしているか否かで表示が変更するよう条件分岐。<br>

#### app/controllers/application_controller.rb
````
<% if user_signed_in? %>
# user_signed_i?メソッド：ユーザーログイン状態→true、ユーザーログアウト状態→falseを返す。
   <div class="nav__right">
     <%= link_to "ログアウト", root_path, class: :nav__logout %>
     <%= link_to "New Proto", root_path, class: :nav__btn %>
   </div>
<% else %>
  <div class="nav__right">
    <%= link_to "ログイン", root_path, class: :nav__btn %>
    <%= link_to "新規登録", root_path, class: :nav__btn %>
  </div>
<% end %>

````

### １０）ログイン・ログアウトができるよう設定
・ヘッダーのログイン・ログアウトボタンに適切なパスを記載。

#### application.html.erb
````
<% if user_signed_in? %>
  <div class="nav__right">
    <%= link_to "ログアウト", destroy_user_session_path, method: :delete, class: :nav__logout %>

    <%= link_to "New Proto", root_path, class: :nav__btn %>
  </div>
<% else %>
  <div class="nav__right">
    <%= link_to "ログイン", user_session_path, class: :nav__btn %>
    <%= link_to "新規登録", root_path, class: :nav__btn %>
  </div>
<% end %>
````

・ログインしているユーザー名の表示

#### index.html.erb
````
<div class="greeting"> 
こんにちは、
  <%= link_to current_user.name, root_path, class: :greeting__link%>です。
  # 上記パスの部分にて、`current_user.name`を使用すると、現在ログインしているユーザー名を適用・表示できる。
</div> 
````

