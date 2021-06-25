# Permission Handler - 請求權限

寫 app 總是逃不掉需要去面對客戶訪問權限的情境 ex: 麥克風、推播、定位...等，使用優質的套件可以幫助我們無痛逃課。

### 踩坑

詳細的使用流程可以參考最下面的文檔不再贅述，這邊我只分享我遇到的坑。

1. 如何在開發環境測試權限? 待補
2. 請求**使用 APP 期間開啟定位的權限**時遇到閃退 - 第一次遇到我直接黑人問號???，納悶哪個步驟漏掉了?結果後來查了一下 issue 才發現我漏掉要在 `Info.plist` 加上 description 的步驟。用 XCode 開啟 `Info.plist` 直接選 addRow，查一下 IOS APP Permission 問題也迎刃而解。

### 範例

以下是開啟**推播**跟在**使用 APP 期間開啟定位**的請求權限的簡易範例。

> ios/Podfile

```Podfile
# ...
post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    target.build_configurations.each do |config|
      config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
        '$(inherited)',

         # dart: [PermissionGroup.location, PermissionGroup.locationAlways, PermissionGroup.locationWhenInUse]
         'PERMISSION_LOCATION=1',

        # dart: PermissionGroup.notification
        'PERMISSION_NOTIFICATIONS=1',
      ]
    end
  end
end
```

> ios/Runner/Info.plist

```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  ...
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>若您關閉「定位服務」，APP的部分功能將會功能會停止運作。</string>
</dict>
</plist>

```

> lib/example.dart

```dart
import 'package:my_app/services/network.dart';
import 'package:permission_handler/permission_handler.dart';

void register(payload) async {
  final user = await NetworkService.register(payload);
  await Permission.notification.request();
  await Permission.locationWhenInUse.request();
  // ...
}
```

### 參考

- [Flutter - permission_handler](https://pub.dev/packages/permission_handler)
- [Flutter Permission Handler Issue 130 - iOS crashes on check permission](https://github.com/Baseflow/flutter-permission-handler/issues/130)
- [info.plist reference: iOS Permission Keys - iOS Dev Recipes](https://www.iosdev.recipes/info-plist/permissions/)

###### tags: `Flutter`
