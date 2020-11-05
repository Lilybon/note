# 單例模式 (Singleton)
單例模式的定義是：
> *保證一個類別**僅有一個實例**，並提供一個存取它的**全域存取點***。

適用於確保實例或行為被執行的**唯一性**，實例一旦經由靜態方法創建過一次就不會再被重新指派，而是返回先前已創建的實例。

e.g. 執行緒池、全域快取、登入彈窗

![銅鑼灣只有一個浩南](https://i.imgur.com/83jm2aY.jpg)
### 實作單例模式
以 class 語法糖實作：
```javascript
// 實例被保存在類別函數 Singleton 的 instance 屬性中
class Singleton {
  constructor (name) {
    this.name = name
  }
  getName () {
    return this.name
  }
  static getInstance (name) {
    if (!this.instance) {
      // this 指向類別函數 Singleton
      this.instance = new Singleton(name)
    }
    return this.instance
  }
}
```
以 class 語法糖結合閉包實作：
```javascript
class Singleton {
  constructor (name) {
    this.name = name
  }
  getName () {
    return this.name
  }
  //IIFE
  static getInstance = (function () {
    // 實例被保存在 getInstance 的閉包環境裡的 instance 變數
    let instance = null
    return function (name) {
      if (!instance) {
        instance = new Singleton(name)
      }
      return instance
    }
  })()
}
```
兩者都可以得到如下的結果：
```javascript
const a = Singleton.getInstance('sven1')
const b = Singleton.getInstance('sven2')
console.log(a === b) //true
```
上述兩種實作的缺點是使用者**必須要知道此類別是一個單例類別**，且須要透過 `Singleton.getInstance` 呼叫才能拿到實例，不像一般創建類別用 `new Xxx` 的方式即可取回實例那麼直觀，增加了使用上的「**不透明性**」。

![old man shrugging shoulders](https://i.imgur.com/0S2YLHf.jpg)
### 透明的單例模式
為了讓使用者如同平常宣告使用的類別一樣直覺(也可以說是提升「**透明性**」)，可以透過 IIFE 將 instance 變數藏於閉包，並回傳真正的類別函數給使用者呼叫。
```javascript
// IIFE
const CreateDiv = (function () {
  let instance
  class CreateDiv {
    constructor (html) {
      // 保證只有一個物件
      if (instance) {
        return instance
      }
      // 建立物件、執行初始化
      this.html = html
      this.init()
      return instance = this
    }
    init () {
      const div = document.createElement('div')
      div.innerHTML = this.html
      document.body.append(div)
    }
  }
  return CreateDiv
})()

const a = new CreateDiv('sven1')
const b = new CreateDiv('sven2')
console.log(a === b) // true
```
但此設計仍然有瑕疵，建構子的設計不符合「**單一職責原則**」，假設不久的將來這個 `createDiv` 函數要被拿來重複利用、製造多的 div 元素時，上面保證只有一個物件的程式邏輯勢必要被從現在的建構子抽離出來。
### 用代理實作單例模式
回到前一節的透明單例上，我們把保證只有一個物件的邏輯抽掉，確保這個 `createDiv` 是一個可復用的類別。
```javascript
const CreateDiv = (function () {
  let instance
  class CreateDiv {
    constructor (html) {
      this.html = html
      this.init()
      return instance = this
    }
    init () {
      const div = document.createElement('div')
      div.innerHTML = this.html
      document.body.appendChild(div)
    }
  }
  return CreateDiv
})()
```
接下來引入代理類別 `ProxySingletonCreateDiv`
```javascript
// IIFE
const ProxySingletonCreateDiv = (function () {
  let instance
  return function (html) {
    if (!instance) {
      instance = new CreateDiv(html)
    }
    return instance
  }
})()

const a = new ProxySingletonCreateDiv('sven1')
const b = new ProxySingletonCreateDiv('sven2')
console.log(a === b) // true
```
透過把判斷單例和儲存實例的邏輯統一由 `ProxySingletonCreateDiv` 類別管理，將 `CreateDiv` 權責單純化，兩者結合以達到單例模式的實現。
### 惰性單例
> *惰性單例指的是在**需要的時候**才建立物件實例。*

對於 JavaScript 這種事件驅動 (Event-driven) 的語言來說非常有用，有用程度超乎我們的想像，可以達到按需載入的效用。

e.g. 只在點擊了登入按鈕才渲染登入彈窗
```htmlembedded
<html>
  <body>
    <button id="login-button">登入</button>
  </body>
</html>
```
```javascript
const createLoginLayer = (function () {
  let div
  return function () {
    if (!div) {
      div = document.createElement('div')
      div.innerHTML = '我是登入彈窗！'
      div.style.display = 'none'
      document.body.appendChild(div)
    }
    return div
  }
})()
const loginButton = document.getElementById('login-button')
// 只有實際事件觸發事件時才建立 DOM
loginButton.onClick = function () {
  const loginLayer = createLoginLayer()
  loginLayer.style.display = 'block'
}
```
### 通用惰性單例
透過 JavaScript **閉包**和函數可作為**頭等公民**被當成參數傳遞的特性我們可以客製化自己想要的單例。
```javascript
const Singleton = function (fn) {
  let result // 用於保存實例或行為完成的註記
  return function () {
    return result || (result = fn(...arguments))
  }
}
```
透過改寫前面一節的 `createLoginLayer` ，用單例模式確保 DOM 物件**實例的唯一性**。
```javascript
const createLoginLayer = function () {
  const div = document.createElement('div')
  div.innerHTML = '我是登入彈窗！'
  div.style.display = 'none'
  document.body.appendChild(div)
  // 用 DOM 表達實例被創建的唯一性
  return div
}

const createSingleLoginLayer = Singleton(createLoginLayer)

const loginButton = document.getElementById('login-button')
loginButton.onClick = function () {
  const loginLayer = createSingleLoginLayer()
  loginLayer.style.display = 'block'
}
```
也可以確保像是綁定事件等**行為被執行的唯一性**。
```javascript
const bindEvent = function () {
  const div1 = document.getElementById('div1')
  div1.onClick = function () {
    console.log('綁定完成')
  }
  div1.style.display = 'none'
  document.body.appendChild(div1)
  // 用布林值 true 表達行為被執行的唯一性
  return true
}

const bindSingleEvent = Singleton(bindEvent)

// 僅首次執行觸發事件綁定
bindSingleEvent()
bindSingleEvent()
bindSingleEvent()
```
### 小結
JavaScript 相較於其他編譯式語言，可以不用依賴類別堆疊，以**閉包**和**高階函數**的概念即可優雅設計出高彈性的單例；且實務上常會遇到必須懶載入的場景，惰性單例可以避免初始化時不必要的開銷，還可以確保全域物件的唯一性。

![spiderman pointing](https://i.imgur.com/3412GeT.jpg =500x)
###### tags: `JavaScript 設計模式與開發實踐` `設計模式` `JavaScript`