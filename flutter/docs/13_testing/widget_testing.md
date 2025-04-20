# Flutter ウィジェットテスト メモ

## 0. テスト分類の全体像と対応関係

テストは「開発工程の視点」と「Flutter技術の視点」の2つの軸で理解すると整理しやすい。

| 開発工程の視点 | Flutter技術の視点 | 実行環境 | 対象範囲 | 実行者 |
|--------------|-----------------|---------|---------|--------|
| **単体テスト** | ユニットテスト | IDE/CIサーバー | 関数・クラス | 開発者 |
|              | ウィジェットテスト | IDE/CIサーバー | 単一UI要素 | 開発者 |
| **結合テスト** | インテグレーションテスト | 実機/エミュレータ | 複数機能連携 | 開発者/QA |
|              | 手動テスト | 実機 | 特定シナリオ | 開発者/QA |
| **システムテスト** | E2Eテスト | 実機/エミュレータ | 全体フロー | QA/自動化 |
|                 | QAテスト | 実機 | 要件適合性 | QA |

### テスト手法の使い分け

| テストタイプ | 目的 | 速度 | コスト | 依存関係 |
|------------|------|------|--------|---------|
| ユニットテスト | ロジック検証 | 最速 | 最低 | モック化 |
| ウィジェットテスト | UI動作検証 | 速い | 低い | 部分的モック化 |
| インテグレーションテスト | 機能連携検証 | 遅い | 中程度 | 実際の依存関係 |
| E2Eテスト | 全体フロー検証 | 最遅 | 最高 | 実際の依存関係 |

## 1. ソフトウェア開発工程におけるウィジェットテストの位置づけ

| 開発工程 | 説明 |
|---------|------|
| 1. システム要件定義 | システム全体の要件を定義 |
| 2. 方式設計 | システム構成・アーキテクチャを設計 |
| 3. 詳細設計 | コンポーネントレベルの詳細を設計 |
| 4. ソフトウェア構築 | コードの実装 |
| 5. 単体テスト（ユニットテスト） | 個々の関数・クラスの動作検証 |
| **6. ウィジェットテスト** | **単一または少数のUI要素の動作検証** |
| 7. 結合テスト | コンポーネント間の連携を検証 |
| 8. システム適格性確認テスト | システム全体の要件適合性を検証 |

## 2. Flutterにおける自動テストの比較

| 特性 | ユニットテスト | ウィジェットテスト | 統合テスト |
|------|--------------|----------------|-----------|
| 対象範囲 | 関数・メソッド・クラス | 単一～少数ウィジェット | アプリ全体・複数画面 |
| 実行環境 | Dartランタイム | Flutterテスト環境 | 実機/エミュレータ |
| 実行速度 | 高速 | 中速 | 低速 |
| フォーカス | ロジック検証 | UI表示・相互作用 | エンドツーエンドの流れ |
| 外部依存 | 分離（モック化） | 分離（モック化） | 実際の依存関係 |
| 記述方法 | `test()` | `testWidgets()` | `testWidgets()` + integration_test |

## 3. ウィジェットテストの基本構造

```dart
void main() {
  testWidgets('カウンターのインクリメントテスト', (WidgetTester tester) async {
    // ARRANGE: ウィジェットの構築と表示
    await tester.pumpWidget(MaterialApp(home: MyCounter()));

    // ASSERT: 初期状態の検証
    expect(find.text('0'), findsOneWidget);
    expect(find.text('1'), findsNothing);

    // ACT: アクションの実行
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump(); // 状態更新を反映

    // ASSERT: 更新後の状態検証
    expect(find.text('0'), findsNothing);
    expect(find.text('1'), findsOneWidget);
  });
}
```

## 4. テスト環境のセットアップ

| 用途 | コード例 |
|------|---------|
| 基本構成 | `await tester.pumpWidget(MaterialApp(home: MyWidget()));` |
| テーマ指定 | `await tester.pumpWidget(MaterialApp(theme: ThemeData.light(), home: MyWidget()));` |
| 多言語環境 | `await tester.pumpWidget(MaterialApp(localizationsDelegates: [...], home: MyWidget()));` |
| 状態管理接続 | `await tester.pumpWidget(ProviderScope(child: MaterialApp(home: MyWidget())));` |
| スクリーンサイズ設定 | `tester.binding.window.physicalSizeTestValue = Size(1080, 1920);` |
| メディアクエリ | `MediaQuery(data: MediaQueryData(), child: MyWidget())` |

## 5. tester.pump() の重要性と使用タイミング

| 使用タイミング | 説明 | コード例 |
|--------------|------|---------|
| ユーザーインタラクション後 | タップ・スワイプ・入力後にUIを更新 | `await tester.tap(find.byType(Button)); await tester.pump();` |
| 状態変更の反映時 | setState後やProvider更新後 | `await tester.tap(find.byText('更新')); await tester.pump();` |
| 非同期処理完了後 | Future完了後のUI更新 | `await tester.pump(Duration(seconds: 1));` |
| アニメーション中 | 特定時間後のフレーム描画 | `await tester.pump(Duration(milliseconds: 16));` |
| アニメーション完了まで | すべてのアニメーション完了を待機 | `await tester.pumpAndSettle();` |
| 画面遷移後 | Navigator操作後の新画面表示 | `await tester.tap(find.byType(Button)); await tester.pumpAndSettle();` |

## 6. Finder によるウィジェット検索方法

| 検索方法 | 使用例 | 説明 |
|---------|-------|------|
| テキスト | `find.text('ログイン')` | 表示テキストで検索 |
| タイプ | `find.byType(ElevatedButton)` | ウィジェットの型で検索 |
| キー | `find.byKey(Key('submit_button'))` | キーで検索 |
| アイコン | `find.byIcon(Icons.add)` | アイコンで検索 |
| ウィジェット | `find.byWidget(myWidgetInstance)` | インスタンスで検索 |
| セマンティクス | `find.bySemanticsLabel('ログインボタン')` | アクセシビリティラベルで検索 |
| ツールチップ | `find.byTooltip('追加')` | ツールチップで検索 |
| 子孫 | `find.descendant(of: find.byType(Card), matching: find.text('タイトル'))` | 親ウィジェット内の子検索 |
| 先祖 | `find.ancestor(of: find.text('項目'), matching: find.byType(ListTile))` | 指定子を持つ先祖を検索 |
| 複数要素指定 | `find.byType(ListTile).at(2)` | 複数一致時にインデックス指定 |

## 7. マッチャーによる検証

| マッチャー | 使用例 | 説明 |
|-----------|-------|------|
| findsOneWidget | `expect(find.text('タイトル'), findsOneWidget);` | 要素が1つだけ存在 |
| findsNothing | `expect(find.text('エラー'), findsNothing);` | 要素が存在しない |
| findsWidgets | `expect(find.byType(Icon), findsWidgets);` | 複数の要素が存在（1つ以上） |
| findsNWidgets | `expect(find.byType(ListTile), findsNWidgets(3));` | 指定数の要素が存在 |
| findsAtLeastNWidgets | `expect(find.byType(Chip), findsAtLeastNWidgets(2));` | 指定数以上の要素が存在 |
| findsAnyWidget | `expect(find.text('非表示項目'), findsAnyWidget);` | オフスクリーンも含めて検索 |

## 8. ユーザー操作のシミュレーション

| 操作 | コード例 | 説明 |
|-----|---------|------|
| タップ | `await tester.tap(find.byType(FloatingActionButton));` | 要素をタップ |
| テキスト入力 | `await tester.enterText(find.byType(TextField), 'ユーザー名');` | 入力欄にテキスト入力 |
| ドラッグ | `await tester.drag(find.byType(Slider), Offset(50.0, 0.0));` | 要素をドラッグ |
| スクロール | `await tester.dragFrom(Offset(200, 300), Offset(0, -500));` | 座標から指定方向にスクロール |
| スクロールして要素表示 | `await tester.scrollUntilVisible(find.text('項目10'), 500.0);` | 要素が見えるまでスクロール |
| 長押し | `await tester.longPress(find.byType(ListTile).at(2));` | 要素を長押し |
| フォーム送信 | `await tester.tap(find.text('送信')); await tester.pumpAndSettle();` | 送信ボタンをタップしアニメーション完了まで待機 |
| キー入力 | `await tester.sendKeyEvent(LogicalKeyboardKey.enter);` | キーボードイベント送信 |

## 9. ウィジェットの詳細検証

| 検証対象 | コード例 | 説明 |
|---------|---------|------|
| プロパティ検証 | `expect(tester.widget<Text>(find.byKey(Key('title'))).data, 'ホーム');` | ウィジェットのプロパティ値を検証 |
| スタイル検証 | `expect(tester.widget<Text>(find.byKey(Key('title'))).style?.color, Colors.blue);` | スタイル属性の検証 |
| サイズ検証 | `final size = tester.getSize(find.byType(Container)); expect(size.width, 100.0);` | 要素サイズの検証 |
| 位置検証 | `final topLeft = tester.getTopLeft(find.byType(AppBar)); expect(topLeft.dy, 0.0);` | 要素位置の検証 |
| 表示状態検証 | `expect(tester.widget<Opacity>(find.byType(Opacity)).opacity, 0.5);` | 透明度などの視覚効果検証 |
| ツリー構造検証 | `expect(find.descendant(of: find.byType(Card), matching: find.text('タイトル')), findsOneWidget);` | 親子関係の検証 |

## 10. モックとフェイク

### APIサービスのモック

```dart
class MockApiService extends Mock implements ApiService {}

testWidgets('APIデータ表示テスト', (tester) async {
  final mockService = MockApiService();
  when(mockService.fetchData()).thenAnswer((_) async => {'name': 'テスト'});
  
  await tester.pumpWidget(MaterialApp(
    home: MyWidget(apiService: mockService),
  ));
  
  await tester.pump(Duration(seconds: 1));
  expect(find.text('テスト'), findsOneWidget);
});
```

### ナビゲーションのモック

```dart
class MockNavigatorObserver extends Mock implements NavigatorObserver {}

testWidgets('画面遷移テスト', (tester) async {
  final mockObserver = MockNavigatorObserver();
  
  await tester.pumpWidget(MaterialApp(
    home: HomeScreen(),
    navigatorObservers: [mockObserver],
  ));
  
  await tester.tap(find.text('詳細'));
  await tester.pumpAndSettle();
  
  verify(mockObserver.didPush(any, any));
});
```

### Providerのモック

```dart
testWidgets('プロバイダー状態テスト', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        userProvider.overrideWithValue(User(name: 'テスト太郎')),
      ],
      child: MaterialApp(home: ProfileScreen()),
    ),
  );
  
  expect(find.text('テスト太郎'), findsOneWidget);
});
```

## 11. 非同期処理のテスト

| シナリオ | テスト方法 |
|---------|----------|
| データ読み込み | ローディング→データ表示の遷移 |
| エラーハンドリング | エラー表示のテスト |

### データ読み込みテスト

```dart
testWidgets('データ読み込みテスト', (tester) async {
  await tester.pumpWidget(MaterialApp(home: DataLoadingScreen()));
  
  // ローディング表示
  expect(find.byType(CircularProgressIndicator), findsOneWidget);
  
  // データ読み込み完了まで待機
  await tester.pump(Duration(seconds: 1));
  
  // データ表示
  expect(find.byType(CircularProgressIndicator), findsNothing);
  expect(find.byType(ListView), findsOneWidget);
});
```

### エラー表示テスト

```dart
testWidgets('エラー表示テスト', (tester) async {
  final mockService = MockApiService();
  when(mockService.fetchData()).thenThrow(Exception('ネットワークエラー'));
  
  await tester.pumpWidget(MaterialApp(
    home: DataScreen(apiService: mockService),
  ));
  
  // API呼び出し後の状態更新
  await tester.pump(Duration(seconds: 1));
  
  // エラーメッセージ表示
  expect(find.text('エラーが発生しました'), findsOneWidget);
  expect(find.text('再試行'), findsOneWidget);
});
```

## 12. ゴールデンテスト（スクリーンショットテスト）

| 機能 | 説明 |
|-----|------|
| 基本的なゴールデンテスト | UIの外観が期待通りか検証 |
| デバイスサイズ指定 | 異なる画面サイズでの表示検証 |

### プロフィールカードの外観テスト

```dart
testWidgets('プロフィールカードの外観', (tester) async {
  await tester.pumpWidget(MaterialApp(
    home: Scaffold(body: ProfileCard(username: 'テストユーザー')),
  ));
  
  await expectLater(
    find.byType(ProfileCard),
    matchesGoldenFile('profile_card.png'),
  );
});
```

### レスポンシブデザインテスト

```dart
testWidgets('レスポンシブデザインテスト', (tester) async {
  // iPhoneサイズ
  tester.binding.window.physicalSizeTestValue = Size(750, 1334);
  
  await tester.pumpWidget(MaterialApp(home: ResponsiveLayout()));
  await expectLater(
    find.byType(ResponsiveLayout),
    matchesGoldenFile('responsive_iphone.png'),
  );
  
  // iPadサイズ
  tester.binding.window.physicalSizeTestValue = Size(1536, 2048);
  await tester.pumpWidget(MaterialApp(home: ResponsiveLayout()));
  await expectLater(
    find.byType(ResponsiveLayout),
    matchesGoldenFile('responsive_ipad.png'),
  );
});
```

## 13. ナビゲーションとルーティングのテスト

| シナリオ | テスト方法 |
|---------|----------|
| 画面遷移検証 | 遷移前後の画面内容の確認 |
| 引数付き遷移 | 引数が正しく渡されるか検証 |

### 詳細画面遷移テスト

```dart
testWidgets('詳細画面遷移テスト', (tester) async {
  await tester.pumpWidget(MaterialApp(
    routes: {
      '/': (context) => HomeScreen(),
      '/details': (context) => DetailsScreen(),
    },
  ));
  
  // 初期画面確認
  expect(find.text('ホーム画面'), findsOneWidget);
  
  // 詳細ボタンタップ
  await tester.tap(find.text('詳細を見る'));
  await tester.pumpAndSettle();
  
  // 遷移先確認
  expect(find.text('詳細画面'), findsOneWidget);
});
```

### 引数付き画面遷移テスト

```dart
testWidgets('引数付き画面遷移テスト', (tester) async {
  await tester.pumpWidget(MaterialApp(
    home: HomeScreen(),
    onGenerateRoute: (settings) {
      if (settings.name == '/details') {
        final args = settings.arguments as Map;
        return MaterialPageRoute(
          builder: (_) => DetailsScreen(itemId: args['id']),
        );
      }
      return null;
    },
  ));
  
  await tester.tap(find.text('商品1の詳細'));
  await tester.pumpAndSettle();
  
  expect(find.text('商品ID: 1の詳細'), findsOneWidget);
});
```

## 14. フォームとバリデーションのテスト

| シナリオ | テスト方法 |
|---------|----------|
| 入力検証 | エラーメッセージ表示の検証 |
| フォーム送信 | 送信処理の検証 |

### 必須入力検証テスト

```dart
testWidgets('必須入力検証', (tester) async {
  await tester.pumpWidget(MaterialApp(home: LoginForm()));
  
  // 空入力で送信
  await tester.tap(find.text('ログイン'));
  await tester.pump();
  
  // エラーメッセージ検証
  expect(find.text('メールアドレスを入力してください'), findsOneWidget);
  expect(find.text('パスワードを入力してください'), findsOneWidget);
  
  // 正しい入力
  await tester.enterText(find.byKey(Key('email')), 'test@example.com');
  await tester.enterText(find.byKey(Key('password')), 'password123');
  await tester.pump();
  
  // 再送信
  await tester.tap(find.text('ログイン'));
  await tester.pump();
  
  // エラーメッセージ消失
  expect(find.text('メールアドレスを入力してください'), findsNothing);
  expect(find.text('パスワードを入力してください'), findsNothing);
});
```

### フォーム送信テスト

```dart
testWidgets('フォーム送信テスト', (tester) async {
  bool formSubmitted = false;
  String submittedEmail = '';
  
  await tester.pumpWidget(MaterialApp(
    home: LoginForm(
      onSubmit: (email, password) {
        formSubmitted = true;
        submittedEmail = email;
      },
    ),
  ));
  
  await tester.enterText(find.byKey(Key('email')), 'user@example.com');
  await tester.enterText(find.byKey(Key('password')), 'securepass');
  await tester.tap(find.text('ログイン'));
  await tester.pump();
  
  expect(formSubmitted, true);
  expect(submittedEmail, 'user@example.com');
});
```

## 15. ウィジェットテストのベストプラクティス

| カテゴリ | ベストプラクティス |
|---------|------------------|
| テストの構造 | ・一つのテストで一つの動作を検証<br>・AAAパターン（Arrange-Act-Assert）に従う<br>・テスト名は何をテストしているか明確に |
| パフォーマンス | ・不要なウィジェットビルドを避ける<br>・最小限のウィジェットツリーをテスト<br>・pumpAndSettle()の使いすぎに注意 |
| モック | ・外部依存はモック化して分離<br>・テストデータはハードコードせず変数化<br>・複雑なモックにはlibrary/partパターンを使用 |
| メンテナンス | ・テスト専用のWidgetKeyを設定<br>・フォーマットに敏感なテストは避ける<br>・テストヘルパー関数で重複コードを削減 |
| 依存性注入 | ・テスト可能性を考慮した設計<br>・コンストラクタインジェクションを活用<br>・Providerなどで依存関係を抽象化 |

## 16. テストの実行方法

| コマンド | 説明 |
|---------|------|
| `flutter test` | すべてのテストを実行 |
| `flutter test test/widget_test.dart` | 特定のテストファイルを実行 |
| `flutter test --tags=widgets` | タグ付きテストを実行 |
| `flutter test --platform=chrome` | 特定プラットフォーム向けテスト実行 |
| `flutter test --coverage` | カバレッジレポート生成 |
| `flutter test --update-goldens` | ゴールデンファイルを更新 |

## 17. ウィジェットテスト設定（analysis_options.yaml）

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # テスト関連のLintルール
    - prefer_const_constructors  # 定数コンストラクタを推奨
    - avoid_print  # プリント文を避ける
    - prefer_final_locals  # ローカル変数をfinalで宣言することを推奨

analyzer:
  errors:
    # テスト内での実装に関する警告を緩和
    unused_import: warning  # 未使用のインポートを警告
    missing_return: error  # 戻り値の欠落をエラー
```

## 18. テスト実装例: カウンターアプリ

```dart
// カウンターウィジェット
class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('カウンター')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('カウント:'),
            Text(
              '$_counter',
              key: Key('counter_value'),
              style: Theme.of(context).textTheme.headline4,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        key: Key('increment_button'),
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}

// ウィジェットテスト
void main() {
  group('CounterWidget', () {
    testWidgets('初期表示の検証', (WidgetTester tester) async {
      await tester.pumpWidget(MaterialApp(home: CounterWidget()));
      
      expect(find.text('カウント:'), findsOneWidget);
      expect(find.text('0'), findsOneWidget);
      expect(find.byKey(Key('increment_button')), findsOneWidget);
    });
    
    testWidgets('インクリメントボタンタップで値が増加', (WidgetTester tester) async {
      await tester.pumpWidget(MaterialApp(home: CounterWidget()));
      
      // 初期値確認
      expect(find.text('0'), findsOneWidget);
      expect(find.text('1'), findsNothing);
      
      // ボタンタップ
      await tester.tap(find.byKey(Key('increment_button')));
      await tester.pump();
      
      // 増加後の値確認
      expect(find.text('0'), findsNothing);
      expect(find.text('1'), findsOneWidget);
      
      // 再度タップ
      await tester.tap(find.byKey(Key('increment_button')));
      await tester.pump();
      
      // 再度増加確認
      expect(find.text('1'), findsNothing);
      expect(find.text('2'), findsOneWidget);
    });
  });
}
```

## 19. テスト実装例: ログインフォーム

```dart
// ログインフォームウィジェット
class LoginForm extends StatefulWidget {
  final Function(String email, String password)? onSubmit;
  
  LoginForm({this.onSubmit});
  
  @override
  _LoginFormState createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  String? _emailError;
  String? _passwordError;
  
  void _validateForm() {
    setState(() {
      _emailError = _emailController.text.isEmpty 
          ? 'メールアドレスを入力してください' 
          : null;
      
      _passwordError = _passwordController.text.isEmpty 
          ? 'パスワードを入力してください' 
          : null;
    });
    
    if (_emailError == null && _passwordError == null) {
      widget.onSubmit?.call(
        _emailController.text, 
        _passwordController.text
      );
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('ログイン')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextField(
                key: Key('email'),
                controller: _emailController,
                decoration: InputDecoration(
                  labelText: 'メールアドレス',
                  errorText: _emailError,
                ),
              ),
              SizedBox(height: 16),
              TextField(
                key: Key('password'),
                controller: _passwordController,
                obscureText: true,
                decoration: InputDecoration(
                  labelText: 'パスワード',
                  errorText: _passwordError,
                ),
              ),
              SizedBox(height: 24),
              ElevatedButton(
                onPressed: _validateForm,
                child: Text('ログイン'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// ログインフォームのウィジェットテスト
void main() {
  group('LoginForm', () {
    testWidgets('空フォーム送信でエラーメッセージが表示される', (tester) async {
      await tester.pumpWidget(MaterialApp(home: LoginForm()));
      
      // 初期状態ではエラーなし
      expect(find.text('メールアドレスを入力してください'), findsNothing);
      expect(find.text('パスワードを入力してください'), findsNothing);
      
      // 空フォーム送信
      await tester.tap(find.text('ログイン'));
      await tester.pump();
      
      // エラーメッセージ検証
      expect(find.text('メールアドレスを入力してください'), findsOneWidget);
      expect(find.text('パスワードを入力してください'), findsOneWidget);
    });
    
    testWidgets('正しい入力でコールバックが呼ばれる', (tester) async {
      String? submittedEmail;
      String? submittedPassword;
      
      await tester.pumpWidget(MaterialApp(
        home: LoginForm(
          onSubmit: (email, password) {
            submittedEmail = email;
            submittedPassword = password;
          },
        ),
      ));
      
      // フォーム入力
      await tester.enterText(find.byKey(Key('email')), 'test@example.com');
      await tester.enterText(find.byKey(Key('password')), 'password123');
      await tester.pump();
      
      // 送信
      await tester.tap(find.text('ログイン'));
      await tester.pump();
      
      // コールバック検証
      expect(submittedEmail, 'test@example.com');
      expect(submittedPassword, 'password123');
      
      // エラーメッセージなし
      expect(find.text('メールアドレスを入力してください'), findsNothing);
      expect(find.text('パスワードを入力してください'), findsNothing);
    });
  });
}
```