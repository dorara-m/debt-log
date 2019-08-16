# mochi-mochi
## what?
### 構想
立て替え金記録アプリ。

### 由来
持ちつ持たれつ、より。

## hosting
https://mochi-mochi-demo.firebaseapp.com/

## Project setup
```
npm install
```

### Add your firebase-key
Firebase Realtime Databaseの使用を前提としている。
クローンして使う場合は、`config/key.js`をルート直下に追加すること。
```config/key.js
export default {
  apiKey: "",
  authDomain: "",
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
