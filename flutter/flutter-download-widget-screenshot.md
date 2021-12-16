# 下載組件截圖

不是全螢幕截圖，<br/>
而是要把特定範圍的組件存成圖片。<br/>
類似 web 端的 html2canvas<br/>
要完成這件事需要三個步驟：
1. 用 `RepaintBoundary` 包圍想擷取成圖片的組件，並綁定一個 `GlobalKey`
2. `Widget` 轉 `Unit8List`
3. 下載 `Uint8List` 至裝置的圖庫

### Widget 轉 Uint8List

傳入之 key 為 flutter 的 `GlobalKey`，方便我們像是操作 DOM 一樣可以指到想截圖的 `RepaintBoundary` 組件。

##### 程式碼

```dart
import 'dart:typed_data';
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';
import 'dart:ui' as ui;

Future<Uint8List> captureImage(
  GlobalKey key, [
  double pixelRatio = 3.0,
  ui.ImageByteFormat format = ui.ImageByteFormat.png,
]) async {
  try {
    RenderRepaintBoundary boundary =
      key.currentContext!.findRenderObject()! as RenderRepaintBoundary;
    // Issue: https://github.com/flutter/flutter/issues/22308
    await Future.delayed(const Duration(milliseconds: 200));
    ui.Image image = await boundary.toImage(pixelRatio: pixelRatio);
    ByteData? byteData = await image.toByteData(format: format);
    final bytes = byteData!.buffer.asUint8List();
    return bytes;
  } catch (e) {
    throw e;
  }
}
```

### 下載 Uint8List 至裝置的圖庫

要寫入剛剛轉換成 unit8List 到圖庫，<br>
我們僅需要使用 `image_gallery_saver` 這個插件並補上兩個平台對應所需要的權限字串即可。

##### 權限

###### Android

AndroidManifest.xml

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<application android:requestLegacyExternalStorage="true" ...>...</application>
```

###### iOS

Info.plist

```plist
<key>NSPhotoLibraryAddUsageDescription</key>
<string>This app requires permission to save images to gallery.</string>
```

##### 程式碼

```dart
import 'package:image_gallery_saver/image_gallery_saver.dart';
import 'package:permission_handler/permission_handler.dart';

Future<bool> saveImage(
  bytes,
  String name, {
  quality = 60,
}) async {
  try {
    final isGranted = await Permission.storage.request().isGranted;
    if (!isGranted) return false;
    final res = await ImageGallerySaver.saveImage(
      bytes,
      quality: quality,
      name: name,
    );
    return res['isSuccess'];
  } catch (e) {
    throw e;
  }
}
```

### 在組件中使用

在想擷取的組件外包一個 `RepaintBoundary`，並設定一個 `GlobalKey`。

##### 程式碼

```dart
import 'package:flutter/material.dart';
import 'package:cadddy/utils/image.dart';

class Example extends StatefulWidget {
  const Example({ Key? key }): super(key: key);
  
  @override
  _ExampleState createState() => _ExampleState();
}

class _ExampleState extends State<Example> {
  const _ExampleState({
    Key? key,
  }): super(key: key);
  final _globalKey = new GlobalKey();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Container(
          child: RepaintBoundary(
            key: _globalKey,
            // 這邊裡面的 Widget 就是你想擷取的部分
            child: Text('邀请卡'),
          ),
        ),
      ),
      floatActionButton: FloatActionButton(
        onPressed: () async {
          final bytes = await captureImage(_globalKey);
          final res = await saveImage(bytes, '邀请卡.png');
          print('下载成功');
        }
      ),
    );
  }
}
```

##### 成果

![download button](https://i.imgur.com/I7j5LOH.png =200x) ![download image success](https://i.imgur.com/Q34riZM.png =200x)

### 參考
- [Export your widget to image with flutter](https://medium.com/flutter-community/export-your-widget-to-image-with-flutter-dc7ecfa6bafb)
- [image_gallery_saver](https://pub.dev/packages/image_gallery_saver)

###### tags: `Flutter`
