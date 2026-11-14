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
「Firestore → ルール」に貼り付けて公開：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    match /users/{uid}/sessions/{id} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

## 3. 公開（Firebase Hosting）
```
npm i -g firebase-tools
firebase login
firebase init hosting   # public ディレクトリに index.html を置く / SPA設定は No
firebase deploy
```
公開URLのドメインは Authentication の「承認済みドメイン」に自動で入ります。

## 4. タブレット／Chromebookで使う
公開URLをChromeで開き、メニューから「ホーム画面に追加」（Chromebookは「インストール」）。

## 注意：子ども用Googleアカウント(Family Link)
外部アプリへのログインに保護者の承認が必要な場合があります。
ログインできないときは、ログイン画面の「ログインせず この端末だけで使う」で代用できます。
