# Flutter ユニットテスト メモ

## 1. ソフトウェア開発工程におけるユニットテストの位置づけ

| 開発工程 | 説明 |
|---------|------|
| 1. システム要件定義 | システム全体の要件を定義 |
| 2. 方式設計 | システム構成・アーキテクチャを設計 |
| 3. 詳細設計 | コンポーネントレベルの詳細を設計 |
| 4. ソフトウェア構築 | コードの実装 |
| **5. 単体テスト（ユニットテスト）** | **個々の関数・クラスの動作検証** |
| 6. 結合テスト | コンポーネント間の連携を検証 |
| 7. システム適格性確認テスト | システム全体の要件適合性を検証 |

## 2. ユニットテストの重要性

- **バグの早期発見**: 実装直後に問題を検出
- **リファクタリングの安全性**: コード変更後も期待通りに動作するか確認
- **ドキュメンテーション**: テストコードが実装の使用例を提供
- **設計の改善**: テスト駆動開発(TDD)により堅牢な設計が促進される

## 3. Flutterにおける自動テストの種類

| テスト種類 | 対象 | 特徴 |
|-----------|------|------|
| ユニットテスト | 関数・メソッド・クラス | 最小単位の機能検証、高速、依存を分離 |
| ウィジェットテスト | 単一ウィジェット | UIコンポーネントの動作検証、中速 |
| インテグレーションテスト | 複数ウィジェット・システム全体 | エンドツーエンドの動作検証、低速 |

## 4. 必要なパッケージと役割

| パッケージ | バージョン例 | 役割 | 使用例 |
|-----------|------------|------|--------|
| flutter_test | SDK標準 | Flutter向け基本テスト環境 | 基本的なテスト・アサート機能 |
| test | ^1.24.0 | Dart言語用汎用テストフレームワーク | 非UIコードのテスト |
| mockito | ^5.4.0 | モックオブジェクト作成 | 外部依存のモック化 |
| build_runner | ^2.4.0 | mockitoのコード生成 | モッククラスの自動生成 |

### その他の有用なテストパッケージ

| パッケージ | 用途 |
|-----------|------|
| fake_async | 時間依存コードのテスト |
| mocktail | コード生成不要のモッキング |
| golden_toolkit | スクリーンショットテスト |
| bloc_test | BLoCパターン用テスト |

## 5. テストファイル構造

```
your_project/
├── lib/
│   ├── src/
│   │   ├── models/
│   │   ├── services/
│   │   └── ...
│   └── main.dart
├── test/
│   ├── unit/
│   │   ├── models/
│   │   └── services/
│   ├── widget/
│   ├── integration/
│   └── helpers/
└── analysis_options.yaml
```

## 6. テストファイルの基本構造

```dart
// test/calculator_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app/calculator.dart';

void main() {
  group('Calculator', () {
    late Calculator calculator;
    
    setUp(() {
      calculator = Calculator();
    });
    
    test('加算が正しく動作する', () {
      expect(calculator.add(2, 3), equals(5));
    });
    
    test('減算が正しく動作する', () {
      expect(calculator.subtract(5, 2), equals(3));
    });
  });
}
```

## 7. セットアップとティアダウン

```dart
group('DatabaseTest', () {
  late Database db;
  
  setUp(() {
    // 各テスト前に実行
    db = Database();
    db.connect();
  });
  
  tearDown(() {
    // 各テスト後に実行
    db.close();
  });
  
  test('データ挿入', () { /* ... */ });
});
```

## 8. モック化の例

モック化とは外部依存（APIサーバー、データベースなど）をテスト用の擬似オブジェクトに置き換える技術。

```dart
// モック作成用のアノテーション
@GenerateMocks([HttpClient])
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:your_app/api_service.dart';
import 'package:test/test.dart';
import 'api_service_test.mocks.dart';

void main() {
  group('ApiService', () {
    late ApiService apiService;
    late MockHttpClient mockHttpClient;
    
    setUp(() {
      mockHttpClient = MockHttpClient();
      apiService = ApiService(client: mockHttpClient);
    });
    
    test('データ取得成功', () async {
      // モックの振る舞いを定義
      when(mockHttpClient.get(any))
          .thenAnswer((_) async => MockResponse(200, '{"data": "test"}'));
      
      final result = await apiService.fetchData();
      
      expect(result, equals({'data': 'test'}));
    });
  });
}
```

### モック化の具体例（天気APIの場合）

```dart
// 実際のAPIクライアントクラス
class WeatherApiClient {
  Future<Map<String, dynamic>> getWeather(String city) async {
    // 実際には外部サーバーにHTTPリクエストを送る処理
    final response = await http.get(Uri.parse('https://api.weather.com/data/$city'));
    return jsonDecode(response.body);
  }
}

// このAPIクライアントを使うサービスクラス
class WeatherService {
  final WeatherApiClient client;
  
  WeatherService(this.client);
  
  Future<String> getWeatherDescription(String city) async {
    final data = await client.getWeather(city);
    return data['description'] ?? '不明';
  }
}

// テストコード
@GenerateMocks([WeatherApiClient])
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'weather_service_test.mocks.dart';

void main() {
  group('WeatherService', () {
    late MockWeatherApiClient mockClient;
    late WeatherService weatherService;
    
    setUp(() {
      mockClient = MockWeatherApiClient();
      weatherService = WeatherService(mockClient);
    });
    
    test('晴れの日の天気説明を返すこと', () async {
      when(mockClient.getWeather('東京'))
          .thenAnswer((_) async => {'description': '晴れ'});
      
      final result = await weatherService.getWeatherDescription('東京');
      
      expect(result, equals('晴れ'));
      verify(mockClient.getWeather('東京')).called(1);
    });
  });
}
```

## 9. 非同期テスト

```dart
test('非同期データ取得', () async {
  // 非同期処理のテスト
  final data = await userRepository.fetchUserData(1);
  expect(data.name, equals('田中太郎'));
});

test('Future処理のエラーハンドリング', () {
  // エラー発生のテスト
  expect(
    () => userRepository.fetchUserData(-1),
    throwsA(isA<InvalidIdException>()),
  );
});
```

## 10. パラメタライズドテスト

```dart
void main() {
  final testCases = [
    {'input': [1, 2], 'expected': 3},
    {'input': [0, 0], 'expected': 0},
    {'input': [-1, 1], 'expected': 0},
  ];
  
  group('Calculator.add', () {
    late Calculator calculator;
    
    setUp(() {
      calculator = Calculator();
    });
    
    for (final testCase in testCases) {
      final input = testCase['input'] as List<int>;
      final expected = testCase['expected'] as int;
      
      test('$input の合計が $expected になる', () {
        expect(calculator.add(input[0], input[1]), equals(expected));
      });
    }
  });
}
```

## 11. コードカバレッジの測定

| カバレッジ種類 | 説明 |
|--------------|------|
| ライン（行）カバレッジ | コード行の何%が実行されたか |
| 関数カバレッジ | 関数の何%が呼び出されたか |
| 分岐カバレッジ | 条件分岐の何%が実行されたか |
| 命令カバレッジ | 個々の命令の何%が実行されたか |

```bash
# カバレッジレポート生成
flutter test --coverage

# HTMLレポート生成（lcovがインストールされている場合）
genhtml coverage/lcov.info -o coverage/html

# ブラウザでレポートを開く
open coverage/html/index.html
```

## 12. テスト駆動開発（TDD）の流れ

1. **先にテストを書く**: 実装前にテストコードを書き、失敗することを確認
2. **最小限の実装**: テストが通る最小限のコードを実装
3. **リファクタリング**: テストが通ることを確認しながらコードを改善

```dart
// 1. まずテストを書く
test('ユーザー名のバリデーション - 空文字列はエラー', () {
  final validator = UserValidator();
  expect(() => validator.validateName(''), throwsA(isA<ValidationError>()));
});

// 2. 最小限の実装
class UserValidator {
  void validateName(String name) {
    if (name.isEmpty) {
      throw ValidationError('名前は空にできません');
    }
  }
}

// 3. リファクタリング
```

## 13. ユニットテストのベストプラクティス

| プラクティス | 説明 |
|------------|------|
| テストの独立性 | テスト間で依存関係を作らない |
| テストの高速性 | 短時間で実行できるようにする |
| 明確な命名 | テスト名で何をテストしているか明確にする |
| 1テスト1アサーション | テストの目的を明確にする |
| AAAパターン | Arrange（準備）、Act（実行）、Assert（検証） |
| テスト可能な設計 | 依存性注入などでテスト可能なコードを設計 |

## 14. Lintとコーディング規約（analysis_options.yaml）

```yaml
# プロジェクトルートに配置する設定ファイル
include: package:flutter_lints/flutter.yaml  # 基本的なFlutterのLintルールを含む

analyzer:
  exclude:
    - "**/*.g.dart"  # 生成されたコードを除外
    - "**/*.freezed.dart"  # freezedで生成されたコードを除外
  errors:
    # 特定のエラーの扱いをカスタマイズ
    missing_return: error  # 戻り値の欠落をエラーとして扱う
    unused_import: warning  # 未使用のインポートを警告として扱う
    todo: info  # TODOコメントを情報として扱う

linter:
  rules:
    # テスト関連のLintルール
    - avoid_print  # テスト中にprint文を使わない
    - prefer_const_constructors  # 定数コンストラクタを推奨
    - prefer_final_locals  # ローカル変数をfinalで宣言することを推奨
    - sort_pub_dependencies  # pubspec.yamlの依存関係をソート
    - always_declare_return_types  # 常に戻り値の型を宣言する
```

## 15. IDEでのテスト設定（Android Studio / IntelliJ IDEA）

### テスト実行設定手順

1. **メインメニューから**：`main.dart` を開く
2. **Edit Configurations...** を選択
3. **Run/Debug Configuration** ダイアログが表示される
4. **+ボタン** をクリック
5. **Flutter Test** を選択
6. **Test scope** セクションで設定：
   - **All in directory** を選択
   - **Test directory** フィールドに `test` ディレクトリへのパスを設定

## 16. CI/CDの基本

| フェーズ | ステップ | 説明 |
|---------|---------|------|
| **CI** | コード更新 | 開発者がコードをリポジトリにプッシュ |
|  | Lint | 静的解析によるコード品質チェック |
|  | ビルド | コードのコンパイルとビルド検証 |
|  | ユニットテスト | 自動テストの実行と検証 |
|  | アラート | 問題があれば開発者に通知 |
| **CD** | デプロイ | テスト環境、ステージング環境への配置 |
|  | リリース | ストアや本番環境への公開 |

### 主なツール

- **GitHub Actions**: GitHubに統合された比較的新しいCI/CDサービス
- **Jenkins**: 広く使われているオープンソースのCI/CDツール

### Flutter CI/CDの基本的な流れ

```
コードのコミット/プッシュ
↓
CI処理開始
  ↓ 
  Lint/静的解析 (flutter analyze)
  ↓
  依存関係の取得 (flutter pub get)
  ↓
  ビルド検証 (flutter build)
  ↓
  ユニットテスト (flutter test)
  ↓
  [任意] コードカバレッジレポート生成
  ↓
  [問題があれば] 開発者に通知
↓
CD処理開始（CIが成功した場合）
  ↓
  環境別ビルド生成
  ↓
  テスト環境へのデプロイ
  ↓
  [承認後] ステージング環境へのデプロイ
  ↓
  [承認後] 本番環境/ストアへの公開
```