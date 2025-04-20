# Firestore の基本知識メモ

## 1. Firestoreの主要特徴比較

| 特徴 | 説明 | 利点 |
|------|------|------|
| **柔軟なデータモデル** | スキーマレスなNoSQLデータベース | データ構造の事前定義不要、変更に強い |
| **リアルタイム更新** | データ変更をリアルタイムで検知・同期 | チャットやリアルタイム機能の実装が容易 |
| **オフラインサポート** | ネットワーク未接続時も動作 | 接続が不安定な環境でも一貫したUX提供 |
| **スケーラビリティ** | 自動的にスケール | トラフィック増加に対応、インフラ管理不要 |
| **セキュリティルール** | 宣言的なルール言語で細かなアクセス制御 | クライアント側から直接安全なアクセスが可能 |

## 2. Firestoreのデータモデル

| 構成要素 | 説明 | SQLデータベースとの比較 |
|---------|------|------------------------|
| **コレクション** | ドキュメントのコンテナ | テーブルに相当 |
| **ドキュメント** | JSONライクなデータオブジェクト | レコード/行に相当 |
| **フィールド** | キーと値のペア | カラムと値に相当 |
| **サブコレクション** | ドキュメント内のコレクション | 外部キー関連付けに近いが階層的 |

### データ構造例

```
コレクション（users）
  └── ドキュメント（user_id_1）
        ├── フィールド（name: "鈴木太郎"）
        ├── フィールド（age: 28）
        └── サブコレクション（posts）
              └── ドキュメント（post_id_1）
                    ├── フィールド（title: "こんにちは"）
                    └── フィールド（content: "はじめての投稿です"）
```

### データ型サポート

- 文字列、数値、ブール値
- 配列、マップ（ネスト可能）
- タイムスタンプ、日付
- 地理的位置情報（GeoPoint）
- ドキュメント参照
- バイナリデータ（Blob）

## 3. スキーマレスの概念

| 特性 | リレーショナルDB | Firestore (NoSQL) |
|------|-----------------|-------------------|
| **スキーマ定義** | 事前に厳格に定義必須 | 不要、柔軟に追加可能 |
| **構造の一貫性** | 同じテーブル内で統一 | ドキュメントごとに異なる構造可 |
| **フィールド追加** | ALTER TABLE文が必要 | 単純に新フィールドを追加可能 |
| **データ整合性** | DB側で強制 | アプリケーション側で担保 |
| **NULL制約** | 厳格に設定可能 | デフォルトでは緩やか |

### スキーマレスとスキーマあり比較コード例

**リレーショナルDB (スキーマあり):**
```sql
-- テーブル定義が必要
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL,
  age INT
);

-- 新規ユーザー追加
INSERT INTO users (id, name, email, age) VALUES (1, '山田太郎', 'yamada@example.com', 30);

-- 必須フィールドが欠けているとエラー
INSERT INTO users (id, name) VALUES (2, '鈴木花子'); -- エラー：emailは必須

-- 新フィールド追加にはスキーマ変更が必要
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
```

**Firestore (スキーマレス):**
```dart
// ユーザー1：標準的なフィールドセット
FirebaseFirestore.instance.collection('users').doc('user1').set({
  'name': '山田太郎',
  'email': 'yamada@example.com',
  'age': 30
});

// ユーザー2：異なるフィールドセット (emailなし、phoneあり)
FirebaseFirestore.instance.collection('users').doc('user2').set({
  'name': '鈴木花子',
  'phone': '090-1234-5678',
  'isPremium': true
});

// 後から新しいフィールドを追加 (スキーマ変更不要)
FirebaseFirestore.instance.collection('users').doc('user1').update({
  'address': '東京都渋谷区',
  'favoriteColors': ['青', '緑']
});
```

### スキーマレスのユースケース

- プロトタイプ開発
- 頻繁に変更されるデータモデル
- ユーザーごとに異なる設定を持つアプリ
- コンテンツ管理システム

## 4. Firestoreの基本操作

| 操作 | 説明 | コード例（概略） |
|------|------|----------------|
| **単一ドキュメント取得** | 特定IDのドキュメント読み取り | `collection('users').doc(userId).get()` |
| **コレクション取得** | コレクション内全ドキュメント取得 | `collection('users').get()` |
| **クエリとフィルタ** | 条件によるドキュメント検索 | `collection('users').where('age', '>=', 20).get()` |
| **リアルタイムリスナー** | データ変更検知用のリスナー設定 | `collection('users').snapshots().listen(...)` |
| **ドキュメント作成** | 新規ドキュメント作成 | `collection('users').add(data)` |
| **ドキュメント更新** | 既存ドキュメント更新 | `doc(userId).update({'name': newName})` |
| **ドキュメント削除** | ドキュメント削除 | `doc(userId).delete()` |

### コード例: データの読み取り

```dart
// 単一ドキュメントの取得
Future<void> getUser(String userId) async {
  DocumentSnapshot snapshot = await FirebaseFirestore.instance
      .collection('users')
      .doc(userId)
      .get();
  
  if (snapshot.exists) {
    Map<String, dynamic> data = snapshot.data() as Map<String, dynamic>;
    print('User name: ${data['name']}');
  } else {
    print('ユーザーが存在しません');
  }
}

// コレクション内のすべてのドキュメントを取得
Future<void> getAllUsers() async {
  QuerySnapshot snapshot = await FirebaseFirestore.instance
      .collection('users')
      .get();
  
  for (var doc in snapshot.docs) {
    Map<String, dynamic> data = doc.data() as Map<String, dynamic>;
    print('User ID: ${doc.id}, name: ${data['name']}');
  }
}

// クエリとフィルタリング
Future<void> getAdultUsers() async {
  QuerySnapshot snapshot = await FirebaseFirestore.instance
      .collection('users')
      .where('age', isGreaterThanOrEqualTo: 20)
      .orderBy('age', descending: true)
      .limit(10)
      .get();
  
  for (var doc in snapshot.docs) {
    Map<String, dynamic> data = doc.data() as Map<String, dynamic>;
    print('User: ${data['name']}, age: ${data['age']}');
  }
}
```

### コード例: リアルタイムアップデート

```dart
// ドキュメントのリアルタイムリスナー
void listenToUserChanges(String userId) {
  FirebaseFirestore.instance
      .collection('users')
      .doc(userId)
      .snapshots()
      .listen((snapshot) {
    if (snapshot.exists) {
      Map<String, dynamic> data = snapshot.data() as Map<String, dynamic>;
      print('User updated: ${data['name']}');
    } else {
      print('ユーザーが削除されました');
    }
  });
}

// コレクションへのリアルタイムリスナー
void listenToAllUsers() {
  FirebaseFirestore.instance
      .collection('users')
      .snapshots()
      .listen((snapshot) {
    for (var change in snapshot.docChanges) {
      Map<String, dynamic> data = change.doc.data() as Map<String, dynamic>;
      
      switch (change.type) {
        case DocumentChangeType.added:
          print('新しいユーザー: ${data['name']}');
          break;
        case DocumentChangeType.modified:
          print('ユーザー更新: ${data['name']}');
          break;
        case DocumentChangeType.removed:
          print('ユーザー削除: ${data['name']}');
          break;
      }
    }
  });
}
```

### コード例: データの書き込み

```dart
// 新しいドキュメントを作成（IDを自動生成）
Future<void> addUser(String name, int age) async {
  await FirebaseFirestore.instance.collection('users').add({
    'name': name,
    'age': age,
    'createdAt': FieldValue.serverTimestamp(), // サーバー側のタイムスタンプ
  });
}

// 特定のIDでドキュメントを作成
Future<void> createUser(String userId, String name, int age) async {
  await FirebaseFirestore.instance.collection('users').doc(userId).set({
    'name': name,
    'age': age,
    'createdAt': FieldValue.serverTimestamp(),
  });
}

// ドキュメント全体を更新
Future<void> updateUser(String userId, String name, int age) async {
  await FirebaseFirestore.instance.collection('users').doc(userId).set({
    'name': name,
    'age': age,
    'updatedAt': FieldValue.serverTimestamp(),
  });
}

// 特定のフィールドのみ更新
Future<void> updateUserAge(String userId, int age) async {
  await FirebaseFirestore.instance.collection('users').doc(userId).update({
    'age': age,
    'updatedAt': FieldValue.serverTimestamp(),
  });
}

// ドキュメントの削除
Future<void> deleteUser(String userId) async {
  await FirebaseFirestore.instance.collection('users').doc(userId).delete();
}
```

## 5. 高度な操作

| 機能 | 説明 | ユースケース |
|------|------|-------------|
| **トランザクション** | 複数操作を原子的に実行 | ポイント移動、座席予約など |
| **バッチ処理** | 複数書き込みをグループ化 | 一括更新処理 |
| **ドキュメント参照** | 他ドキュメントへの参照型 | ユーザーと投稿の関連付けなど |
| **データ非正規化** | パフォーマンス向上のためのデータ複製 | 頻繁に読み取る情報の埋め込み |

### コード例: トランザクション

```dart
// トランザクションの例：ポイント転送
Future<void> transferPoints(String fromUserId, String toUserId, int points) async {
  FirebaseFirestore firestore = FirebaseFirestore.instance;
  
  return firestore.runTransaction((transaction) async {
    // 両方のユーザードキュメントを取得
    DocumentSnapshot fromUserSnapshot = 
        await transaction.get(firestore.collection('users').doc(fromUserId));
    DocumentSnapshot toUserSnapshot = 
        await transaction.get(firestore.collection('users').doc(toUserId));
    
    // ドキュメントが存在するか確認
    if (!fromUserSnapshot.exists || !toUserSnapshot.exists) {
      throw Exception('ユーザーが存在しません');
    }
    
    // 現在のポイントを取得
    int fromPoints = (fromUserSnapshot.data() as Map<String, dynamic>)['points'] ?? 0;
    int toPoints = (toUserSnapshot.data() as Map<String, dynamic>)['points'] ?? 0;
    
    // ポイントが十分あるか確認
    if (fromPoints < points) {
      throw Exception('ポイントが不足しています');
    }
    
    // 両方のドキュメントを更新
    transaction.update(
      firestore.collection('users').doc(fromUserId), 
      {'points': fromPoints - points}
    );
    
    transaction.update(
      firestore.collection('users').doc(toUserId), 
      {'points': toPoints + points}
    );
  });
}
```

### コード例: バッチ処理

```dart
// バッチ処理の例：複数ユーザーの一括更新
Future<void> batchUpdateUsers(List<String> userIds, Map<String, dynamic> data) async {
  WriteBatch batch = FirebaseFirestore.instance.batch();
  
  for (String userId in userIds) {
    DocumentReference userRef = FirebaseFirestore.instance.collection('users').doc(userId);
    batch.update(userRef, data);
  }
  
  return batch.commit();
}
```

### コード例: ドキュメント参照とデータの非正規化

```dart
// ドキュメント参照の使用例
Future<void> createPost(String userId, String title, String content) async {
  DocumentReference userRef = FirebaseFirestore.instance.collection('users').doc(userId);
  
  await FirebaseFirestore.instance.collection('posts').add({
    'title': title,
    'content': content,
    'author': userRef,  // ドキュメント参照を保存
    'createdAt': FieldValue.serverTimestamp(),
  });
}

// データの非正規化の例
Future<void> createPostWithUserInfo(String userId, String title, String content) async {
  // まずユーザー情報を取得
  DocumentSnapshot userSnapshot = await FirebaseFirestore.instance
      .collection('users')
      .doc(userId)
      .get();
  
  Map<String, dynamic> userData = userSnapshot.data() as Map<String, dynamic>;
  
  // 投稿を作成し、必要なユーザー情報を埋め込む
  await FirebaseFirestore.instance.collection('posts').add({
    'title': title,
    'content': content,
    'authorId': userId,
    'authorName': userData['name'],  // 検索やリスト表示用に埋め込み
    'authorPhotoUrl': userData['photoUrl'],  // 表示用に埋め込み
    'createdAt': FieldValue.serverTimestamp(),
  });
}
```

## 6. FirebaseとAWSの対応関係

| Firebase サービス | AWS 相当サービス | 特徴比較 |
|-------------------|------------------|----------|
| **Cloud Firestore** | **DynamoDB** | Firestoreはドキュメント指向、DynamoDBはキーバリュー重視 |
| **Authentication** | **Cognito** | Firebaseは設定が簡単、Cognitoはカスタマイズ性が高い |
| **Cloud Functions** | **Lambda** | 同様のサーバーレス関数だがFirebaseはモバイル連携重視 |
| **Storage** | **S3** | どちらもオブジェクトストレージだがFirebaseはモバイル最適化 |
| **Hosting** | **S3 + CloudFront** | Firebaseは設定が簡単、AWSはより高度なカスタマイズ可能 |
| **Cloud Messaging** | **SNS** | FCMはモバイル特化、SNSはより広範なメッセージング |

## 7. モバイルアプリ開発でのデータベース選択

| 観点 | NoSQL (Firestore) | リレーショナルDB |
|------|-------------------|-----------------|
| **開発スピード** | 早い（スキーマレス、SDKで直接アクセス） | 比較的遅い（APIレイヤー構築が必要なケースが多い） |
| **データ整合性** | 弱い（アプリ側で担保） | 強い（ACID特性） |
| **複雑なクエリ** | 制限あり（結合操作に難あり） | 強力（複雑な結合、集計が可能） |
| **スケーラビリティ** | 高い（自動スケーリング） | 追加設定が必要なケースが多い |
| **リアルタイム性** | 組み込み機能 | 追加開発が必要 |
| **オフラインサポート** | 組み込み機能 | 追加開発が必要 |
| **学習曲線** | 比較的緩やか | 比較的急（SQL知識必要） |

### 適したユースケース

| アプリタイプ | 推奨データベース | 理由 |
|-------------|-----------------|------|
| **プロトタイプ/MVP** | **Firestore** | 速い開発サイクル、少ない初期設定 |
| **チャット/リアルタイムアプリ** | **Firestore** | リアルタイム同期、オフラインサポート |
| **金融/会計アプリ** | **リレーショナルDB** | 高いデータ整合性、トランザクション保証 |
| **データ分析重視** | **リレーショナルDB** | 複雑なクエリ、集計機能 |
| **コンテンツ管理** | **Firestore/両方** | コンテンツ種類によって異なる |

### ハイブリッドアプローチ例

```
モバイルアプリ
    │
    ├── Firestore
    │   ├── UIに表示するデータ
    │   ├── ユーザー設定
    │   ├── リアルタイム必要データ
    │   └── オフライン必要データ
    │
    └── リレーショナルDB (APIサーバー経由)
        ├── トランザクション重要データ
        ├── 複雑な関係を持つデータ
        ├── レポート/分析データ
        └── 集計処理
```

## 8. Firestoreの制限と注意点

| 制限事項 | 詳細 | 対策 |
|---------|------|------|
| **ドキュメントサイズ** | 最大1MB | 大きなデータはStorageに保存して参照 |
| **インデックス制限** | 複合クエリにはインデックスが必要 | クエリ設計の最適化 |
| **クエリの複雑さ** | 結合操作やOR条件に制限あり | データ非正規化、複数クエリの結合 |
| **コスト** | 読み書き操作回数ベースの課金 | キャッシュ活用、一括操作の利用 |
| **セキュリティ** | クライアント直接アクセスの管理 | 適切なセキュリティルール設定 |

### セキュリティルール例

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // ユーザーコレクションのルール
    match /users/{userId} {
      // ユーザー自身のドキュメントのみ読み書き可能
      allow read, write: if request.auth != null && request.auth.uid == userId;
      
      // ユーザーの投稿サブコレクションは誰でも読み取り可能、作成者のみ書き込み可能
      match /posts/{postId} {
        allow read: if true;
        allow write: if request.auth != null && request.auth.uid == userId;
      }
    }
    
    // 公開データは誰でも読み取り可能
    match /public/{document=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

## 9. Flutter+Firestoreの実践例

### リアルタイムデータ表示Widget

```dart
class UserListScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('ユーザー一覧')),
      body: StreamBuilder<QuerySnapshot>(
        stream: FirebaseFirestore.instance.collection('users').snapshots(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }
          
          if (snapshot.hasError) {
            return Center(child: Text('エラーが発生しました'));
          }
          
          if (!snapshot.hasData || snapshot.data!.docs.isEmpty) {
            return Center(child: Text('ユーザーがいません'));
          }
          
          return ListView.builder(
            itemCount: snapshot.data!.docs.length,
            itemBuilder: (context, index) {
              DocumentSnapshot doc = snapshot.data!.docs[index];
              Map<String, dynamic> data = doc.data() as Map<String, dynamic>;
              
              return ListTile(
                title: Text(data['name'] ?? 'No Name'),
                subtitle: Text('年齢: ${data['age'] ?? 'N/A'}'),
                trailing: IconButton(
                  icon: Icon(Icons.delete),
                  onPressed: () {
                    FirebaseFirestore.instance
                        .collection('users')
                        .doc(doc.id)
                        .delete();
                  },
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          // 新規ユーザー追加画面に遷移
        },
      ),
    );
  }
}
```

## 10. まとめ：プロジェクトによるデータベース選択ガイド

| 要件 | Firestore向き | リレーショナルDB向き |
|------|--------------|-------------------|
| **開発速度重視** | ✓ | |
| **リアルタイム機能必須** | ✓ | |
| **オフライン対応必須** | ✓ | |
| **厳格なデータ整合性** | | ✓ |
| **複雑な集計/レポート** | | ✓ |
| **大規模トランザクション** | | ✓ |
| **柔軟なデータ構造** | ✓ | |
| **モバイルアプリ中心** | ✓ | |
| **既存システム連携** | | ✓ |

最終的には、プロジェクトの具体的な要件、チームのスキルセット、開発期間、予算などを総合的に考慮して選択するのが望ましい。