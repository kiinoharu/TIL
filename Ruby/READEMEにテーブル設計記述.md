# READEMEにテーブル設計記述

### Typeの種類と解説

`Type`はカラムのデータ型を指定する。以下が主な種類。<br>
・string：文字列を格納する。通常、255文字までの短いテキストに使用する。（例: ユーザー名、メールアドレス）<br>
・text：長文テキストを格納する。（例: 記事本文、コメント）<br>
・integer：整数を格納する。（例: 年齢、ID）<br>
・float：浮動小数点数を格納する。（例: 小数点を含む数値、価格）<br>
・decimal：精度が重要な数値（小数点を含む）を格納する。（例: 金額計算）<br>
・boolean：`true`または`false`の値を格納する。（例: 有効フラグ、公開/非公開）<br>
・date：日付を格納する。（例: 誕生日、イベントの日付）<br>
・time：時間を格納する。（例: 開始時間、終了時間）<br>
・datetime：日時を格納する。（例: 作成日時、更新日時）<br>
・references：外部キー用に使用され、他のテーブルのレコードと関連付けるために使用する。（例: user_id）<br>

### Optionsの種類と解説

`Options`はカラムに対して適用する制約やオプションを指定する。<br>
・null: false：このかたむは必ず値が入力されなければならないことを意味する。（NOT NULL制約）<br>
・unique: true：このカラムの値はテーブル内で一意でなければならないことを意味する。（ユニーク制約）<br>
・default: “値”：カラムにデフォルトの値を設定する。（例: `default: "guest”`）<br>
・index: true：このカラムにインデックスを付与し、検索を高速化する。（例:メールアドレスやIDに適用）<br>
・foreign_key: true：このカラムが外部機ーであることを指定し、他のテーブルのレコードと関連付ける。<br>

### 具体例

以下に`user`テーブルのカラム例を示す。

| Column       | Type        | Options | 
| ----------   | --------- | ----------------------------- | 
| email           | string       | null: false, unique: true | 
| age             | integer     | null: false, default: 0 | 
| admin.        | boolean   | null: false, default: false | 
| created_at | datetime | null: false |

### 解説:

・`email`カラムは必須で一意（unique）であること。<br>
・`age`カラムは整数で、デフォルト値が0。<br>
・`admin`カラムはブール型で、デフォルトで`false`。<br>
・`create_at`カラムは日時型で、必ず値が必要。<br>
