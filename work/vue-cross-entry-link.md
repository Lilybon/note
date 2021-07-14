# 跨入口連結

Vue 有提供複數個 entry 的功能，<br/>
好處是可以讓那複數個網站共用同樣的商業邏輯並根據網站規模各自打包自己的 vendor chunk，<br/>

以下我們用 **A** (標準網站)、 **B** (簡易網站)來說明今天遇到的小需求：<br/>
在 **B** 上面提供一個連結導回 **A** 的特定頁面。
注意 **A** 和 **B** 兩個是不同的 Vue 實例所以是這沒辦法用 Vue Router 做溝通的喔，請回歸 `window.location` 的懷抱。

以下為簡單的範例：

1. 整包專案共用的函式庫

```javascript
// utils.js
export const getMainEntryLink = (redirect = '') => {
  redirect = encodeURIComponent('/#/' + redirect)
  return `${ process.env.domain }?redirect=${ redirect }`
}
```

2. **B** 某頁的組件

```html
<a rel="noopener" :href="getMainEntryLink('foo/')">回主頁的 foo 頁面</a>
```

```javascript
import { getMainEntryLink } from '@/utils'

export default {
  setup() {
    return {
      getMainEntryLink
    }
  }
}
```

3. **A** 實例創建的進入點

```javascript
// main.js
import qs from 'qs'

const url = window.location.href
const queries = url.match(/\?([^/#]*)/)
const params = queries ? qs.parse(queries[1]) : {}

if (params.redirect) {
  window.location = decodeURIComponent(params.redirect)
}

// ...

```

###### tags: `Work` `JavaScript` `Vue`

