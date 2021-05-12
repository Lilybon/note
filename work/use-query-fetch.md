# 開獎網 - useQueryFetch

![彩球](http://pic.pimg.tw/beckham1985/fa315ec6dee5dfacaf767fba6bd6c617.png =400x)

因為客戶想要一個可以賺廣告收入的形象網站<br/>
新的越南彩票開獎網 **Aphid** 就這麼出生了<br/>
(中國彩叫 **Ant** 但我不想浪費時間想專案名就隨興取一個跟螞蟻互利共生的昆蟲了)

### 業務需求

初次拿到的頁面請求要印在 **html** 上，<br/>
其他的一律用 JS 去打請求，<br/>
對～很典型的 SSR 情境。

因為有開獎相關的資訊會有分頁，<br/>
且選到其他控因都要**重置頁碼**，<br/>
所以我們一樣封裝一個 hook 處理，<br/>
可以參考我以前的文章：<br/>
[前台重構 - 使用 Composition API 封裝頁碼邏輯](https://hackmd.io/@lilybon/use-pagination)

不過跟之前不同，<br/>
這次專案的規模很小，<br/>
且後端並沒有用到 **LimitOffsetPagination**，<br/>
所以不需要用 computed 額外處理邏輯 page 的邏輯。(讚喔)

身為這次專案的 TeamLead，<br/>
為了組員幸福及害怕重功惡夢，<br/>
只好再造一次給 Nuxt 用的業務輪子。

![Ah shit, here we go again.](https://memes.tw/user-template/3c868dfa0d6fbe73178ec786504a5a56.png =400x)

### 思路

1. 事件順序是 conditions 或 page 產生變化 → 更新 router query → 打請求
2. 有別於之前 usePagination 的設計 後來覺得既然 conditions 都跟 query 高度相關，乾脆直接強制使用者定義 **setupConditions** 讓 query 知道如何去初始化或更新 conditions；此外，考慮到依賴 route query 這個核心變數，不用 useAsync 而用 **useFetch**。

### 困境與解決方法

1. nuxtjs/composition-api 的 useFetch **不提供傳參數**，必須要自己在 hook 裡面拼接成它要求的規格。
2. 要考慮從 `nuxt-link` 觸發改動 route query 的情境，要回流更新 conditions 跟 page 讓 UI 保持一致，但要避免不必要的 router push 和避免 watch 無窮迴圈。

### 實作

```javascript
import {
  computed,
  ref,
  reactive,
  useRoute,
  useRouter,
  useFetch,
  watch,
} from "@nuxtjs/composition-api";
import { isSameObj } from "~/utils";

export default function ({
  setupConditions,
  defaultParams,
  promiseFn,
  afterFetch,
}) {
  // 防呆
  setupConditions =
    setupConditions ||
    (() => {
      throw new TypeError(
        "You must add setupConditions to teach useQueryFetch how router query turns into conditions!"
      );
    });
  afterFetch =
    afterFetch ||
    (() => {
      throw new TypeError(
        "You must add afterFetch to receive promise response data!"
      );
    });

  const router = useRouter();
  const route = useRoute();
  const query = computed({
    get: () => route.value.query,
    set: () => {},
  });

  const { page: parsePage, ...parseConditions } = query.value;
  const page = ref(+parsePage || 1);
  const conditions = reactive(setupConditions({}, parseConditions));

  // 在 hook 中以匿名函數組成符合 useFetch 用法的函數
  const { fetch, fetchState } = useFetch(async () => {
    const data = await promiseFn({
      ...query.value,
      ...defaultParams,
    });
    afterFetch(data);
  });
  const updateRouteQuery = (newQuery) => {
    // 避免回流觸發無意義的 router push
    if (isSameObj(newQuery, query.value)) return;
    router.push({
      name: router.currentRoute.name,
      query: newQuery,
    });
  };

  watch(
    [() => ({ ...conditions }), page],
    ([newConditions, newPage]) => {
      updateRouteQuery({
        ...newConditions,
        page: newPage,
      });
    },
    { deep: true }
  );
  watch(query, (newQuery) => {
    // 考慮直接點選 nuxt-link 時要回流更新 conditions 跟 page
    const { page: parsePage, ...parseConditions } = newQuery;
    setupConditions(conditions, parseConditions);
    page.value = +parsePage || 1;
    fetch();
  });

  return {
    page,
    conditions,
    fetch,
    fetchState,
  };
}
```

### 使用方式

以下為任意一個 page 使用 useQueryFetch 的範例：

```javascript
import { defineComponent, ref, useContext } from "@nuxtjs/composition-api";
import useQueryFetch from "~/hooks/useQueryFetch";

export default defineComponent({
  setup() {
    const resultData = ref([]);
    const totalPages = ref(0);

    const { $axios } = useContext();
    const { page, conditions, fetchState } = useQueryFetch({
      setupConditions: (conditions, routeQuery) => {
        conditions.a = routeQuery.a;
        return conditions;
      },
      defaultParams: {
        b: 30,
        c: false,
      },
      promiseFn: (params) => $axios.get("/foo/", { params }),
      afterFetch: (data) => {
        resultData.value = data.result_data;
        totalPages.value = data.total_pages;
      },
    });

    return {
      resultData,
      totalPages,
      page,
      conditions,
      fetchState,
    };
  },
});
```

###### tags: `Work` `JavaScript` `Vue`
