# i18n 重構(2) - 尋找可替換的 key

經過(1)拉平 i18n 的物件後<br/>
我們需要在合併專案底下的 local JSON 跟 npm JSON 前做以下前置處理：
1. 抓出 local JSON 跟 npm JSON 重複命名的 key 值並給予 hash ( ex : xxx_1 、 xxx_2 ) 區分。
這個很簡單就不對該步驟多做贅述了
2. **抓出專案 locale 底下有哪些 key 可以被npm locale 取代。**

我們有三個語系 cn 、 en 、 vn 的 JSON 需要做處理<br/>
好險只有三包而且現在 JSON 還不夠肥大

![怕](https://i.imgur.com/Cc7wt8I.png =400x)

~~因為懶~~，直接在專案根目錄底下直接寫個 js 腳本解決

流程如下進行：
1. 遍歷 localCn 的 key 收集跟 localCn [ key ]值相同的 npmCn key 做候選人
2. 遍歷候選人篩選出全部 npm 跟 local 對應詞彙完全匹配者 (無痛替換)
3. 收集 before ( 原始 local 的 key 值 ) 跟 after (可替換成 npm 的 key 值 ) 的結果
4. 看是要人力或寫腳本將前端專案的 key 替換掉

```javascript
const npmCn = require('./node_modules/@unnotech/locale/src/zh-CN.json')
const npmEn = require('./node_modules/@unnotech/locale/src/en.json')
const npmVn = require('./node_modules/@unnotech/locale/src/vi-VN.json')
const localCn = require('./src/i18n/locale/zh-CN.json')
const localEn = require('./src/i18n/locale/en.json')
const localVn = require('./src/i18n/locale/vi-VN.json')

const npmList = [npmEn, npmVn]
const localList = [localEn, localVn]
const results = []

for (let lk in localCn) {
  let candidateList = []
  for (let nk in npmCn) {
    if (localCn[lk] === npmCn[nk]) candidateList.push(nk)
  }
  for (let nk of candidateList) {
    let isSame = true
    for (let i = 0; i < npmList.length; i++) {
      let npm = npmList[i]
      let local = localList[i]
      if (!npm[nk] || !local[lk] || npm[nk].toLowerCase() !== local[lk].toLowerCase()) {
        isSame = false
        break
      }
    }
    if (isSame) results.push({ before: lk, after: nk })
  }
}
console.log(results)

```

結果如下：
因為列表太長我截片段就好
大概長這樣

![結果](https://i.imgur.com/juDI8kM.png)

雖然寫得真是夠隨意<br/>
但至少不用再勞心勞力找重複的 key 慢慢刪掉<br/>
蒸蚌


###### tags: `Refactor` `JavaScript` `i18n`