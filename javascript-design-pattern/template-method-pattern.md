# 範本方法模式(Template Method)

![TemplateMethodDesignPattern](https://dz2cdn1.dzone.com/storage/temp/14064500-templatemethoddesignpattern.png =600x)

範本方法模式的定義是：<br />

> _一種只需要繼承就可以實作的非常簡單的模式，由兩個部份結構組成，第一部分是抽象父類別，第二部分是具體的實作子類別。_

假設我們有兩個平行的子類別，各子類別分別有一些相同的行為，為了避免重複，可以將相同的行為的實作搬到父類別，不同的行為則由子類別實作，很好地體現泛化的思想。

### Coffee or Tea

![wrong it's tea](https://i.kym-cdn.com/photos/images/facebook/001/851/747/502 =300x)

這邊直接引用書上的例子，實際感受一下這兩個類別的異同之處。

下面是一個咖啡類別 `Coffee`：

```javascript
class Coffee {
  boilWater() {
    console.log('把水煮沸')
  }
  brewCoffeeGriends() {
    console.log('用沸水沖泡咖啡')
  }
  pourInCup() {
    console.log('把咖啡倒進杯子')
  }
  addSugarAndMilk() {
    console.log('加糖和牛奶')
  }
  init() {
    this.boilWater()
    this.brewCoffeeGriends()
    this.pourInCup()
    this.addSugarAndMilk()
  }
}

const coffee = new Coffee()
coffee.init()
```

下面是一個茶類別 `Tea`：

```javascript
class Tea {
  boilWater() {
    console.log('把水煮沸')
  }
  steepTeaBag() {
    console.log('用沸水浸泡茶葉')
  }
  pourInCup() {
    console.log('把茶水倒進杯子')
  }
  addLemon() {
    console.log('加檸檬')
  }
  init() {
    this.boilWater()
    this.steepTeaBag()
    this.pourInCup()
    this.addLemon()
  }
}

const tea = new Tea()
tea.init()
```

嗯...有在寫程式的工程師們應該已經開始頭痛了，這兩個類別相似的點也太多了吧，先冷靜分析一下兩者的異同－除了把 `boilWater` 相同以外，其他三個方法可以被抽象化成意近的函式，並由子類別實作。

開始以範本方法重構吧～<br />
首先，定義抽象父類別 `Beverage` 並將相同的方法統一放置：

```javascript
class Beverage {
  boilWater() {
    console.log('把水煮沸')
  }
  // brewCoffeeGriends 和 steepTeaBag 被抽象成「泡」的動作
  brew() {}
  // pourInCup 取得不錯不用改 不過泡法不同依然要由子類別實作
  pourInCup() {}
  // addSugarAndMilk 和 addLemon 被抽象成「加調味料」的動作
  addCondiments() {}
  init() {
    this.boilWater()
    this.brew()
    this.pourInCup()
    this.addCondiments()
  }
}
```

接著，設計具體實作的子類別，<br />
以下是重構後的咖啡類別 `Coffee`：

```javascript
class Coffee extends Beverage {
  brew() {
    console.log('用沸水沖泡咖啡')
  }
  pourInCup() {
    console.log('把咖啡倒進杯子')
  }
  addCondiments() {
    console.log('加糖和牛奶')
  }
}

const coffee = new Coffee()
coffee.init()
```

以下是重構後的茶類別 `Tea`：

```javascript
class Tea extends Beverage {
  brew() {
    console.log('用沸水浸泡茶葉')
  }
  pourInCup() {
    console.log('把茶水倒進杯子')
  }
  addCondiments() {
    console.log('加檸檬')
  }
}

const tea = new Tea()
tea.init()
```

如此改動不只讓程式顯得一致，也利用原型鏈將相同的方法繼承避免重複。那麼，這個模式所指的範本方法到底是誰呢？在上面這個例子中，答案是 `Beverage.prototype.init`，它作為演算法的腳本，**指導子類別用何種順序去執行哪些方法**。

### 使用場景

該模式經常被架構師用於搭建專案的框架，架構師定好框架的骨架，程式設計師繼承框架的結構之後，負責往裡面填空。

### 真的需要繼承嗎?

範本方法是為數不多的基於繼承的設計模式，但 JavaScript 實際上沒有提供真正的類別式繼承，不過憑藉[好萊塢原則](https://baike.baidu.com/item/%E5%A5%BD%E8%8E%B1%E5%9D%9E%E5%8E%9F%E5%88%99/16019700)－**高層元件呼叫底層元件**的心法，我們也可以通過**物件和物件之間的委託**來實作，形式上地借鑿了提供類別式的語言。

```javascript
const Beverage = function (params) {
  const boilWater = function () {
    console.log('把水煮沸')
  }
  // 若一定要由子物件實作可以在預設 function 動手腳防呆
  const brew =
    params.brew ||
    function () {
      throw new Error('必須傳遞 brew 方法')
    }
  const pourInCup =
    params.pourInCup ||
    function () {
      throw new Error('必須傳遞 pourInCup 方法')
    }
  const addCondiments =
    params.addCondiments ||
    function () {
      throw new Error('必須傳遞 addCondiments 方法')
    }

  const F = function () {}
  // 注意，箭頭函式沒有 prototype，請乖乖使用 function () {}
  F.prototype.init = function () {
    boilWater()
    brew()
    pourInCup()
    addCondiments()
  }
  return F
}

const Coffee = Beverage({
  brew: function () {
    console.log('用沸水沖泡咖啡')
  },
  pourInCup: function () {
    console.log('把咖啡倒進杯子')
  },
  addCondiments: function () {
    console.log('加糖和牛奶')
  },
})

const Tea = Beverage({
  brew: function () {
    console.log('用沸水浸泡茶葉')
  },
  pourInCup: function () {
    console.log('把茶倒進杯子')
  },
  addCondiments: function () {
    console.log('加檸檬')
  },
})

const coffee = new Coffee()
coffee.init()

const tea = new Tea()
tea.init()
```

###### tags: `JavaScript 設計模式與開發實踐` `設計模式` `JavaScript`
