# 後台重構 - 以設定檔自動化生成第三方遊戲欄位

最近公司的需求變得沒有這麼急迫，可以花點時間還債了QQ

我們的產品主要有彩票和第三方遊戲－
彩票泛指像是**六合彩、七星彩、飛艇**等常見的博弈遊戲；
第三方遊戲則是介接第三方廠商的API服務在前台用iframe打請求拿取URL遊玩的遊戲，像是**棋牌、視訊、電子、捕魚、體育**都是我們有提供的遊戲類別。

~~沒錯，我都跟朋友戲稱我是負責串接接性感荷官在線發牌的前端工程師。~~

![AE視訊](https://i.imgur.com/5OxP2Va.png =400x)


我認真思考了一下後台在快速迭代下設計上的缺失，最主要會遇到的麻煩是**每次添加新的第三方遊戲類別都要浪費一堆時間找到組件補上欄位、新增路由**，還常常會東漏西漏，讓我改到快發瘋。

### 自動化

想到了一個非常簡單暴力的解法：
何不嘗試建立一個**設定檔**直接讓所有組件、路由、選項都依賴此設定檔生成就好？

![genius](https://i.imgur.com/E1ayrAM.png =300x)

我一直有在思考是不是可以這樣做，但是苦於新創迭代需求太頻繁害我來不及擦屁股QQ，既然今天終於抽出時間讓我重構了，那就開始著手嘗試吧！

```javascript
// @/config/external-groups.js
export default [
  { id: 1, name: '棋牌', code: 'poker' },
  { id: 2, name: '視訊', code: 'live' },
  { id: 3, name: '電子', code: 'slot' },
  { id: 4, name: '捕魚', code: 'fish' },
  { id: 5, name: '體育', code: 'sport' }
]
```

對，就是這麼樸實無華， KISS 對每個人都好，把要用到的 attr 用陣列包一包即可。

以下是各 attr 的實際用途：
* id - 生成各遊戲類別之注單的預設 query (ex: ?group=1)
* name - 生成各遊戲類別的欄位 label 跟選項的 label
* code - 生成各遊戲類別的欄位 attr 和選項的 value

分析一下以這個方式重構帶來的利弊：
* 好處－生成遊戲類別一律只要改動設定檔就好，**節省大量時間**，且類別排序能完全一致。
* 壞處－程式碼**可讀性變差了一點**，新人可能需要追 code 搞懂我搞了什麼花樣。

覺得能省掉至少半天到一天的時程這點還蠻划算的，且後端的欄位 key 都有一致讓我很溫馨，於是開始動工！

後台專案目前是以 Vue2, Vue Router 和 element-ui 為技術棧，下面的例子會以這些技術為基礎做講解。

### Router

接下來就是一路無聊的 for-loop 到底。
感謝 ES6 的 **展開運算符**，我們可以依賴設定檔自動生成注單路由。
```javascript
// @/router.js
import Router from 'vue-router'
import EXTERNAL_GROUPS from '@/config/external-groups'
export default new Router ({
  routes: [
    // 其他路由
    ...EXTERNAL_GROUPS.map(group => ({
      path: `/report/${group.code}-bet-record`,
      name: `report-${group.code}-bet-record`,
      component: () => import('./views/Report/ExternalBetRecord'),
      meta: {
        title: `${group.name}注单`, // 頁面標題
        providerCategory: group.id // 注單頁 API 的 default query
        // 其他 meta key 如權限、頁面類別...等ㄎ 
      }
    }))
    // 其他路由
  ]
})
```
### Options

要讓 html template 渲染記得把設定檔放進 data 裡。
```htmlembedded
<html>
  <el-select v-model="query.type" placeholder="全部">
    <el-option label="全部" value="" />
    <el-option label="彩票佣金收入" value="lottery_commission" />
    <el-option
      v-for="type in EXTERNAL_GROUPS"
      :key="`external-${ type.code }-commission`"
      :label="`${ type.name }佣金收入`"
      :value="`${ type.code }_commission`"
    />
    <el-option label="佣金取款" value="withdraw" />
    <el-option label="佣金提取" value="transfer" />
  </el-select>
</html>
```
### Table

```htmlembedded
<html>
  <el-table-column
    v-for="type in EXTERNAL_GROUPS"
    :key="`external-${ type.code }-profit-and-amount`"
    :label="type.name"
    align="right"
    v-slot="{ row }"
  >
    <p
      :class="profitColor(row[`${ type.code }_profit`])"
      @click="routerGo(row.time, `${ type.code }_record`)"
    >
      {{ row[`${ type.code }_profit`] | currency('￥') }}
    </p>
    <p>{{ row[`${ type.code }_amount`] | currency('￥') }}</p>
  </el-table-column>
</html>
```

### Filters

 之前用 filters 寫 switch-case 的部分打掉改用 methods 由 map 去找對應值。
```javascript
// @/views/Agent/CommissionRecord.vue
import EXTERNAL_GROUPS from '@/config/external-groups'
export default {
  name: 'CommissionRecord',
  data () {
    return {
      typeMap: {
        withdraw: '佣金取款',
        transfer: '佣金提取',
        profitspace_commission: '彩票佣金收入',
        revenue_commission: '彩票佣金收入',
        ...EXTERNAL_GROUPS.reduce((map, type) => {
          map[`${type.code}_commission`] = `${type.name}佣金收入`
          return map
        }, {})
      },
    }
  },
  methods: {
    formatType (value) {
      return this.typeMap[value] || ''
    }
  }
}
```

大功告成，同事以後加第三方的遊戲類別時應該會很舒爽。
不過因為這包專案沒 i18n 所以才這麼輕鬆，之後還要煩惱怎麼自動加 i18n 跟客製化注單的欄位 (不是每個欄位都這麼簡單跑個 for-loop 就能輕鬆搞定的 QQ)

###### tags: `Refactor` `JavaScript` `Vue` `Vue Router` `element-ui`