# HTTPメソッド関連エラー（Routing Error）

### ・事象：ログアウトボタンのHTTPメソッドにdeleteを指定しているが、認識されずGETで取得される。
#### エラー発生時のコード
````
application.html.erb

<%= link_to "ログアウト", destroy_user_session_path, method: :delete , class: :nav__logout %>
````
````
rails routes
 Prefix               Verb   URI Pattern                  Controller#Action
 destroy_user_session DELETE /users/sign_out(.:format)    devise/sessions#destroy
````

#### 調べた内容
１）JavaScriptのライブラリが正しく読み込まれていないことが原因であること。<br>
２）HTTPメソッドの指定の記述がRailsのVerごとで違いがあること。

#### 試した内容
・１）の試した内容<br>
rails-ujsまたはjquery-ujsというJavaScriptライブラリが必要であるところ、適用されていないため、発生している可能性がある。
以下コード追記を実施。
````
application.js
import "@hotwired/turbo-rails"
import "controllers"
import Rails from "@rails/ujs";
Rails.start();
#上記2行追記
````
→解消せず。<br>
<br>
・２）の試した内容<br>
１）の記述はそのままで、HTTPメソッドの記述を変更。<br>
`method: :delete`→`data: { turbo_method: :delete }`
````
application.html.erb
<%= link_to "ログアウト", destroy_user_session_path, data: { turbo_method: :delete }, class: :nav__logout %>
````
→解消せず。<br>
<br>
#### ・１）の追記部分のみ削除
#### →解消。ログアウト可能になる。
<br>
<br>

#### エラー発生原因分析
・前提として、RailsのVerによって、記述が異なる部分がある。<br>
今回でいえば、`２）の試した内容`が該当する。<br>
また、１）にて追記した`ujs`については、Rails 7.0.0以前の記述設定であり、Rails 7.0.0以降では、`import "@hotwired/turbo-rails"`にて`turbo`を設定・使用するものとなっている。<br>
・コードは上から順番にコードがプログラムされるため、`application.js`に古いVerの記述設定をしてしまい、`turbo`が`ujs`に上書きされている状況であった。<br>
<br>
### 教訓
・verごとの記述設定など、異なる記述や設定があることの理解・把握ができた。<br>
・ChatGPTや参考文献を元にエラー解消する際は、使用中のフレームワークのverや導入したgemのverを把握した上で、トラシューしないと一生エラーは解消できない。<br>
