# 學習列表

近期的筆記會以掌握前端相關的底層實作原理跟使用方法為主題去撰寫，<br />
如果時間還夠就寫開發 side-project 的紀錄。

後面的小括號代表認真閱讀過整篇文章的次數。

### 打包

- Webpack
  - [揭秘 webpack 插件工作流程和原理](https://juejin.cn/post/6844904161515929614) (0)
  - [Github - CompressionWebpackPlugin](https://github.com/webpack-contrib/compression-webpack-plugin) (0)
  - [Github - CopyWebpackPlugin](https://github.com/webpack-contrib/copy-webpack-plugin) (0)
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
- Dynamic Import
- Lazy Load
- Webpack Bundle Analyzer
  - [利用 webpack-bundle-analyzer 來分析檔案大小](https://medium.com/coding-hot-pot/%E5%88%A9%E7%94%A8webpack-bundle-analyzer%E4%BE%86%E5%88%86%E6%9E%90%E6%AA%94%E6%A1%88%E5%A4%A7%E5%B0%8F-627801925604) (1)
- Virtualized List
  - [浅说虚拟列表的实现原理](https://blog.csdn.net/chern1992/article/details/107146268) (0)
- Tree Shakng
- Preload、Prefetch and other in \<link\> tag
  - preload
  - preconnect
  - dns-prefetch
- CDN & Cache
  - [Web 前端性能优化之 CDN](https://www.cnblogs.com/wasbg/p/10921373.html) (1)
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

### 前端框架(須了解主流框架實作方式)

- React
  - Hooks
- Next
  - SSR 原理
- Vue
  - Composition API
  - Watch
  - Computed
  - NextTick
- Nuxt
  - SSR 原理

### App

- Flutter
  - [Build flavors in Flutter (Android and iOS) with different Firebase projects per flavor](https://medium.com/@animeshjain/build-flavors-in-flutter-android-and-ios-with-different-firebase-projects-per-flavor-27c5c5dac10b) (0)

### HTML5

- Svg
- Canvas
- IFrame
  - [Iframe 跨域傳值在 iOS 失效的解法﹍利用網址 + localStorage + cookie 並用](https://www.wfublog.com/2017/12/iframe-cross-domain-ios-localstorage-cookie-url.html) (1)
- Map Area

### 通訊協議

- HTTP、HTTPS
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

### 實作(或閱讀原始碼)

- Promise
  - [JavaScript Promise | 為了瞭解原理，那就來實作一個 Promise 吧!](https://medium.com/%E6%89%8B%E5%AF%AB%E7%AD%86%E8%A8%98/implement-promise-aed55f3e84e9) (0)
- MVVM
- Map
  - [map 底层为什么用红黑树实现](https://blog.csdn.net/Ternence_zq/article/details/109821769) (1)
- Debounce & Throttle
  - [Debounce & Throttle — 那些前端開發應該要知道的小事(一)](https://medium.com/@alexian853/debounce-throttle-%E9%82%A3%E4%BA%9B%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC%E6%87%89%E8%A9%B2%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E5%B0%8F%E4%BA%8B-%E4%B8%80-76a73a8cbc39) (0)

### Side-Project

- GoChat (Nuxt + Golang)
- N-hxntxi (Flutter + Golang)

### 其他

- WebAssembly
- Golang (做 Side-Project 用)
- [牛客網 - 前端校招面试题目合集](https://www.nowcoder.com/ta/review-frontend) (30/501)(0)

###### tags: `Interview` `Front-end`
