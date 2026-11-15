# パッと算数：セットアップ手順

`index.html` 1ファイルだけで動きます。
最初の `FIREBASE_CONFIG = null` のままなら「この端末だけ」モード（名前を選ぶ方式、記録は端末内に保存）です。
Googleログインで子どもを判別したい場合は、以下を行ってください。

## 1. Firebaseプロジェクト作成（無料のSparkプランでOK）
1. https://console.firebase.google.com でプロジェクトを作成（Analyticsは不要）
2. 「構築 → Authentication → ログイン方法」で **Google** を有効化
3. 「構築 → Firestore Database」を作成（ロケーションは `asia-northeast1`）
4. 「プロジェクトの設定 → マイアプリ → ウェブ（</>）」でアプリを登録し、表示される `firebaseConfig` をコピー
5. `index.html` の `const FIREBASE_CONFIG = null;` をコピーした内容に置き換える

## 2. Firestoreルール
「Firestore → ルール」に貼り付けて公開します。
メールアドレスはこのリポジトリ（公開）には書かず、Firebase管理画面にだけ書いてください。
`PARENT_EMAIL_*` と `CHILD_EMAIL_*` は実際のGoogleアカウントに置き換えます。

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function verified() {
      return request.auth != null && request.auth.token.email_verified == true;
    }
    function isParent() {
      return verified() && request.auth.token.email in ['PARENT_EMAIL_1', 'PARENT_EMAIL_2'];
    }
    function allowed() {
      return isParent() || (verified() && request.auth.token.email in ['CHILD_EMAIL_1', 'CHILD_EMAIL_2']);
    }
    match /users/{uid}/sessions/{sessionId} {
      allow create: if allowed() && request.auth.uid == uid;
      allow read: if (allowed() && request.auth.uid == uid) || isParent();
      allow update, delete: if false;
    }
    match /users/{uid}/state/{docId} {
      allow get: if (allowed() && request.auth.uid == uid) || isParent();
      allow create, update: if allowed() && request.auth.uid == uid
        && request.resource.data.keys().hasOnly(['data', 'updated']);
      allow delete: if false;
    }
    match /profiles/{uid} {
      allow create, update: if allowed() && request.auth.uid == uid
        && request.resource.data.keys().hasOnly(['name', 'email', 'updated'])
        && request.resource.data.email == request.auth.token.email;
      allow get: if (allowed() && request.auth.uid == uid) || isParent();
      allow list: if isParent();
      allow delete: if false;
    }
  }
}
```

`users/{uid}/state/progress` は、復習する問題と「おまかせ」のレベルを保存します。

親用ビュー: 親のアカウントでログインすると、ホームに「親用ビュー」が出ます（`profiles` の一覧を読めるのは親だけです）。

## 3. 公開（Firebase Hosting）
```
npm i -g firebase-tools
firebase login
firebase init hosting   # public ディレクトリに index.html を置く / SPA設定は No
firebase deploy
```
GitHub Pagesで公開する場合は、Authentication →「設定」→「承認済みドメイン」に `ユーザー名.github.io` を追加します。

## 4. タブレット／Chromebookで使う
公開URLをChromeで開き、メニューから「ホーム画面に追加」（Chromebookは「インストール」）。

## 注意：子ども用Googleアカウント(Family Link)
外部アプリへのログインに保護者の承認が必要な場合があります。
ログインできないときは、ログイン画面の「ログインせず この端末だけで使う」で代用できます。
