# Riverpod - Provider 的那些坑

提出幾個用 Riverpod 開發需要留意的常見問題，<br/>
歡迎大家提出更多坑讓我增加篇幅。

### 命名

Provider 變數加上 Provider 的後綴 (ex: XXXProvider)<br/>
Notifier 變數加上 Notifier 的後綴 (ex: XXXNotifier)<br/>
理由是統一且你在組件 build 裡 ref.watch 時不會詞窮，統一命名其他人也比較好追蹤問題。

###  ref.read 和 ref.watch 的使用時機

ref.read：
1. 一般函式、Notifier 函式。
2. 組件只在一開始生成時讀取 Provider 的值，後續的變化不用即時更新視圖。

ref.watch：
1. 定義的 Provider 須要基於其他 Provider 的結果即時更新。
2. 組件在 Provider 的值發生變化時需要監測到變化並即時更新視圖。

### 如何監聽 Provider

Riverpod 1.0.3 版本把 Provider listen 做了一些調整，現在可以在值觸發改變時吃到新值和舊值了。

```dart
final count1Provider = StateProvider<int>((_) => 0);
final count2Provider = StateProvider<int>((_) => 0);

final totalCountProvider = Provider((ref) {
  final count1 = ref.watch(count1Provider.state).state;
  final count2 = ref.watch(count2Provider.state).state;
  return count1 + count2;
})
```

```dart
@override
Widget build(BuildContext context) {
  ref.listen<int?>(totalCountProvider, (newValue, oldValue) {
    print('$oldValue -> $newValue');
  });
  // ...
}
```

### FutureProvider 直接在組件取值


想在組件取用 FutureProvider 的值有兩種做法，在簡單的使用情境下，使用

```dart
@override
Widget build(BuildContext context) {
  final config = ref.watch(configProvider).asData?.value;
  return configProvider.when(
    data: (response) { ... },
    err: (error, stackTrace) { ... },
    loading: () { ... },
  );
}
```

但如果組件嵌套的關係過於複雜，上面的做法彈性不夠時，你可以使用以下做法：

```dart
@override
Widget build(BuildContext context) {
  // 組件有提及這個 configProvider 時就會觸發單例初始化，先到先得，資料 fetch 到結果前都會是 null 需要處理好防呆。
  final config1 = ref.watch(config1Provider).asData?.value;
  final config2 = ref.watch(config1Provider).asData?.value;
  // ...
}
```

### FutureProvider 更新值

某些情況你可能會想跟後端重 fetch API 的結果更新 Provider。

```dart
ref.refresh(configProvider);
```

### FutureProvider 俄羅斯套娃

如果有一個 FutureProvider 是基於另一個 FutureProvider 的結果，<br/>
你可以試試看以下作法：
```dart
final AFutureProvider =
    FutureProvider<AModel>((ref) async {
  // 會觸發 BFutureProvider 撈資料，資料回來前都是 AFutureProvider 的 asData?.value 為 null
  final BModel = await ref.watch(BFutureProvider.future);
  return AModel(
    foo: BModel,
  );
});
```

![俄羅斯套娃](http://img.mp.itc.cn/upload/20170517/6fa2c1f07cfa418e89f1a75281c4deff_th.jpg =400x)


### StateNotifier 的 state 拆到粒度越接近原生型別越好

一個 StateNotifier **只負責做一件事**。你可以直接在 StateNotifier 存一些非 state 的變數且關聯的變數(ex: 存放倒數的 timer 和要響應的計時數字 state 在一個 StateNotifier 是 ok 的)，但記住，非 state 的變數是**無法響應更新的**。

### StateNotifier 善用 ref 和其他 Provider 建立聯繫

StateNotifier 是可以透過 constructor 傳遞 ref 的，這個 ref 的功用很強，它可以幫助你去讀取其他 provider 的值，當業務邏輯複雜到趨近崩潰時一定要想起來用他重構。

![與人連結](https://i1.momoshop.com.tw/1619273786/goodsimg/0008/668/174/8668174_L_m.webp =300x)

### StateNotifier 觸發渲染更新

記住一件事，**只有 state 賦值可以觸發渲染更新**。<br/>

首先我來示範一下如何更新 NameList：
```dart
class NameListNotifier extends StateNotifier<List<String>> {
  NameListNotifier() : super([]);
  
  void update(int targetIndex, String newName) {
    state = [
      for (var index = 0; index < state.length; index++)
        if (index == targetIndex)
          newName
        else
          state[index],
    ];
  }
}
```
對，就是這麼麻煩。<br/>
如果直接改 `state[targetIndex]` 的 值會改變 state 的實際值，<br/>
**卻無法觸發重渲**，<br/>
所有陣列常用的方法直接操作 state 都不會觸發重渲，<br/>
但你以為結束了嗎？<br/>

再讓我們示範更新一個 TodoList：
```dart
class Todo {
  const Todo({
    required this.name,
    required this.completed,
  });

  final String this.name;
  final bool this.completed;
}

class TodoListNotifier extends StateNotifier<List<Todo>> {
  NameListNotifier() : super([]);
  
  void update(int targetIndex, Todo newTodo) {
    state = [
      for (var index = 0; index < state.length; index++)
        if (index == targetIndex)
          newTodo
        else
          state[index],
    ];
  }
}
```

什麼？你說只是換一個 newTodo 根本是小菜一碟？<br/>
那請你思考一下這個 Todo 是一個有至少20幾個 attribute 的 class。<br/>
當然可以先對原本的 state 做修改再重新賦值給整個陣列，但那樣可讀性極爛。<br/>
沒人會知道你幹嘛有事沒事把陣列重新賦值。<br/>

你可以試試看在 model class 加上一個自定義的 `copyWith` 方法**來重新產生基於自己的實體的新實體**，不過要記得有 Model 有新增 attribute 要拓展這個方法。

```dart
class Todo {
  const Todo({
    required this.name,
    required this.completed,
  });

  final String this.name;
  final bool this.completed;
  
  Todo copyWith({
    String? name,
    bool? completed,
  }) {
    return Todo(
      name: name ?? this.name,
      completed: completed ?? this.completed,
    );
  }
}
```

### 不要在 build() 中直接觸發 state 改變

如果你有監聽 Provider 做響應更新又在 build() 觸發 state 更新，可能會觸發無窮迴圈，使用函式並確保它們只在特定事件被觸發。

### 不要在 ConsumerStatefulWidget 的 initState 或 dispose 週期觸發重渲

這個 issue 非常尷尬，先用 state provider 來一段範例秀下限。

```dart
@override
void dispose() {
  ref.read(myLifeProvider.state).state = '終於得到解脫';
}

@override
Widget build(BuildContext context) {
  final myLife = ref.watch(myLifeProvider.state).state;
  // ...
}
```

很抱歉，你的修改是沒有成功被觸發的，<br/>
結果偵錯主控台會跑出一段錯誤告訴你不要在這兩個週期毛手毛腳，<br/>
因為你在理應新增實體和註銷實體時觸發渲染，<br/>
老實說這個可以直接開噴是狀態管理設計上的缺陷吧 LUL。

![know-your-place-trash](https://c.tenor.com/oHceX7JBg6AAAAAd/know-your-place-trash.gif =400x)


目前暴力有效的解法是，<br/>
1. 保險起見，dispose 時使用全局的 ProviderContainer 參照 provider，避免 dispose 時提及本該被註銷的實體的 ref 引發 memory leak。
    ```dart
    final globalProvider =PtroviderContainer();
    ```

    ```dart
    runApp(
      UncontrolledProviderScope(
        container: globalProvider,
        child: MyApp(),
      ),
    );
    ```
2. 此外，用 Future.delayed 延遲 state 操作或是讓 clear state 操作不是賦值操作(因為不會觸發重渲)。

    **Future.delayed + StateProvider 範例**
    ```dart
    @override
    void dispose() {
      Future.delayed(Duration.zero, () {
        // 你不包 Future.delayed 一樣報錯給你看，因為會觸發渲染。
        globalContainer.read(myLifeProvider.state).state = '終於得到解脫';
      });
    }
    ``` 

    **StateNotifier 範例**
    ```dart
    class MarksNotifier extends StateNotifier<Map<String, bool>> {
      MarksNotifier() : super({});
      void clear () {
        state.clear();
      }
    }

    final marksProvider =
        StateNotifierProvider<MarksNotifier, Map<String, bool>>(
      (_) => MarksNotifier(),
    );
    ``` 

    ```dart
    @override
    void dispose() {
      // 這個不包 Future.delayed 也沒差，因為根本沒觸發渲染。
      globalContainer.read(marksProvider.notifier).clear();
    }
    ``` 


我不認為上面這套解法是好實踐，但它能讓我確實完成複雜的前端的業務。


#### 總結

如果你也是正在使用 Riverpod 的玩家，<br/>
那你應該也會面臨 Provider 寫到快發瘋的處境，<br/>
遵守上面的一些開發原則可以讓你的同事跟你自己少一點痛苦。<br/>
有任何更好的建議也歡迎讓我知道。

###### tags: `Flutter` `Riverpod`
