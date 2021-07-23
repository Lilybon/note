# Automated Headless Browser Testing

### Get Started

```javascript
const pupeteer = require('puppeteer')

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
})
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

###### tags: `Node JS: Advanced Concepts` `NodeJS` `JavaScript`
