# IT 鬼故事 - 套件依賴導致頁面異常

整理一下發生問題到解決問題的經過：

1. PM 反應早上看到新頁面正常運作，下午整頁看不到任何東西。
2. 檢查線上的新頁面，顯示 **TypeError**，問題報錯顯示在該頁 **vue-condition-watcher** 初始化的段落。
3. 在本地跑一次 `yarn build` 配合 live-server 檢查新頁面，顯示正常無任何異狀。
4. 將問題和發現的線索具體描述給其他同事，大家一起討論。發現另一個同事也在其他產品用到 **vue-condition-watcher** 的地方出現相同問題。
   ![TypeError](https://i.imgur.com/Vm5BHZH.jpg)

5. 鎖定問題可能出在部署時的 **node** 版本和 **vue-condition-watcher** 的依賴，嘗試在本地用 nvm 降版本跑 `yarn install`，結果爆出 **vue-demi** 版本跟目前運行 **node** 不兼容的訊息。
   ![Dependencies warning](https://i.imgur.com/gdyxtWA.png)

6. 看 **vue-demi** 的 repo 最近的 commit 發現 engines 裡的 **node** 變成 >= 12，然後檢查 **vue-condition-watcher** 的依賴發現 **vue-demi** 是用 latest，茅塞頓開。
   ![](https://i.imgur.com/L12cNey.jpg)

7. 和全體同事討論要升專案的版本還是 **vue-condition-watcher** 做一個兼容版本，最後考慮現在常用的套件普遍依賴 **node** >= 12 ，先將開發環境的 **node** 升級結案。

### 結論

開發套件請小心使用 latest 這個標籤，怕。

###### tags: `Work` `yarn`
