# InAppWebView

InAppWebView 的功用好比 HTML 的 iframe，<br/>
提供黑科技讓我們內嵌網頁以解決某些特別棘手的問題。<br/>
來討論一下它的用途和如何使用。

先來個在杰難逃鎮樓。

![領域展開](https://i.ytimg.com/vi/y20CLJyW74Q/hqdefault.jpg =300x)

### 為何使用

簡述一下使用場景：
* Hybrid APP 為了減小體積而把部分業務邏輯遷移到 WEB 專案上實現。
* 想使用的圖表庫 dart 不支援。媽的說的就是你，TradingView。

### 前置作業

簡述一下這個做法會需要準備哪些東西：
* 本地 Server 或 CDN 供應網站資源
* APP 端通信的邏輯
* WEB 端通信的邏輯

### 組件

記得給容器的尺寸，不然有可能會報錯。<br/>
關鍵是 `initialUrlRequest` 必須是**請求**，這就是為何本地的網站 assets 會必須額外開啟一個本地的 server 之緣故。

```dart
@override
Widget build(BuildContext context) {
  return Container(
    height: 360,
    child: InAppWebView(
      initialUrlRequest: URLRequest(
        url: 'third-party-website-url',
      ),
      initialOptions: InAppWebViewGroupOptions(
        crossPlatform: InAppWebViewOptions(
          clearCache: false,
          supportZoom: false,
          transparentBackground: true,
        ),
      ),
      // 將所有手勢傳給 WebView
      gestureRecognizers: {
        Factory(() => EagerGestureRecognizer()),
      },
      onWebViewCreated: (controller) {
        // 綁定客製化事件
        _attachHandler(controller);
      },
      onLoadError: (_controller, _url, _code, message) {
        print('[InAppWebView]: failed to load webview, $message');
      },
    ),
  );
}
```

### 通信方式

建議先粗略看一遍下面的範例再回來這邊比較好搞懂脈絡。

一句話總結 InAppWebView 做的事情：<br/>
**APP 和 WEB 兩端會各有一個 controller 進行跨端的發佈和訂閱。**

若兩端彼此傳遞的參數為 object 建議配合 dart 和 typescript 兩邊定義好描述物件的 model，有點費工但至少下一個維護你程式碼的人會很感激你這麼做。

#### APP 端

基本上在 `onWebViewCreated` 事件後即可綁定自定義的訂閱事件。

透過 `controller.addEventListner` 訂閱來自 WEB 端的訊息。<br/>
**`callback` 可以是非同步的**( WEB 端的發布訊息會用 JavaScript 的 `Promise` 來等待 APP 端回傳的參數)。

```dart
// WEB -> [APP]

void _attachController (controller) {
  _controller = controller;
  
  _controller?.addEventListener(
    handlerName: 'restaurantIsReady',
    callback: (arguments) {
      // 建議把 model 訂清楚
      return RestaurantOptions(...);
    }
  );
  
  _controller?.addEventListener(
    handlerName: 'cook',
    callback: (arguments) async {
      // arguments 型別為陣列
      final Chef chef = arguments[0];
      final Order order = arguments[1];

      final course = await _cook(chef, order);
      return course;
    }
  );
}
```

透過 `controller.evaluateJavaScript` 發布訊息給 WEB 端。

```dart
// [APP] -> WEB

void footTheBill(int tableNumber) {
  _controller.evaluateJavaScript(
    source: 'window.checkout($tableNumber)'
  );
}
```

#### WEB 端

html 跟 typescript 引入的部分沒什麼重點我省略不講，<br/>
我們來看看 Window 裡面都被我定義了些什麼：

```typescript
declare global {
  interface Window {
    flutter_inappwebview: {
      // WEB 端透過 callHandler 發布訊息給 APP 端
      // 第一個變數為想觸發的 APP 端 handlerName
      // 第二個以後的變數為要傳遞給該事件之 callback 的變數
      callHandler: (handlerName: string, ...args: any) => Promise<any>
    },
    // WEB端透過自定義 window 上的函式訂閱 APP 端發布的訊息
    // 小心不要跟 window 的預設函式撞名發生慘劇
    checkout: (tableNamber :int) => void
  }
}
```

透過 `window.flutter_inappwebview.callHandler` 發布訊息給 APP 端。<br/>

```typescript
// [WEB] -> APP

window.addEventListener('flutterInAppWebViewPlatformReady', (event) => {
  window.flutter_inappwebview.callHandler('restaurantIsReady')
    .then((options: RestaurantOptions) => {
      // ...
    })
});
```

透過在 window 上的自定義事件訂閱來自 APP 端的訊息。

```typescript
// APP -> [WEB]

function checkout (tableNamber :int) {
  // ...
}
window.checkout = checkout;
```

### 結語

以上就是 InAppWebView 不負責任的介紹，<br/>
如果你的 APP 真的很肥但還想要更肥，<br/>
或是你有一些很難很刁鑽的功能 APP 無法實現，<br/>
也許可以考慮接個 InAppWebView 讓人生輕鬆一點。

![並沒有](https://memeprod.sgp1.digitaloceanspaces.com/user-resource/a31207a4daab475e4d64655ea6577364.png =300x)

### 參考

* [Flutter InAppWebView DOC](https://inappwebview.dev/docs/)

###### tags: `Flutter`
