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
8. Deep Clone
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
9. call, apply

```javascript
Function.prototype.call2 = function (context, ...args) {
  context = context || window
  context.fn = this
  const result = context.fn(...args)
  delete context.fn
  return result
}

Function.prototype.apply2 = function (context, args = []) {
  context = context || window
  context.fn = this
  const result = context.fn(...args)
  delete context.fn
  return result
}
```

10. new
    ```javascript
    function objectFactory (Constructor, ...args) {
      const obj = new Object()
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
12. X

###### tags: `Interview` `Front-end`
