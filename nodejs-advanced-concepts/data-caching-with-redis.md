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
   A: 主要來說更動頻率不高的 GET 最優先，其他不可逆操作如 POST、PATCH 等則不用。

### Practice

定義快取服務，主要功能有四個：

1. 自定義請求是否需被快取
2. 利用 user id 和 query key 當 hash 作簡易的快取
3. 具有時效性的快取
4. 清除對應 user 之快取

```javascript
// services/cache.js
const mongoose = require('mongoose')
const redis = require('redis')
const util = require('util')

const redisUrl = 'redis://127.0.0.1:6379'
const client = redis.createClient(redisUrl)
client.hget = util.promisify(client.hget)
// 透過裝飾者模式劫持 ORM
const exec = mongoose.Query.prototype.exec

// 用 function chaining 和物件變數設定請求是否需要被快取
mongoose.Query.prototype.cache = function (options = {}) {
  this.useCache = true
  this.hashKey = JSON.stringify(options.key || '')
  return this
}

mongoose.Query.prototype.exec = function () {
  if (!this.useCache) {
    return exec.apply(this, arguments)
  }

  // 用 query 參數跟 collection 作快取 key
  const key = JSON.stringify(
    Object.assign({}, this.getQuery(), {
      collection: this.mongooseCollection.name,
    })
  )

  const cacheValue = await client.hget(this.hashKey, key)

  if (cacheValue) {
    const doc = JSON.parse(cacheValue)

    // 如果 response 太 nested 可以考慮用遞迴遍歷處理
    return Array.isArray(doc)
      ? doc.map((d) => new this.model(d))
      : new this.model(doc)
  }

  const result = await exec.apply(this, arguments)

  client.hset(this.hashKey, key, JSON.stringify(result), 'EX', 10)

  return result
}

module.exports = {
  clearHash(hashKey) {
    client.del(JSON.stringify(hashKey))
  },
}
```

並設計一個 middleware 方面提供給 POST、PATCH、DELETE 等 API 完成後執行，<br/>
確保使用者下次拿 GET 不會拿到過時的資料。

```javascript
// middlewares/cleanCache
const { clearHash } = require('../services/cache')

module.exports = async (req, res, next) => {
  await next()

  clearHash(req.user.id)
}
```

API 使用快取和清除快取的範例：

```javascript
// routes/blogRoutes.js
const mongoose = require('mongoose')
const requireLogin = require('../middlewares/requireLogin')
const { cleanHash } = require('../middlewares/cleanCache')

const Blog = mongoose.model('Blog')

module.exports = (app) => {
  app.get('/api/blogs', requireLogin, async (req, res) => {
    const blogs = await Blog.find({ _user: req.user.id }).cache({
      key: req.user.id,
    })

    res.send(blogs)
  })

  app.post('/api/blogs', requireLogin, cleanCache, async (req, res) => {
    const { title, content } = req.body

    const blog = new Blog({
      title,
      content,
      _user: req.user.id,
    })

    try {
      await blog.save()
      res.send(blog)
    } catch (err) {
      res.send(400, err)
    }
  })
}
```

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript`
