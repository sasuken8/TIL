# StatefulWidgetの基本

## StatelessWidgetとの構文の違い

| 項目 | StatelessWidget | StatefulWidget |
|------|----------------|----------------|
| クラス構成 | 単一クラス | ウィジェットクラスとStateクラスの2つ |
| 状態保持 | 不可 (イミュータブル) | 可能 (Stateクラスで保持) |
| 再描画トリガー | 親ウィジェットの再構築時のみ | 親ウィジェット再構築時・setState()呼び出し時 |
| 実装の複雑さ | シンプル | やや複雑 |

### コード例

```dart
// StatelessWidget
class MyStatelessWidget extends StatelessWidget {
  final int value;
  
  const MyStatelessWidget({Key? key, required this.value}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Text('$value');
  }
}

// StatefulWidget
class MyStatefulWidget extends StatefulWidget {
  final int initialValue;
  
  const MyStatefulWidget({Key? key, required this.initialValue}) : super(key: key);

  @override
  State<MyStatefulWidget> createState() => _MyStatefulWidgetState();
}

class _MyStatefulWidgetState extends State<MyStatefulWidget> {
  late int _counter;

  @override
  void initState() {
    super.initState();
    _counter = widget.initialValue;
  }

  @override
  Widget build(BuildContext context) {
    return Text('$_counter');
  }
}
```

## 画面描画タイミング

| タイミング | 詳細 |
|----------|------|
| 初回画面構築時 | ウィジェットがツリーに最初に追加されたとき (initState → didChangeDependencies → build) |
| 親ウィジェット更新時 | 親が再構築され、このウィジェットも再構築される場合 (didUpdateWidget → build) |
| 状態更新時 | setState()が呼ばれて内部状態が変更されたとき (setState → build) |

## 主要メソッドとライフサイクル

| メソッド | 呼び出しタイミング | 目的 |
|--------|-----------------|------|
| コンストラクタ | ウィジェットのインスタンス化時 | 不変プロパティの設定 |
| createState() | StatefulWidgetのインスタンス作成後 | 関連するStateオブジェクトの生成 |
| initState() | Stateがツリーに挿入されたとき (一度のみ) | 初期化処理 |
| didChangeDependencies() | initState()の後、依存するInheritedWidgetが変更されたとき | コンテキスト依存の初期化 |
| build() | 初期化後、setState()の後 | UIの構築 |
| didUpdateWidget() | 親ウィジェットが再構築され、同じruntimeTypeの新ウィジェットで更新されるとき | 古いウィジェットとの比較・更新 |
| setState() | 状態変更時 | 状態更新・UI再構築のトリガー |
| dispose() | ウィジェットがツリーから削除されるとき | リソース解放 |

## ライフサイクル状態

| 状態 | 説明 | 対応メソッド |
|-----|------|------------|
| Created | ウィジェットが作成された状態 | コンストラクタ、createState() |
| Initialized | 初期化された状態 | initState(), didChangeDependencies() |
| Ready | 表示準備完了状態 | build() |
| Defunct | ツリーから削除された状態 | dispose() |

## setState()の重要性

### 基本的な役割

1. **状態の更新**: クラス内の変数（状態）を変更
2. **UI再構築のトリガー**: Flutterフレームワークに再描画の必要性を通知

### setState()の内部動作

1. 状態変数が更新される
2. 現在のStateオブジェクトが「dirty」としてマークされる
3. 次のフレームでbuild()メソッドが呼ばれ、UIが再構築される

### ベストプラクティス

| 推奨事項 | 悪い例 | 良い例 |
|---------|-------|-------|
| 最小限の範囲で使用 | setState内で重い計算を含める | 変更する状態のみを含める |
| 非同期処理との連携 | マウント解除後にsetStateを呼ぶ | mountedフラグで確認してからsetState |
| 副作用を避ける | setState内でI/O操作を行う | setState外で副作用を処理 |

```dart
// 非同期処理での適切な使用例
void _fetchData() async {
  final result = await apiService.getData();
  if (mounted) { // Widgetがまだツリー上に存在するか確認
    setState(() {
      _data = result;
    });
  }
}
```

## 実践的な例：カウンターアプリ

```dart
class CounterPage extends StatefulWidget {
  const CounterPage({Key? key}) : super(key: key);

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Counter App')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('You have pushed the button this many times:'),
            Text('$_counter', style: Theme.of(context).textTheme.headline4),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

## StatefulWidgetの使用シナリオ

| 使用ケース | 例 |
|-----------|-----|
| ユーザー入力が必要な場合 | フォーム、テキスト入力 |
| 動的データの表示 | カウンター、タイマー、プログレスバー |
| アニメーション | アニメーションコントロール |
| 非同期データロード | APIからのデータ取得と表示 |
| インタラクティブなUI | ドラッグ可能なウィジェット、ゲーム |

## 注意点とパフォーマンス考慮事項

- 不必要なStatefulWidgetの使用を避け、可能な限りStatelessWidgetを使用する
- 大きなウィジェットツリーでsetState()を呼ぶと、全体の再構築コストが高くなる
- 複雑なアプリケーションでは、Riverpodなどの外部状態管理ソリューションを検討する
- setState()内での重い処理や副作用を避ける