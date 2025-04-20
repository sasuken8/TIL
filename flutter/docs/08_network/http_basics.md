# HTTP通信の基本

## 1. HTTPプロトコルの基礎

### HTTPとは
- **定義**: Hypertext Transfer Protocol - クライアントとサーバー間の通信プロトコル
- **主な特徴**: リクエスト/レスポンスモデル、ステートレス

### HTTPとHTTPSの比較

| 項目 | HTTP | HTTPS |
|------|------|-------|
| 暗号化 | なし | SSL/TLSによる暗号化あり |
| ポート | 80 | 443 |
| セキュリティ | 低 | 高 |
| データ保護 | なし | 盗聴・改ざん防止 |
| 認証 | なし | サーバー認証あり |

## 2. HTTPリクエストとレスポンス

### HTTPリクエスト構成要素

| 要素 | 説明 | 例 |
|------|------|-----|
| メソッド | 操作の種類 | GET, POST, PUT, DELETE |
| URL | リソースの場所 | https://api.example.com/users |
| ヘッダー | メタデータ | Content-Type, Authorization |
| ボディ | 送信データ | JSONデータ |

### HTTPメソッド

| メソッド | 用途 | べき等性 | 安全性 |
|--------|------|---------|-------|
| GET | データ取得 | ◯ | ◯ |
| POST | データ作成 | × | × |
| PUT | データ更新（全体） | ◯ | × |
| PATCH | データ更新（一部） | × | × |
| DELETE | データ削除 | ◯ | × |

### HTTPレスポンス構成要素

| 要素 | 説明 | 例 |
|------|------|-----|
| ステータスコード | 処理結果 | 200, 404, 500 |
| ヘッダー | メタデータ | Content-Type, Content-Length |
| ボディ | 返却データ | JSONデータ |

### 具体的なリクエスト・レスポンス例

#### GETリクエスト例
```
GET /api/users?page=1&limit=10 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### GETレスポンス例
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 357

{
  "data": [
    {"id": 1, "name": "John Doe", "email": "john@example.com"},
    {"id": 2, "name": "Jane Smith", "email": "jane@example.com"}
  ],
  "meta": {
    "current_page": 1,
    "total_pages": 5,
    "total_records": 42
  }
}
```

#### POSTリクエスト例
```
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Length: 75

{
  "name": "Alice Brown",
  "email": "alice@example.com",
  "password": "secureP@ss123"
}
```

#### POSTレスポンス例
```
HTTP/1.1 201 Created
Content-Type: application/json
Location: https://api.example.com/api/users/3
Content-Length: 157

{
  "id": 3,
  "name": "Alice Brown",
  "email": "alice@example.com",
  "created_at": "2025-04-19T12:34:56Z"
}
```

### 主要なステータスコード

| コード | カテゴリ | 説明 | 例 |
|-------|---------|------|-----|
| 2xx | 成功 | リクエスト成功 | 200 OK, 201 Created |
| 3xx | リダイレクト | 追加アクション必要 | 301 Moved, 304 Not Modified |
| 4xx | クライアントエラー | リクエスト側の問題 | 400 Bad Request, 404 Not Found |
| 5xx | サーバーエラー | サーバー側の問題 | 500 Internal Error, 503 Service Unavailable |

### ステータスコードの詳細

| コード | 名前 | 意味 | APIでの用途 |
|--------|------|------|------------|
| **2xx - 成功** |
| 200 | OK | リクエスト成功 | GETでのデータ取得成功 |
| 201 | Created | リソース作成成功 | POSTでの新規作成成功 |
| 202 | Accepted | 処理受付済み | 非同期処理開始の受付 |
| 204 | No Content | 成功だがコンテンツなし | DELETE成功など |
| **3xx - リダイレクト** |
| 301 | Moved Permanently | 恒久的な移動 | APIのバージョン移行 |
| 302 | Found | 一時的な移動 | 非推奨 |
| 304 | Not Modified | 変更なし | キャッシュ利用可能 |
| **4xx - クライアントエラー** |
| 400 | Bad Request | 不正なリクエスト | リクエスト形式不備 |
| 401 | Unauthorized | 認証エラー | 認証情報の不足・不正 |
| 403 | Forbidden | アクセス権限なし | 権限不足 |
| 404 | Not Found | リソースが存在しない | 該当データなし |
| 405 | Method Not Allowed | 許可されていないメソッド | 不適切なHTTPメソッド |
| 409 | Conflict | 競合 | 同時編集の衝突 |
| 422 | Unprocessable Entity | 処理不可能 | バリデーションエラー |
| 429 | Too Many Requests | リクエスト過多 | レート制限超過 |
| **5xx - サーバーエラー** |
| 500 | Internal Server Error | サーバー内部エラー | 予期せぬサーバーエラー |
| 502 | Bad Gateway | ゲートウェイエラー | バックエンドサービス障害 |
| 503 | Service Unavailable | サービス利用不可 | メンテナンス中など |
| 504 | Gateway Timeout | ゲートウェイタイムアウト | バックエンド処理遅延 |

## 3. ヘッダーと用途

### よく使うHTTPヘッダー

| ヘッダー名 | 方向 | 用途 |
|------------|--------|------|
| **一般的なヘッダー** |
| Content-Type | 双方向 | データ形式の指定 |
| Content-Length | 双方向 | ボディサイズ |
| **リクエストヘッダー** |
| Authorization | リクエスト | 認証情報 |
| Accept | リクエスト | 受け入れ可能な形式 |
| User-Agent | リクエスト | クライアント情報 |
| Accept-Language | リクエスト | 言語設定 |
| **レスポンスヘッダー** |
| Location | レスポンス | リダイレクト先URL |
| Cache-Control | レスポンス | キャッシュ制御 |
| ETag | レスポンス | リソース識別子 |
| **セキュリティ関連** |
| X-XSS-Protection | レスポンス | XSS対策 |
| X-Content-Type-Options | レスポンス | MIMEタイプ強制 |
| Strict-Transport-Security | レスポンス | HTTPS強制 |

### Content-Type一覧

| Content-Type | 用途 | データ形式 |
|--------------|------|-----------|
| application/json | JSONデータ | `{"name": "value"}` |
| application/x-www-form-urlencoded | フォームデータ | `name=value&key=data` |
| multipart/form-data | ファイルアップロード | バイナリデータ + テキスト |
| text/plain | プレーンテキスト | `plain text` |

## 4. エンドポイントとクエリパラメータ

### エンドポイント構造

```
https://api.example.com/users/123/posts
└─ ベースURL ─┘└ パス ┘
```

### クエリパラメータ

```
https://api.example.com/products?category=electronics&sort=price&order=asc
                           └───────── クエリパラメータ ─────────┘
```

| 特徴 | 説明 |
|------|------|
| 区切り記号 | URLの後に`?`で始まり、複数の場合は`&`で連結 |
| エンコード | 特殊文字はURLエンコードが必要 |
| 用途 | フィルタリング、ソート、ページネーションなど |

## 5. Flutterでの実装

### パッケージの追加
```yaml
dependencies:
  http: ^1.1.0
```

### HTTPメソッドとFlutterコード対応表

| HTTPメソッド | Flutter実装 | 用途 |
|-------------|-------------|------|
| GET | `http.get(url)` | データ取得 |
| POST | `http.post(url, body: jsonEncode(data))` | データ作成 |
| PUT | `http.put(url, body: jsonEncode(data))` | データ更新(全体) |
| PATCH | `http.patch(url, body: jsonEncode(data))` | データ更新(一部) |
| DELETE | `http.delete(url)` | データ削除 |

### 基本的なリクエスト処理
```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

// GETリクエスト
Future<Map<String, dynamic>> fetchData() async {
  final url = Uri.parse('https://api.example.com/data');
  final response = await http.get(url);
  
  if (response.statusCode == 200) {
    return jsonDecode(response.body);
  } else {
    throw Exception('Failed: ${response.statusCode}');
  }
}

// POSTリクエスト
Future<Map<String, dynamic>> createData(Map<String, dynamic> data) async {
  final url = Uri.parse('https://api.example.com/data');
  final response = await http.post(
    url,
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode(data)
  );
  
  if (response.statusCode == 201) {
    return jsonDecode(response.body);
  } else {
    throw Exception('Failed: ${response.statusCode}');
  }
}
```

### クエリパラメータの設定
```dart
// クエリパラメータを含むURL
final url = Uri.https(
  'api.example.com',
  '/products',
  {
    'category': 'electronics',
    'sort': 'price',
    'order': 'asc'
  }
);
```

### JSON処理
```dart
// オブジェクト -> JSON文字列
Map<String, dynamic> user = {'name': 'John', 'age': 30};
String jsonString = jsonEncode(user);

// JSON文字列 -> オブジェクト
Map<String, dynamic> decodedUser = jsonDecode(jsonString);
```

## 6. エラーハンドリング

### エラーの種類と対応方法

| エラー種類 | 原因 | 対応方法 |
|-----------|------|---------|
| ネットワークエラー | 接続問題 | 再接続、オフラインモード提供 |
| タイムアウト | 長時間応答なし | リトライ機構、タイムアウト設定 |
| サーバーエラー | サーバー側の問題 | エラーメッセージ表示、後ほど再試行 |
| クライアントエラー | リクエスト形式の問題 | 入力検証、適切なガイダンス |
| パースエラー | 無効なレスポンス形式 | 例外処理、デバッグ情報収集 |

### エラーコードとUIフィードバックの対応表

| エラーコード | ユーザーへの表示メッセージ | UI対応 | 
|-------------|-------------------------|--------|
| 400 | 入力内容に問題があります。入力項目を確認してください。 | フォームエラー表示 |
| 401 | セッションの有効期限が切れました。再ログインしてください。 | ログイン画面へ遷移 |
| 403 | この操作を行う権限がありません。 | アクセス制限の説明 |
| 404 | 指定したデータが見つかりません。 | 空の状態表示 |
| 409 | データが競合しています。最新の情報を取得して再度試行してください。 | 再読み込みボタン表示 |
| 422 | 入力内容が正しくありません。以下の項目を確認してください。 | バリデーションエラー詳細表示 |
| 429 | リクエスト制限を超えました。しばらく待ってから再度試行してください。 | カウントダウンタイマー表示 |
| 5xx | サーバーでエラーが発生しました。時間をおいて再度試行してください。 | 再試行ボタン表示 |

### 実装例
```dart
Future<dynamic> apiRequest(String endpoint) async {
  try {
    final url = Uri.parse('https://api.example.com/$endpoint');
    final response = await http.get(url)
        .timeout(const Duration(seconds: 10));
    
    if (response.statusCode >= 200 && response.statusCode < 300) {
      return jsonDecode(response.body);
    } else {
      _handleErrorStatus(response.statusCode);
    }
  } on SocketException {
    throw '接続エラー: ネットワーク接続を確認';
  } on TimeoutException {
    throw 'タイムアウト: 後でもう一度試行';
  } on FormatException {
    throw 'データ形式エラー: 不正なレスポンス形式';
  }
}

void _handleErrorStatus(int statusCode) {
  switch (statusCode) {
    case 400: throw '無効なリクエスト';
    case 401: throw '認証エラー: 再ログインが必要';
    case 403: throw 'アクセス権限がありません';
    case 404: throw 'リソースが見つかりません';
    case 429: throw 'リクエスト制限超過: しばらく待機';
    case 500: throw 'サーバーエラー: システム管理者に連絡';
    default: throw 'エラー: ステータスコード $statusCode';
  }
}
```

## 7. セキュリティとHTTPS

### HTTPSの設定

| プラットフォーム | 設定方法 |
|---------------|---------|
| Android | `usesCleartextTraffic="false"` (AndroidManifest.xml) |
| iOS | App Transport Security (Info.plist) |

### 証明書検証
```dart
// 注意: 本番環境では自己署名証明書を許容しないこと
HttpClient httpClient = HttpClient()
  ..badCertificateCallback = 
      ((X509Certificate cert, String host, int port) => false);
```

### Bearer認証
```dart
final response = await http.get(
  url,
  headers: {
    'Authorization': 'Bearer $token',
    'Content-Type': 'application/json'
  }
);
```

## 8. APIサービス設計パターン

```dart
class ApiService {
  final String baseUrl;
  final http.Client client;
  
  ApiService({
    required this.baseUrl,
    http.Client? client
  }) : client = client ?? http.Client();
  
  Future<T> get<T>(
    String endpoint,
    T Function(dynamic) fromJson
  ) async {
    final url = Uri.parse('$baseUrl/$endpoint');
    final response = await client.get(url);
    
    return _processResponse(response, fromJson);
  }
  
  Future<T> post<T>(
    String endpoint,
    dynamic data,
    T Function(dynamic) fromJson
  ) async {
    final url = Uri.parse('$baseUrl/$endpoint');
    final response = await client.post(
      url,
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(data)
    );
    
    return _processResponse(response, fromJson);
  }
  
  T _processResponse<T>(
    http.Response response,
    T Function(dynamic) fromJson
  ) {
    if (response.statusCode >= 200 && response.statusCode < 300) {
      return fromJson(jsonDecode(response.body));
    } else {
      throw HttpException(
        'API Error: ${response.statusCode}',
        uri: response.request?.url
      );
    }
  }
  
  void dispose() {
    client.close();
  }
}
```

## 9. UI連携パターン

### ローディング・エラーハンドリングの状態管理
```dart
enum RequestState { initial, loading, success, error }

class ApiState<T> {
  final RequestState state;
  final T? data;
  final String? errorMessage;
  
  ApiState.initial() : 
    state = RequestState.initial, 
    data = null, 
    errorMessage = null;
    
  ApiState.loading() : 
    state = RequestState.loading, 
    data = null, 
    errorMessage = null;
    
  ApiState.success(this.data) : 
    state = RequestState.success, 
    errorMessage = null;
    
  ApiState.error(this.errorMessage) : 
    state = RequestState.error, 
    data = null;
}
```

### UI実装例
```dart
Widget buildContent() {
  switch (apiState.state) {
    case RequestState.initial:
      return Container();
    case RequestState.loading:
      return Center(child: CircularProgressIndicator());
    case RequestState.success:
      return ListView.builder(
        itemCount: apiState.data!.length,
        itemBuilder: (context, index) => ListTile(
          title: Text(apiState.data![index].title),
        ),
      );
    case RequestState.error:
      return Center(
        child: Column(
          children: [
            Text('エラー: ${apiState.errorMessage}'),
            ElevatedButton(
              onPressed: fetchData,
              child: Text('再試行'),
            ),
          ],
        ),
      );
  }
}
```

## 10. ベストプラクティス

| カテゴリ | ベストプラクティス |
|---------|------------------|
| 設計 | APIサービスの抽象化、Repository パターンの導入 |
| セキュリティ | HTTPS使用、認証トークンの安全な保存 |
| パフォーマンス | コネクションプーリング、キャッシュ機構 |
| エラー処理 | 階層的なエラー処理、UIフィードバック |
| テスト | モックサーバー、単体テスト、統合テスト |
| バックグラウンド処理 | バックグラウンドでの通信、通知機構 |