# API統合に関する詳細メモ

## 1. APIの基本概念

APIとは、アプリケーション間でデータやサービスを交換するための仕組み。

| 用語 | 説明 | 補足 |
|------|------|------|
| API | Application Programming Interfaceの略。システム間の接続点や通信方法を定義したもの | プログラムが他のプログラムと通信するための「窓口」 |
| WebAPI | HTTP/HTTPSプロトコルを使用してインターネット経由でアクセスできるAPI | ブラウザやアプリからサーバーリソースにアクセスする手段 |
| Interface(I/F) | 2つのシステム間の接続点や通信方法を定義したもの | 内部実装を隠蔽し、決められた方法での通信のみを許可 |
| エンドポイント | APIにアクセスするための特定のURL | 例: `https://api.example.com/users` |
| リソース | APIを通じてアクセスできるデータやサービス | ユーザー情報、商品データなど |

## 2. APIの種類

### Web API

| 種類 | 特徴 | 長所 | 短所 | 例 | Flutter実装方法 |
|------|------|------|------|------|------|
| **REST API** | HTTPメソッドベース、主にJSON形式、ステートレス | シンプル、広く採用、キャッシュ可能 | オーバーフェッチ問題、複数リクエスト必要なケースも | Twitter API, GitHub API, Weather API | httpパッケージ |
| **GraphQL** | 単一エンドポイント、必要なデータのみ指定可能、柔軟なクエリ | 必要なデータのみ取得可能、1リクエストで複雑なデータ取得可能 | サーバー実装が複雑、キャッシュが難しい | GitHub v4 API, Shopify API | graphql_flutterパッケージ |
| **SOAP** | XMLベース、プロトコル指定あり、ステートフル可能 | 厳格な型定義、セキュリティ機能豊富 | 冗長、重い、実装複雑 | 銀行システム、エンタープライズシステム | カスタムXML処理 |
| **WebSocket** | 双方向リアルタイム通信、コネクション維持 | リアルタイム通信可能、低レイテンシ | コネクション維持コスト、スケール難 | チャットアプリ、取引システム、リアルタイムダッシュボード | web_socket_channelパッケージ |
| **WebHook** | イベント駆動型、サーバーからクライアントへの通知 | リアルタイム更新、ポーリング不要 | エンドポイント公開必要、セキュリティ懸念 | GitHub Webhooks, PayPal IPN | サーバー側実装必要 |

### その他のAPI

| 種類 | 特徴 | 使用例 | Flutter実装方法 |
|------|------|------|------|
| **Platform API** | iOSやAndroidのネイティブ機能 | カメラ、センサー、位置情報、通知 | プラットフォームチャネル、プラグイン |
| **Browser API** | ブラウザ機能へのアクセス (Flutter Web) | ローカルストレージ、位置情報、カメラ | js相互運用、特定プラグイン |
| **Firebase API** | Googleバックエンドサービス | 認証、Firestore、Storage、FCM | firebase関連パッケージ |
| **SDK API** | 特定サービス用ライブラリ | 支払い処理、マップ、分析 | 専用SDKパッケージ |
| **Internal API** | アプリ内モジュール間連携 | 機能モジュール間のデータ共有 | クラス設計、依存性注入 |

## 3. リクエスト/レスポンス構造

### リクエスト要素詳細

| 要素 | 説明 | 例 | 補足 |
|------|------|------|------|
| URL | APIのエンドポイント | `https://api.example.com/users?page=1&limit=10` | ベースURL + パス + クエリパラメータ |
| メソッド | HTTPリクエストタイプ | GET: 取得<br>POST: 作成<br>PUT: 完全更新<br>PATCH: 部分更新<br>DELETE: 削除 | RESTfulAPIでは操作と対応 |
| ヘッダ | リクエストに関するメタデータ | `Authorization: Bearer token123`<br>`Content-Type: application/json`<br>`Accept-Language: ja-JP`<br>`Cache-Control: no-cache` | 認証、コンテンツタイプ、言語など指定 |
| ボディ | 送信するデータ本体 | JSONデータ、フォームデータ、ファイルなど | GET/DELETEでは通常不使用 |
| クエリパラメータ | URLに含まれるパラメータ | `?search=keyword&sort=asc&page=2` | フィルタリング、ソート、ページネーションなど |
| パスパラメータ | URLパスに含まれる変数 | `/users/{id}/posts` | リソース識別子など |

### リクエストボディ形式詳細

| 形式 | MIME Type | 特徴 | 用途 | Flutter実装 |
|------|------|------|------|------|
| JSON | application/json | 構造化されたデータ、読みやすい | 最も一般的、多くのAPIで標準 | `jsonEncode/jsonDecode` |
| XML | application/xml | タグベース構造化データ | SOAP、レガシーシステム、複雑な構造 | XMLパーサーライブラリ |
| Form Data | application/x-www-form-urlencoded | キー値ペア、URL形式エンコード | フォーム送信、シンプルなデータ | http.post body直接指定 |
| Multipart | multipart/form-data | ファイルとデータ混在 | ファイルアップロード | MultipartRequest |
| Plain Text | text/plain | 構造化されていないテキスト | シンプルなメッセージ | 文字列直接指定 |
| Binary | application/octet-stream | バイナリデータ | ファイル、画像、音声など | バイト配列 |
| CSV | text/csv | カンマ区切りデータ | 表形式データ、一括インポート | 文字列処理 |

### レスポンス要素詳細

| 要素 | 説明 | 例 | 補足 |
|------|------|------|------|
| ステータスコード | リクエスト結果を示す数字 | 1xx: 情報<br>2xx: 成功 (200: OK, 201: Created)<br>3xx: リダイレクト<br>4xx: クライアントエラー (400: Bad Request, 401: Unauthorized, 404: Not Found)<br>5xx: サーバーエラー (500: Internal Server Error) | HTTPステータスコード |
| ヘッダ | レスポンスに関するメタデータ | `Content-Type: application/json`<br>`Cache-Control: max-age=3600`<br>`X-Rate-Limit-Remaining: 98` | コンテンツタイプ、キャッシュ指示、カスタムヘッダ |
| ボディ | サーバーからの応答データ | JSONデータ、HTMLなど | 実際のレスポンスデータ |
| Content-Length | ボディのサイズ | `Content-Length: 1024` | バイト単位のサイズ |

### 一般的なレスポンス構造（JSON）

```json
{
  "status": "success",           // 処理結果
  "data": {                      // メインデータ
    "id": 123,
    "name": "Example Product",
    "price": 1999
  },
  "meta": {                      // メタデータ
    "total": 256,
    "page": 1,
    "limit": 20
  },
  "links": {                     // リンク（HATEOAS）
    "self": "/products/123",
    "related": "/products/123/reviews"
  }
}
```

## 4. FlutterでのAPI連携実装詳細

### HTTP依存関係

```yaml
dependencies:
  http: ^1.1.0  # Dart 3.0以上に対応
```

### リクエストの詳細実装

```dart
// GETリクエスト（詳細版）
Future<Map<String, dynamic>> fetchData({
  required String endpoint,
  Map<String, String>? headers,
  Map<String, dynamic>? queryParams,
  Duration timeout = const Duration(seconds: 10),
}) async {
  // クエリパラメータの追加
  var uri = Uri.parse(endpoint);
  if (queryParams != null && queryParams.isNotEmpty) {
    final queryString = queryParams.entries
        .map((e) => '${Uri.encodeComponent(e.key)}=${Uri.encodeComponent(e.value.toString())}')
        .join('&');
    uri = Uri.parse('$endpoint?$queryString');
  }
  
  // デフォルトヘッダー
  final requestHeaders = {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  };
  
  // カスタムヘッダー追加
  if (headers != null) {
    requestHeaders.addAll(headers);
  }
  
  try {
    // タイムアウト付きリクエスト
    final response = await http.get(
      uri,
      headers: requestHeaders,
    ).timeout(timeout);
    
    // レスポンス処理
    if (response.statusCode >= 200 && response.statusCode < 300) {
      // 成功時
      if (response.body.isEmpty) return {};
      return jsonDecode(response.body);
    } else {
      // エラー時
      _logError('HTTPエラー', response);
      throw ApiException(
        code: response.statusCode,
        message: _parseErrorMessage(response),
        endpoint: endpoint,
      );
    }
  } on TimeoutException {
    _logError('タイムアウト', endpoint);
    throw ApiException(
      code: 408,
      message: 'リクエストがタイムアウトしました',
      endpoint: endpoint,
    );
  } on SocketException {
    _logError('ネットワークエラー', endpoint);
    throw ApiException(
      code: 0,
      message: 'ネットワーク接続エラー',
      endpoint: endpoint,
    );
  } catch (e) {
    _logError('未分類エラー', e);
    throw ApiException(
      code: -1,
      message: '予期しないエラー: ${e.toString()}',
      endpoint: endpoint,
    );
  }
}
```

```dart
// POSTリクエスト（詳細版）
Future<Map<String, dynamic>> postData({
  required String endpoint,
  required Map<String, dynamic> body,
  Map<String, String>? headers,
  Duration timeout = const Duration(seconds: 10),
}) async {
  final uri = Uri.parse(endpoint);
  
  final requestHeaders = {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  };
  
  if (headers != null) {
    requestHeaders.addAll(headers);
  }
  
  try {
    final response = await http.post(
      uri,
      headers: requestHeaders,
      body: jsonEncode(body),
    ).timeout(timeout);
    
    // レスポンスボディをログ（開発用）
    _logResponse(response);
    
    if (response.statusCode >= 200 && response.statusCode < 300) {
      if (response.body.isEmpty) return {};
      return jsonDecode(response.body);
    } else {
      throw ApiException(
        code: response.statusCode,
        message: _parseErrorMessage(response),
        endpoint: endpoint,
      );
    }
  } catch (e) {
    // エラーハンドリング
    if (e is ApiException) rethrow;
    throw ApiException(
      code: -1,
      message: e.toString(),
      endpoint: endpoint,
    );
  }
}
```

### カスタム例外クラス

```dart
class ApiException implements Exception {
  final int code;
  final String message;
  final String endpoint;
  final dynamic data;
  
  ApiException({
    required this.code,
    required this.message,
    required this.endpoint,
    this.data,
  });
  
  @override
  String toString() => 'APIエラー [$code]: $message (エンドポイント: $endpoint)';
  
  // エラータイプ判別
  bool get isNetworkError => code == 0;
  bool get isAuthError => code == 401 || code == 403;
  bool get isServerError => code >= 500;
  bool get isClientError => code >= 400 && code < 500;
}
```

## 5. 応用実装パターン詳細

### 完全なAPIクライアント抽象化

```dart
// api_client.dart
abstract class ApiClient {
  // 基本メソッド
  Future<Map<String, dynamic>> get(
    String endpoint, {
    Map<String, String>? headers,
    Map<String, dynamic>? queryParams,
  });
  
  Future<Map<String, dynamic>> post(
    String endpoint, {
    required Map<String, dynamic> body,
    Map<String, String>? headers,
  });
  
  Future<Map<String, dynamic>> put(
    String endpoint, {
    required Map<String, dynamic> body,
    Map<String, String>? headers,
  });
  
  Future<Map<String, dynamic>> patch(
    String endpoint, {
    required Map<String, dynamic> body,
    Map<String, String>? headers,
  });
  
  Future<Map<String, dynamic>> delete(
    String endpoint, {
    Map<String, dynamic>? body,
    Map<String, String>? headers,
  });
  
  // アップロード用
  Future<Map<String, dynamic>> uploadFile(
    String endpoint,
    File file, {
    String fieldName = 'file',
    Map<String, String>? fields,
    Map<String, String>? headers,
  });
  
  // クリーンアップ
  void close();
}

// 実装クラス
class HttpApiClient implements ApiClient {
  final http.Client _client;
  final String _baseUrl;
  final Duration _timeout;
  final Map<String, String> _defaultHeaders;
  
  HttpApiClient({
    required String baseUrl,
    http.Client? client,
    Duration timeout = const Duration(seconds: 15),
    Map<String, String>? defaultHeaders,
  }) : _client = client ?? http.Client(),
       _baseUrl = baseUrl,
       _timeout = timeout,
       _defaultHeaders = defaultHeaders ?? {
         'Content-Type': 'application/json',
         'Accept': 'application/json',
       };
  
  @override
  Future<Map<String, dynamic>> get(
    String endpoint, {
    Map<String, String>? headers,
    Map<String, dynamic>? queryParams,
  }) async {
    // 実装
  }
  
  // その他メソッド実装
  
  @override
  void close() {
    _client.close();
  }
}
```

### 完全なリポジトリパターン

```dart
// ユーザーモデル
class User {
  final int id;
  final String name;
  final String email;
  final String? avatarUrl;
  final DateTime createdAt;
  final List<String> roles;
  final Map<String, dynamic> metadata;
  
  User({
    required this.id,
    required this.name,
    required this.email,
    this.avatarUrl,
    required this.createdAt,
    this.roles = const [],
    this.metadata = const {},
  });
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
      avatarUrl: json['avatar_url'],
      createdAt: DateTime.parse(json['created_at']),
      roles: List<String>.from(json['roles'] ?? []),
      metadata: json['metadata'] ?? {},
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
      'avatar_url': avatarUrl,
      'created_at': createdAt.toIso8601String(),
      'roles': roles,
      'metadata': metadata,
    };
  }
  
  // イミュータブルアップデート
  User copyWith({
    String? name,
    String? email,
    String? avatarUrl,
    List<String>? roles,
    Map<String, dynamic>? metadata,
  }) {
    return User(
      id: this.id,
      name: name ?? this.name,
      email: email ?? this.email,
      avatarUrl: avatarUrl ?? this.avatarUrl,
      createdAt: this.createdAt,
      roles: roles ?? this.roles,
      metadata: metadata ?? this.metadata,
    );
  }
}

// リポジトリインターフェース
abstract class UserRepository {
  Future<List<User>> getUsers({int page = 1, int limit = 20});
  Future<User> getUserById(int id);
  Future<User> createUser(User user);
  Future<User> updateUser(User user);
  Future<bool> deleteUser(int id);
  Future<User> updateUserAvatar(int id, File avatarFile);
}

// リポジトリ実装
class ApiUserRepository implements UserRepository {
  final ApiClient _apiClient;
  final Logger _logger;
  
  ApiUserRepository({
    required ApiClient apiClient,
    Logger? logger,
  }) : _apiClient = apiClient,
       _logger = logger ?? Logger();
  
  @override
  Future<List<User>> getUsers({int page = 1, int limit = 20}) async {
    try {
      final response = await _apiClient.get(
        'users',
        queryParams: {'page': page, 'limit': limit},
      );
      
      final List usersJson = response['data'] as List;
      return usersJson.map((json) => User.fromJson(json)).toList();
    } catch (e) {
      _logger.error('ユーザー一覧の取得に失敗', e);
      rethrow;
    }
  }
  
  @override
  Future<User> getUserById(int id) async {
    try {
      final response = await _apiClient.get('users/$id');
      return User.fromJson(response['data']);
    } catch (e) {
      _logger.error('ユーザー(ID: $id)の取得に失敗', e);
      rethrow;
    }
  }
  
  // 他のメソッド実装
}
```

### 詳細なエラーハンドリング

```dart
// 結果を表すジェネリッククラス（詳細版）
class Result<T> {
  final T? data;
  final ErrorType? errorType;
  final String? errorMessage;
  final int? errorCode;
  final bool isSuccess;
  final StackTrace? stackTrace;
  final DateTime timestamp;
  
  Result._({
    this.data, 
    this.errorType, 
    this.errorMessage,
    this.errorCode,
    required this.isSuccess,
    this.stackTrace,
    DateTime? timestamp,
  }) : timestamp = timestamp ?? DateTime.now();
  
  factory Result.success(T data) {
    return Result._(data: data, isSuccess: true);
  }
  
  factory Result.failure(
    ErrorType type, 
    String message, {
    int? code,
    StackTrace? stackTrace,
  }) {
    return Result._(
      errorType: type, 
      errorMessage: message,
      errorCode: code,
      isSuccess: false,
      stackTrace: stackTrace,
    );
  }
  
  // ユーティリティメソッド
  bool get isFailure => !isSuccess;
  
  bool get isNetworkError => 
      errorType == ErrorType.network || 
      errorType == ErrorType.timeout;
  
  R when<R>({
    required R Function(T data) success,
    required R Function(ErrorType type, String message) failure,
  }) {
    if (isSuccess) {
      return success(data as T);
    } else {
      return failure(errorType!, errorMessage!);
    }
  }
}

enum ErrorType {
  network,
  timeout,
  serverError,
  unauthorized,
  forbidden,
  notFound,
  validation,
  conflict,
  unknown,
}

// APIエラーからResultへの変換
Result<T> _handleApiException<T>(ApiException e) {
  switch (e.code) {
    case 0:
      return Result.failure(ErrorType.network, 'ネットワーク接続エラー');
    case 401:
      return Result.failure(ErrorType.unauthorized, '認証が必要です');
    case 403:
      return Result.failure(ErrorType.forbidden, 'アクセス権限がありません');
    case 404:
      return Result.failure(ErrorType.notFound, 'リソースが見つかりません');
    case 408:
      return Result.failure(ErrorType.timeout, 'リクエストがタイムアウトしました');
    case 409:
      return Result.failure(ErrorType.conflict, 'データの競合が発生しました');
    case 422:
      return Result.failure(ErrorType.validation, '入力データが無効です: ${e.message}');
    default:
      if (e.code >= 500) {
        return Result.failure(ErrorType.serverError, 'サーバーエラーが発生しました', code: e.code);
      }
      return Result.failure(ErrorType.unknown, e.message, code: e.code);
  }
}
```

## 6. API認証詳細

| 認証方式 | 説明 | 実装 | セキュリティレベル |
|------|------|------|------|
| API Key | 単一のキーを使用 | ヘッダ/クエリパラメータ | 低～中 |
| Basic Auth | ユーザー名:パスワードをBase64エンコード | Authorizationヘッダ | 低（HTTPS必須） |
| Bearer Token | トークンをヘッダに添付 | `Authorization: Bearer {token}` | 中～高 |
| JWT | 署名付きJSONトークン | Bearer Tokenとして送信 | 高 |
| OAuth 2.0 | 認可フレームワーク、アクセストークン発行 | 複雑なフロー、外部サービス認証 | 最高 |
| API Key + Secret | キーとシークレットの組み合わせ | リクエスト署名、HMAC生成 | 高 |

### トークン管理の実装例

```dart
class TokenManager {
  final SecureStorage _storage;
  
  String? _accessToken;
  String? _refreshToken;
  DateTime? _expiryTime;
  
  TokenManager({required SecureStorage storage}) : _storage = storage;
  
  // 初期化
  Future<void> initialize() async {
    _accessToken = await _storage.read('access_token');
    _refreshToken = await _storage.read('refresh_token');
    final expiry = await _storage.read('token_expiry');
    _expiryTime = expiry != null ? DateTime.parse(expiry) : null;
  }
  
  // トークン取得
  Future<String?> getAccessToken() async {
    // トークン有効期限チェック
    if (_isTokenExpired()) {
      // リフレッシュが必要
      final success = await refreshTokens();
      if (!success) return null;
    }
    return _accessToken;
  }
  
  // トークン保存
  Future<void> setTokens({
    required String accessToken,
    required String refreshToken,
    required Duration expiresIn,
  }) async {
    _accessToken = accessToken;
    _refreshToken = refreshToken;
    _expiryTime = DateTime.now().add(expiresIn);
    
    await _storage.write('access_token', accessToken);
    await _storage.write('refresh_token', refreshToken);
    await _storage.write('token_expiry', _expiryTime!.toIso8601String());
  }
  
  // トークンリフレッシュ
  Future<bool> refreshTokens() async {
    if (_refreshToken == null) return false;
    
    try {
      // APIでトークンリフレッシュ
      final response = await _refreshTokenFromApi(_refreshToken!);
      
      // 新トークン保存
      await setTokens(
        accessToken: response['access_token'],
        refreshToken: response['refresh_token'],
        expiresIn: Duration(seconds: response['expires_in']),
      );
      
      return true;
    } catch (e) {
      // リフレッシュ失敗時はクリア
      await clearTokens();
      return false;
    }
  }
  
  // トークンクリア（ログアウト時など）
  Future<void> clearTokens() async {
    _accessToken = null;
    _refreshToken = null;
    _expiryTime = null;
    
    await _storage.delete('access_token');
    await _storage.delete('refresh_token');
    await _storage.delete('token_expiry');
  }
  
  // トークン有効期限チェック
  bool _isTokenExpired() {
    if (_accessToken == null || _expiryTime == null) return true;
    
    // 期限の10分前を有効期限切れとする（マージン）
    final now = DateTime.now();
    return now.isAfter(_expiryTime!.subtract(Duration(minutes: 10)));
  }
  
  // 実際のAPIリクエスト
  Future<Map<String, dynamic>> _refreshTokenFromApi(String refreshToken) async {
    // APIリクエスト実装
  }
}
```

## 7. API統合のデザインパターン

| パターン | 説明 | 用途 |
|------|------|------|
| リポジトリ | データアクセスを抽象化 | ビジネスロジックと通信の分離 |
| Service Locator | 依存関係を中央管理 | サービスインスタンス管理 |
| Factory | オブジェクト生成を抽象化 | API呼び出し処理の構築 |
| アダプター | インターフェース変換 | 異なるAPIを統一形式で利用 |
| プロキシ | オブジェクトアクセス制御 | キャッシュ、認証、ロギング |
| Command | 処理をオブジェクト化 | 非同期処理、リトライ、キュー |
| Strategy | アルゴリズム切り替え | 認証方式、シリアライズ方式選択 |
| Builder | 複雑オブジェクトの構築 | リクエスト構築の簡素化 |

### DIを使用した実装例

```dart
// 依存性注入（DI）を使用した例
class ApiModule {
  static ApiClient provideApiClient() {
    return HttpApiClient(
      baseUrl: Config.API_BASE_URL,
      timeout: Duration(seconds: 20),
      defaultHeaders: {
        'X-API-Key': Config.API_KEY,
        'Content-Type': 'application/json',
      },
    );
  }
  
  static TokenManager provideTokenManager() {
    return TokenManager(
      storage: SecureStorageImpl(),
    );
  }
  
  static AuthService provideAuthService(
    ApiClient apiClient,
    TokenManager tokenManager,
  ) {
    return AuthServiceImpl(
      apiClient: apiClient,
      tokenManager: tokenManager,
    );
  }
  
  static UserRepository provideUserRepository(
    ApiClient apiClient,
    TokenManager tokenManager,
  ) {
    return ApiUserRepository(
      apiClient: AuthenticatedApiClient(
        delegate: apiClient,
        tokenManager: tokenManager,
      ),
    );
  }
}

// RiverpodでのDI設定例
final apiClientProvider = Provider<ApiClient>((ref) {
  return ApiModule.provideApiClient();
});

final tokenManagerProvider = Provider<TokenManager>((ref) {
  return ApiModule.provideTokenManager();
});

final authServiceProvider = Provider<AuthService>((ref) {
  final apiClient = ref.watch(apiClientProvider);
  final tokenManager = ref.watch(tokenManagerProvider);
  return ApiModule.provideAuthService(apiClient, tokenManager);
});

final userRepositoryProvider = Provider<UserRepository>((ref) {
  final apiClient = ref.watch(apiClientProvider);
  final tokenManager = ref.watch(tokenManagerProvider);
  return ApiModule.provideUserRepository(apiClient, tokenManager);
});
```

## 8. 詳細なAPI実装パターン（続き）

### ページネーション処理

```dart
class PaginatedResponse<T> {
  final List<T> data;
  final int currentPage;
  final int totalPages;
  final int totalItems;
  final int itemsPerPage;
  final bool hasNextPage;
  final bool hasPreviousPage;
  final String? nextPageUrl;
  final String? previousPageUrl;
  
  PaginatedResponse({
    required this.data,
    required this.currentPage,
    required this.totalPages,
    required this.totalItems,
    required this.itemsPerPage,
    this.nextPageUrl,
    this.previousPageUrl,
  }) : 
    hasNextPage = currentPage < totalPages,
    hasPreviousPage = currentPage > 1;
  
  factory PaginatedResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromJson
  ) {
    final List<dynamic> items = json['data'] ?? [];
    return PaginatedResponse(
      data: items.map((item) => fromJson(item)).toList(),
      currentPage: json['meta']['current_page'] ?? 1,
      totalPages: json['meta']['total_pages'] ?? 1,
      totalItems: json['meta']['total_items'] ?? items.length,
      itemsPerPage: json['meta']['per_page'] ?? items.length,
      nextPageUrl: json['links']['next'],
      previousPageUrl: json['links']['previous'],
    );
  }
}

// リポジトリでの使用例
Future<PaginatedResponse<User>> getUsers({int page = 1, int limit = 20}) async {
  try {
    final response = await _apiClient.get(
      'users',
      queryParams: {'page': page, 'limit': limit},
    );
    
    return PaginatedResponse.fromJson(
      response,
      (json) => User.fromJson(json),
    );
  } catch (e) {
    _logger.error('ユーザー一覧の取得に失敗', e);
    rethrow;
  }
}
```

### キャッシュ実装

```dart
class CachedApiClient implements ApiClient {
  final ApiClient _delegate;
  final ApiCache _cache;
  
  CachedApiClient({
    required ApiClient delegate,
    required ApiCache cache,
  }) : _delegate = delegate,
       _cache = cache;
  
  @override
  Future<Map<String, dynamic>> get(
    String endpoint, {
    Map<String, String>? headers,
    Map<String, dynamic>? queryParams,
  }) async {
    final cacheKey = _generateCacheKey(endpoint, queryParams);
    
    // キャッシュチェック
    final cachedData = await _cache.get(cacheKey);
    if (cachedData != null) {
      return cachedData;
    }
    
    // APIから取得
    final response = await _delegate.get(
      endpoint,
      headers: headers,
      queryParams: queryParams,
    );
    
    // キャッシュ保存
    await _cache.set(cacheKey, response);
    
    return response;
  }
  
  // その他のメソッド実装
  
  String _generateCacheKey(String endpoint, Map<String, dynamic>? params) {
    if (params == null || params.isEmpty) {
      return endpoint;
    }
    
    final sortedParams = SplayTreeMap<String, dynamic>.from(params);
    final queryString = sortedParams.entries
        .map((e) => '${e.key}=${e.value}')
        .join('&');
    
    return '$endpoint?$queryString';
  }
}

// キャッシュインターフェース
abstract class ApiCache {
  Future<Map<String, dynamic>?> get(String key);
  Future<void> set(String key, Map<String, dynamic> data, {Duration? ttl});
  Future<void> remove(String key);
  Future<void> clear();
}

// キャッシュ実装（例：SharedPreferences使用）
class SharedPrefsApiCache implements ApiCache {
  final SharedPreferences _prefs;
  final String _prefix;
  
  SharedPrefsApiCache({
    required SharedPreferences prefs,
    String prefix = 'api_cache_',
  }) : _prefs = prefs,
       _prefix = prefix;
  
  @override
  Future<Map<String, dynamic>?> get(String key) async {
    final fullKey = _prefix + key;
    final data = _prefs.getString(fullKey);
    if (data == null) return null;
    
    final cacheItem = jsonDecode(data);
    
    // 有効期限チェック
    final expiry = cacheItem['expiry'] as int?;
    if (expiry != null && expiry < DateTime.now().millisecondsSinceEpoch) {
      // 期限切れ
      await remove(key);
      return null;
    }
    
    return cacheItem['data'];
  }
  
  @override
  Future<void> set(
    String key,
    Map<String, dynamic> data, {
    Duration? ttl,
  }) async {
    final fullKey = _prefix + key;
    
    final cacheItem = {
      'data': data,
      'timestamp': DateTime.now().millisecondsSinceEpoch,
    };
    
    if (ttl != null) {
      cacheItem['expiry'] = DateTime.now()
          .add(ttl)
          .millisecondsSinceEpoch;
    }
    
    await _prefs.setString(fullKey, jsonEncode(cacheItem));
  }
  
  @override
  Future<void> remove(String key) async {
    final fullKey = _prefix + key;
    await _prefs.remove(fullKey);
  }
  
  @override
  Future<void> clear() async {
    final keys = _prefs.getKeys().where((key) => key.startsWith(_prefix));
    for (final key in keys) {
      await _prefs.remove(key);
    }
  }
}
```

### バッチリクエスト

```dart
Future<List<User>> getUsersBatch(List<int> userIds) async {
  if (userIds.isEmpty) return [];
  
  // バッチサイズ定義
  const int batchSize = 20;
  final results = <User>[];
  
  // リクエストのバッチ処理
  for (int i = 0; i < userIds.length; i += batchSize) {
    final end = (i + batchSize < userIds.length) 
        ? i + batchSize 
        : userIds.length;
    final batchIds = userIds.sublist(i, end);
    
    // バッチリクエスト
    try {
      final ids = batchIds.join(',');
      final response = await _apiClient.get(
        'users',
        queryParams: {'ids': ids},
      );
      
      final List<dynamic> batchData = response['data'];
      final batchUsers = batchData
          .map((userData) => User.fromJson(userData))
          .toList();
      
      results.addAll(batchUsers);
    } catch (e) {
      _logger.error('バッチユーザー取得エラー', e);
      // エラー発生時も処理継続
    }
    
    // バッチ間隔を設ける（APIレート制限対策）
    if (end < userIds.length) {
      await Future.delayed(Duration(milliseconds: 200));
    }
  }
  
  return results;
}
```

### リトライメカニズム

```dart
Future<T> withRetry<T>({
  required Future<T> Function() operation,
  int maxAttempts = 3,
  Duration initialDelay = const Duration(milliseconds: 500),
  double backoffFactor = 2.0,
  List<int> retryStatusCodes = const [408, 429, 500, 502, 503, 504],
  List<Type> retryExceptions = const [TimeoutException, SocketException],
}) async {
  Duration delay = initialDelay;
  
  for (int attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (e) {
      // 最終試行でエラーの場合は再スロー
      if (attempt == maxAttempts) rethrow;
      
      bool shouldRetry = false;
      
      // APIエラー判定
      if (e is ApiException) {
        shouldRetry = retryStatusCodes.contains(e.code);
      } else {
        // 一般例外判定
        for (final type in retryExceptions) {
          if (e.runtimeType == type) {
            shouldRetry = true;
            break;
          }
        }
      }
      
      if (!shouldRetry) rethrow;
      
      // リトライ前に待機（指数バックオフ）
      await Future.delayed(delay);
      delay = delay * backoffFactor;
      
      _logger.warn('リトライ実行 ($attempt/$maxAttempts)', e);
    }
  }
  
  // コンパイルエラー回避のためのダミー（実際には実行されない）
  throw StateError('unexpected code path');
}

// 使用例
Future<User> getUserById(int id) async {
  return withRetry(
    operation: () async {
      final response = await _apiClient.get('users/$id');
      return User.fromJson(response['data']);
    },
    maxAttempts: 3,
    retryStatusCodes: [429, 500, 503],
  );
}
```

## 9. API統合のセキュリティ

| セキュリティ対策 | 説明 | 実装方法 |
|------|------|------|
| HTTPS使用 | 暗号化通信 | SSL証明書、SSL Pinning |
| 入力検証 | データの妥当性確認 | バリデーション、サニタイズ |
| トークン管理 | 認証情報の安全な扱い | 安全なストレージ、有効期限 |
| リクエスト署名 | リクエスト改ざん防止 | HMACによる署名付与 |
| レート制限 | DoS攻撃対策 | リトライ制御、バックオフ |
| SSL Pinning | 中間者攻撃防止 | 証明書検証、証明書固定 |
| 機密データ保護 | 個人情報などの保護 | 暗号化、最小権限 |

### SSL Pinning実装

```dart
import 'package:http/http.dart' as http;
import 'package:http/io_client.dart';
import 'dart:io';

class SecureHttpClient extends http.BaseClient {
  final HttpClient _client;
  final http.Client _inner;
  
  SecureHttpClient._(this._client, this._inner);
  
  static Future<SecureHttpClient> create() async {
    final client = HttpClient()
      ..badCertificateCallback = _validateCertificate;
    
    return SecureHttpClient._(client, IOClient(client));
  }
  
  @override
  Future<http.StreamedResponse> send(http.BaseRequest request) {
    return _inner.send(request);
  }
  
  @override
  void close() {
    _inner.close();
    _client.close();
  }
  
  // 許可された証明書のSHA-256フィンガープリント
  static const _validCertificateFingerprints = [
    '5B:CB:33:EC:F9:82:A5:A0:5B:17:E3:7B:D3:7D:22:48:6A:36:3D:41:97:AA:46:06:41:2C:A4:B9:59:81:1E:30',
    // バックアップや開発環境用の指紋を追加
  ];
  
  // 証明書検証
  static bool _validateCertificate(X509Certificate cert, String host, int port) {
    final fingerprint = _getFingerprint(cert);
    return _validCertificateFingerprints.contains(fingerprint);
  }
  
  // 証明書のフィンガープリントを取得
  static String _getFingerprint(X509Certificate cert) {
    // 実際の実装ではSHA-256ハッシュ生成
    // この例では簡易的に証明書データを使用
    return cert.sha1.toUpperCase();
  }
}

// 使用例
void initApiClient() async {
  final secureClient = await SecureHttpClient.create();
  final apiClient = HttpApiClient(
    baseUrl: 'https://api.example.com',
    client: secureClient,
  );
  
  // apiClientを使用
}
```

## 10. API統合のテスト戦略

| テスト種類 | 目的 | 実装方法 | ツール |
|------|------|------|------|
| 単体テスト | クラス/メソッド単位でテスト | モック使用、依存性分離 | mockito, mocktail |
| 統合テスト | 複数コンポーネントの連携テスト | テスト用API、偽物実装 | fake_api_client |
| エンドツーエンドテスト | 実際のAPIとの通信テスト | テスト専用環境 | flutter_driver |
| 契約テスト | API仕様との整合性確認 | スキーマ検証、自動化 | pact |
| マニュアルテスト | エッジケースの検証 | 手動操作、例外条件確認 | - |

### モックを使用した単体テスト

```dart
import 'package:mocktail/mocktail.dart';
import 'package:test/test.dart';

class MockApiClient extends Mock implements ApiClient {}
class MockTokenManager extends Mock implements TokenManager {}

void main() {
  group('ApiUserRepository', () {
    late MockApiClient mockApiClient;
    late MockTokenManager mockTokenManager;
    late ApiUserRepository repository;
    
    setUp(() {
      mockApiClient = MockApiClient();
      mockTokenManager = MockTokenManager();
      repository = ApiUserRepository(
        apiClient: mockApiClient,
        tokenManager: mockTokenManager,
      );
    });
    
    test('getUsers は正しいユーザーリストを返すこと', () async {
      // モックの設定
      when(() => mockApiClient.get(
        'users',
        queryParams: any(named: 'queryParams'),
        headers: any(named: 'headers'),
      )).thenAnswer((_) async => {
        'data': [
          {
            'id': 1,
            'name': 'Test User',
            'email': 'test@example.com',
            'created_at': '2023-01-01T00:00:00Z',
            'roles': ['user'],
            'metadata': {},
          }
        ],
        'meta': {
          'current_page': 1,
          'total_pages': 1,
          'total_items': 1,
          'per_page': 20,
        }
      });
      
      // テスト実行
      final result = await repository.getUsers();
      
      // 検証
      expect(result.length, 1);
      expect(result[0].id, 1);
      expect(result[0].name, 'Test User');
      expect(result[0].email, 'test@example.com');
      
      // モックの呼び出し検証
      verify(() => mockApiClient.get(
        'users',
        queryParams: {'page': 1, 'limit': 20},
        headers: any(named: 'headers'),
      )).called(1);
    });
    
    test('APIエラー時は適切な例外がスローされること', () async {
      // モックの設定
      when(() => mockApiClient.get(
        'users',
        queryParams: any(named: 'queryParams'),
        headers: any(named: 'headers'),
      )).thenThrow(ApiException(
        code: 401,
        message: '認証エラー',
        endpoint: 'users',
      ));
      
      // テスト実行と検証
      expect(
        () => repository.getUsers(),
        throwsA(isA<ApiException>().having(
          (e) => e.code,
          'code',
          401,
        )),
      );
    });
  });
}
```

## 11. モバイル固有のAPI連携注意点

| 項目 | 説明 | 対策 |
|------|------|------|
| バッテリー消費 | API通信によるバッテリー消費 | バッチ処理、キャッシュ |
| データ使用量 | モバイルデータの節約 | 圧縮、必要最小限データ |
| オフライン対応 | ネットワーク非接続時の動作 | オフラインキャッシュ、同期機能 |
| バックグラウンド処理 | バックグラウンドでの通信 | バックグラウンドサービス、通知 |
| 接続状態管理 | ネットワーク変化への対応 | 接続監視、自動リトライ |

### ネットワーク状態監視

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

class NetworkAwareApiClient implements ApiClient {
  final ApiClient _delegate;
  final Connectivity _connectivity;
  final StreamController<bool> _connectionStatusController;
  
  Stream<bool> get connectionStatus => _connectionStatusController.stream;
  bool _isConnected = true;
  
  NetworkAwareApiClient({
    required ApiClient delegate,
    Connectivity? connectivity,
  }) : 
    _delegate = delegate,
    _connectivity = connectivity ?? Connectivity(),
    _connectionStatusController = StreamController<bool>.broadcast() {
    _initConnectivityListener();
  }
  
  Future<void> _initConnectivityListener() async {
    // 初期状態確認
    _isConnected = await _checkConnection();
    _connectionStatusController.add(_isConnected);
    
    // 接続状態変化リスナー
    _connectivity.onConnectivityChanged.listen((result) async {
      final isConnected = result != ConnectivityResult.none;
      if (_isConnected != isConnected) {
        _isConnected = isConnected;
        _connectionStatusController.add(_isConnected);
      }
    });
  }
  
  Future<bool> _checkConnection() async {
    final result = await _connectivity.checkConnectivity();
    return result != ConnectivityResult.none;
  }
  
  @override
  Future<Map<String, dynamic>> get(
    String endpoint, {
    Map<String, String>? headers,
    Map<String, dynamic>? queryParams,
  }) async {
    if (!_isConnected) {
      throw ApiException(
        code: 0,
        message: 'ネットワーク接続がありません',
        endpoint: endpoint,
      );
    }
    
    return _delegate.get(
      endpoint,
      headers: headers,
      queryParams: queryParams,
    );
  }
  
  // その他メソッド実装
  
  void dispose() {
    _connectionStatusController.close();
  }
}
```

## 12. APIドキュメント活用と解析

| ドキュメント形式 | 特徴 | ツール |
|------|------|------|
| OpenAPI (Swagger) | REST API標準仕様形式 | Swagger UI, Swagger Codegen |
| GraphQL Schema | GraphQL型定義 | GraphQL Playground, GraphiQL |
| API Blueprint | マークダウンベース | Apiary, Aglio |
| RAML | YAML形式 | API Console, RAML Tools |
| 自動生成ドキュメント | コードからの生成 | Dartdoc, Compodoc |

### OpenAPI定義からのコード生成

```yaml
# openapi.yaml (Swagger/OpenAPI 3.0)
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: ユーザー一覧を取得
      parameters:
        - name: page
          in: query
          schema:
            type: integer
        - name: limit
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
      required:
        - id
        - name
        - email
```

```dart
// 生成されたコードの例（OpenAPI Generator使用）
// user_api.dart
class UserApi {
  final ApiClient apiClient;
  
  UserApi({required this.apiClient});
  
  /// ユーザー一覧を取得
  ///
  /// パラメータ:
  /// * [page] 
  /// * [limit] 
  Future<UserList> getUsers({ int? page, int? limit }) async {
    // 実装
  }
}

// models/user.dart
class User {
  final int id;
  final String name;
  final String email;
  
  User({
    required this.id,
    required this.name,
    required this.email,
  });
  
  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }
}
```

## 13. 高度なAPI統合例

### フォールバック機能

```dart
class ResilientApiClient implements ApiClient {
  final List<ApiClient> _clients;
  final Logger _logger;
  
  ResilientApiClient({
    required List<ApiClient> clients,
    Logger? logger,
  }) : 
    _clients = List.from(clients),
    _logger = logger ?? Logger();
  
  @override
  Future<Map<String, dynamic>> get(
    String endpoint, {
    Map<String, String>? headers,
    Map<String, dynamic>? queryParams,
  }) async {
    ApiException? lastException;
    
    // 順番に各クライアントを試行
    for (final client in _clients) {
      try {
        return await client.get(
          endpoint,
          headers: headers,
          queryParams: queryParams,
        );
      } catch (e) {
        if (e is ApiException) {
          lastException = e;
          _logger.warn('API呼び出し失敗、フォールバック試行', e);
        } else {
          rethrow;
        }
      }
    }
    
    // すべてのクライアントが失敗した場合
    throw lastException ?? ApiException(
      code: -1,
      message: '利用可能なAPIクライアントがありません',
      endpoint: endpoint,
    );
  }
  
  // その他メソッド実装
}
```

### 依存関係フリーなAPI設計

```dart
// プラットフォーム非依存の抽象化レイヤー
abstract class HttpClient {
  Future<HttpResponse> get(String url, {Map<String, String>? headers});
  Future<HttpResponse> post(String url, {Map<String, String>? headers, dynamic body});
  Future<HttpResponse> put(String url, {Map<String, String>? headers, dynamic body});
  Future<HttpResponse> delete(String url, {Map<String, String>? headers});
  void close();
}

class HttpResponse {
  final int statusCode;
  final Map<String, String> headers;
  final dynamic body;
  
  HttpResponse({
    required this.statusCode,
    required this.headers,
    required this.body,
  });
}

// Dart httpパッケージの実装
class DartHttpClient implements HttpClient {
  final http.Client _client;
  
  DartHttpClient({http.Client? client}) : _client = client ?? http.Client();
  
  @override
  Future<HttpResponse> get(String url, {Map<String, String>? headers}) async {
    final response = await _client.get(Uri.parse(url), headers: headers);
    return _convertResponse(response);
  }
  
  // その他メソッド実装
  
  HttpResponse _convertResponse(http.Response response) {
    return HttpResponse(
      statusCode: response.statusCode,
      headers: response.headers,
      body: response.body,
    );
  }
  
  @override
  void close() {
    _client.close();
  }
}

// Dio実装（別のHTTPクライアント）
class DioHttpClient implements HttpClient {
  final Dio _dio;
  
  DioHttpClient({Dio? dio}) : _dio = dio ?? Dio();
  
  @override
  Future<HttpResponse> get(String url, {Map<String, String>? headers}) async {
    final response = await _dio.get(url, options: Options(headers: headers));
    return _convertResponse(response);
  }
  
  // その他メソッド実装
  
  @override
  void close() {
    _dio.close();
  }
}

// API実装
class PlatformAgnosticApiClient implements ApiClient {
  final HttpClient _httpClient;
  final String _baseUrl;
  
  PlatformAgnosticApiClient({
    required HttpClient httpClient,
    required String baseUrl,
  }) : 
    _httpClient = httpClient,
    _baseUrl = baseUrl;
  
  @override
  Future<Map<String, dynamic>> get(
    String endpoint, {
    Map<String, String>? headers,
    Map<String, dynamic>? queryParams,
  }) async {
    // 実装
  }
  
  // その他メソッド
}
```

## 14. まとめと注意点

| カテゴリ | ベストプラクティス |
|------|------|
| アーキテクチャ | レイヤー分離、抽象化、依存性逆転 |
| パフォーマンス | キャッシュ、バッチ処理、並行リクエスト |
| エラーハンドリング | 一貫した例外、リトライ、ユーザーフィードバック |
| セキュリティ | HTTPS、トークン管理、SSL Pinning |
| テスト | モック利用、ユニットテスト、E2Eテスト |
| モバイル最適化 | バッテリー・データ消費考慮、オフライン対応 |
| ドキュメント | OpenAPI活用、自動生成、スキーマ検証 |
| フォールバック | 代替手段、デグレード操作、耐障害性 |

API統合は、モダンなFlutterアプリケーション開発の中核となるスキルで、適切な実装は堅牢なアプリケーションの基盤となります。単なるデータ取得だけでなく、ユーザー体験、パフォーマンス、セキュリティに大きく影響するため、適切な設計と実装が求められます。