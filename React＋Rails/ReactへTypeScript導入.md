# 背景
　react-scriptにてアプリの雛形を作成後にTypeScriptをインストールした際にVer5以上のTypeScriptでは、`依存関係の
警告（ERESOLVE overriding peer dependency）`が発生した。<br>

# 参考記事

[【React】react-scriptsからviteへの移行](https://zenn.dev/xronotech/articles/11e671bf0315e7 "【React】react-scriptsからviteへの移行")

# 条件
・react-scripts：バージョン5.0.1<br>
・TypeScript：バージョン5.6.3

## 事象おさらい　
・react-scriptsの最新バージョンは5.0.1を適用。<br>→TypeScript 5.x系を公式にはサポートをしていないため、競合発生。
<br><br>
**【解決案】**<br>
①TypeScriptのバージョンを4.x系にダウングレードする：<br>
　react-scripts@5.0.1はTypeScript 4.x系と互換性があるため、TypeScriptのバージョンを4.9.5などに下げることで依存関係の競合を解消できる。<br>
②react-scriptsの代替としてviteを使用する：<br>
　viteは高速なビルドツールであり、最新のTypeScriptバージョンにも対応している。create-react-appからの移行も比較的容易。<br>
<br>
→解決案②を実行。TypeScriptの学習も兼ねてアプリケーション作成を行う目的のため、バージョンの古いTypeScriptを使用するメリットを感じなかったため。

## 解決案②実行内容
（１）**参考記事**の実行<br>
　参考記事参照。<br>
（２）微修正<br>
　・`App.js`（元々作成していたもの）を`App.tsx`にリネーム<br>
　・`index.tsx`からAppを正しい拡張子でインポート
```index.tsx
import App from './App';
```
　・`viteサーバー`再起動
```
# ターミナル
npm start
```
