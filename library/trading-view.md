# TradingView

TradingView 是一門成熟的繪製 K 線的庫，<br/>
它提供了客製化的接口讓工程師能**盡量無痛**的生成線圖。<br/>
我原本以為離開前公司(券商的網頁開發部門)後就再也沒機會碰到了，<br/>
結果事隔兩年最近工作上的業務需求又遇到了。

在邊看 [chart-libray-tutorial](https://github.com/tradingview/charting_library) 和 github issue 邊研究下，<br/>
我整理了一段小範例避免自己或其他人之後要接 K 線不會對它這麼陌生、這麼痛苦。

![人心最後終究是要回到故鄉來的](http://i.imgur.com/8HFDuqKh.jpg =500x)

順帶一提，<br/>
由於這個庫是非公開的，<br/>
因此**想取得存取權限需要到官網申請**。

### # Widget Constructor

提供非常多樣化的屬性設定，<br/>
請務必配合 chart library 的 wiki 配置。<br/>
再提醒一次，記得去官網申請此 repo 的權限才看得到下面的連結喔。

[TradingView Wiki - Widget Constructor]((https://github.com/tradingview/charting_library/wiki/Widget-Constructor))

```javascript
import Datafeed from './datafeed.js';

const upColor = '#5dc787';
const downColor = '#e45561';

const chart = new TradingView.widget({
  debug: false,
  // Container - 綁定之 DOM 元素 ID
  container: 'tv-chart',
  /* 
   * DataFeed - 資料源 (TradingView 最難搞的一塊)，你必須照著它的玩法
   * 才能使圖表正常運作，這部分會在後續的 DataFeed 章節講解
   * */
  datafeed: DataFeed,
  // Library Path - 指定 chart_library 所在的對應路徑
  library_path: '/charting_library/',
  // Locale - 圖表使用的語系
  locale: 'zh_TW',
  // TimeZone - 請自行參考 wiki TZ database name
  timezone: 'Asia/Taipei',
  // Symbol - 預設的指標或股票對應的標記
  symbol: 'binance:ETHUSDT',
  /* 
   * Preset - 針對平台預設的 feature set，建議先針對是平台設置該屬性再
   * 逐一設定啟禁用其他 feature。
   * */
  preset: 'mobile',
  // Disabled Features - 禁用的功能
  disabled_features: [
    'symbol_search_hot_key', // 搜尋 symbol 輸入框快捷鍵
    'header_symbol_search', // 切換 symbol (股票,指標)的按鈕
    'header_fullscreen_button',
    'header_screenshot',
    'header_compare',
    'header_indicators',
    'header_undo_redo',
    'header_chart_type',
    'header_settings',
    'left_toolbar',
    'legend_widget',
    'use_localstorage_for_settings',
    'go_to_date',
    'timezone_menu',
    'timeframes_toolbar',
    'volume_force_overlay' // k 線和成交量顯示在同一張圖表
  ],
  // Disabled Features - 啟用的功能
  enabled_features: [
    'header_widget', // 顯示表頭
    'header_resolutions', // 表頭切換 K 線時間解析度按鈕
  ],
  // Theme - 圖表主題
  theme: 'dark', 
  /* 
   * Overrides - 如果預設的樣式不合意，官方強烈建議透過 overrides 改寫
   * 圖表樣式以符合最佳實踐。
   * */
  overrides: {
    // 蠟燭身顏色
    'mainSeriesProperties.candleStyle.upColor': upColor,
    'mainSeriesProperties.candleStyle.downColor': downColor,
    // 蠟燭身邊界顏色
    'mainSeriesProperties.candleStyle.borderUpColor': upColor,
    'mainSeriesProperties.candleStyle.borderDownColor': downColor,
    // 蠟燭芯顏色
    'mainSeriesProperties.candleStyle.wickUpColor': upColor,
    'mainSeriesProperties.candleStyle.wickDownColor': downColor,
  }
});
```


### # 技術指標

如果想畫一些技術指標，可以參考 `createStudy` 的文檔，下面示範如何畫出7、25、99日均線。

```javascript
const chart = new TradingView.widget(options);

chart.onChartReady(() => {
  // add MA indicators
  [7, 25, 99].forEach(days => {
    chart
      .activeChart()
      .createStudy('Moving Average', false, false, [days]);
  });
  // ...
});
```

### # DataFeed

![所以我說那個醬汁呢](https://images.chinatimes.com/newsphoto/2018-08-17/656/20180817004531.jpg =400x)

##### 生命週期

todo

##### K 線資料格式

```json
{
  "time": 1632996240000,
  "open": 2991.48,
  "high": 2994.18,
  "low": 2989.13,
  "close": 2989.38,
  "volume": 2001104,
}
```

##### 歷史 K 線

###### DataFeed

```javascript
const lastBarsCache = new Map();

export default {
  onReady: (callback) => {
    // todo
  },

  searchSymbols: async (
    userInput,
    exchange,
    symbolType,
    onResultReadyCallback
  ) => {
    // todo
  },

  resolveSymbol: async (
    symbolName,
    onSymbolResolvedCallback,
    onResolveErrorCallback,
  ) => {
    // todo
  },

  /* 
   * 圖表第一次 resolveSymbol 後和其之後把 K 線圖往左滑或縮小時，若沒有指定時間段的
   * 資料時，TradingView 就會觸發這個鉤子去要新的資料。
   * */
  getBars: async (
    symbolInfo,
    resolution,
    periodParams,
    onHistoryCallback,
    onErrorCallback,
  ) => {
    /* 
     * from: 新的時間段起點 (UNIX 時間戳)
     * to: 新的時間段終點 (UNIX 時間戳)
     * firstDataRequest: 是否為此 symbol 的第一次 getBars (布林值)
     * */
    const { from, to, firstDataRequest } = periodParams;
    try {
      /* 
       * 這邊的格式需要跟後端合作完成，
       * 若以下參數格式跟後端不合，請自己寫配接器轉換。
       * */
      const klines = await fetchKLines({
        resolution: resolution,
        symbol: symbolInfo.full_name,
        start_time: from,
        end_time: to,
        limit: 2000,
      });
      const bars = klines.map(kline => ({
        time: kline.time,
        open: kline.open,
        high: kline.high,
        low: kline.low,
        close: kline.close,
        volume: kline.volume,
      }));

      // 當 noData 為 true 被回傳時，之後滑到更前面就不會再觸發 getBars 鉤子
      onHistoryCallback(bars, {
        noData: bars.length === 0,
      });
    } catch (error) {
      // 若取得歷史資料的過程失敗要使用這個 callback 傳回錯誤資訊
      onErrorCallback(error);
    }
  }
}
```

##### 即時 K 線

###### DataFeed

```javascript
import {
  subscribeOnStream,
  unsubscribeFromStream,
} from './streaming.js';

const lastBarsCache = new Map();

export default {
  getBars: (
    symbolInfo,
    resolution,
    periodParams,
    onHistoryCallback,
    onErrorCallback,
  ) {
    // ...
    try {
      /* 
       * 只在第一次將目前 symbol 的最後一根 K 線快取，
       * 因為第一次拿到的最後一根就是最新的一根 K 線。
       * */
      if (firstDataRequest) {
        lastBarsCache.set(symbolInfo.full_name, {
          ...bars[bars.length - 1],
        });
      }
      onHistoryCallback(bars, {
        noData: bars.length === 0,
      });
    } catch (error) {
      // ...
    }
  },
  subscribeBars: (
    symbolInfo,
    resolution,
    onRealtimeCallback,
    subscribeUID,
    onResetCacheNeededCallback,
  ) => {
    subscribeOnStream(
      symbolInfo,
      resolution,
      onRealtimeCallback,
      subscribeUID,
      onResetCacheNeededCallback,
      /* 
       * 將目前 symbol 的最後一根 k 線傳給 streaming 快取作為比較初始值，
       * 這樣之後 streaming 就知道要如何更新 K 線了
       * */
      lastBarsCache.get(symbolInfo.full_name),
    );
  },

  unsubscribeBars: (subscriberUID) => {
    unsubscribeFromStream(subscriberUID);
  }
}
```

###### Streaming

```javascript
const socket = io('WSS_SERVICE_URL_HERE');
const channelToSubscription = new Map();

socket.on('m', data => {
  console.log('[socket] Message:', data);
  // todo
});

export function subscribeOnStream (
  symbolInfo,
  resolution,
  onRealtimeCallback,
  subscribeUID,
  onResetCacheNeededCallback,
  lastBar,
) {
  // todo 
}

export function unsubscribeFromStream (subscriberUID) {
  // todo
}
```

### # 開發需要留意的事項

下面講的幾點只看說明可能有點抽象，我覺得先試著看懂上面的範例再理解就好。

1. race condition
    - 若想確保 `DataFeed` 即時更新 K 線的鉤子 `subscribeOnStream` 和 `unsubscribeFromStream` 正常運作，請確定 web socket 完成連線再渲染圖表。
2. lastBarCache
    - 以 Map 格式儲存，用 `symbolInfo` 的可辨識字串(ex: 全名)作為 key，最後一根 K 線作為 value。
    - 分別須要在第一次 `getBars` 、 `subscribeOnStream` 提及，**為的是在之後 web socket 可以在收到新訊息時即時更新訂閱的最後一根 K 線**。
3. channelToSubscription
    - 從 web socket 接收資料時，請做出跟 `subscribeOnStream` 時一樣的 `subscribeUID` ，格式須要跟後端溝通討論。 
4. resolution
    - 請求歷史 K 和訂閱即時 K 的提交時間格式參數可能跟 `resolution` 的表示方法不同，最好寫兩支簡單的配接器處理，或請後端參考 TradingView 提供的格式。
    - 記得根據 **當前的最後一根 k 線的時間戳** 和 **圖表的 resolution** 推算 **下一根 k 線的時間戳** 。
5. timezone & locale
    - 取得使用者當前的 timezone 和語系做舒服的初始化。

### # 參考

* [TradingView 官網](https://www.tradingview.com/)
* [tradingview/chart_library](https://github.com/tradingview/charting_library)
* [tradingview/chart-library-tutorial](https://github.com/tradingview/charting-library-tutorial)
* [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

###### tags: `Library` `JavaScript`