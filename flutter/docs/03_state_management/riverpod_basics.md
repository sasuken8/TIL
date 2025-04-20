# Riverpodによる状態管理入門

## 1. Riverpodの基本概念

Riverpodは、Flutter向けの強力な状態管理ソリューションで、Providerの作者によって開発された次世代の状態管理ライブラリだ。（Riverpod 公式サイト：https://riverpod.dev/ja/）

### Riverpodの特徴

| 特徴 | 説明 |
|------|------|
| 型安全性 | 完全な型安全性による開発時のエラー検出 |
| グローバルアクセス | ウィジェットツリーに依存しない状態へのアクセス |
| コンパイル時安全性 | コンパイル時の安全性チェックによる早期エラー検出 |
| 自動破棄 | プロバイダーの自動破棄によるメモリリーク防止 |
| テスト容易性 | モック化や依存性の注入が容易 |

## 2. 依存関係とパッケージの役割

### dependencies

| パッケージ | バージョン | 役割 |
|------------|------------|------|
| flutter_riverpod | ^2.4.0 | Riverpodのメインパッケージ。`ProviderScope`、`ConsumerWidget`などを提供 |
| riverpod_annotation | ^2.1.0 | コード生成のためのアノテーションを提供 |

### dev_dependencies

| パッケージ | バージョン | 役割 |
|------------|------------|------|
| build_runner | ^2.4.0 | Dartのコード生成エンジン |
| riverpod_generator | ^2.2.0 | Riverpodのコード生成ツール |
| riverpod_analyzer_units | ^1.3.0 | Riverpodコードの静的解析とリント |

## 3. セットアップ手順

### pubspec.yamlの設定

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.4.0
  riverpod_annotation: ^2.1.0

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^2.2.0
  riverpod_analyzer_units: ^1.3.0
```

### アプリのルート設定

```dart
void main() {
  runApp(
    // ProviderScopeでアプリ全体をラップ
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

## 4. プロバイダーの種類

| プロバイダー | 用途 | 特徴 |
|--------------|------|------|
| Provider | 変更されない値/オブジェクト | 最も基本的なプロバイダー |
| StateProvider | 単純な状態 | UIからの直接的な状態変更が可能 |
| NotifierProvider | 複雑な状態管理 | ロジックと状態を分離 |
| FutureProvider | 非同期データ | Future型の値を扱う |
| StreamProvider | ストリームデータ | Stream型の値を扱う |

## 5. コード生成

### コード生成コマンド

| コマンド | 用途 |
|---------|------|
| `flutter pub run build_runner build` | 1回限りのビルドを実行 |
| `flutter pub run build_runner watch` | 継続的に監視し変更時に再生成 |
| `flutter pub run build_runner build --delete-conflicting-outputs` | 競合ファイルを削除して再生成 |

### 生成されるファイル

| ファイル拡張子 | 内容 |
|----------------|------|
| `.g.dart` | Riverpodジェネレーターによって生成されたファイル |

## 6. VueとRiverpodの比較

| Flutter/Dart | Vue.js | 機能 |
|--------------|--------|------|
| Riverpod | Pinia | 状態管理 |
| build_runner | Vue CLI | ビルドツール |
| ConsumerWidget | Composition API | 状態へのアクセス |

## 7. サンプルコード

### 基本的なプロバイダー

```dart
// カウンター状態のプロバイダー (コード生成なし)
final counterProvider = StateProvider<int>((ref) {
  return 0; // 初期値
});

// カウンター状態のプロバイダー (コード生成あり)
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    return 0; // 初期状態
  }
  
  void increment() {
    state++;
  }
}
```

### ConsumerWidgetの使用例

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // プロバイダーから状態を読み取る
    final counter = ref.watch(counterProvider);
    
    return Scaffold(
      appBar: AppBar(title: Text('Riverpod Demo')),
      body: Center(
        child: Text(
          'カウント: $counter',
          style: TextStyle(fontSize: 24),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider.notifier).state++,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

## 8. refオブジェクトの使用方法

| メソッド | 説明 | 使用例 |
|---------|------|--------|
| `ref.watch(provider)` | プロバイダーの値を監視し、変更時に再ビルド | `final count = ref.watch(counterProvider);` |
| `ref.read(provider)` | プロバイダーの値を一度だけ読み取る | `ref.read(counterProvider.notifier).increment();` |
| `ref.listen(provider, callback)` | プロバイダーの値の変更を監視し、コールバックを実行 | `ref.listen(counterProvider, (previous, next) { print("$previous → $next"); });` |

## 9. プロジェクト構成の例

| ディレクトリ/ファイル | 内容 |
|----------------------|------|
| `lib/providers/` | プロバイダー定義ファイル |
| `lib/screens/` | 画面ウィジェット |
| `lib/widgets/` | 再利用可能なウィジェット |
| `lib/repositories/` | データアクセス層 |
| `lib/services/` | ビジネスロジック |

# Freezed - イミュータブルなデータモデリング（補足）

Freezedは、Riverpodとは直接関係はないが、多くのFlutterプロジェクトでRiverpodと併用されるイミュータブルなデータクラス生成ライブラリだ。

## 1. Freezedの概要

| 特徴 | 説明 |
|------|------|
| イミュータブル | 状態の予期しない変更を防止 |
| `copyWith` | オブジェクトの一部のみを変更するメソッド |
| イコール実装 | 正確な値の比較が可能 |
| パターンマッチング | ユニオン型のような使い方が可能 |
| JSON変換 | JSONとの相互変換が自動生成される |

## 2. Freezedの依存関係

```yaml
dependencies:
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1  # JSONシリアライゼーション用

dev_dependencies:
  freezed: ^2.4.5
  json_serializable: ^6.7.1  # JSONシリアライゼーション用
```

## 3. Freezedの使用例

```dart
// todo_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import 'package:flutter/foundation.dart';

part 'todo_model.freezed.dart';
part 'todo_model.g.dart';

@freezed
class Todo with _$Todo {
  const factory Todo({
    required String id,
    required String title,
    @Default(false) bool completed,
  }) = _Todo;
  
  factory Todo.fromJson(Map<String, dynamic> json) => _$TodoFromJson(json);
}
```

## 4. FreezedとRiverpodの連携例

```dart
// todo_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'todo_model.dart';

part 'todo_provider.g.dart';

@riverpod
class TodoList extends _$TodoList {
  @override
  List<Todo> build() {
    return []; // 初期状態は空のリスト
  }
  
  void addTodo(String title) {
    final newTodo = Todo(
      id: DateTime.now().toString(),
      title: title,
    );
    state = [...state, newTodo];
  }
  
  void toggleTodo(String id) {
    state = state.map((todo) {
      if (todo.id == id) {
        return todo.copyWith(completed: !todo.completed);
      }
      return todo;
    }).toList();
  }
  
  void removeTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }
}
```

これらの機能を活用することで、型安全で保守性の高いFlutterアプリケーションを構築できる。