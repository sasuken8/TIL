# Flutter 基本的な画面遷移 (Navigation Basics)

## 画面遷移の基本概念

Flutter では画面（ページ）は「Route」と呼ばれ、これらの Route を管理するのが「Navigator」である。Navigator は Route のスタックを管理し、プッシュ（追加）やポップ（削除）などの操作を提供する。Navigatorは一種のスタック構造になっており、後入れ先出し（LIFO: Last In, First Out）の原則で動作する。

## 主な画面遷移のタイプ

| 遷移タイプ | 概要 | 適した用途 | 実装方法 |
|------------|------|------------|----------|
| **プッシュ遷移** | 新しい画面を現在の画面の上に積み重ねる | 階層的なナビゲーション（商品一覧→商品詳細など） | `Navigator.push()`, `Navigator.pushNamed()` |
| **モーダル遷移** | 現在の画面の上に部分的または全面的に新しい画面を表示 | 現在のコンテキストを維持したまま追加情報や操作を提供（設定、確認ダイアログなど） | `fullscreenDialog: true`, `showModalBottomSheet()`, `showDialog()` |
| **ポップ遷移** | 現在の画面を閉じて前の画面に戻る | ユーザーが操作を完了・キャンセルした時など | `Navigator.pop()` |

## プッシュ遷移の実装

```dart
// 基本的なプッシュ遷移
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondScreen()),
);

// 名前付きルートを使用したプッシュ遷移
Navigator.pushNamed(context, '/second');
```

名前付きルートを使用する場合のルート定義：

```dart
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => HomeScreen(),
    '/second': (context) => SecondScreen(),
    '/third': (context) => ThirdScreen(),
  },
);
```

## モーダル遷移の種類と実装

| モーダルタイプ | 特徴 | 実装方法 |
|----------------|------|----------|
| **フルスクリーンモーダル** | 画面全体を覆うモーダル<br>下から上へスライドするアニメーション<br>閉じるボタン（✕）表示 | `MaterialPageRoute(builder: ..., fullscreenDialog: true)` |
| **ボトムシートモーダル** | 画面下部から表示されるモーダル<br>部分的に元の画面が見える | `showModalBottomSheet()` |
| **ダイアログモーダル** | 画面中央に表示される小さなモーダル<br>背景がぼかされる | `showDialog()` |

### fullscreenDialog: true の具体的な違い

| 項目 | 通常の遷移 | fullscreenDialog: true |
|------|------------|------------------------|
| **アニメーション** | 右から左へスライド（iOS）<br>下から上へフェード（Android） | 下から上へスライド（iOS/Android共通） |
| **AppBar の表示** | 戻るボタン（←） | 閉じるボタン（✕） |
| **適した用途** | 一般的な画面遷移 | フォーム入力、設定画面、フィルター設定など |

```dart
// フルスクリーンモーダル実装例
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => SettingsScreen(),
    fullscreenDialog: true,
  ),
);

// ボトムシートモーダル実装例
showModalBottomSheet(
  context: context,
  builder: (context) => Container(
    height: 300,
    child: Center(child: Text('モーダルの内容')),
  ),
);

// ダイアログモーダル実装例
showDialog(
  context: context,
  builder: (context) => AlertDialog(
    title: Text('タイトル'),
    content: Text('内容'),
    actions: [
      TextButton(
        onPressed: () => Navigator.pop(context),
        child: Text('閉じる'),
      ),
    ],
  ),
);
```

## ポップ遷移（Navigator.pop）の機能

| 機能 | コード例 | 説明 |
|------|----------|------|
| 基本的な戻る操作 | `Navigator.pop(context)` | 単純に前の画面に戻る |
| データを戻す | `Navigator.pop(context, '結果')` | 前の画面にデータを返す |
| 戻る操作の制御 | `WillPopScope` と組み合わせる | 戻るボタンの挙動をカスタマイズ |

```dart
// 戻る操作の制御例
WillPopScope(
  onWillPop: () async {
    // 戻る前に確認ダイアログを表示
    final shouldPop = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('確認'),
        content: Text('このページを離れますか？'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: Text('キャンセル'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: Text('OK'),
          ),
        ],
      ),
    );
    return shouldPop ?? false;
  },
  child: Scaffold(/* ... */),
);
```

## 画面遷移時のデータ受け渡し

| 方法 | 用途 | 実装方法 |
|------|------|----------|
| **コンストラクタ経由** | 遷移先にデータを渡す | 遷移先クラスのコンストラクタに引数を渡す |
| **名前付きルートのarguments** | 遷移先にデータを渡す<br>定義済みの名前付きルート使用時 | `Navigator.pushNamed(context, '/detail', arguments: data)` |
| **戻り値（結果）** | 遷移先から戻ってきたときに結果を受け取る | `final result = await Navigator.push()` と `Navigator.pop(context, result)` |

### コンストラクタ経由のデータ渡し

```dart
// 遷移元
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(itemId: 123, title: '商品詳細'),
  ),
);

// 遷移先
class DetailScreen extends StatelessWidget {
  final int itemId;
  final String title;
  
  DetailScreen({required this.itemId, required this.title});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(title)),
      body: Center(child: Text('Item ID: $itemId')),
    );
  }
}
```

### 名前付きルートでのデータ渡し

```dart
// 遷移元
Navigator.pushNamed(
  context,
  '/detail',
  arguments: {'itemId': 123, 'title': '商品詳細'},
);

// 遷移先
class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as Map<String, dynamic>;
    final itemId = args['itemId'];
    final title = args['title'];
    
    return Scaffold(
      appBar: AppBar(title: Text(title)),
      body: Center(child: Text('Item ID: $itemId')),
    );
  }
}
```

### 戻り値（結果）の受け取り

```dart
// 遷移元（結果を待機）
void _navigateAndGetResult() async {
  final result = await Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => SelectionScreen()),
  );
  
  // 結果を処理
  if (result != null) {
    setState(() {
      selectedItem = result;
    });
  }
}

// 遷移先（結果を返す）
class SelectionScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('選択画面')),
      body: ListView.builder(
        itemCount: items.length,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text(items[index]),
            onTap: () {
              // 選択した項目を返してポップ
              Navigator.pop(context, items[index]);
            },
          );
        },
      ),
    );
  }
}
```

## 遷移時のアニメーションカスタマイズ

カスタムアニメーションの代表的なタイプ:

| アニメーションタイプ | 効果 | 実装クラス |
|----------------------|------|------------|
| スライド | 指定方向からスライドイン | `SlideTransition` |
| フェード | 透明度の変化 | `FadeTransition` |
| スケール | サイズの拡大・縮小 | `ScaleTransition` |
| 回転 | 要素の回転 | `RotationTransition` |

```dart
// カスタムアニメーション実装例（右から左へのスライド）
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => SecondScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      var begin = Offset(1.0, 0.0);
      var end = Offset.zero;
      var curve = Curves.ease;
      var tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));
      return SlideTransition(
        position: animation.drive(tween),
        child: child,
      );
    },
  ),
);
```

## ナビゲーションのベストプラクティス

| ベストプラクティス | 説明 |
|-------------------|------|
| **一貫性の維持** | アプリ全体で一貫したナビゲーションパターンを使用する |
| **適切な遷移タイプの選択** | コンテンツやユーザーフローに応じた適切な遷移方法を選択する |
| **ユーザーの期待への対応** | プラットフォームの標準的な動作に沿った実装をする |
| **適切なデータ管理** | 画面間でのデータ受け渡しを効率的に行う |

## 実践的なサンプルコード

複数の遷移方法を組み合わせた例:

```dart
class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  String selectedItem = '未選択';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('ホーム画面')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('選択されたアイテム: $selectedItem'),
            SizedBox(height: 20),
            
            // 標準的なプッシュ遷移
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => DetailScreen(itemName: selectedItem)),
                );
              },
              child: Text('詳細画面へ'),
            ),
            
            // 結果を返すプッシュ遷移
            ElevatedButton(
              onPressed: () async {
                final result = await Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => SelectionScreen()),
                );
                if (result != null) {
                  setState(() {
                    selectedItem = result;
                  });
                }
              },
              child: Text('アイテムを選択'),
            ),
            
            // モーダル遷移 (fullscreenDialog: true)
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => SettingsScreen(),
                    fullscreenDialog: true,
                  ),
                );
              },
              child: Text('設定画面を開く'),
            ),
            
            // ボトムシートモーダル
            ElevatedButton(
              onPressed: () {
                showModalBottomSheet(
                  context: context,
                  builder: (context) => Container(
                    height: 200,
                    padding: EdgeInsets.all(16),
                    child: Center(child: Text('ボトムシートモーダル')),
                  ),
                );
              },
              child: Text('ボトムシートを表示'),
            ),
          ],
        ),
      ),
    );
  }
}
```