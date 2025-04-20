# Go Routerを使ったルーティング

## パッケージのインストール

Go Routerをプロジェクトに追加するには、以下のコマンドを実行する：

```bash
flutter pub add go_router
```

または、`pubspec.yaml`ファイルに直接追加：

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^13.0.0  # 最新バージョンを使用
```

追加後は、パッケージを取得：

```bash
flutter pub get
```

## 命令的アプローチと宣言的アプローチの比較

| 特徴 | Navigator 1.0 (命令的) | Navigator 2.0/Go Router (宣言的) |
|------|------------------------|----------------------------------|
| 概念 | **何をするか**を指示する | **どのような状態であるべきか**を記述する |
| 実装方法 | `Navigator.push()`, `pop()` | ルート定義とURLパスによる状態表現 |
| コード配置 | アプリ各所に散らばる | 一箇所に集中 |
| 拡張性 | 複雑なケースで管理困難 | 複雑なケースでも整理しやすい |
| Web対応 | 難しい | 容易 |
| ディープリンク | 実装が複雑 | 簡単に対応可能 |

### 命令的アプローチの例（Navigator 1.0）

```dart
// ボタンクリック時：詳細画面に進む
ElevatedButton(
  onPressed: () {
    Navigator.push(
      context, 
      MaterialPageRoute(builder: (context) => DetailsScreen(id: '123')),
    );
  },
  child: Text('詳細を見る'),
),

// 別の場所：戻るボタンの処理
IconButton(
  icon: Icon(Icons.arrow_back),
  onPressed: () {
    Navigator.pop(context);
  },
),
```

### 宣言的アプローチの例（Go Router）

```dart
// GoRouterの定義（一箇所に集約）
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
    ),
    GoRoute(
      path: '/details/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return DetailsScreen(id: id);
      },
    ),
  ],
);

// ボタンクリック時：URLが変わるだけ
ElevatedButton(
  onPressed: () => context.go('/details/123'),
  child: Text('詳細を見る'),
),
```

## ディープリンクの種類と特徴

| 種類 | 形式例 | 特徴 | 用途 |
|------|--------|------|------|
| カスタムURLスキーム | `myapp://product/123` | - 独自のプロトコル識別子<br>- 古くからある方法<br>- 他アプリと競合の可能性 | - アプリ内遷移<br>- 簡易なディープリンク |
| Universal Links (iOS)<br>App Links (Android) | `https://example.com/product/123` | - 通常のウェブURLを使用<br>- インストール状態で振り分け<br>- 高いセキュリティ | - Webアプリとの連携<br>- ソーシャルシェア<br>- マーケティング |
| Firebase Dynamic Links | カスタマイズ可能 | - 多機能<br>- プラットフォーム間の一貫性<br>- インストール誘導機能 | - 複雑なディープリンク<br>- インストール前後で一貫した体験 |

## Go Routerの主要機能

### 基本的なルーター設定

```dart
final GoRouter _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/details',
      builder: (context, state) => const DetailsScreen(),
    ),
  ],
);

void main() {
  runApp(MaterialApp.router(
    routerConfig: _router,
    title: 'Go Router Example',
  ));
}
```

### 画面遷移メソッド

| メソッド | 用途 | 挙動 |
|----------|------|------|
| `context.go('/path')` | 新しい画面に遷移 | スタックをリセット |
| `context.push('/path')` | 新しい画面をスタックに追加 | 戻るボタンで前画面に戻れる |
| `context.pop()` | 前の画面に戻る | スタックから現在の画面を削除 |
| `context.goNamed('name')` | 名前付きルートに遷移 | スタックをリセット |

### パラメータの受け渡し

#### パスパラメータ
```dart
GoRoute(
  path: '/user/:id',
  builder: (context, state) {
    final userId = state.pathParameters['id'];
    return UserScreen(userId: userId);
  },
),

// 使い方
context.go('/user/123');
```

#### クエリパラメータ
```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'];
    return SearchScreen(query: query);
  },
),

// 使い方
context.go('/search?q=flutter');
```

### トランジション（画面遷移アニメーション）

| トランジション | 用途 | 実装例 |
|----------------|------|--------|
| フェード | 徐々に表示/非表示 | `FadeTransition(opacity: animation, child: child)` |
| スライド | 画面の端からスライドイン | `SlideTransition(position: offsetAnimation, child: child)` |
| スケール | 拡大/縮小 | `ScaleTransition(scale: animation, child: child)` |
| 回転 | 回転しながら表示 | `RotationTransition(turns: animation, child: child)` |
| 複合 | 複数の効果を組み合わせ | フェード + スライドなど |

#### トランジション実装例

```dart
GoRoute(
  path: '/details',
  pageBuilder: (context, state) => CustomTransitionPage<void>(
    key: state.pageKey,
    child: const DetailsScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      // スライドトランジション
      const begin = Offset(1.0, 0.0);
      const end = Offset.zero;
      final tween = Tween(begin: begin, end: end);
      final offsetAnimation = animation.drive(tween);
      
      return SlideTransition(
        position: offsetAnimation,
        child: child,
      );
    },
  ),
),
```

## プロジェクト構成のベストプラクティス

一般的なフォルダ構成：

```
lib/
  ├── main.dart             // アプリのエントリーポイント
  ├── app.dart              // MaterialAppを含むアプリのルート
  ├── config/               // アプリの設定
  │   ├── router.dart       // GoRouterの設定
  │   ├── theme.dart        // テーマ設定
  │   └── api_config.dart   // APIエンドポイント定義
  ├── models/               // データモデル
  ├── pages/                // UIページ
  │   ├── home_page.dart
  │   ├── login_page.dart
  │   ├── detail_page.dart
  │   └── splash_page.dart
  ├── widgets/              // 再利用可能なウィジェット
  │   └── menu.dart
  ├── services/             // サービス（認証、APIなど）
  │   ├── auth_service.dart
  │   └── api_service.dart
  └── utils/                // ユーティリティ関数
      ├── date_formatter.dart
      └── validators.dart
```

### ディレクトリの役割

| ディレクトリ | 役割 | 例 |
|-------------|------|-----|
| config/ | アプリ全体の設定を管理 | API URL、テーマ、ルーター設定 |
| services/ | 外部システム連携や複雑なロジック | API通信処理、認証、データベース操作 |
| utils/ | 汎用的な機能（状態を持たない） | 日付フォーマット、バリデーション、計算 |
| pages/ | 画面全体のレイアウトと状態管理 | 各画面のクラス定義 |
| widgets/ | 再利用可能なUI部品 | カスタムボタン、メニュー、カード |

### APIエンドポイントとAPI通信処理の違い

| 項目 | APIエンドポイント | API通信処理 |
|------|-----------------|------------|
| 何か | 通信先のURL | 通信を行うコードやロジック |
| どこに定義 | config/api_config.dart | services/api_service.dart |
| 性質 | 静的な情報、設定値 | 動的な処理、ビジネスロジック |
| 例 | `static const String login = '/auth/login';` | `Future<LoginResult> login(String username, String password) {...}` |

#### APIエンドポイント定義例

```dart
// lib/config/api_config.dart
class ApiConfig {
  // 環境によって切り替え可能なベースURL
  static const String baseUrl = 'https://api.example.com/v1';
  
  // 各種エンドポイント
  static const String login = '$baseUrl/auth/login';
  static const String register = '$baseUrl/auth/register';
  static const String getUserProfile = '$baseUrl/users/profile';
  static const String updateUserProfile = '$baseUrl/users/profile';
  static const String getProducts = '$baseUrl/products';
  static const String getProductDetails = '$baseUrl/products/'; // + product_id を後で追加
}
```

#### API通信処理実装例

```dart
// lib/services/api_service.dart
import 'package:http/http.dart' as http;
import '../config/api_config.dart';

class ApiService {
  Future<Map<String, dynamic>> login(String username, String password) async {
    final response = await http.post(
      Uri.parse(ApiConfig.login),
      body: {
        'username': username,
        'password': password,
      },
    );
    
    if (response.statusCode == 200) {
      return jsonDecode(response.body);
    } else {
      throw Exception('ログインに失敗しました');
    }
  }
}
```

## Splash画面の役割

Splash画面（スプラッシュスクリーン）は、アプリ起動時に最初に表示される画面で、以下の役割がある：

1. ブランディング表示 - アプリのロゴや名前を表示
2. 初期化処理 - アプリの起動に必要なデータの読み込みや初期化処理
3. ルート決定 - ユーザーのログイン状態などに基づき、最初に表示すべき画面を決定

## SharedPreferencesを使ったセッション管理

SharedPreferencesは、キーバリュー形式で簡易的なデータをデバイスに保存するためのプラグイン。ログイン状態やユーザー設定など、小規模なデータの永続化に適している。

### 導入方法

```bash
flutter pub add shared_preferences
```

### 基本的な使い方

```dart
import 'package:shared_preferences/shared_preferences.dart';

class SessionManager {
  // キー定義
  static const String KEY_IS_LOGGED_IN = 'is_logged_in';
  static const String KEY_USER_ID = 'user_id';
  static const String KEY_AUTH_TOKEN = 'auth_token';

  // ログイン情報の保存
  static Future<bool> saveLoginInfo(String userId, String token) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool(KEY_IS_LOGGED_IN, true);
    await prefs.setString(KEY_USER_ID, userId);
    await prefs.setString(KEY_AUTH_TOKEN, token);
    return true;
  }

  // ログイン状態の確認
  static Future<bool> isLoggedIn() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getBool(KEY_IS_LOGGED_IN) ?? false;
  }

  // ログアウト
  static Future<bool> logout() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool(KEY_IS_LOGGED_IN, false);
    await prefs.remove(KEY_AUTH_TOKEN);
    return true;
  }
}
```

### Go Routerとの連携例

```dart
final GoRouter _router = GoRouter(
  initialLocation: '/',
  redirect: (BuildContext context, GoRouterState state) async {
    // ログイン状態に基づいてリダイレクト
    final bool isLoggedIn = await SessionManager.isLoggedIn();
    final bool isLoggingIn = state.matchedLocation == '/login';

    if (!isLoggedIn && !isLoggingIn) {
      return '/login';
    }

    if (isLoggedIn && isLoggingIn) {
      return '/home';
    }

    return null;
  },
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => SplashScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginScreen(),
    ),
    GoRoute(
      path: '/home',
      builder: (context, state) => HomeScreen(),
    ),
  ],
);
```

## FlutterのGoRouterとVue RouterのUI/UX比較

FlutterのGo RouterとVue Routerは、異なるプラットフォームでありながら、類似した宣言的なルーティング概念を持つ。

| 機能 | Go Router (Flutter) | Vue Router |
|------|---------------------|------------|
| ルート定義 | `GoRoute(path: '/path')` | `{ path: '/path', component: Component }` |
| パラメータ | `'/user/:id'` | `'/user/:id'` |
| クエリーパラメータ | `state.uri.queryParameters['q']` | `this.$route.query.q` |
| ルート名 | `name: 'home'` | `name: 'home'` |
| 遷移方法 | `context.go('/path')` | `this.$router.push('/path')` |
| 名前付きルート遷移 | `context.goNamed('home')` | `this.$router.push({ name: 'home' })` |
| ネストされたルート | `routes: [GoRoute(...)]` | `children: [{ path: '...' }]` |
| ガード/リダイレクト | `redirect: (context, state) {...}` | `beforeEnter: (to, from, next) => {...}` |
| 遷移アニメーション | `CustomTransitionPage` | `<transition>` コンポーネント |
| URLの扱い | 内部的にパスを管理 | ブラウザのURLを直接操作 |

### 主な類似点

1. 宣言的なルート定義
2. パスパラメータとクエリパラメータのサポート
3. ネストされたルート構造
4. リダイレクト機能
5. 名前付きルートによるナビゲーション

### 主な相違点

1. FlutterはネイティブアプリのためURLバーは表示されないが、内部的には同様の概念で管理
2. Vue Routerはブラウザ履歴APIを直接利用
3. アニメーションの実装方法が異なる
4. Flutter/Dartは強い型付け言語のため、型安全性が高い