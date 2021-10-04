# Riverpod 簡介

網路上中文資源有點少，只好看影片整理起來當 cheat sheet 用。

### Provider 類型

#### Provider

型別簡單、值不可被修改的 provider 。

```dart
final valueProvider = Provider<int>((ref) {
  return 0;
});
```

#### State Provider

型別簡單、值可被修改的 provider 。

```dart
final counterStateProvider = StateProvider<int>((ref) {
  return 0;
});
```

#### State Notifier Provider

型別複雜、值的修改方法可以自定義的 provider 。

```dart
class Clock extends StateNotifier<DateTime> {
  Clock(): super(DateTime.now()) {
    _timer = Timer.periodic(Duration(seconds: 1), (_) {
      state = DateTime.now();
    })
  }

  late final Timer _timer;

  @override
  void dispose() {
    _timer.cancel();
    super.dispose();
  }
}

final clockProvider = StateNotifierProvider<Clock>((ref) {
  return Clock();
});
```

#### Future Provider

當你的狀態來自於非同步的資料且你需要針對不同的狀態(loading, error, data) 加以區分時，這個 provider 幫你省去了一些麻煩。

```dart
class Book {
  final String title;
  final String author;

  Record({
    required this.title,
    required this.author,
  });
}

final bookShelfProvider = FutureProvider<List<Book>>((ref) {
  return Future.value([
    Book(
      title: 'widget tree sucks',
      author: 'lilybon',
    ),
    Book(
      title: 'use google to pass google interview, brilliant!',
      author: 'google',
    ),
  ]);
});
```

#### Stream Provider

```dart
final streamProvider = StreamProvider<int>((ref) {
  return Stream.fromIterable([0, 1]);
});
```

### Provider 後綴

#### .autoDispose

未完成

#### .family

未完成

### ProviderObserver

未完成

### 範例

以下範例用到的 provider 都可以從上面 Provider 類別介紹找到，就不再複製貼上了。

#### Example1: 計數器 (StateProvider)

ConsumerWidget - 無狀態組件要用到 riverpod 的狀態管理時需繼承該類別。
ProviderListener - 適合監聽簡單的 state provider，若組件的業務邏輯過複雜則不適用。

```dart
class Counter extends ConsumerWidget {
  @override
  Widget build (BuildContext context, ScopedReader watch) {
    final counter = watch(counterStateProvider);
    return ProviderListener<StateController<int>>(
      provider: counterStateProvider,
      onChange: (context, counterState) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Value is ${counterState.state}')),
        );
      },
      child: Scaffold(
        body: Center(
          child: Text(
            'Value: ${counter.state}',
            style: Theme.of(context).textTheme.headline4,
          ),
        ),
        floatActionButton: FloatActionButton(
          onPressed: () => context.read(counterStateProvider).state++
          child: Icon(Icons.add),
        )
      ),
    );
  }
}
```

#### Example2: 計時器 (StateNotifierProvider)

```dart
class Timer extends ConsumerWidget {
  @override
  Widget build (BuildContext context, ScopedReader watch) {
    final currentTime = watch(clockProvider.state);
    final timeFormatted = DateFormat.Hms().format(currentTime);
    return Scaffold(
      body: Center(
        child: Text(
          timeFormatted,
          style: Theme.of(context).textTheme.headline4,
        ),
      ),
    );
  }
}
```

#### Example3: 書櫃 (FutureProvider)

```dart
class BookShelf extends ConsumerWidget {
  @override
  Widget build (BuildContext context, ScopedReader watch) {
    final bookShelf = watch(bookShelfProvider);
    return Scaffold(
      body: Center(
        // 可針對 future 的狀態作不同的渲染
        child: bookShelf.when(
          data: (books) => Column(
            children: books.map((book) => Row(
              children: [
                Text(book.title),
                Text(book.author),
              ],
            )).toList()
          ),
          loading: () => CircularProgressIndicator(),
          error: (e, st) => Text('Error: $e'),
        ),
      ),
    );
  }
}
```

### 測試

#### 環境設定

未完成

#### 模擬及取代依賴

未完成

### 參考

- [Riverpod](https://riverpod.dev/)
- [Flutter State Management with Riverpod: The Essential Guide](https://www.youtube.com/watch?v=J2iFYZUabVM&t=1143s)

###### tags: `Flutter` `Riverpod`
