# Mitt

![Mitt](https://i.imgur.com/PO2zokr.png =200x)

最近發現有一個適合取代 EventBus 的函式庫，原始碼不僅簡單易懂、檔案又小(500b)，但卻能輕鬆達成發佈/訂閱模式的效果，來看看原始碼是如何實作的吧～

### 初始化

看看函數的宣告式，先忽略回傳介面 `Emitter` 不看，我們從型別定義觀察 `all` 的結構：

- `all` 的型態為 ES6 的 Map。

```typescript
export type EventHandlerMap = Map<
  EventType,
  EventHandlerList | WildCardEventHandlerList
>;

export default function mitt(all?: EventHandlerMap): Emitter {
  all = all || new Map();
  return {
    all,
    // ...
  };
}
```

1. `all` 的 key 為 string 或 symbol。

```typescript
export type EventType = string | symbol;
```

2. `all` 的 value 為儲存 handler 的陣列。

```typescript
export type EventHandlerList = Array<Handler>;
export type WildCardEventHandlerList = Array<WildcardHandler>;
```

3. hanlder 分成兩種，使用者須依照文檔對 eventType 綁事件和呼叫事件。

```typescript
export type Handler<T = any> = (event?: T) => void;
export type WildcardHandler = (type: EventType, event?: any) => void;
```

看完的 OS：

![還有這種操作](https://i.imgur.com/tSl5mLg.png =200x)

哇靠！對欸，現在 Functional Programming 的觀念逐漸成為前端主流 meta ，**在不需要煩惱 this 指向哪裡的情況下**確實是可以用一張單例的 Map 和存 handler 的 Array 打發的。

### 監聽事件

```typescript
// TODO
```

### 觸發事件

```typescript
// TODO
```

### 實戰

這邊用我在公司重構的部分程式碼當範例，重構開始：

1. 確保 Mitt 單例 − 所有專案要用的 `mittBus` 在 utils 生成實例，有想裸用(?)可以直接引入使用。

   決定單例的理由如下：

   - 因為現在前端**用到的機會偏少**，寫過 Vue 的人都知道 Vuex 因為有模塊維護起來很容易。現在大概只有通用組件在某些情況才會需要透過它來傳遞事件。
   - 有**一致規範命名**就可以輕鬆繞開綁錯事件的情形。

```javascript
// @/utils/mitt
import mitt from "mitt";
export const mittBus = mitt();
```

2. 設計 hook 。

```javascript
// @/hooks/useMitt
import { mittBus } from "@/utils/mitt";
import { onUnmounted } from "@vue/composition-api";

export default function useMitt() {
  // 用 Set 記憶組件內的局部事件 key
  const set = new Set();

  // 組件不用時註銷不再需要監聽局部事件
  onUnmounted(() => {
    [...set.keys()].forEach((eventType) => {
      mittUnlisten(eventType);
    });
  });

  function mittListen(eventType, handler) {
    set.add(eventType);
    mittBus.on(eventType, handler);
  }

  function mittUnlisten(eventType) {
    set.delete(eventType);
    mittBus.off(eventType);
  }

  function mittLaunch(eventType, payload) {
    mittBus.emit(eventType, payload);
  }

  return {
    mittBus,
    mittListen,
    mittUnlisten,
    mittLaunch,
  };
}
```

心動不如趕快行動，現在就把專案的 EventBus 換成 Mitt 試試吧！

### 參考

[Mitt](https://github.com/developit/mitt)

###### tags: `Library` `TypeScript` `Mitt`
