# 面試題

下面會提及一些面試考題，<br />
值得在面試前消化一下。

1. Create a sum function that will calculate the sum of arguments. if there aren’t any arguments, return the result, if there are any arguments, return a function that can be used for the next calculation.

   ```javascript
   function sum(...args) {
     let result = 0
     return helper(...args)
     function helper(...values) {
       for (let value of values) result += +value
       return values.length ? helper : result
     }
   }

   sum() // return 0
   sum(1)() // return 1
   sum(1)(1)(1)() // return 3
   sum(1)(1, 1, 1, 1)(1)() // return 6
   sum(1) // return a function
   // sum(1)...(1, 1, 1, ...1)    // return a function
   ```

2. Triplets <br />

   ![triplets](https://scontent-hkt1-2.xx.fbcdn.net/v/t1.15752-9/168222709_2038100552997744_3212583619549286926_n.png?_nc_cat=105&ccb=1-3&_nc_sid=ae9488&_nc_ohc=mIrkAbDARB8AX_-b4Tz&_nc_ht=scontent-hkt1-2.xx&oh=148778de78613aae9485294b68c57318&oe=609AE185 =500x)

   ```javascript
   function triplets(d, t) {
     let result = 0
     helper(0, 0, 0)
     return result
     function helper(startIndex, sum, count) {
       if (sum > t) return
       if (count === 3) result++
       else {
         for (let i = startIndex; i < d.length; i++) {
           helper(i + 1, sum + d[i], count + 1)
         }
       }
     }
   }

   triplets([1, 2, 3, 4, 5], 8) // return 4
   ```

3. Fisher–Yates Shuffle

   ```javascript
   function shuffle(arr) {
     let n = arr.length;
     while (n > 0) {
       let index = (Math.floor(Math.random() * n);
       [arr[index], a[n - 1]] = [arr[n - 1], arr[index]];
       n--;
     }
     return arr;
   }

   shuffle([1, 2, 3, 4, 5, 6, 7, 8, 9]);
   ```

4. TrxndMxcrx Codility 1 <br />
   ![TrxndMxcrx Codility 1](https://i.imgur.com/6qVwhJG.png =500x)

   ```javascript
   function solution(s) {
     let sum = 0,
       count = 0
     for (let char of s) {
       sum += char
     }
     if (sum % 3 === 0) count++
     for (let char of s) {
       let tmpSum = sum - char
       for (let i = 0; i <= 9; i++) {
         if (String(i) !== char && (tmpSum + i) % 3 === 0) count++
       }
     }
     return count
   }

   solution('23') // return 7
   solution('0081') // return 11
   solution('022') // return 9
   ```

5. TrxndMxcrx Codility 2 <br />
   ![TrxndMxcrx Codility 2](https://i.imgur.com/hoWNWVc.png =500x)

   ```python
   # you can write to stdout for debugging purposes, e.g.
   # print("this is a debug message")

   def solution(A, K):
       # write your code in Python 3.6
       if K > len(A):
           return -1

       _max = 0
       even = []
       odd = []

       for i in range(len(A)):
           if A[i] % 2 == 0:
               even.append(A[i])
           else:
               odd.append(A[i])

       even.sort()
       odd.sort()

       e_idx = len(even) - 1
       o_idx = len(odd) - 1

       while K > 0:
           if K % 2 == 1:
               if e_idx >= 0:
                   _max += even[e_idx]
                   e_idx -= 1
               else:
                   return -1
               K -= 1
           elif e_idx >= 1 and o_idx >= 1:
               if even[e_idx] + even[e_idx-1] >= odd[o_idx] + odd[o_idx-1]:
                   _max += even[e_idx] + even[e_idx-1]
                   e_idx -= 2
               else:
                   _max += odd[o_idx] + odd[o_idx-1]
                   o_idx -= 2

               K -= 2
           elif e_idx >= 1:
               _max += even[e_idx] + even[e_idx-1]
               e_idx -= 2
               K -= 2
           elif o_idx >= 1:
               _max += odd[o_idx] + odd[o_idx-1]
               o_idx -= 2
               K -= 2

       return _max
   ```

   這題我還沒自己實作，先備份我同事的 pyhton 解答在這邊。

6. Function Cache
    ```javascript
    function cached (fn) {
      const memo = {}
      return function (...args) {
        const key = JSON.stringify(args)
        if (key in memo) return memo[key]
        return memo[key] = fn(...args)
      }
    }
    ```
7. Parse CSV
    ```javascript
    const input = 'Number, City, Location\n\
    1, City 1, Taichung\n\
    2, City 2, "Taipei, Taiwan, aaa, bbb"'
    
    const result = parseCSV(input)
    result[0][0] // Number
    result[1][1] // City 1
    result[2][2] // Taipei, Taiwan, aaa, bbb

    function parseCSV (text) {
      return text.split(/\n/)
      // todo: how to split comma but without double quote comma
    }
    ```
8. 實作 Deep Clone
    ```javascript
    function deepClone (object) {
      if (!object) return
      const clonedObject = Array.isArray(object) ? [] : {}
      for (let [key, value] of Object.entries(object)) {
        clonedObject[key] = typeof value === 'object' ? deepClone(value) : value
      }
      return clonedObject
    }
    ```
9. 實作 call, apply, bind

```javascript
Function.prototype.call2 = function (context, ...args) {
  if (typeof this !== 'function') {
    throw new Error('call2 - keyword "this" is not callable')
  }
  context = context || window
  context.fn = this
  const result = context.fn(...args)
  delete context.fn
  return result
}

Function.prototype.apply2 = function (context, args = []) {
  if (typeof this !== 'function') {
    throw new Error('apply2 - keyword "this" is not callable')
  }
  context = context || window
  context.fn = this
  const result = context.fn(...args)
  delete context.fn
  return result
}

Function.prototype.bind2 = function (context, ...bindingArgs) {
  if (typeof this !== 'function') {
    throw new Error('bind2 - keyword "this" is not callable')
  }

  const self = this

  const fBound = function (...execArgs) {
    const args = bindingArgs.concat(execArgs)
    // 當作為構造函數時： this 指向實例，條件為 true，綁定實例本身為目標
    // 當作一班函數時： this 指向 window，條件為 false，綁定 context 為目標
    return self.apply(this instanceof fBound ? this : context, args)
  }
  fBound.prototype = Object.create(self.prototype)
  fBound.prototype.constructor = fBound

  return fBound
}
```

10. 實作 new
    ```javascript
    function objectFactory (Constructor, ...args) {
      if (typeof Constructor !== 'function') {
        throw new Error('objectFactory - Constructor is not callable')
      }
      const obj = {}
      obj.__proto__ = Constructor.prototype
      const result = Constructor.apply(obj, args)
      return typeof result === 'object' ? result : obj
    }
    ```

11. 請用 React-hook 實作可按住 Shift 多選的 checkbox 功能
    
    可參考以下文章嘗試
    - React use
    - React Element UI
    - [這篇文章的實作方式](https://tj.ie/multi-select-checkboxes-with-react/)

12. 使用你習慣的現代前端框架做一個有無限捲動的 NFT 列表頁和詳情頁，可以的話順便加上 MetaMask 和 web3.js 做登入。

13. this 指向 - 請說明畫面會印出什麼結果
    ```javascript
    const value = 2
    const foo = {
        value: 1,
        bar: bar.bind(null)
    }

    function bar() {
        console.log(this.value);
    }

    foo.bar() // 2
    ```

14. 實作 throttle
```javascript
function throttle (callback, wait) {
  let timeout
  return function (...args) {
    let context = this
    if (timeout) return
    timeout = setTimeout(function () {
      callback.apply(context, args)
      timeout = null
    })
  }    
}
```
15. 實作 debounce
    ```javascript
    function debounce (callback, wait, immediate) {
      let timeout, result

      function helper (...args) {
        let context = this
        clearTimeout(timeout)
        if (immediate) {
          if (!timeout) result = callback.apply(context, args)
          timeout = setTimeout(function () {
            timeout = null
          }, wait)
          return result
        } 
        timeout = setTimeout(function () {
          result = callback.apply(context, args)
        }, wait)
        return result
      }
      
      helper.cancel = function () {
        clearTimeout(timeout)
        timeout = null
      }
      
      return helper
    }
    ```

16. callee - 請說明畫面會印出什麼結果
    ```javascript
    const data = []
    for (let i = 0; i < 3; i++) {
      (data[i] = function () {
        console.log(arguments.callee.i)
      }).i = i
      data[i]()
    }
    // 0
    // 1
    // 2
    ```

17. 實作 ES5 Object.create
    ```javascript
    function createObject (o) {
      const F = function () {}
      F.prototype = o
      return new F() 
    }
    ```

18. 請解釋 JavaScript 是如何繼承的？原型鏈是什麼？寫一段繼承看看。
    ```javascript
    function Parent (name) {
      this.name = name
      this.colors = ['red', 'blue', 'green']
    }

    Parent.prototype.getName = function () {
      console.log(this.name)
    }

    function Child (name, age) {
      Parent.call(this, name);
      this.age = age;
    }

    Child.prototype = Object.create(Parent.prototype)
    Child.prototype.constructor = Child


    var child1 = new Child('kevin', 18)

    console.log(child1)
    ```

19. 實作 Array 的 Iterator。物件本身是不可以被 iterate 的，要如何使物件可以執行 forOf
    ```javascript
    function createIterator (items) {
      let index = 0
      return {
        next: function () {
          const done = index >= items.length
          const value = done ? undefined : items[index++]
          return {
            value,
            done
          }
        }
      }
    }

    const iterator = createIterator([1, 2, 3]);

    console.log(iterator.next()); // { done: false, value: 1 }
    console.log(iterator.next()); // { done: false, value: 2 }
    console.log(iterator.next()); // { done: false, value: 3 }
    console.log(iterator.next()); // { done: true, value: undefined }
    
    const obj = {
      [Symbol.iterator]: function () {
        return createIterator([1, 2, 3])
      }
    }
    for (let v of obj) console.log(v)
    // 1
    // 2
    // 3
    ```

21. 實現函數惰性
    ```javascript
    // IIFE
    let lazyFn = (function () {
      let date
      return function () {
        if (date) return date
        date = new Date()
        return date
      }
    })()
    // 函數對象
    let lazyFn = function () {
      if (lazyFn.date) return lazyFn.date
      lazyFn.date = new Date()
      return lazyFn.date
    }

    // 惰性函數
    let lazyFn = function () {
      const date = new Date()
      lazyFn = function () {
        return date
      }
      return lazyFn()
    }
    ```

21. 請解釋 SPA 的優缺點。請說明 SPA 如何做 SEO。
22. 實作 mergePromise
    ```javascript
    const time = timer => 
      new Promise(resolve => {
        setTimeout(() => {
          resolve()
        }, timer)
      })

    const ajax1 = () => time(2000).then(() => {
      console.log(1);
      return 1
    })

    const ajax2 = () => time(1000).then(() => {
      console.log(2);
      return 2
    })

    const ajax3 = () => time(1000).then(() => {
      console.log(3);
      return 3
    })

    const mergePromise = ajaxs => {
      const data = []
      let promise = Promise.resolve()
      ajaxs.forEach(ajax => {
        promise = promise.then(ajax).then(res => {
          data.push(res)
          return data
        })
      })
      return promise
    }

    mergePromise([ajax1, ajax2, ajax3]).then(data => {
      console.log('done')
      console.log(data)
    })
    // 1
    // 2
    // 3
    // done
    // [1, 2, 3]
    ```

23. 使用Promise實現紅綠燈交替重複亮
    ```javascript
    const red = () => {
      console.log('red')
    }

    const green = () => {
      console.log('green')
    }

    const yellow = () => {
      console.log('yellow')
    }

    const light = (timer, cb) =>
      new Promise(resolve => {
        setTimeout(() => {
          cb()
          resolve()
        }, timer)
      })

    const step = () =>
      Promise.resolve()
        .then(() => light(3000, red))
        .then(() => light(2000, green))
        .then(() => light(1000, yellow))
        .then(() => step())

    step()
    ```

24. 實作 `Promise.all` `Promise.race`
    ```javascript
    Promise.prototype.all2 = function (...promises) {
      const results = []
      const merged = promises.reduce((acc, promise) =>
        acc
          .then(() => promise)
          .then((result) => results.push(result)),
        Promise.resolve(null)
      )
      return merged.then(_ => results)
    }

    Promise.prototype.race2 = function (...promises) {
      return new Promise((resolve, reject) => {
        promises.forEach(promise =>
          promise
            .then(resolve)
            .catch(reject)
      })
    }
    ```

25. 設計一個控制 promise 倂發數上限的函式
    ```javascript
    function promiseConcurrencyLimit (limit, tasks) {
      const queue = [...tasks]
      const initialLength = Math.min(limit, queue.length)
      for (let i = 0; i < initialLength; i++) {
        helper(queue)
      }
    }

    function helper (queue) {
        if (!queue.length) return
        const promiseFn = queue.shift()
        promiseFn().finally(() => {
          helper(queue)
        })
    }
    ```

26. 實作 promise
```javascript
const HANDLERS = Symbol('handlers')
const QUEUE = Symbol('queue')
const STATE = Symbol('state')
const VALUE = Symbol('value')

const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  constructor (fn) {
    this[HANDLER] = new Handlers()
    this[QUEUE] = []
    this[STATE] = PENDING
    this[VALUE] = undefined

    const fnType = typeof fn
    if (fnType === 'function') {
      try {
        // 直接代入對應的 resolve 跟 reject 嘗試執行
        fn(
          value => resolvePromise(this, value),
          reason => transition(this, REJECTED, reason)
        )
      } catch (error) {
        transition(this, REJECTED, error)
      }
    } else if (fnType !== undefined) {
      resolvePromise(this, fn)
    }
  }
    
  get state () {
    return this[STATE]
  }

  get value () {
    return this[VALUE]
  }
    
  then (onFulfilled, onRejected) {
    const next = new MyPromise()
    if (typeof onFulfilled === 'function') {
       next[HANDLERS].fulfill = onFulfilled
    }
    if (typeof onRejected === 'function') {
       next[HANDLERS].reject = onRejected
    }
    this[QUEUE].push(next)
    process(this)
    return next
  }

  catch (onRejected) {
    return this.then(null, onRejected)
  }
}

class Handlers {
  constructor () {
    this.fulfill = null
    this.reject = null
  }
}

function resolvePromise (promise, x) {
  // TODO: how?
}

function transition (promise, state, value) {
  if (promise[STATE] === state || promise[STATE] !== PENDING) return
  promise[STATE] = state
  promise[VALUE] = value
  return process(promise)
}

function process (promise) {
  if (promise[STATE] !== PENDING) return
  // we skip the environment IIFE async task binding
  setTimeout(processNextTick, 0, promise)
  return promise
}

function processNextTick (promise) {
  while (promise[QUEUE].length) {
    const queuePromise = promise[QUEUE].shift()
    let handler

    if (queuePromise[STATE] === FULFILLED) {
      handler = queuePromise[HANDLERS].fulfill || value => value
    } else if (queuePromise[STATE] === REJECTED) {
      handler = queuePromise[HANDLERS].reject || reason => { throw reason }
    }

    try {
      resolvePromise(queuePromise, handler(promise[VALUE]))
    } catch (error) {
      transition(queuePromise, REJECTED, error)
      continue
    }
  }
}
```

27. X


###### tags: `Interview` `Front-end`
