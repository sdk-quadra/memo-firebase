# Vue CLI memo app（firebase）
* Vue CLIのmemoアプリ
* 保存先はfirebaseです

## セットアップ
```
$ npm install
```

## firebase
* 自身のfirebase設定をしてください
```
$ npm install -g firebase-tools
```
* firebaseへのログイン
```
$ firebase login
```

## 環境変数
* top階層に`.env.development.local`を作成し、firebaseの情報を以下を記述してください
```
VUE_APP_API_KEY="****"
VUE_APP_PROJECT_ID="****"
VUE_APP_SENDER_ID="****"
VUE_APP_APP_ID="****"
VUE_APP_MEASUREMENT="****"
```

## 起動
```
$ npm run serve
```

## screenshot

![image](https://user-images.githubusercontent.com/39234092/105281792-43a38280-5bf0-11eb-8863-121134d4bde8.png)
