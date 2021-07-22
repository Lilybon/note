# 策略模式 (Strategy)

單例模式的定義是：

> _定義一系列的演算法，把它們一個個**封裝**起來，並且使它們**可以相互替換**。_

現實中，很多時候也是有多種途徑到達同一個目的地。
比如我們可以根據自己的經濟狀況選擇買可以負擔得起的手機。

- 石油王選 iphone12。
- 小康選 iphoneSE。
- 吃土選 Nokia3310。

![為所欲為](https://5b0988e595225.cdn.sohucs.com/images/20180813/0ba7bb6faa984cefb447931162a6ece6.gif =500x)

### 使用策略模式計算獎金

以下是一個直覺的計算獎金函數 `calculateBonus` 。

```javascript
/**
 * 計算獎金數額
 * @param {string} performanceLevel - 工資數額
 * @param {number} salary - 績效考核等級
 * @return {number}
 */
function calculateBonus (performanceLevel, salary) {
  switch (performanceLevel) {
    case: 'S': return salary * 4
    case: 'A': return salary * 3
    case: 'B': return salary * 2
  }
}

console.log(calculateBonus('B', 20000)) // 40000
console.log(calculateBonus('S', 6000)) // 24000
```

這段程式碼隱含以下缺點：

- `calculateBonus` 函數比較龐大，包含太多 if-else 類的述句，這些述句需要涵蓋所有的邏輯分支。
- `calculateBonus` 函數缺乏彈性，如果要把績效的參數做修正，只能深入 `calculateBonus` 內部實作，違反「**開放－封閉原則**」
- 演算法的重用性差，如果專案有其他地方需要**重用**計算獎金的邏輯，他們只有用複製貼上一途。

我們可以使用策略模式重構程式碼，目的是將**演算法的使用**(不變)和**演算法的實作**(變化)分離開來。

我們試著先用 class 語法糖模仿強型別語言的實作。
將計算績效的規則封裝在對應的策略類別中：

```javascript
class performaceS {
  calculate(salary) {
    return salary * 4
  }
}

class performaceA {
  calculate(salary) {
    return salary * 3
  }
}

class performaceB {
  calculate(salary) {
    return salary * 2
  }
}
```

接下來定義獎金類別 `Bonus` ：

```javascript
class Bonus {
  constructor() {
    this.salary = null
    this.strategy = null
  }
  setSalary(salary) {
    this.salary = salary
  }
  setStrategy(strategy) {
    this.strategy = strategy
  }
  getBonus() {
    // 注意！Bonus本身沒有能力進行計算
    // 把計算獎金的操作委託給對應的策略物件
    return this.stragegy.calculate(this.salary)
  }
}
```

試著使用重構完畢的程式碼，將**演算法實作物件**餵給**演算法執行物件**進行計算：

```javascript
const bonus = new Bonus()

bonus.setSalary(20000)
bonus.setStrategy(new PerformanceB())
console.log(bonus.getBonus()) // 40000

bonus.setSalary(6000)
bonus.setStrategy(new PerformanceB())
console.log(bonus.getBonus()) // 24000
```

經過重構後，每個類別職責變得更加鮮明清晰。

### JavaScript 版本的策略模式

在 JavaScript 中，**函數也是物件**，寫起來**超 ♂︎ 自 ♂︎ 由**。
可以更簡單和直接的把每個策略定義為函數，也不需要額外實體化實例來執行策略，結構上非常簡潔又看得舒服。

![那個自由的男人](https://media1.tenor.com/images/38e5262d801737950f5204669aeff197/tenor.gif?itemid=13534337)

```javascript
const strategy = {
  S: function (salary) {
    return salary * 4
  },
  A: function (salary) {
    return salary * 3
  },
  B: function (salary) {
    return salary * 2
  },
}

function calculateBonus(level, salary) {
  return strategy[level](salary)
}

console.log(calculateBonus('B', 20000)) // 40000
console.log(calculateBonus('S', 6000)) // 24000
```

由於現在的 JavaScript 現在也支持 `import` 及 `export` 的語法，我們也可以不借助 `strategy` 物件包裝演算法，讓策略模式變得更加隱形，做為**高階函數**將演算法按需載入。

```javascript
// strategy.js
export function S(salary) {
  return salary * 4
}
export function A(salary) {
  return salary * 3
}
export function B(salary) {
  return salary * 2
}
export function calculateBonus(fn, salary) {
  return fn(salary)
}
```

### 表單校驗

### 策略模式的優缺點

### 小結

###### tags: `JavaScript 設計模式與開發實踐` `設計模式` `JavaScript`
