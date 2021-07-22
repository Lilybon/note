# Enhancing Node Performance

### Blocking in Event Loop

以下為在 server 底下模擬一個耗時的 task 的範例。

```javascript
// index.js
const express = require("express");
const app = express();

function doWork(duration) {
  const start = Date.now();
  while (Date.now() - start < duration) {}
}

app.get("/", (req, res) => {
  doWork(5000);
  res.send("Hi there");
});

app.listen(3000);
```

### Benchmarking Server Performance

使用 **Apache Benchmark** 對 API 進行壓力測試<br/>
**-c** concurrency 表示**併發請求數**<br/>
**-n** requests 表示**請求總數**

```bash
ab -c 50 -n 500 localhost:3000/fast
```

### Clustering

以下為手動設置 Cluster 的範例。

```javascript
// index.js
// 一個 child 有一個 thread 可使用。
process.env.UV_THREADPOOL_SIZE = 1;
const cluster = require("cluster");

// index.js 是在 master 模式下被執行的嗎?
if (cluster.isMaster) {
  // 觸發 index.js **再次**被執行，但是在 child 模式下被執行。
  cluster.fork();
} else {
  // child 模式，這邊負責當 server ，除此之外沒處理別的事情。
  const express = require("express");
  const crypto = require("crypto");
  const app = express();

  app.get("/", (req, res) => {
    crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
      res.send("Hi there");
    });
  });

  app.listen(3000);
}
```

注意，並不是無腦地增加 child instance 就可以優化效能，<br/>
硬體是有極限的，本質上你必須考量電腦 CPU 最多能同時處理多少任務。<br/>
ex: 就算 1 個 child 開到 6 個 thread ，在雙核心的電腦下同時只能處理最多 2 個 thread 的任務就毫無意義，還不如開 2 個 thread 的效能來得快。

### PM2

```
yarn global add pm2
```

以下為用 PM2 設置 Cluster 的範例，<br/>
把前面一節有關 Cluster 的程式碼清掉。

```javascript
// index.js
const express = require("express");
const crypto = require("crypto");
const app = express();

app.get("/", (req, res) => {
  crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
    res.send("Hi there");
  });
});

app.listen(3000);
```

```bash
# 設置 -i 0 表示創建等同於邏輯核 (Locgical CPU Cores) 數量的 Cluster
pm2 start index.js -i 0

# 刪除所有 index Cluster
pm2 delete index

# 檢視所有 Cluster
pm2 list

# 檢視所有 index Cluster 的詳情
pm2 show index

# 檢視所有 Cluster 的 Log
pm2 monit
```

### Worker Threads

```bash
yarn add webworker-thread
```

```javascript
const express = require("express");
const crypto = require("crypto");
const app = express();
const Worker = require("webworker-threads").Worker;

app.get("/", (req, res) => {
  /*
   * 注意!
   * 1. 這個 worker 的 callback 不要牽涉到外面 scope 的變數，
   *    你應該把這個 worker 視為分離於主線程的另一個程序看待。
   * 2. 不要使用 arrow function，否則無法使用 this.onMessage
   *    接主線程的 postMessage 。
   * */
  const worker = new Worker(function () {
    this.onmessage = function () {
      let counter = 0;
      while (counter < 1e9) {
        counter++;
      }

      postMessage(counter);
    };
  });

  worker.onmessage = function (message) {
    console.log(message.data);
    res.send("" + message.data);
  };

  worker.postMessage();
});

app.listen(3000);
```

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript`
