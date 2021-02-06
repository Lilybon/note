# i18n 重構(1) - 格式化物件

因為公司各專案 i18n 重複的字詞太多了<br/>
同事希望能把 JSON 併到同一個 repository 統一管理<br/>
但之前手機端跟電腦端產品的命名規則不同( nested, plain )

![怎麼這樣](https://i.imgur.com/r1a2v65.png)

於是只能寫一個簡單的小腳本整理 JSON<br/>
因為拉平比較不用人力就決定統一將命名規則改成 plain JSON

流程如下：
1. 遞迴拉平嵌套 property
2. 排序 property
3. property 前加 prefix
4. 匯出 JSON 字串

```javascript
function flattenObj (obj) {
   const result = {}
   helper(obj)
   return result
   function helper(obj = {}, prefix = '') {
     for (let key in obj) {
       let nextPrefix = (prefix ? prefix + '_' : '') + key
       if (typeof obj[key] !== 'object') result[nextPrefix] = obj[key]
       else helper(obj[key], nextPrefix)
     }
   }
}

function sortObj (obj) {
  return Object
    .keys(obj)
    .sort()
    .reduce((acc, key) => {
      acc[key] = obj[key]
      return acc
    }, {})
}

function markPrefix (prefix) {
  return function (obj) {
    return Object
      .keys(obj)
      .reduce((acc, key) => {
        acc[prefix + key] = obj[key]
        return acc
      }, {})
  }
}

function formatJSONString (obj) {
  return [
    flattenObj,
    sortObj,
    markPrefix('prefix:'),
    JSON.stringify
   ].reduce((acc, cb) => cb(acc), obj)
}

// example
formatJSONString({
  a: {
    b: {
      c: 123
    },
    d: 456
  },
  f: true
})
// {"prefix:a_b_c":123,"prefix:a_d":456,"prefix:f":true}
```
後來想想 makePrefix 有點冗就拿掉了

###### tags: `Refactor` `JavaScript` `i18n`