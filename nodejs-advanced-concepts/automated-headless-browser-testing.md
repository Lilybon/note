# Automated Headless Browser Testing

### Global Jest Setup

Setup

```javascript
// test/setup
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

User Factory

```javascript
// test/factories/userFactory
const mongoose = require('mongoose')
// 由於 test setup 已處理過 mongo 連線所以不會報錯
const User = mongoose.model('User')

module.export = () => new User({}).save()
```

Session Factory

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

### Practice

```javascript
const pupeteer = require('puppeteer')
const userFactory = require('./factories/userFactory')
const sessionFactory = require('./factories/sessionFactory')

let browser, page

beforeEach(async () => {
  browser = await pupeteer.launch({
    headless: false,
  })
  page = await browser.newPage()
  await page.goto('localhost:3000')
})

afterEach(async () => {
  await browser.close()
})

test('the header has the correct text', async () => {
  const text = await page.$eval('a.brand-logo', (el) => el.innerHTML)
  expect(text).toEqual('Blogster')
})

test('clicking login starts oauth flow', async () => {
  await page.click('.right a')

  const url = await page.url()
  
  expect(url).toMatch(/accounts\.google\.com/)
})

test('When signed in, shows logout button', async () => {
  const user = await userFactory()
  const { session, sig } = sessionFactory(user)
  
  await page.setCookie({ name: 'session', value: session })
  await page.setCookie({ name: 'session.sig', value: sig })
  await page.goto('localhost:3000')
  // 等指定 selector 的 element 生成後才繼續執行
  await page.waitFor('a[href="/auth/logout"]')
  
  const text = await page.$eval('a[href="/auth/logout"]', el => el.innerHTML)
  
  expect(text).toEqual('Logout')
})
```

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript`
