# Data Caching with Redis

### Get Start

基本的 redis 指令如下：

```javascript
const redis = require('redis');
const redisUrl = 'redis://127.0.0.1:6379';
const client = redis.creatClient(redisUrl);

client.set('colors', { red: 'rojo' }); // 錯誤示範 - 不要用 set 存非純值，會被強制轉成非你預期的字串
client.get('colors', console.log); // null '[object Object]'

client.set('colors', JSON.stringify{ red: 'rojo' });
client.get('colors', console.log); // null '{"red":"rojo"}'

client.hset('spanish', 'red', 'rojo');
client.hget('spanish', 'red', (err, val) => console.log(val)); // rojo

client.hset('german', 'red', 'rot');
client.get('german', 'red', console.log); // null 'rot'

client.flushall(); // 清除全部資料
```

### Cache Solution

實際上在設計快取時需要考慮諸多情境：
1. Q: 如何通用地設定快取?
    A: 以**裝飾者模式**劫持 ORM
2. Q: 如何避免快取無限膨脹?
    A: 設定**快取時效**
3. Q: 如何設定快取的 Key ?
    A: 用 **SQL query** 或一些關鍵唯一的資訊 hash
4. Q: 哪些請求需要快取? 哪些不用?
    A: 待捕


```javascript
// cache.js
const mongoose = require('mongoose');
const redis = require('redis');
const util = require('util');

const redisUrl = 'redis://127.0.0.1:6379';
const client = redis.createClient(redisUrl);
client.get = util.promisify(client.get);
// 透過裝飾者模式劫持 ORM
const exec = mongoose.Query.prototype.exec;

mongoose.Query.prototype.exec = function () {
  const key = JSON.stringify(Object.assign({}, this.getQuery(), {
    collection: this.mongooseCollection.name
  }));
  
  const cacheValue = await client.get(key);
  
  if (cacheValue) {
    const doc = JSON.parse(cacheValue);

    // 如果 response 太 nested 可以考慮用遞迴遍歷處理
    return Array.isArray(doc)
      ? doc.map(d => new this.model(d))
      : new this.model(doc);
  }

  const result = await exec.apply(this, arguments);
  
  client.set(key, JSON.stringify(result));
  
  return result;
};
```

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript`