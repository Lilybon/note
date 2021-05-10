# 開獎網 - 多狀態與計時器管理

![晚安... 早點睡吧!](http://5b0988e595225.cdn.sohucs.com/images/20190227/ef842cd760074d09bcc663410a80f7d9.jpeg =500x)

最近因為都在忙著跟同事打魔物獵人崛起(真 D 齁勝)，<br />
有點怠慢了哈哈，<br />
只好拿上班遇到的酷炫需求來筆記一下刷個存在感。

### 業務需求

給你一支當日開獎時間資訊的 API，<br />
其中 `schedule_open`、`schedule_close`、`schedule_result` 有順向的時間意義。

時間軸如下：

1. →{顯示黑色圓點 loading }→
2. → 封盤 (**schedule_open**) →{顯示黑色圓點 loading }→
3. → 開獎中 (**schedule_close**) →{顯示紅色方形 loading }→
4. → 開獎完成 (**schedule_result**) →{顯示綠色勾勾}→

API 的格式如下：

```json
{
    "code": 2000,
    "msg": null,
    "data": {
        "foo_group_tag": [
            {
                "game_code": "foo game Code",
                "display_name": "foo game display name",
                "issue_number": "20210510",
                "schedule_open": "2021-05-03T12:15:00Z",
                "schedule_close": "2021-05-10T12:10:00Z",
                "schedule_result": "2021-05-10T12:15:00Z"
            },
            ...
        ],
        ...
    }
}
```

請設計一個可儲存多遊戲開獎狀態和計時器的架構。<br />
看完當下覺得好像不太難，但實際做下去才發現...<br />
啊我的頭好痛。

![Professor X's Mind Rays](https://i.kym-cdn.com/entries/icons/original/000/030/110/xmen.jpg =400x)

### 實作

專案用 Nuxt 加 composition api，<br />
關於 `nuxtjs/composition-api` 的 hook 的東西我不會贅述，<br />
請有興趣自己去看文檔。

下面是組件中的變數宣告：<br />
核心是以兩張 map 分別存放遊戲對應之**狀態**和 **timeoutId**

```javascript
const { $axios, $dayjs } = useContext()

// 指示一個順向時間軸的 key 陣列
const DRAW_TIME_LINE = [
  'schedule_open',
  'schedule_close',
  'schedule_result',
]

// 後端 API 回傳之最新開獎結果
const latestSchedule = ref([])

// 以 gameCode 為 key，下一個狀態指針為 value
const gameNextScheduleIndexMap = ref({})

// 以 gameCode 為 key，當前計時之 timeout id 為 value
const gameTimeoutIdMap = {}

// 中止當前所有遊戲的 timeoutId
const clearUpdateStatusTimeout = () => {
  Object.entries(gameTimeoutIdMap).forEach([gameCode, timeoutId] => {
    clearTimeout(timeoutId)
    delete gameTimeoutIdMap[gameCode]
  })
}

// 遞迴更新個別遊戲的 timeoutId
const setUpdateStatusTimeout = (gameCode, millisecondsToNextSchedule, nextScheduleIndex) => {
  const helper = (sIndex) => {
    gameNextScheduleIndexMap.value[gameCode] = sIndex
    gameTimeoutIdMap[gameCode] =
      sIndex === -1 || millisecondsToNextSchedule.length <= sIndex
        ? null
        : setTimeout(() => {
            helper(sIndex + 1)
          }, millisecondsToNextSchedule[sIndex])
  }
  helper(nextScheduleIndex)
}

const { fetch, fetchState } = useFetch(async () => {
  latestSchedule.value = await $axios.get('/latest_schedule_info/', {
    params: {
      date: date.value,
    },
  })
})
```

每次觸發新的計時器的邏輯：

```javascript
// I. 清空前一個開獎結果的 timeoutId
clearUpdateStatusTimeout();

// II. 計算各狀態彼此之間所需要之毫秒數
const today = $dayjs();
const millisecondsToNextScheduleMap = Object.values(newLatestSchedule)
  .flat()
  .reduce((result, gameInfo) => {
    result[gameInfo.game_code] = DRAW_TIME_LINE.map((scheduleKey) =>
      $dayjs(gameInfo[scheduleKey]).diff(today)
    )
      /**
       *  我們只在意還沒到達的下一個最近的倒數計時和其之後的倒數
       *  將相對今天為正之毫秒數轉換成可以被 setTimeout 使用的格式
       */
      .map(
        (milliseconds, index, self) => milliseconds - (self[index - 1] || 0)
      );
    return result;
  }, {});

// III. 一次性更新每個遊戲的下一個狀態的指針位置
gameNextScheduleIndexMap.value = Object.entries(
  millisecondsToNextScheduleMap
).reduce((result, [gameCode, milliseconds]) => {
  result[gameCode] = milliseconds.findIndex((value) => value > 0);
  return result;
}, {});

// IV. 對每個遊戲個別設置遞迴計時器
Object.entries(gameNextScheduleIndexMap.value).forEach(
  ([gameCode, nextScheduleIndex]) => {
    setUpdateStatusTimeout(
      gameCode,
      millisecondsToNextScheduleMap[gameCode],
      nextScheduleIndex
    );
  }
);
```

### 困境與解決方式

1. 職責劃分 - 計時器只能由 client 執行，但又不能直接放在 useFetch 的鉤子裡，**因為 client 初始化組件不會觸發 fetch** (server side 已經幫你打過請求並渲染)。目前想到的可行解是用 immediate watch 在 client 端觸發計時即可。

   ```javascript
   const { fetch, fetchState } = useFetch(async () => {
     latesetSchedule.value = await $axios.get("/latest-schedule-info/");
   });
   if (process.client) {
     watch(
       latesetSchedule,
       () => {
         // 計時器邏輯
       },
       { immediate: true }
     );
   }
   ```

2. 遞迴執行 timeout - 最難的部分，需要逐一思考如何**定義時間軸**、**設計遞迴終止條件**、**轉換後端 API 的格式成可行的計時區間**，本來一直糾結在怎麼寫才能寫得漂亮，結果花了一整個早上還是沒頭緒，下午決定直接憑直覺再事後最佳化就完成了，不一定是最好的解法，但至少足以解決當下的問題。
3. 避免 Memory Leak - 一開始設計時就要考慮進去，只要記得組件沒用到時跟重打 API 時都要去主動終止 timeout id 就好。

   ```javascript
   onUnmounted(clearUpdateStatusTimeout);
   ```

###### tags: `Work` `Front-end`
