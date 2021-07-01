# The Internals of Node

### Node 如何被執行?

以執行 pbkdf2 為例，我們會經歷以下六個歷程。

1. **我們撰寫的 JavaScript**

```javascript
app.get("/", (req, res) => {
  crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
    res.send("Hello World");
  });
});
```

2. **lib 的原始碼** (JavaScript 函式庫)

```javascript
// lib/internal/crypto/pbkdf2.js
const { validateCallback } = require("internal/validators");

function pbkdf2(password, salt, iterations, keylen, digest, callback) {
  // 檢驗參數是否符合格式
  if (typeof digest === "function") {
    callback = digest;
    digest = undefined;
  }

  ({ password, salt, iterations, keylen, digest } = check(
    password,
    salt,
    iterations,
    keylen,
    digest
  ));

  validateCallback(callback);

  // 透過 internalBinding 呼叫 C++ 函式庫
  // ...
}
```

3. **internalBinding** (連接 JavaScript 和 C++ 的橋樑)

```javascript
// lib/internal/crypto/pbkdf2.js
const { PBKDF2Job } = internalBinding("crypto");

function pbkdf2(password, salt, iterations, keylen, digest, callback) {
  // 檢驗參數是否符合格式
  // ...

  // 透過 internalBinding 呼叫 C++ 函式庫
  const job = new PBKDF2Job(
    kCryptoJobAsync,
    password,
    salt,
    iterations,
    keylen,
    digest
  );

  const encoding = getDefaultEncoding();
  job.ondone = (err, result) => {
    if (err !== undefined) return FunctionPrototypeCall(callback, job, err);
    const buf = Buffer.from(result);
    if (encoding === "buffer")
      return FunctionPrototypeCall(callback, job, null, buf);
    FunctionPrototypeCall(callback, job, null, buf.toString(encoding));
  };

  job.run();
}
```

4. **V8** (轉換 JavaScript 和 C++ 間的變數)

5. **src 的原始碼** (C++ 函式庫)

6. **libuv** (提供 Node 接觸底層 OS 的權限)

### Node Event Loop

```javascript
const pendingTimers = [];
const pendingOSTasks = [];
const pendingOperations = [];

// New timers, tasks, operations 會在 myFile 被執行時被記錄下來
myFile.runContents();

function shouldContinue() {
  /*
   * 檢查1: 任何待處理的 setTimeout, setInterval, setImmediate?
   * 檢查2: 任何待處理的 OS tasks? (Like server listening to port)
   * 檢查3: 任何待處理的 long running operations? (Like fs module)
   * */
  return (
    pendingTimers.length || pendingOSTasks.length || pendingOperations.length
  );
}

// 整段程式在一個'tick'被執行的情境
while (shouldContinue()) {
  // 1) Node 看向 pendingTimers 並檢查是否有函式準備好被執行 (setTimeout, setInterval)
  // 2) Node looks at pendingOSTasks and pendingOperations and calls relevant callbacks
  // 3) 暫停執行，只在有以下情況下繼續執行...
  // - 一個新的 pendingOSTask 完成
  // - 一個新的 pendingOperation 完成
  // - 一個 timer 準備要完成
  // 4) 看向 pendingTimers. 呼叫任何 setImmediate
  // 5) 處理任何 'close' 事件
}

// 退出返回 terminal
```

### Node 是單線程嗎?

Node **Event Loop** 是單線程。

**有一些** Node **框架/Std Lib 不是**單線程。

### Libuv Thread Pool

```javascript
// 調整 libuv thread pool 大小
process.env.UV_THREADPOOL_SIZE = 2;

const crypto = require("crypto");

crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
  console.log("1:", Date.now() - start);
});

crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
  console.log("2:", Date.now() - start);
});

crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
  console.log("3:", Date.now() - start);
});

crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
  console.log("4:", Date.now() - start);
});

crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
  console.log("5:", Date.now() - start);
});
```

結果可能會隨你電腦 CPU 的核心數、 **libuv thread pool** 的大小而影響，不過重要的是 Node Event Loop 指派任務是單線程，但是底層的 C++ 被執行則可能是多線程。

以下提供 task 的行進路線幫助理解底層的運作機制：
Task → Thread Pool → OS Task Scheduler → CPU Core

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript`
