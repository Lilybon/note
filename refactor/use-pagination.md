# 前台重構 - 使用 Composition API 封裝頁碼邏輯

![I love writing report](https://www.memecreator.org/static/images/memes/4796468.jpg =400x)

終於有點時間可以弄 OKR ，和公司的資深前端討論了一下，決定優先把前台報表頁的程式碼統一成一致的 style。

### 重構過程和思路

1. 因為每個同事寫報表都有自己的 coding style，但其實做的邏輯大多雷同，所以我同事寫了一個很厲害的 **vue-condition-watcher** 打算統一報表頁寫法，這樣之後出大事只維護核心的抽象邏輯即可。
2. 公司產品的後端是用 **django** 寫的，而分頁報表 API 用的是 **LimitOffsetPagination** ，它有兩個特別的 query - **limit** 跟 **offset**，這兩個 query 的功用如下：
   - **limit** - 撈多少資料
   - **offset** - 後端從 table 的哪個 index 撈取資料
3. 因為 vue-conditional-watcher 只針對 conditions 監聽，很難介入和辨別使用者的操作是以下何者：
   - 切換頁碼 (只觸發改變 offset)
   - 改變 filter 欄位 (觸發 filter 欄位改變且 offset 要歸零)
   ```javascript
   const conditions = reactive({
     // 頁碼控制欄位
     limit: 20,
     offset: 0,
     // filter 欄位
     createdAt: ["2020-03-30", "2020-03-31"],
     transaction_type: "",
   });
   ```
4. 決定寫一個代理去幫忙 vue-condition-watcher 判斷上面這件事，**核心的思想是把 vue-condition-watcher 的 conditions 解構失去 reactive 特性再重新用 ref 跟 reactive 包裝**，這樣決定何時更新 conditions 的主導權就會重回我們手上。

```javascript
// @/hooks/usePagination.js
export default function (
  config,
  queryOptions = {
    sync: "router",
    /*
     * 我們不能讓使用者可以透過 route query 更改 limit
     * 因為可能會有瘋子一次打個超大數字直接讓後端爆炸
     */
    ignore: ["limit"],
  },
  limit = 20
) {
  const { proxy } = getCurrentInstance();
  config.conditions.offset = +proxy.$route.query.offset || 0;
  config.defaultParams = config.defaultParams || {};
  config.defaultParams.limit = limit;
  queryOptions.sync = queryOptions.sync || "router";
  if (!queryOptions.ignore.includes("limit")) queryOptions.ignore.push("limit");

  const {
    conditions: watcherConditions,
    data,
    loading,
    refresh,
  } = useConditionWatcher(config, { sync, ignore });

  /*
   * 核心 I：
   * 解構並重新以 ref 和 reactive 建立活性
   * 之後使用者 mutate 到的數據都會是這些代理變數
   */
  const {
    offset: parseOffset,
    limit: parseLimit,
    ...parseConditions
  } = watcherConditions;
  const offset = ref(parseOffset);
  const conditions = reactive(parseConditions);

  /*
   * 頁碼相對於 offset 和 limit 的計算關係
   * 拿來綁定給 element-ui 的 el-pagination
   */
  const currentPage = computed({
    get: () => offset.value / limit + 1,
    set: (page) => {
      offset.value = (page - 1) * limit;
    },
  });

  /*
   * 核心 II：
   * 由這些代理變數去改變實際的 watcherConditions 觸發打請求
   */
  watch(offset, (newOffset) => {
    watcherConditions.offset = newOffset;
  });

  watch(
    conditions,
    (newConditions) => {
      Object.keys(newConditions).forEach((key) => {
        watcherConditions[key] = newConditions[key];
      });
      watcherConditions.offset = 0;
      offset.value = 0;
    },
    { deep: true }
  );

  return {
    data,
    loading,
    refresh,
    limit,
    offset,
    conditions,
    currentPage,
  };
}
```

### 使用方式

```htmlembedded
<template>
  <div>
    <el-form :inline="true" :model="conditions">
      <el-form-item label="選擇日期">
        <el-date-picker
          v-model="conditions.createdAt"
          size="mini"
          type="daterange"
          start-placeholder="開始日期"
          end-placeholder="結束日期"
          value-format="yyyy-MM-dd"
          :disabled="loading"
        >
      </el-date-picker>
    </el-form-item>
    ...
    <el-pagination
      v-if="totalCount > limit"
      :current-page.sync="currentPage"
      :page-size="limit"
      layout="total, prev, pager, next"
      :total="totalCount"
     >
  </el-pagination>
  </div>
<template>
```

```javascript
import usePagination from '@/hooks/usePagination'
export default {
  setup () {
    const { data, loading, refresh, limit, currentPage, condiitons } = usePagination({
      fetcher: () => axios.get('xxx'),
      conditions: {
        createdAt: ['2021-03-29', '2021-03-30']
      },
      beforeFetch (conditions) {
        conditions.created_at_after = conditions.createdAt[0]
        conditions.created_at_before = conditions.createdAt[1]
        delete conditions.createdAt
        return conditions
      },
      afterFetch (data) {
        totalCount.value = data.counts
      }
    })
    return {
      totalCount,
      data,
      loading,
      refresh,
      limit,
      currentPage,
      condiitons
    }
  }
}
 </script>
```

### 可以繼續拓展的方向

- 報表可能會更齊全，limit 可能有朝一日也會變成一個可控制變因。
- 可以對 afterFetch 用裝飾者模式去封裝有固定 format 的 response data (比如上面例子的 totalCount )

呃，然後我真的有點不確定這個 hook 要叫什麼名字好。<br/>
試著取名成`useAPIFetcher`、`useLimitOffset`、`useReport`過，但都覺得怪怪的。

### 參考

[Django - Pagination](https://www.django-rest-framework.org/api-guide/pagination/#limitoffsetpagination)<br/>
[Wiki - Proxy Pattern](https://en.wikipedia.org/wiki/Proxy_pattern)

###### tags: `Refactor` `JavaScript` `Vue` `element-ui`
