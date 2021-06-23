# Dio - 功能強大的 Http 請求庫

公司黑客松打到一半紀錄一下，<br />
因為覺得 GetXConnect 要切太多格式雷同的 providers 有點冗，<br />
畢竟服務規模如果不大的情境下其實用一個 network service 頂著就好，<br />
所以就決定用 dio 接請求了，相對簡單又跟 axios 87%像，真香。

![真是讓人High到不行啊](https://i.imgur.com/QxiITJ4.gif =400x)

### 優點

1. interceptor - 提供跟 axios 差不多簡易的鉤子，方便做統一擷取資料格式跟處理字定義錯誤格式。
2. 簡單的參數 - `FormData`、`Map`、含有 `MultipartFile` 的 `FormData` 都在守備範圍中。

### 踩坑

1. 定義模型 - 我之前完全沒用過 Flutter，很常遇到拿 attribute 編譯直接報錯的情況，這裡我們需要特別定義回傳的格式才行，做法我在後面會繼續說明。
2. `onResponse` 裡的 `handler.next` 的參數**必須是 `Response<dynamic>`** - 跟以前使用 JS 的 axios 認知不同，就算你想直接回傳 `response.data.data` 編譯也不會讓你過，我參考了官方的範例做法是把 `response.data` 直接改掉傳回去，因為 data 是被 Response 是先定義的合法 attribute。
3. 兩層 `xxxModel.fromJson` 的 parsing - `onResponse` 時做**請求的 parsing**，判斷後端服務客製化的 code 合法後，對**請求得到的資料做的 parsing**。

### 定義模型

下面我們用 LoginModel 當範例：

1. 安裝依賴到 pubspec.yaml
   ```yaml
   dependecies:
     json_annotation: ^4.0.0
   dev_dependencies:
     build_runner: ^2.0.0
     json_serializable: ^4.0.0
   ```
2. 定義 `LoginModel`，相對位置於`lib/models/login_model.dart`。

   ```dart
   import 'package:json_annotation/json_annotation.dart';
   import 'package:my_app/models/token_model.dart';

   part 'login_model.g.dart';

   @JsonSerializable()
   class LoginModel {
     LoginModel({
       required this.token,
       required this.created,
     });

     /**
       *  注意兩件事
       *  1. 可以把用 JsonKey 把 data attribute 轉成駝峰式命名。
       *  2. 當模型較深層，可以引用其他 model class。
       */
     @JsonKey(name: 'token')
     TokenModel token;

     @JsonKey(name: 'created')
     bool created;

     factory LoginModel.fromJson(Map<String, dynamic> json) =>
         _$LoginModelFromJson(json);
     Map<String, dynamic> toJson() => _$LoginModelToJson(this);
   }

   ```

3. 執行 `pub run build_runner build` 生成 `login_model.g.dart`
4. 在專案中快樂使用 `LoginModel.fromJson` 將 JSON 資料轉成 Class

之後想要新增任意 model 都可以依樣畫葫蘆，讚讚。

### 使用

以下為 Network Service 的 Class 實作：

```dart
import 'package:dio/dio.dart';
import 'package:get_storage/get_storage.dart';
import 'package:my_app/models/login_model.dart';
import 'package:my_app/models/response_wrapper_model.dart';

Dio initNetworkServiceDio() {
  var dio = Dio(
    BaseOptions(
      baseUrl: 'BASE_URL',
      connectTimeout: 30000,
      receiveTimeout: 30000,
      responseType: ResponseType.json,
    ),
  );
  dio.interceptors.add(
    InterceptorsWrapper(
      onRequest: (options, handler) {
        GetStorage box = GetStorage();
        final accessToken = box.read('access_token');
        /*
          登入若有提供 token 的效期的話
          可以在打請求前判斷效期是否已過做重 fetch 請求的事
          不過目前公司黑客松沒提供 refresh token 的 API
         */
        if (accessToken is String) {
          options.headers['Authorization'] = 'Bearer $accessToken';
        }
        handler.next(options);
      },
      onResponse: (Response response, handler) {
        // 請求的 parsing
        final body = ResponseWrapperModel.fromJson(response.data);
        if (body.code != 2000) {
          return handler.reject(
            DioError(
              requestOptions: response.requestOptions,
              response: response,
              type: DioErrorType.other,
              error: body.msg,
            ),
          );
        }
        // 改寫 response.data
        response.data = body.data;
        return handler.next(response);
      },
      onError: (DioError e, handler) {
        // invalid responseStatusCode 的錯誤處理在這邊
        return handler.next(e);
      },
    ),
  );
  return dio;
}

class NetworkService {
  static final Dio _dio = initNetworkServiceDio();

  // 註冊/登入
  static Future<LoginModel> login(Map form) async {
    final response = await _dio.post('login/', data: form);
    // 請求得到的資料的 parsing
    return LoginModel.fromJson(response.data);
  }

  // 取得SMS驗證碼
  static Future<Response<dynamic>> smsValidation(Map form) =>
      _dio.post('sms-verification/', data: form);
}
```

### 雜談

浪費了不少時間在踩坑上，希望以後要接請求的朋友們不會這麼痛苦。<br />
如果有更好的做法也希望能交流一下。<br />
題外話，用手機測 SMS 登入失敗了很多次，不小心名正言順地當了一回薪水小偷，<br />
好險 PM 說沒事。

![SMS, SMS everywhere](https://i.imgur.com/R9ptV5w.jpg =200x)

### 參考

- [Flutter - Dio](https://pub.dev/packages/dio)
- [Flutter - json_serializable](https://pub.dev/packages/json_serializable)
- [Flutter - json_annotation](https://pub.dev/packages/json_annotation)
- [Flutter - build_runner](https://dart.dev/tools/build_runner)
- [Youtube - Flutter Api Calling and JSON Parsing using Dio and Json Serializable](https://www.youtube.com/watch?v=lvRsi3PjckI)

###### tags: `Flutter`


