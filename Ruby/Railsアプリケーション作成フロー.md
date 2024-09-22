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

### ３）devise導入
1.`Gemfile`編集

````Gemfile
# 中略
gem ‘devise’
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

### ４）devise設定ファイル作成

````ターミナル
# deviseの設定ファイルを作成
% rails g devise:install
````
このコマンドは追加したdeviseの「設定関連に使用するファイル」を自動で生成するコマンド。

### deviseのUserモデル作成

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
````user.rb
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
````ターミナル
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
````app/controllers/application_controller.rb
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
````app/controllers/application_controller.rb
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
````application.html.erb
<% if user_signed_in? %>
  <div class="nav__right">
    <%= link_to "ログアウト", destroy_user_session_path, data: { turbo_method: :delete }, class: :nav__logout %>

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
````index.html.erb
<div class="greeting"> 
こんにちは、
  <%= link_to current_user.name, root_path, class: :greeting__link%>です。
  # 上記パスの部分にて、`current_user.name`を使用すると、現在ログインしているユーザー名を適用・表示できる。
</div> 
````

### ---プロトタイプ情報投稿機能作成---
### １１）モデル及びテーブル作成
・`rails g model prototype`にてPrptptypeモデル作成<br>
・マイグレーションファイルにカラム追加
#### 2024XXXXXXXX_create_prototypes.rb
````2024XXXXXXXX_create_prototypes.rb
class CreatePrototypes < ActiveRecord::Migration[7.0]
  def change
    create_table :prototypes do |t|
      t.string     :title,              null: false, default: ""
      t.text       :catch_copy,         null: false
      t.text       :concept,            null: false
      t.references :user,               null: false, foreign_key: true
　　　　　　　　　　　　　# 上記４項目追加
      t.timestamps
    end
  end
end
````
・`rails db:migrate`実行<br>
・DB上にてテーブル及びカラムが存在することを確認
<br>
### １２）アソシエーション記述
#### prototye.rb
````prototye.rb
class Prototype < ApplicationRecord
  belongs_to :user
end
````
#### user.rb
````　user.rb
class User < ApplicationRecord
  # 中略
  has_many :prototypes
end
````
### １３）Active Storage導入
・mini_magickとimage_processingのGemをGemfileに記述し、bundle installを実行
####　Gemfile
````Gemfile
# 上略
gem 'mini_magick'
gem 'image_processing', '~> 1.2'
````
・`rails active_storage:install`でActive Storageを導入<br>
・`rails db:migrate`を実行
・Prototypeモデルに、has_one_attachedを使用してimageカラムとのアソシエーションを記述
#### prototype.rb
````prototype.rb
class Prototype < ApplicationRecord
  belongs_to :user
  has_one_attached :image
end
````
### １４）バリデーション設定
#### prototype.rb
````prototype.rb
class Prototype < ApplicationRecord
  belongs_to :user
  has_one_attached :image

  validates :title, presence: true
  validates :catch_copy, presence: true
  validates :concept, presence: true
  validates :image, presence: true
end
````
### １５）投稿機能実装
<br>
・ヘッダーの「New Proto」ボタンから、新規投稿ページに遷移するようにパスを設定
#### application.html.erb
````application.html.erb
<%= link_to "New Proto", new_prototype_path, class: :nav__btn %>
# `rails routes`にて、新規投稿ページに遷移するパスを確認。`new_prototype_path`を指定。
````
<br>

・newアクションにインスタンス変数@prototypeを定義し、Prototypeモデルの新規オブジェクトを代入
#### prototype_controller.rb
````
  def new
    @prototype = Prototype.new
  end
````
<br>

・new.html.erbから部分テンプレートである、_form.html.erbを呼び出す記述
#### new.html.erb
````new.html.erb
 <%= render partial: "form",  locals: { prototype: @prototype } %>
# render:部分テンプレートの呼び出しメソッド,partial:部分テンプレートの指定,locals:部分テンプレートにローカル変数を渡し、ローカル変数にコントローラーにて定義したインスタンス変数を格納することで、フォームの入力データにアクセスできる。
````
<br>

・_form.html.erbのform_withのモデル名を正しく記述
#### _form.html.erb
````_form.html.erb
<%= form_with model: prototype, local: true do |f|%>
　# 中略
<% end %>
````

<br>

・prototypesコントローラーのprivateメソッドにストロングパラメーターをセットし、特定の値のみを受け付けるようにした。且つ、user_idもmerge
#### prototypes_controller.rb
````prototypes_controller.rb
# 上略

private

def prototype_params
  params.require(:prototype).permit(:title, :catch_copy, :concept, :image).merge(user_id: current_user.id)
end

#　`params.require(:prototype) `:フォームのデータから`:prototype`キーのデータを取得。これがないとエラーになる。
# `permit(:title, :catch_copy, :concept, :image)`:許可するフィールドの指定。他のフィールドは無視され、セキュリティ強化。
# `.merge(user_id: current_user.id)`:ログインしているユーザーのIDを追加する。
````
<br>

・createアクションにデータ保存のための記述をし、保存されたときはルートパスに戻るような記述
　＋　createアクションに、データが保存されなかったときは新規投稿ページへ戻るようrenderを用いて記述
#### prototypes_controller.rb
````prototypes_controller.rb
  def create
    @prototype = Prototype.new(prototype_params)
    if @prototype.save
      redirect_to root_path
    else
      render :new
    end
  end
````
### １６）投稿したプロトタイプをトップページに表示させる
・各プロトタイプを表示するための部分テンプレート準備。今回は`_prototype.html.erb`とする。<br>
・indexアクションに、インスタンス変数を定義し、すべてのプロトタイプの情報を代入
#### prototypes_controller.ｒｂ
````prototypes_controller.ｒｂ
  def index
    @prototypes = Prototype.all
  end
````
・index.html.erbから_prototype.html.erbを呼び出し、プロトタイプ毎に、画像・プロトタイプ名・キャッチコピー・投稿者の名前を表示できるよう記述<br>
※rederメソッドにてcollectionオプションを用いた実装。
#### index.html.erb
````index.html.erb
      <%# 投稿機能実装後、部分テンプレートでプロトタイプ投稿一覧を表示する %>
      <%= render partial: "prototype", collection: @prototypes, as: :prototype %>
      # render partial: "prototype":prototypeファイルを指定
      # collection: @prototypes:@prototypesコレクション内の各プロトタイプに対して、`_prototype.html.erb`部分テンプレートを繰り返し適用。
      # as: :prototype:部分テンプレート内で `prototype` という変数名が使われる。
````
#### _prototype.html.erb
````
<div class="card">
  <%= link_to image_tag(prototype.image, class: :card__img ), root_path%>
  <div class="card__body">
    <%= link_to prototype.title, root_path, class: :card__title%>
    <p class="card__summary">
      <%= prototype.catch_copy %>
    </p>
    <%= link_to prototype.user.name, root_path, class: :card__user %>
  </div>
</div>
````
### ---プロトタイプの詳細ページ実装---
### １７）アクションとルーティング設定
#### prototypes_controller.rb
````prototypes_controller.rb
  def show  
  end
  # showアクション定義
````
#### routes.rb
````routes.rb
  resources :prototypes, only: [:index, :new, :create, :show]
  # resourceｓへ`:show`追加
````
### １８）トップページのプロトタイプをクリック→詳細ページへ遷移する実装
#### _prototype.html.erb
````_prototype.html.erb
<div class="card">
  <%= link_to image_tag(prototype.image, class: :card__img ), prototype_path(prototype.id) %>
  <div class="card__body">
    <%= link_to prototype.title, prototype_path(prototype.id), class: :card__title%>
    <p class="card__summary">
      <%= prototype.catch_copy %>
    </p>
    <%= link_to prototype.user.name, prototype_path(prototype.id), class: :card__user %>
  </div>
</div>

# 各link_toのパスに`prototype_path(prototype.id)`を記述。引数に`prototype.id`を設定することで、idが適切に渡され、URLが正常に生成される。
````
### １９）詳細ページで投稿情報表示
・showアクションにインスタンス変数定義＋pathパラメータで送信されるIDで、prototypeモデルの特定のオブジェクトを取得するように記述し、インスタンス変数へ代入。
#### prototypes_controller.rb
````prototypes_controller.rb
  def show
    @prototype = Prototype.find(params[:id])
  end
````
・`show.html.erb`をユーザー情報とトップページにて選択したプロトタイプが表示される＋投稿したユーザーだけに「編集・削除」ボタンが表示されるように記述
#### show.html.erb
````show.html.erb
<main class="main">
  <div class="inner">
    <div class="prototype__wrapper">
      <p class="prototype__hedding">
        <%= @prototype.title%>
      </p>
      <%= link_to @prototype.user.name, root_path, class: :prototype__user %>
      <%# プロトタイプの投稿者とログインしているユーザーが同じであれば以下を表示する %>
      <% if user_signed_in? && current_user.id == @prototype.user.id %>
        <div class="prototype__manage">
          <%= link_to "編集する", root_path, class: :prototype__btn %>
          <%= link_to "削除する", root_path, class: :prototype__btn %>
        </div>
      <% end %>
      <%# // プロトタイプの投稿者とログインしているユーザーが同じであれば上記を表示する %>
      <div class="prototype__image">
        <%=  image_tag @prototype.image %>
      </div>
      <div class="prototype__body">
        <div class="prototype__detail">
          <p class="detail__title">キャッチコピー</p>
          <p class="detail__message">
            <%= @prototype.catch_copy %>
          </p>
        </div>
        <div class="prototype__detail">
          <p class="detail__title">コンセプト</p>
          <p class="detail__message">
            <%= @prototype.concept %>
          </p>
        </div>
      </div>
````
### ２０）プロトタイプ情報の編集機能実装
・アクションとルーティング設定<br>
#### prototype_controller.rb
````prototype_controller.rb
  def edit
  end

  def update
  end
````
#### routes.rb
````routes.rb
  resources :prototypes, only: [:index, :new, :create, :show, :edit, :update]
  # `:edit`、`：update`追記
````
・編集機能に関するビューファイル作成<br>
・詳細ページから編集ページに遷移できるよう設定
#### show.html.erb
````show.html.erb
<%= link_to "編集する", edit_prototype_path, class: :prototype__btn %>
# パスを`rails routes`にて確認して記述
````
・編集機能実装<br>
①editアクションにインスタンス変数@prototypeを定義した。且つ、Pathパラメータで送信されるID値で、Prototypeモデルの特定のオブジェクトを取得するように記述、それを@prototypeへ代入。<br>
②updateアクションにデータを更新する記述をし、更新されたときはそのプロトタイプの詳細ページに戻るような記述。<br>
 　＋データが更新されなかったときは、編集ページに戻るようにrenderを用いて記述。<br>
③動作確認。バリデーションによって更新ができず編集ページへ戻ってきた場合でも、入力済みの項目（画像以外）は消えないことを確認。<br>
#### prototype_controller.rb
````prototype_controller.rb
  def edit
    @prototype = Prototype.find(params[:id])
  end

  def update
    @prototype = Prototype.find(params[:id])
    if @prototype.update(prototype_params)
      redirect_to prototype_path(@prototype)
    else
      render :edit
    end
  end
````
### ２１）プロトタイプ削除機能実装
・アクションとルーティング設定
#### prototype_controller.rb
````prototype_controller.rb
  def destroy
  end
````
#### routes.rb
````routes.rb
resources :prototypes
# アクションをすべて設定したため、onlyオプション省略
````
・「削除する」ボタンから、先ほど作成したルーティングが呼び込まれるよう設定
#### show.html.erb
````show.html.erb
<%= link_to "削除する", prototype_path(@prototype), data: { turbo_method: :delete },class: :prototype__btn %>
# HTTPメソッドに`DELETE`を指定
````
・destroyアクションに、プロトタイプを削除し、トップページに戻るよう記述
#### prototype_controller.rb
````prototype_controller.rb
def destroy
  @prototype = Prototype.find(params[:id])
  @prototype.destroy
  redirect_to root_path
end
````
### ---コメント投稿機能実装---
### ２２）Commentモデル及びテーブル作成
・Commentモデル作成
####ターミナル
````ターミナル
rails g model comment
````
・マイグレーションファイルに必要カラム追加
#### 2024XXXXXXXXXX_create_comments.rb
````2024XXXXXXXXXX_create_comments.rb
  def change
    create_table :comments do |t|
      t.references :user,      null: false, foreign_key: true
      t.references :prototype, null: false, foreign_key: true
      t.text       :text
      t.timestamps
    end
  end
````
・マイグレーション実行
#### ターミナル
````ターミナル
rails db:migrate
# コマンド実行後、DBにてカラムが追加されているか確認。
````
・各モデルにアソシエーション及びバリデーション設定
#### comment.rb
````comment.rb
class Comment < ApplicationRecord
  belongs_to :user
  belongs_to :prototypes

  validates :text, presence: true
end
````
#### prototype.rb
````prototype.rb
  has_many :comments, dependent: :destroy　#追記
````
#### user.rb
````user.rb
  has_many :comments, dependent: :destroy　#追記
````
※`dependent: :destroy`オプションとは、具体的には、親モデル（`has_many`などで記述されたモデル）が削除された際、関連する子モデルも一緒に削除するオプションです。
### ２３）コントローラー作成、アクション設定及びルーティング設定
#### ターミナル
````ターミナル
　rails g controllers comments
　# コントローラー作成コマンド
````
#### comments_controller.rb
````comments_controller.rb
class CommentsController < ApplicationController
  def create
    
  end
　　　　# アクション定義
end
````
#### routes.rb
````routes.rb
  resources :prototypes do
    resources :comments, only: :create
  end
  # `resources :prototypes`にネスティングさせる。「あるプロトタイプに対してのコメント」という親子関係を表現したパスにできる。
　　　　# 記述後、`rails routeｓ`にてルーティング確認。
````
### ２４）コメント投稿機能実装
・`prototype_controller.rb`のshowアクションに、@commentというインスタンス変数を定義し、Commentモデルの新規オブジェクトを代入
#### prototype_controller.rb
````prototype_controller.rb
  def show
    @prototype = Prototype.find(params[:id])
    @comment = Comment.new　　# 左記を追記
  end
````
・`show.html.erb`のコメント投稿フォーム記述＋ログインしているユーザーにのみ表示するよう設定
#### show.html.erb
````show.html.erb
<% if user_signed_in? %>
  <%= form_with model: @comment, url: prototype_comments_path(@prototype),local: true do |f|%>
  <div class="field">
    <%= f.label :text, "コメント" %><br />
    <%= f.text_field :text, id:"comment_content" %>
  </div>
    <div class="actions">
      <%= f.submit "送信する", class: :form__btn  %>
    </div>
  <% end %>
<% end %>
````
・`commenｔ_controlleｒ.rb`のprivateメソッドにストロングパラメーターをセットし、特定の値のみを受け付けるようにした。且つ、user_idとprototype_idもmerge
#### commenｔ_controlleｒ.rb
````commenｔ_controlleｒ.rb
private 

def comment_params
  params.require(:comment).permit(:text).merge(user_id: current_user.id, prototype_id: params[:prototype_id])
end
````
・createアクションに、データが保存されたときは詳細ページにリダイレクトされるよう記述＋データが保存されなかったときは詳細ページに戻るようrenderを用いて記述
#### comments_controller.rb
````comments_controller.rb
class CommentsController < ApplicationController
  def create
    @comment = Comment.create(comment_params)
    if @comment.save
      redirect_to prototype_path(@comment.prototype)
    else
      @prototype = @comment.prototype
      @comments = @prototype.comments
      render "prototypes/show", status: :unprocessable_entity
    end
  end
end
````
### ２５）投稿したコメントが詳細ページで表示されるよう実装
・showアクションにインスタンス変数@commentsを定義し、その投稿に紐づくすべてのコメントを代入するための記述
#### prototype_controller.rb
````prototype_controller.rb
  def show
    @prototype = Prototype.find(params[:id])
    @comment = Comment.new
    @comments = @prototype.comments.includes(:user)　# 左記を追記
  end
````
・｀show.html.erb`に投稿に紐づくコメントおよびその投稿者を表示する記述
#### show.html.erb
````show.html.erb
<% @comments.each do |comment| %>
　# @commetsに含まれるすべてのコメントをループ処理で1つずつ取り出し、それを`commet`変数として使う。
  <li class="comments_list">
    <%= comment.text %> 
    <%= link_to comment.user.name, root_path, class: :comment_user %> 
  </li>
<% end %>
````
### ---ユーザー詳細ページ実装---

### ２６）userコントローラー作成、アクション及びルーティング設定
#### ターミナル
````ターミナル
　rails g controller users
````
#### users_controller.rb
````users_controller.rb
class UsersController < ApplicationController
  def show
  end
end
````
#### routes.rb
````routes.rb
   resources :users, only: :show
````
・ユーザー詳細ページのビューファイルの設置<br>
・各ページのユーザー名をクリックすると、ユーザー詳細ページに遷移するよう遷移先記述
#### index.html.erb
````index.html.erb
<div class="greeting">
  こんにちは、
  <%= link_to current_user.name, user_path(current_user), class: :greeting__link%>です。
</div> 
````
#### _prototype.html.erb
````_prototype.html.erb
 <%= link_to prototype.user.name, user_path(prototype.user), class: :card__user %>
````
#### show.html.erb
````show.html.erb
<%= link_to @prototype.user.name, user_path(@prototype.user), class: :prototype__user %>
・・・
<% @comments.each do |comment| %>
  <li class="comments_list">
    <%= comment.text %> 
    <%= link_to comment.user.name, user_path(@prototype.user), class: :comment_user %> 
  </li>
<% end %>
````
### ２７）ユーザー詳細ページでユーザーの情報表示
・usersコントローラーのshowアクションにインスタンス変数@userを定義。且つ、Pathパラメータで送信されるID値で、Userモデルの特定のオブジェクトを取得するように記述し、それを@userに代入。＋部分テンプレートにて`@prototypes`を使用するため、定義。
#### users_controller.rb
````users_controller.rb
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    @prototypes = @user.prototypes
  end
end
````
・ユーザー詳細ページでユーザーの各情報が表示できるよう記述＋部分テンプレート呼び出し。
#### users/show.html.erb
````users/show.html.erb
<%= @user.name %>
・・・
<%= @user.profile %>
・・・
<%= @user.occupation %>
・・・
<%= @user.position %>
・・・
# 部分テンプレートでそのユーザーが投稿したプロトタイプ投稿一覧を表示する。 
<%= render partial: "prototypes/prototype", collection: @prototypes %>
````
### ２８）ユーザーのアクセス範囲制限
・ユーザーのログイン状態によってページ遷移を制限するページと制限しないページを把握
#### 
````
# ログインしていない状態で遷移できないページ
・新規投稿ページ
・編集ページ
・削除機能
# ログインしていない状態でも遷移できるページ
・トップページ
・プロトタイプ詳細ページ
・ユーザー詳細ページ
・ユーザー新規投稿ページ
・ログインページ
````
・`prototype_controller.rb`にアクセス制限を設定
#### prototype_controller.rb
````prototype_controller.rb
  before_action :authenticate_user!,only: [:new,:edit,:destroy]　# 上記項目にて"ログインしていない状態で遷移できないページ"を`only`オプションにて指定。
````
・投稿者以外のユーザーが投稿者専用のページに遷移できないよう設定
#### prototype_controller.rb
````prototype_controller.rb
  def edit
    @prototype = Prototype.find(params[:id])
    unless @prototype.user == current_user
      redirect_to action: :index
    end
  end
````



### ---デプロイ---

### ２９）Renderを活用したデプロイ準備
・PostgerSQLをインストール　※１度インストールすれば、２回目以降のデプロイ時はインストールの必要なし。
#### ターミナル
````ターミナル
% brew install postgresql@14
・・・
# 上記インストール後、以下コマンド実施して、バージョン表示確認。
% psql --version
# =>psql (PostgreSQL) 14.5 (Homebrew)の様に表示されれば、OK
````
・デプロイするアプリのGemにPostgerSQLを使用できるGemを追加
#### Gemfile
````Gemfile
group :production do
  gem 'pg'
end
````
・Gemインストール
#### ターミナル
````ターミナル
% bundle install
````
・デプロイ用の設定ファイル（`bin/render-build.sh`）を作成、記述
#### bin/render-build.sh
````bin/render-build.sh
#!/usr/bin/env bash
# exit on error
set -o errexit

bundle install
bundle exec rake assets:precompile
bundle exec rake assets:clean
bundle exec rake db:migrate
````
・DBの設定を変更
#### config/database.yml
````config/database.yml
default: &default
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  adapter: mysql2
  username: root
  password:
  host: localhost
  database: #アプリケーション名

test:
  <<: *default
  adapter: mysql2
  username: root
  password:
  host: localhost
  database: #アプリケーション名

production:
  <<: *default
  adapter: postgresql
  url: <%= ENV['DATABASE_URL'] %>
````
・`Gemfile.lock`の設定を変更
#### ターミナル
````ターミナル
% bundle lock --add-platform x86_64-linux
# =>上記コマンド実行後、`credentials.yml.enc`,`master.key`の2つのファイルが作成される。
````
・秘密情報管理ファイルを作成<br>
 GitHubからクローンしたアプリには`master.key`が含まれていないため、削除し、以下コマンドを実行し、新たに`master.key`を作成。
 #### ターミナル
````ターミナル
% EDITOR="vi" bin/rails credentials:edit
# 新しいファイルが作成されたことを確認した後、「escキー」→「:」→「q」と入力し、「enterキー」を押してファイルを閉じる。
````
・GitHubにコードをプッシュ
### ３０）Renderでのデプロイ手順
・DB作成
#### Render
````Render
①ヘッダーの「＋New」ボタンの「PostgreSQL」を選択。
②「Name→アプリ名、Region→Ohio」に設定。　※Regionについては、デフォルトのOregonは障害の発生率が高いため、回避する目的でOhioに設定。
③「Create Database」をクリック
　　　Statusが「Creating」→「Available」に変われば、DB作成完了。
④「Internal Database URL」をコピー。後の工程で使用。
````
・アプリ新規作成
#### Render
````Render
①ヘッダーの「＋New」ボタンの「Web Service」を選択。
②GitHubのリポジトリと連携させる設定画面が表示されるため、デプロイするアプリを選択し、「Connect」ボタンをクリック。
③「Name→アプリ名、Region→Ohio」に設定。
④「Build Command：./bin/render-build.sh、Start Command：bundle exec puma -C config/puma.rb」に記述変更。
⑤環境変数設定
→「RAILS_MASTER_KEY：アプリのマスターキーを入力」
→環境変数の入力欄追加
→左側「DATABASE_URL」、右側「Internal Database URL」の内容をコピペ
⑥「Deploy Web Service」のボタンをクリック
→ターミナルが表示され、デプロイ作業が開始されたらOK
→デプロイが完了すると、緑色のアイコンで「Live」と表示され、ターミナルに「Puma starting in cluster mode...」という文字が見える。
→デプロイ完了後、リンクをクリックし、挙動確認。
````
