# Flutterウィジェットについて2

## ウィジェットの基本構造

Flutterではすべての画面要素がウィジェットで構成されている。各ウィジェットは特定の引数を受け取るように設計されており、それらを組み合わせてUIを構築する。

### ウィジェットと引数

| 概念 | 説明 |
|------|------|
| ウィジェット | Flutterの画面要素。すべてがウィジェットで構成される |
| 引数（パラメータ） | ウィジェットを設定するための値 |
| 名前付き引数 | Dartでの引数指定方法。`name: value`の形式で指定する |
| コンストラクタ | ウィジェットのインスタンスを作成するための特別なメソッド |

### 例: Scaffoldウィジェットの使用

```dart
Scaffold(
  appBar: AppBar(title: Text('タイトル')),
  body: Center(child: Text('本文')),
  floatingActionButton: FloatingActionButton(onPressed: () {}),
)
```

この例では:
- `Scaffold`はウィジェット
- `appBar`, `body`, `floatingActionButton`は名前付き引数
- 各引数に渡される`AppBar()`, `Center()`, `FloatingActionButton()`も別のウィジェット

## ウィジェットの定義

各ウィジェットがどのような引数を受け取れるかは、ウィジェットのクラス定義で指定されている。

### コンストラクタの例 (Scaffold)

```
Scaffold({Key? key, PreferredSizeWidget? appBar, Widget? body, Widget? floatingActionButton, ... })
```

この定義は:
- `Scaffold`ウィジェットが受け取れる引数を列挙している
- 各引数の型も指定されている（`appBar`は`PreferredSizeWidget?`型など）

## オブジェクト指向の概念

Flutter/Dartはオブジェクト指向言語であり、以下の概念が重要になる。

| 概念 | 説明 | Flutter/Dartでの例 |
|------|------|-------------------|
| クラス | オブジェクトの設計図 | `Scaffold`, `AppBar`, `Text` |
| インスタンス | クラスから作成された実体 | `Scaffold()` |
| インターフェース | クラスが実装すべきメソッド・属性の契約 | `Widget`, `PreferredSizeWidget` |
| 継承・実装 | 上位クラスの特性を受け継ぐ仕組み | `implements Widget` |

### インターフェースの例

```dart
abstract class PreferredSizeWidget implements Widget {
  Size get preferredSize;
}
```

これは:
- `PreferredSizeWidget`というインターフェースを定義
- これは`Widget`インターフェースも実装する
- `preferredSize`というプロパティを持つことを要求

### ウィジェットの階層関係

```
Object (すべてのDartオブジェクトの基本)
  └── Widget (Flutterの基本UI要素)
      ├── StatelessWidget (状態を持たないウィジェット)
      │   └── Container, Text など
      ├── StatefulWidget (状態を持つウィジェット)
      │   └── TextField, Checkbox など
      └── PreferredSizeWidget (サイズが予測できるウィジェット)
          └── AppBar, TabBar など
```

## Dartの特徴

- コンストラクタ呼び出し時に`new`キーワードは省略可能（推奨）
- 名前付き引数を多用する設計
- 型安全性が高い

## 重要な基本ウィジェット

| カテゴリ | 代表的なウィジェット |
|---------|-------------------|
| レイアウト | Container, Row, Column, Stack, Expanded, Padding |
| 表示 | Text, Image, Icon |
| 入力 | TextField, Checkbox, Radio, Switch |
| コンテナ | Scaffold, AppBar, BottomNavigationBar |
| アニメーション | AnimatedContainer, Hero |

## まとめ

- Flutterはウィジェットを組み合わせるパズルのような開発スタイル
- 各ウィジェットの引数と役割を理解することが重要
- 既存ウィジェットを組み合わせてカスタムウィジェットを作ることで複雑なUIも実現可能
- オブジェクト指向の概念を理解するとFlutterの設計思想が見えてくる