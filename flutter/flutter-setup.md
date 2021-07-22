# MacOS 環境設定

![Flutter](https://miro.medium.com/max/1000/1*ilC2Aqp5sZd1wi0CopD1Hw.png =100x)

最近公司要打一個為期不短的黑客松，
因為同事跟我都是懶人，喜歡用全面又設計好的 UI-Kit 和一魚三吃 ( Android、 IOS、 Web App )，所以全公司的前端都要開始學 Flutter 了。

![驚不驚喜 意不意外](https://i.imgur.com/4ktpHJF.png =300x)

怕我之後要用會忘記，也順便幫一下還沒入門就放棄的新手，簡短紀錄哪些東西會用到。

### 安裝 Flutter SDK

1. 去官網安裝 SDK 並解壓縮，你會得到一個 `flutter` 的資料夾。
2. 在 user 資料夾建一個叫 `development` 的資料夾，並將 `flutter` 資料夾貼進去。其實也可以依喜好改名，只要第 4 步驟能參照到對應路徑就好。
3. 開啟 terminal 輸入 `vim ~/.zshrc` 指令，按下 `i` 編輯，在 `.zshrc` 最底下貼上下面這段 code：

   ```
   # Flutter
   export PATH="$PATH:$HOME/development/flutter/bin"
   ```

   按下 `esc` ，再依序輸入 `shift` 和 `wq!` 指令儲存改動。

4. 重啟 terminal，輸入 `flutter --version` 確認是否安裝成功。

### 安裝 Android Studio

1. 去官網安裝 IDE。
2. 打開 IDE，點擊右下角 Configuration > Plugins 輸入 `Flutter` ，下載名字就叫做 `Flutter` 的那個 plugin 。
3. 重啟 IDE ，可以看到 `Create New Flutter Project` 出現在選項中，之後要啟 flutter 的 boilderplate 專案就靠它了。
4. 設定模擬器 **AVD Manager**，這步驟就跟 Flutter 沒什麼關聯，但你要測 App 功能是否符合預期會考慮使用虛擬機測，你可以自己上網查(欸)，或者直接用預設的。

### 安裝 Xcode

1. 去 AppStore 安裝 App (下載檔有夠大包)。
2. 開啟 terminal 輸入以下指令，並輸入你的使用者密碼，配置 Xcode 命令行工具。
   ```bash
   sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
   sudo xcodebuild -runFirstLaunch
   ```
3. 開啟 Xcode 點選上方工具列 Xcode > Open Developer Tool > Simulator ，你就得到一個虛擬的 iPhone 囉。

之後你就可以開心地用 Android Studio 啟一個 Flutter 專案並同時測 iOS 和 Android 的裝置了，大家通通給我上車^^。

![我叫你上車](https://memeprod.sgp1.digitaloceanspaces.com/meme/6f9a32f0ec210b56aed67f4239c43faa.png =400x)

### 參考

[Flutter - get started](https://flutter.dev/docs/get-started/install/macos)
[Android Studio](https://developer.android.com/studio)

###### tags: `Flutter`


