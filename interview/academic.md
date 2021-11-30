# 學習列表

近期的筆記會以掌握前端相關的底層實作原理跟使用方法為主題去撰寫，<br />
如果時間還夠就寫開發 side-project 的紀錄。

後面的小括號代表認真閱讀過整篇文章的次數。

### 瀏覽器

- [浏览器与Node的事件循环(Event Loop)有何区别?](https://juejin.cn/post/6844903761949753352) (0)
- [MDN - 操控瀏覽器歷史紀錄](https://developer.mozilla.org/zh-TW/docs/Web/API/History_API) (1)
- CORS
    - [CORS 完全手冊（二）：如何解決 CORS 問題？](https://blog.huli.tw/2021/02/19/cors-guide-2/) (1)
    - [原來 CORS 沒有我想像中的簡單](https://blog.techbridge.cc/2018/08/18/cors-issue/) (1)
- X

### 資安

- [中間人攻擊](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB) (1)
- [[XSS 1] 從攻擊自己網站學 XSS (Cross-Site Scripting)](https://medium.com/hannah-lin/%E5%BE%9E%E6%94%BB%E6%93%8A%E8%87%AA%E5%B7%B1%E7%B6%B2%E7%AB%99%E5%AD%B8-xss-cross-site-scripting-%E5%8E%9F%E7%90%86%E7%AF%87-fec3d1864e42) (1)
- [身為 Web 工程師，你一定要知道的幾個 Web 資訊安全議題](https://medium.com/starbugs/%E8%BA%AB%E7%82%BA-web-%E5%B7%A5%E7%A8%8B%E5%B8%AB-%E4%BD%A0%E4%B8%80%E5%AE%9A%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E5%B9%BE%E5%80%8B-web-%E8%B3%87%E8%A8%8A%E5%AE%89%E5%85%A8%E8%AD%B0%E9%A1%8C-29b8a4af6e13) (1)

### 打包

- Webpack
  - [揭秘 webpack 插件工作流程和原理](https://juejin.cn/post/6844904161515929614) (0)
  - [Github - CompressionWebpackPlugin](https://github.com/webpack-contrib/compression-webpack-plugin) (0)
  - [Github - CopyWebpackPlugin](https://github.com/webpack-contrib/copy-webpack-plugin) (0)
  - [sass-loader - 開始撰寫 SCSS 語法吧！](https://ithelp.ithome.com.tw/articles/10185426) (1)
- Rollup
- Vite
  - HMR 原理

### 效能優化

- Performance Analyzers
  - LightHouse
  - PageSpeed
- Core Web Vital
  - [Web Vitals](https://web.dev/vitals/) (1)
  - [Optimize Largest Contentful Paint](https://web.dev/optimize-lcp/) (0)
- Code Uglify
- Image Minimize
- Critical Render Path
  - [關鍵轉譯路徑 Critical Rendering Path](https://cythilya.github.io/2018/07/13/critical-rendering-path/) (1)
  - [如何優化像素管道的 Styles 和 Layout？](https://cythilya.github.io/2018/07/19/styles-and-layout/) (1)
  - [Layout thrashing cheatsheet](https://devhints.io/layout-thrashing) (1)
- Code Splitting
  - [Webpack - Code Splitting
    ](https://webpack.js.org/guides/code-splitting/#root) (1)
- Dynamic Import
- Lazy Load
- Webpack Bundle Analyzer
  - [利用 webpack-bundle-analyzer 來分析檔案大小](https://medium.com/coding-hot-pot/%E5%88%A9%E7%94%A8webpack-bundle-analyzer%E4%BE%86%E5%88%86%E6%9E%90%E6%AA%94%E6%A1%88%E5%A4%A7%E5%B0%8F-627801925604) (1)
- Virtualized List
  - [浅说虚拟列表的实现原理](https://blog.csdn.net/chern1992/article/details/107146268) (0)
  - [Building a virtualized list from scratch](https://medium.com/ingeniouslysimple/building-a-virtualized-list-from-scratch-9225e8bec120)
- Tree Shakng
- Preload、Prefetch and other in \<link\> tag
  - preload
  - preconnect
  - dns-prefetch
  - [Preload, Prefetch And Priorities in Chrome](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf) (0)
  - [深入淺出 Preload, Prefetch 和 Preconnect：三種加快網頁載入速度的 Resource Hint 技巧](https://shubo.io/preload-prefetch-preconnect/) (1)
- CDN & Cache
  - [Web 前端性能优化之 CDN](https://www.cnblogs.com/wasbg/p/10921373.html) (1)
  - [前端工程师面试宝典 - CDN](https://fecommunity.github.io/front-end-interview/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/8.CDN.html) (1)
  - Cloudflare
- Write Good Code
  - [Vue 3 Style Guide](https://v3.vuejs.org/style-guide/#simple-computed-properties-strongly-recommended) (0)
- CSSSprite
  - [CSS Sprites: What They Are, Why They’re Cool, and How To Use Them](https://css-tricks.com/css-sprites/) (0)
- Content Encoding
  - [vuecli 開啓 gzip 壓縮以及 nginx 的設定](https://tw511.com/a/01/10382.html) (0)
  - [智能压缩，摆脱用 Gzip 还是 Brotli 的纠结](https://zhuanlan.zhihu.com/p/41467559#:~:text=%E9%92%88%E5%AF%B9%E5%B8%B8%E8%A7%81%E7%9A%84Web%20%E8%B5%84%E6%BA%90,%E9%9D%9E%E5%B8%B8%E9%AB%98%E7%9A%84%E5%8E%8B%E7%BC%A9%E7%8E%87%E3%80%82) (0)
  - [Brotli vs Gzip Compression. How we improved our latency by 37%](https://medium.com/oyotech/how-brotli-compression-gave-us-37-latency-improvement-14d41e50fee4) (0)
  - [简单聊聊 GZIP 的压缩原理与日常应用](https://juejin.cn/post/6844903661575880717) (0)
- [今晚，我想來點 Web 前端效能優化大補帖！](https://medium.com/starbugs/%E4%BB%8A%E6%99%9A-%E6%88%91%E6%83%B3%E4%BE%86%E9%BB%9E-web-%E5%89%8D%E7%AB%AF%E6%95%88%E8%83%BD%E5%84%AA%E5%8C%96%E5%A4%A7%E8%A3%9C%E5%B8%96-e1a5805c1ca2) (0)
- [我是如何將網頁效能提升 5 倍的 — 構建優化篇](https://www.mdeditor.tw/pl/glmW/zh-tw) (1)

### 前端框架(須了解主流框架實作方式)

- React
    - Fiber Tree
        - [React16源码解析(一)- 图解Fiber架构](https://segmentfault.com/a/1190000020736966) (1)
        - [Inside Fiber: in-depth overview of the new reconciliation algorithm in React
](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e) (0)
    - [将 React 作为 UI 运行时](https://overreacted.io/zh-hans/react-as-a-ui-runtime/) (0)
    - SPA
    - DOM
    - Hooks
        - useState
            - [useEffect 的完整指南](https://overreacted.io/zh-hant/a-complete-guide-to-useeffect/) (1)
        - useEffect
        - useContext
        - useRef
    - Library
        - SWR
    - xxx
- Next
     - SSR
    - SSG
    - getStaticPaths
    - getStaticProps
    - getServerSideProps
- Vue
    - [Vue3 源码解析](https://segmentfault.com/a/1190000039848609) (0)
    - [从零手写简易Vue3](https://juejin.cn/post/6891589578008821767) (0)
    - [Vue 元件data為什麼必須是函式？](https://iter01.com/17771.html) (1)
    - Optimization
        - [Vue性能提升之Object.freeze()](https://juejin.cn/post/6844903922469961741) (1)
        - 
    - SPA
    - Virtual DOM
    - Composition API
    - Ref
    - Reactive
    - Watch
    - Computed
    - NextTick
    - Library
        - Vuex
        - Vue-Router
        - Vue-I18n
  - xxx
- Nuxt
    - SSR
    - SSG
    - asyncData
    - fetch
    - middleware

### App

- Flutter
  - [Build flavors in Flutter (Android and iOS) with different Firebase projects per flavor](https://medium.com/@animeshjain/build-flavors-in-flutter-android-and-ios-with-different-firebase-projects-per-flavor-27c5c5dac10b) (0)

### HTML5

- Svg
- Canvas
    - [目錄頁 : 成為Canvas Ninja ～ 理解2D渲染的精髓](https://ithelp.ithome.com.tw/articles/10281906) (0)
- IFrame
  - [Iframe 跨域傳值在 iOS 失效的解法﹍利用網址 + localStorage + cookie 並用](https://www.wfublog.com/2017/12/iframe-cross-domain-ios-localstorage-cookie-url.html) (1)
- Map Area
- ContentEditable
- Input
    - [drag drop files into standard html file input](https://stackoverflow.com/questions/8006715/drag-drop-files-into-standard-html-file-input) (1)

### CSS3

- [當 flexbox 遇到 height: 100%](https://medium.com/@littleDog/%E7%95%B6flexbox%E9%81%87%E5%88%B0height-100-242703fcaab6) (1)
- [overflow: auto not working in Safari OSX](https://stackoverflow.com/questions/32971425/overflow-auto-not-working-in-safari-osx) (1)
- [Chrome / Safari not filling 100% height of flex parent
  ](https://stackoverflow.com/questions/33636796/chrome-safari-not-filling-100-height-of-flex-parent) (1)
- [Applying an ellipsis to multiline text [duplicate]](https://stackoverflow.com/questions/33058004/applying-an-ellipsis-to-multiline-text/41137262) (1)
- [Placeholder in contenteditable - focus event issue](https://stackoverflow.com/questions/9093424/placeholder-in-contenteditable-focus-event-issue/18368720#18368720) (0)

### JavaScript

- Hoisting
    - [我知道你懂 hoisting，可是你了解到多深？
](https://blog.techbridge.cc/2018/11/10/javascript-hoisting/) (1)
- Strict Mode
    - [MDN - Strict mode](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Strict_mode) (1)
- ECMAScript
    - [ES2021 / ES12 New Features](https://backbencher.dev/articles/javascript-es2021-new-features) (1)
- Garbage Collection
    - [垃圾回收](https://zh.javascript.info/garbage-collection) (1)
- Promise
  - [JavaScript Promise | 為了瞭解原理，那就來實作一個 Promise 吧!](https://medium.com/%E6%89%8B%E5%AF%AB%E7%AD%86%E8%A8%98/implement-promise-aed55f3e84e9) (0)
- MVVM
- Map
  - [map 底层为什么用红黑树实现](https://blog.csdn.net/Ternence_zq/article/details/109821769) (1)
- Debounce & Throttle
  - [Debounce & Throttle — 那些前端開發應該要知道的小事(一)](https://medium.com/@alexian853/debounce-throttle-%E9%82%A3%E4%BA%9B%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC%E6%87%89%E8%A9%B2%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E5%B0%8F%E4%BA%8B-%E4%B8%80-76a73a8cbc39) (0)
- MutationObserver
    - [MDN - MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) (1)
- MessageChannel
    - [MDN - MessageChannel](https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel) (1)
- Proxy, Reflect
    - [一起來了解 Javascript 中的 Proxy 與 Reflect
](https://blog.techbridge.cc/2018/05/27/js-proxy-reflect/) (0)
- Array
    - [前端进阶算法2：从Chrome V8源码看JavaScript数组（附赠腾讯面试题）](https://github.com/sisterAn/JavaScript-Algorithms/issues/2) (1)
- Symbol
    - [ES6 系列之模拟实现 Symbol 类型](https://github.com/mqyqingfeng/Blog/issues/87) (1)
- V8
    - [从V8角度揭秘你不知道的面试八股文](https://juejin.cn/post/6942643533417283591) (1)
    - [高性能 JavaScript 引擎 V8 - 垃圾回收](https://juejin.cn/post/6934638688563363853) (0)
- X

   

### 通訊協議

- [网络分层TCP/IP 与HTTP](https://juejin.cn/post/6844903568860774408) (1)
- HTTP、HTTPS
  - [都 2019 年了，还问 GET 和 POST 的区别](https://blog.fundebug.com/2019/02/22/compare-http-method-get-and-post/) (1)
  - [TCP 三向交握 (Three-way Handshake)](https://notfalse.net/7/three-way-handshake) (0)
  - [循序漸進理解 HTTP Cache 機制](https://blog.techbridge.cc/2017/06/17/cache-introduction/) (1)
  - [一文搞懂 HTTP 和 HTTPS 是什麼？兩者有什麼差別](https://tw.alphacamp.co/blog/http-https-difference) (1)
  - [作为前端的你了解多少tcp的内容](https://juejin.cn/post/6844903731704791054) (1)
- WebSocket
- WebRTC

### Web API

- WebWorker
  - [Web Worker 使用教程](http://www.ruanyifeng.com/blog/2018/07/web-worker.html) (1)
- SharedWorker
  - [实现多个标签页之间通信的几种方法(sharedworker)](cnblogs.com/cangqinglang/p/10635707.html) (1)
  - [Github - simple-shared-worker](https://github.com/mdn/simple-shared-worker) (1)
- ServiceWorker
- requestAnimationFrame
  - [Using requestAnimationFrame](https://css-tricks.com/using-requestanimationframe/) (1)
- Notification

### Lint

- [Make linting great again!](https://medium.com/@okonetchnikov/make-linting-great-again-f3890e1ad6b8) (1)
- [建立公司內部使用的 eslint-config package](https://medium.com/onedegree-tech-blog/%E5%BB%BA%E7%AB%8B%E5%85%AC%E5%8F%B8%E5%85%A7%E9%83%A8%E4%BD%BF%E7%94%A8%E7%9A%84-eslint-config-package-4b76c089848) (0)

### Electron

- [iThome - ElectronJS 系列](https://ithelp.ithome.com.tw/users/20111867/ironman/2941) (0)

### Side-Project

- GoChat (Nuxt + Golang)
- nHxntxi (Flutter + Golang)

### 監控

- GTM
  - [How to track Single Page Applications with Google Tag Manager](https://dataenthusiast.it/english-version/how-to-track-single-page-application-with-google-tag-manager/) (0)
- xxx

### 其他

- WebAssembly
- Golang (做 Side-Project 用)
- [牛客網 - 前端校招面试题目合集](https://www.nowcoder.com/ta/review-frontend) (30/501)(0)

###### tags: `Interview` `Front-end`
