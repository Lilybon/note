# Wiring Up Continuous Integration

### Project Structure

以下是一個單純的前後端的架構：<br/>
一個 server project (Express) 底下包著 client project (ReactJS)<br/>
它們各自擁有獨立的 package.json

```
project
   ├── client
   ├──   ├── package.json
   │     └── ...
   ├── package.json
   ├── .travis.yaml
   └── ...
```

project/package.json

```json
{
  "script": {
    "start": "node index.js",
    "build": "NPM_CONFIG_PRODUCTION=false npm install --prefix client npm run build --prefix client"
  }
}
```

project/client/package.json

```json
{
  "script": {
    "build": "react-scripts build"
  }
}
```

**Dev 執行環境**
Browser <- React-Server (Port:3000) <- Express API (Port:5000)

**Prod 和 CI 執行環境**
Browser <- Express API (Port: 3000) & Static React Build File

### Travis YAML Setup

.travis.yaml

```yaml
language: node_js
node_js:
  - 14
dist: trusty
services:
  - mongodb
  - redis-server
env:
  - NODE_ENV=ci PORT=3000
cache:
  directories:
    - node_modules
    - client/node_modules
install:
  - npm install
  - npm run build
script:
  - nohup npm run start &
  - sleep 3
  - npm run test
```

### Server Configuration

1. 整理各環境的 key file 。
2. 確定 key file 有設定 redis URL 和 serving port 。
3. 確定 production 跟 ci 的情境下 express 提供打包好的前端專案。

index.js

```javascript
if (['production', 'ci'].includes(process.env.NODE_ENV)) {
  app.use(express.static('client/build'))
}
```

### Puppeteer Configuration

1. browser 的設定修正 - headless 改為 true ， args 加上 `--no-sandbox`
2. 測試路由路徑修正 − 避免測試在機器上沒辦法順利進行，ex: `localhost:3000/blogs` 改成`http://localhost:3000/blogs`

### Github & Travis

1. 用你的 Github 帳號登入 https://www.travis-ci.com/ 並允許 Travis 對你的 Github 存取權限。
2. 開啟 dashboard 設定哪個 repo 需要啟用 CI 服務。

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript` `Travis CI`
