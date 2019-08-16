# mochi-mochi
## overview
立て替え金記録アプリ。
「いつ、AさんがBさんの分、何に、いくら払ったか」を記録。またそれを編集、削除、絞り込みが可能。

## demo
https://mochi-mochi-demo.firebaseapp.com/

## Project setup
Vue CLIをベースとして制作。

```
npm install
```

### Add your firebase-key
Firebase Realtime Databaseの使用を前提としている。
クローンして使う場合は、`config/key.js`をルート直下に追加すること。内容は下記のように。
```
export default {
  apiKey: "[your apiKey]",
  authDomain: "[your authDomain]",
  …
}
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
