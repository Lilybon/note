# Automated Headless Browser Testing

### Global Jest Setup

可以在進行所有測試前對 mongoose 做最基本的 setup。

Setup

```javascript
// test/setup
// 設定每個 test 的逾時秒數，超過該時現限者會被判定為失敗
jest.setTimeout(30000)

require('../models/User')

const mongoose = require('mongoose')
const keys = require('../config/keys')

mongoose.Promise = global.Promise
mongoose.connect(keys.mongoURI, { useMongoClient: true })
```

package.json

```json
{
  "jest": {
    "setupTestFrameworkScriptFile": "./test/setup.js"
  }
}
```

### Session Signatures

簽章 sig 的存在主要是避免使用者隨意篡改 session 的資料，若有惡意使用者在不知道 sig 是透過什麼 secret 被 hash 出來的情況下竄改 session，他終究會在審核身份的 middleware 被阻擋。
```
set-cookie:session=SESSION; path=/; expires=Fri 23, July 2021 21:40:35 GTM; httponly
set-cookie:session.sig=SESSION_SIGNATURE; path=/; expires=Fri 23, July 2021 21:40:35 GTM; httponly
```

Session + Cookie Signing Key = Session Signature

```javascript
const Keygrip = require('keygrip')

// 考量安全性，cookieSigningKey 通常會被放在專案的 secret 避免被駭
const cookieSigningKey = 'this is a secret'
const keygrip = new Keygrip([cookieSigningKey])

const session = 'SESSION'
const sessionSignature = keygrip.sign('session=' + session)
const maliciousSessionSignature = 'surprise motherf*cker'

// 判斷是否為合法使用者
keygrip.verify('session=' + session, sessionSignature) // true
keygrip.verify('session=' + session, maliciousSessionSignature) // false
```

### Factory Functions

User Factory - 利用 mongoose 製造假的使用者

```javascript
// test/factories/userFactory
const mongoose = require('mongoose')
// 由於 test setup 已處理過 mongo 連線且載入 User model 所以不會報錯
const User = mongoose.model('User')

module.export = () => new User({}).save()
```

Session Factory - 利用假的使用者製造假的 sig 和 session

```javascript
// test/factories/sessionFactory
const Buffer = require('safe-buffer').Buffer
const Keygrip = require('keygrip')
const kes = require('../../config/keys')
const keygrip = new Keygrip([keys.cookieKey])

module.export = (user) => {
  const sessionObject = {
    passport: {
      user: user._id.toString()
    }
  }
  const session = Buffer
    .from(JSON.stringify(sessionObject))
    .toString('base64')
  const sig = keygrip.sign('session=' + session)
  
  return { session, sig }
}
```

### Helper Functions

若想**避免劫持造成第三方套件的 `Class` 被污染**，可以嘗試使用 `Class` 並回傳一個 `Proxy` 決定讀取 property 的優先級。BTW，原型鏈其實也可以達到相近的效果，不過最大的差別是用 Proxy 你可以**自訂義物件的優先級**，即使沒上下關係的物件也可以被列入參考；反之，原型鏈的話必須順著繼承的順序尋找 property。

Page

```javascript
// test/helpers/page
const puppeteer = require('puppeteer')
const userFactory = require('../factories/userFactory')
const sessionFactory = require('../factories/sessionFactory')

class CustomPage {
  static async build () {
    const browser = await puppeteer.launch({
      headless: false
    })
    
    const page = await browser.newPage()
    const customPage = new CustomPage(page)
    
    return new Proxy(customPage, {
      get: function (target, property) {
        return customPage[property] || browser[property] || page[property]
      }
    })
  }

  constructor (page) {
    this.page = page
  }
  
  login () {
    const user = await userFactory()
    const { session, sig } = sessionFactory(user)
  
    await this.page.setCookie({ name: 'session', value: session })
    await this.page.setCookie({ name: 'session.sig', value: sig })
    await this.page.goto('localhost:3000/blogs')
    // 等指定 selector 的 element 生成後才繼續執行
    await this.page.waitFor('a[href="/auth/logout"]')
  }
  
  async getContentsOf (selector) {
    return this.page.$eval(selector, el => el.innerHTML)
  }
  
  get (path) {
    return this.page.evaluate(
      _path =>
        fetch({
          path: _path,
          method: 'GET',
          credentials: 'same-origin',
          headers: {
            'Content-Type': 'application/json'
          }
        })
          .then(res => res.json()),
      path
    )
  }
  
  post (path, data) {
    return this.page.evaluate(
      (_path, _data) => 
        fetch({
          path: _path,
          method: 'POST',
          credentials: 'same-origin',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(_data)
        })
          .then(res => res.json()),
      path,
      data
    )
  }

  execRequests (actions) {
    return Promise.all(
      actions.map(({ method, path, data }) => this[method](path, data))
    )
  }
}

module.export = CustomPage
```

### Practice

Header Test

```javascript
// test/helpers/header.test
const Page = require('./helpers/page')

let page

beforeEach(async () => {
  page = await Page.build()
  await page.goto('localhost:3000')
})

afterEach(async () => {
  await page.close()
})

test('the header has the correct text', async () => {
  const text = await page.getContentsOf('a.brand-logo')
  expect(text).toEqual('Blogster')
})

test('clicking login starts oauth flow', async () => {
  await page.click('.right a')

  const url = await page.url()
  
  expect(url).toMatch(/accounts\.google\.com/)
})

test('When signed in, shows logout button', async () => {
  await page.login()
  
  const text = await page.getContentsOf('a[href="/auth/logout"]')
  
  expect(text).toEqual('Logout')
})
```

Blogs Test

```javascript
const Page = require('./helpers/page')

let page

beforeEach(async () => {
  page = await Page.build()
  await page.goto('localhost:3000')
})

afterEach(async () => {
  await page.close()
})

describe('When logged in', async () => {
  beforeEach(() => {
    await page.login()
    await page.click('a.btn-floating')
  })

  test('can see blog create form', async () => {
    const label = await page.getContentsOf('form label')
  
    expect(label).toEqual('Blog Title')
  })
  
  describe('And using valid inputs', async () => {
    beforeEach(async () => {
      // 模擬輸入表單並提交
      await page.type('.title input', 'My Title')
      await page.type('.content input', 'My Content')
      await page.click('form button')
    })
    
    test('Submitting takes user to review screen', async () => {
      const text = await page.getContentsOf('h5')
      
      expect(text).toEqual('Please confirm your entries')
    })
    
    test('Submitting then saving add blog to index page', async () => {
      await page.click('button.green')
      await page.waitFor('.card')
      
      const title = await page.getContentsOf('.card-title')
      const content = await page.getContentsOf('p')
      
      expect(title).toEqual('My Title')
      expect(content).toEqual('My Content')
    })
  })
  
  describe('And using invalid inputs', async () => {
    beforeEach(async () => {
      await page.click('form button')
    })
    
    test('The form shows an error message', async () => {
      const titleError = await page.getContentsOf('.title .red-text')
      const contentError = await page.getContentsOf('.content .red-text')
      
      expect(titleError).toEqual('You must provide a value')
      expect(contentError).toEqual('You must provide a value')
    })
  })
})

describe('User is not logged in', async () => {
  const actions = [
    {
      method: 'get',
      path: '/api/blogs/'
    },
    {
      method: 'get',
      path: '/api/blogs/',
      data: {
        title: 'T',
        content: 'C'
      }
    }
  ]

  test('Blog related actions are prohibited', async () => {
    // 模擬進入頁面後直接打請求
    const results = await page.execRequests(actions)

    for (let result of results) {
      expect(result).toEqual({ error: 'You must log in!' })
    }
  })
})
```

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript`
