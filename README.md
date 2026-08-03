# マイ本棚アプリ セットアップ手順

`index.html` 1ファイルで動く、スマホのブラウザから使う書籍管理アプリです。
カメラでISBNバーコードを読み取り、openBD / Google Books から書籍情報を取得して本棚に追加できます。データはFirebase(Firestore)に保存するため、自宅PCとスマホなど複数端末で共有・リアルタイム同期されます。

カメラ機能はブラウザの仕様上 **HTTPS(またはlocalhost)でしか動作しません**。そのため、単にファイルをダブルクリックして開くだけでは動きません。下記の手順でFirebase設定 → 公開(ホスティング)まで行ってください。

---

## 1. Firebaseプロジェクトを作成する

1. https://console.firebase.google.com/ にアクセスし、Googleアカウントでログイン
2. 「プロジェクトを追加」→ 任意のプロジェクト名(例: `my-bookshelf`)を入力して作成
   - Googleアナリティクスは不要なのでオフでOK

## 2. Firestore Database を有効化する

1. 左メニュー「構築」→「Firestore Database」→「データベースの作成」
2. ロケーションは `asia-northeast1` (東京) など任意のものを選択
3. 「本番環境モードで開始」を選択
4. 作成後、「ルール」タブを開き、以下に置き換えて「公開」

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /books/{bookId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

このルールは「匿名認証を含め、ログイン済みのアクセスのみ許可」する設定です。URLを知らない第三者からの読み書きを防ぎます。

## 3. 匿名認証(Anonymous Auth)を有効化する

1. 左メニュー「構築」→「Authentication」→「Sign-in method」
2. 「匿名」を選択して有効化

## 4. Webアプリを追加して設定値(firebaseConfig)を取得する

1. プロジェクト概要の歯車アイコン →「プロジェクトの設定」
2. 「マイアプリ」→ `</>`(ウェブ)アイコンをクリックしてアプリを登録(名前は任意)
3. 表示される `firebaseConfig` オブジェクトをコピー

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "xxxx.firebaseapp.com",
  projectId: "xxxx",
  storageBucket: "xxxx.appspot.com",
  messagingSenderId: "...",
  appId: "..."
};
```

## 5. index.html に設定値を埋め込む

`index.html` 内の以下の箇所を、手順4でコピーした値に置き換えてください。

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

## 6. HTTPSで公開する(GitHub Pagesの例)

CLIのインストール不要で一番簡単な方法として GitHub Pages を紹介します。無料のGitHubアカウントがあれば、以下の手順のみで完了します。

### 6-1. GitHubアカウントの準備

すでにアカウントがあればスキップしてください。なければ https://github.com/ の「Sign up」から無料登録します(メールアドレスとパスワードのみでOK)。

### 6-2. リポジトリを作成する

1. 右上の「+」→「New repository」
2. Repository name に任意の名前を入力(例: `my-bookshelf`)
3. 公開設定は **Public** を選択
   - GitHub Pagesは無料プランではPublicリポジトリのみ対応です。Privateで公開したい場合はGitHub Pro等の有料プランが必要になります
   - リポジトリがPublicでも、`index.html` の中身(Firebase設定値含む)は誰でも閲覧できてしまう点に注意してください。Firestoreのルール側(手順2)で認証必須にしているため実害は限定的ですが、気になる場合は後述の「補足」を参照してください
4. 「Create repository」をクリック

### 6-3. index.html をアップロードする

1. 作成したリポジトリのトップページで「Add file」→「Upload files」をクリック
2. `index.html` をドラッグ&ドロップ(またはファイル選択)
3. 画面下部の「Commit changes」をクリックしてアップロード完了

### 6-4. GitHub Pagesを有効化する

1. リポジトリ上部メニューの「Settings」タブを開く
2. 左メニューの「Pages」を選択
3. 「Build and deployment」の Source を **Deploy from a branch** に設定
4. Branch を `main`、フォルダを `/ (root)` に設定して「Save」

### 6-5. 公開URLへアクセスする

1. 保存後、数十秒〜数分待つ(初回は少し時間がかかります)
2. 同じPages設定画面の上部に `https://<ユーザー名>.github.io/<リポジトリ名>/` という形式のURLが表示されるので、それをスマホのChrome/Safariで開きます
3. 開いた直後にカメラ利用の許可を求めるダイアログが出るので「許可」を選択してください

### 6-6. 更新方法

`index.html` を修正したい場合(例: firebaseConfigの再設定や機能追加)は、リポジトリの `index.html` ページを開き、鉛筆アイコン(Edit)から直接編集するか、再度「Upload files」で上書きアップロードし、コミットすれば数分でPagesに反映されます。

### 補足

- Firebase設定値(`apiKey`など)がリポジトリ内で公開状態になりますが、これはFirebaseの仕様上そもそも「公開されて問題ない値」として設計されています(実際のアクセス制御はFirestoreの「ルール」と認証で行う仕組みのため)。手順2のルール設定を必ず行ってください
- どうしてもコードを非公開にしたい場合は、GitHub ProプランでPrivateリポジトリ+Pagesを使うか、Netlify DropやVercel(いずれもUIからのアップロードのみで公開でき、Privateに近い扱いも可能)を検討してください

## 使い方

- トップページ: 登録済みの書籍一覧(未読了 / 読了で分かれています)
- 右下の📷ボタン: カメラを起動してISBNバーコード(裏表紙の13桁のバーコード)を読み取り
- 読み取り後、書籍情報が表示されるので内容を確認して「登録する」を押すと一覧に追加されます
- DBに情報が見つからない場合は手動でタイトル等を入力して登録できます
- 各書籍の「読了にする」ボタンで完了扱いに切り替え、「未読に戻す」で戻せます
- 「削除」で一覧から削除できます

## 注意事項

- Firestore・匿名認証はいずれも無料枠内で個人利用する分には十分な範囲です
- ルールで「認証済みなら誰でも読み書き可能」としているため、URLとFirebase設定を第三者に共有しないでください
- カメラ権限をブラウザで許可する必要があります(初回アクセス時にダイアログが出ます)
