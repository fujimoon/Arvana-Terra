# Arvana Terra - API 仕様書 / API Specification

## 概要 / Overview

Arvana Terra バックエンド API の完全な仕様書です。すべてのエンドポイント・リクエスト/レスポンス形式・エラーコードを記載しています。

Complete API specification for the Arvana Terra backend. Covers all endpoints, request/response formats, and error codes.

---

## ベース情報 / Base Information

```
Base URL (Development):  http://localhost:3000/api/v1
Base URL (Production):   https://api.arvana-terra.jp/api/v1
Content-Type:            application/json
Accept:                  application/json
Charset:                 UTF-8
```

---

## 認証ヘッダー / Authentication Headers

```http
Authorization: Bearer <accessToken>
```

アクセストークンの有効期限: **15分**
リフレッシュトークンの有効期限: **30日**

---

## 共通レスポンス形式 / Common Response Format

### 成功レスポンス

```json
{
  "success": true,
  "data": { ... }
}
```

### ページネーション付きレスポンス

```json
{
  "data": [ ... ],
  "total": 100,
  "page": 1,
  "limit": 20,
  "totalPages": 5
}
```

### エラーレスポンス

```json
{
  "success": false,
  "error": "エラーメッセージ"
}
```

---

## エラーコード / Error Codes

| HTTP Status | 説明 | 対処方法 |
|-------------|------|---------|
| 400 | Bad Request - リクエスト形式が不正 | リクエストボディを確認 |
| 401 | Unauthorized - 認証が必要 | Authorization ヘッダーを確認、トークンをリフレッシュ |
| 403 | Forbidden - 権限が不足 | ユーザーのロールを確認 |
| 404 | Not Found - リソースが存在しない | ID を確認 |
| 409 | Conflict - 既に存在する | email 重複などの確認 |
| 422 | Unprocessable Entity - バリデーションエラー | リクエストデータを確認 |
| 429 | Too Many Requests - レート制限超過 | 一定時間後に再試行 |
| 500 | Internal Server Error - サーバーエラー | バックエンドログを確認 |

---

## レート制限 / Rate Limiting

```
一般エンドポイント: 100 リクエスト / 15分 / IP
認証エンドポイント: 10 リクエスト / 15分 / IP

レスポンスヘッダー:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 95
  X-RateLimit-Reset: 1705300000
```

---

## ページネーション / Pagination

```
クエリパラメータ:
  page:      ページ番号 (デフォルト: 1)
  limit:     1ページあたりの件数 (デフォルト: 20, 最大: 100)
  sortBy:    ソートフィールド (例: createdAt)
  sortOrder: asc | desc (デフォルト: desc)

例:
  GET /my/lands?page=2&limit=10&sortBy=createdAt&sortOrder=desc
```

---

## 認証 API / Authentication API

### POST /auth/register

新規ユーザー登録。

**認証:** 不要

**リクエスト:**
```json
{
  "email": "tanaka@example.com",
  "password": "securePass123",
  "name": "田中 太郎",
  "role": "landlord",
  "phone": "090-1234-5678"
}
```

| フィールド | 型 | 必須 | 説明 |
|-----------|---|:---:|------|
| email | string | ✓ | メールアドレス（一意） |
| password | string | ✓ | パスワード（8文字以上推奨） |
| name | string | ✓ | 氏名 |
| role | string | ✓ | `landlord` / `homeowner` / `employer` |
| phone | string | - | 電話番号 |

**レスポンス (201):**
```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "tanaka@example.com",
    "name": "田中 太郎",
    "role": "landlord"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "3f4e5b6c-7d8e-9f0a-b1c2-d3e4f5a6b7c8"
}
```

---

### POST /auth/login

ログイン。

**認証:** 不要

**リクエスト:**
```json
{
  "email": "tanaka@example.com",
  "password": "securePass123"
}
```

**レスポンス (200):**
```json
{
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "tanaka@example.com",
    "name": "田中 太郎",
    "role": "landlord"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "3f4e5b6c-7d8e-9f0a-b1c2-d3e4f5a6b7c8"
}
```

---

### POST /auth/refresh

アクセストークンのリフレッシュ。

**認証:** 不要

**リクエスト:**
```json
{
  "refreshToken": "3f4e5b6c-7d8e-9f0a-b1c2-d3e4f5a6b7c8"
}
```

**レスポンス (200):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "new-refresh-token-uuid"
}
```

---

### GET /auth/me

現在の認証ユーザー情報を取得。

**認証:** 必須

**レスポンス (200):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "tanaka@example.com",
  "name": "田中 太郎",
  "role": "landlord",
  "phone": "090-1234-5678",
  "address": "大阪府豊中市...",
  "profileImageUrl": "/uploads/profile-123.jpg",
  "bio": "大阪北部エリアを中心に土地・アパートを管理しています。",
  "createdAt": "2024-01-15T10:00:00.000Z",
  "updatedAt": "2024-02-20T15:30:00.000Z"
}
```

---

### PATCH /auth/me

プロフィール更新。

**認証:** 必須

**リクエスト:**
```json
{
  "name": "田中 太郎",
  "phone": "090-9876-5432",
  "address": "大阪府吹田市...",
  "bio": "更新された自己紹介文",
  "profileImageUrl": "/uploads/new-profile.jpg"
}
```

---

## 土地 API / Lands API

### GET /my/lands

自分の土地一覧を取得。

**認証:** 必須
**ロール:** landlord, admin, super_admin

**クエリパラメータ:**

| パラメータ | 型 | 説明 |
|-----------|---|------|
| status | string | `active` / `for_sale` / `sold` |
| search | string | 名称・住所で検索 |
| page | number | ページ番号 |
| limit | number | 件数 |

**レスポンス (200):**
```json
{
  "data": [
    {
      "id": "land-uuid-1",
      "ownerId": "user-uuid",
      "name": "豊中市の土地",
      "address": "大阪府豊中市庄内西町1-1-1",
      "area": 250.5,
      "zoning": "第1種住居地域",
      "status": "active",
      "isPublic": false,
      "thumbnailUrl": "/uploads/land-thumb.jpg",
      "imageUrls": ["/uploads/land-1.jpg", "/uploads/land-2.jpg"],
      "purchasePrice": 35000000,
      "currentValue": 38000000,
      "purchaseDate": "2020-03-15T00:00:00.000Z",
      "tags": ["住宅地", "大阪府"],
      "createdAt": "2024-01-15T10:00:00.000Z",
      "updatedAt": "2024-02-20T15:30:00.000Z"
    }
  ],
  "total": 5,
  "page": 1,
  "limit": 20,
  "totalPages": 1
}
```

---

### POST /my/lands

土地の新規作成。

**認証:** 必須
**ロール:** landlord, admin, super_admin

**リクエスト:**
```json
{
  "name": "豊中市の土地",
  "address": "大阪府豊中市庄内西町1-1-1",
  "area": 250.5,
  "zoning": "第1種住居地域",
  "description": "静かな住宅街に位置する土地。北側道路、整形地。",
  "status": "active",
  "isPublic": false,
  "purchasePrice": 35000000,
  "currentValue": 38000000,
  "purchaseDate": "2020-03-15",
  "notes": "将来的には売却を検討",
  "tags": ["住宅地", "大阪府", "豊中市"]
}
```

**レスポンス (201):** 作成された Land オブジェクト

---

### GET /my/lands/:id

土地詳細取得。

**レスポンス (200):** Land オブジェクト（properties, chatRooms, tasks を含む）

---

### PATCH /my/lands/:id

土地情報の部分更新。

**リクエスト:** 更新したいフィールドのみ

---

### DELETE /my/lands/:id

土地の削除。（関連する contracts, chatRooms, tasks も cascade 削除）

**レスポンス (200):** `{ "success": true, "message": "Land deleted successfully" }`

---

## 物件 API / Properties API

### GET /my/properties

自分の物件一覧。

**クエリパラメータ:**

| パラメータ | 説明 |
|-----------|------|
| status | active / for_sale / sold / under_renovation |
| buildingType | apartment / house / commercial / warehouse / other |
| landId | 特定の土地に紐づく物件のみ |
| page, limit | ページネーション |

**レスポンス例 (1件):**
```json
{
  "id": "property-uuid",
  "ownerId": "user-uuid",
  "landId": "land-uuid",
  "name": "グリーンハイツ豊中",
  "address": "大阪府豊中市庄内西町1-1-2",
  "buildingType": "apartment",
  "floors": 3,
  "totalRooms": 12,
  "builtYear": 2005,
  "area": 800.0,
  "status": "active",
  "isPublic": false,
  "thumbnailUrl": "/uploads/prop-thumb.jpg",
  "purchasePrice": 80000000,
  "currentValue": 85000000,
  "tags": ["アパート", "大阪府"],
  "createdAt": "2024-01-20T10:00:00.000Z",
  "updatedAt": "2024-02-25T12:00:00.000Z"
}
```

---

### GET /my/properties/:id/stats

物件ダッシュボード統計。

**レスポンス (200):**
```json
{
  "totalRooms": 12,
  "occupied": 10,
  "vacant": 2,
  "monthlyIncome": 750000
}
```

---

## 部屋 API / Rooms API

### GET /my/properties/:id/rooms

部屋一覧。

**クエリパラメータ:** `status` (occupied/vacant/maintenance), `floor` (階数), `search`

**レスポンス例 (1件):**
```json
{
  "id": "room-uuid",
  "propertyId": "property-uuid",
  "roomNumber": "101",
  "floor": 1,
  "type": "one_ldk",
  "area": 45.5,
  "rentPrice": 75000,
  "status": "occupied",
  "notes": "南向き。日当たり良好。",
  "tenant": {
    "id": "tenant-uuid",
    "name": "山田 花子",
    "email": "yamada@example.com",
    "phone": "090-1111-2222",
    "moveInDate": "2022-04-01T00:00:00.000Z",
    "contractEndDate": "2024-03-31T00:00:00.000Z",
    "rentAmount": 75000,
    "depositAmount": 150000,
    "paymentStatus": "current"
  },
  "createdAt": "2024-01-20T10:00:00.000Z"
}
```

---

### GET /my/properties/:propertyId/rooms/:roomId/payments

支払い履歴一覧。

**レスポンス例 (1件):**
```json
{
  "id": "payment-uuid",
  "roomId": "room-uuid",
  "propertyId": "property-uuid",
  "tenantId": "tenant-uuid",
  "amount": 75000,
  "dueDate": "2024-02-01T00:00:00.000Z",
  "paidDate": "2024-02-03T14:30:00.000Z",
  "status": "paid",
  "notes": null,
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

---

## 設備 API / Equipment API

### GET /my/properties/:id/equipment

設備一覧。

**クエリパラメータ:** `floor`, `roomId`, `category`, `status`

**レスポンス例 (1件):**
```json
{
  "id": "equipment-uuid",
  "propertyId": "property-uuid",
  "roomId": null,
  "name": "エレベーター",
  "category": "elevator",
  "floor": null,
  "location": "1F ロビー",
  "manufacturer": "三菱電機",
  "model": "HOPE-XV",
  "serialNumber": "EV-2005-0123",
  "installDate": "2005-06-01T00:00:00.000Z",
  "warrantyExpiry": "2025-05-31T00:00:00.000Z",
  "status": "good",
  "lastInspectionDate": "2024-01-15T00:00:00.000Z",
  "nextInspectionDate": "2024-07-15T00:00:00.000Z",
  "repairCostEstimate": null,
  "notes": "年2回の法定点検実施中"
}
```

---

### GET /my/properties/:id/equipment/:deviceId/readings

スマートデバイスの読み取りデータ。

**クエリパラメータ:** `weeks` (デフォルト: 8)

**レスポンス (200):**
```json
[
  {
    "date": "2024-02-12",
    "value": 123.5,
    "unit": "m3"
  },
  {
    "date": "2024-02-19",
    "value": 128.2,
    "unit": "m3"
  }
]
```

---

## チャット API / Chat API

チャット機能は「土地チャット」「物件チャット」「従業員チャット」の3種類をサポートします。トピック（ChatRoom）単位でチャットを作成し、Socket.io によるリアルタイム通信を提供します。

### GET /chats?type={type}&targetId={id}

チャットルーム一覧取得。

**認証:** 必須

| クエリパラメータ | 型 | 必須 | 説明 |
|--------------|---|:---:|------|
| type | string | ✓ | `land` / `property` / `employee` |
| targetId | string | ✓ | 土地ID / 物件ID / 従業員ID |

**レスポンス (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "chatroom-cuid",
      "type": "land",
      "title": "外壁修繕について",
      "description": "2024年秋の外壁塗装工事に関する情報共有",
      "landId": "land-001",
      "propertyId": null,
      "employeeId": null,
      "createdById": "user-cuid",
      "createdBy": { "id": "user-cuid", "name": "田中 一郎" },
      "messages": [
        {
          "id": "msg-cuid",
          "chatRoomId": "chatroom-cuid",
          "senderId": "user-cuid",
          "sender": { "id": "user-cuid", "name": "田中 一郎" },
          "content": "業者の見積もりが届きました",
          "createdAt": "2024-08-20T14:30:00.000Z"
        }
      ],
      "_count": { "messages": 12 },
      "createdAt": "2024-08-01T10:00:00.000Z",
      "updatedAt": "2024-08-20T14:30:00.000Z"
    }
  ]
}
```

---

### POST /chats

チャットルーム作成。

**認証:** 必須

**リクエスト:**
```json
{
  "type": "land",
  "title": "外壁修繕について",
  "description": "2024年秋の外壁塗装工事に関する情報共有",
  "landId": "land-001"
}
```

| フィールド | 型 | 必須 | 説明 |
|-----------|---|:---:|------|
| type | string | ✓ | `land` / `property` / `employee` |
| title | string | ✓ | チャットルーム名（トピック） |
| description | string | - | 説明 |
| landId | string | - | type=land の場合に指定 |
| propertyId | string | - | type=property の場合に指定 |
| employeeId | string | - | type=employee の場合に指定 |

**レスポンス (201):** 作成された ChatRoom オブジェクト

---

### GET /chats/:id

チャットルーム詳細取得。

**認証:** 必須

**レスポンス (200):**
```json
{
  "success": true,
  "data": {
    "id": "chatroom-cuid",
    "type": "land",
    "title": "外壁修繕について",
    "createdBy": { "id": "user-cuid", "name": "田中 一郎" },
    "land": { "id": "land-001", "name": "奈良市東大寺周辺 売地" },
    "property": null,
    "employee": null,
    "createdAt": "2024-08-01T10:00:00.000Z",
    "updatedAt": "2024-08-20T14:30:00.000Z"
  }
}
```

---

### GET /chats/:id/messages

チャットメッセージ一覧（ページネーション）。

**認証:** 必須

**クエリパラメータ:** `page` (デフォルト: 1), `limit` (デフォルト: 50)

**レスポンス (200):**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "msg-cuid",
        "chatRoomId": "chatroom-cuid",
        "senderId": "user-cuid",
        "sender": { "id": "user-cuid", "name": "田中 一郎" },
        "content": "業者の見積もりが届きました",
        "createdAt": "2024-08-20T14:30:00.000Z"
      }
    ],
    "total": 45,
    "page": 1,
    "limit": 50
  }
}
```

---

### POST /chats/:id/messages

メッセージ送信（HTTP フォールバック）。Socket.io が利用できない場合に使用。

**認証:** 必須

**リクエスト:**
```json
{ "content": "業者の見積もりが届きました" }
```

**レスポンス (201):** 作成された ChatMessage オブジェクト。また Socket.io `/chat` 名前空間の `room:{id}` ルームへ `new_message` イベントを broadcast。

---

## タスク API / Tasks API

### POST /tasks

タスクの新規作成。

**リクエスト:**
```json
{
  "propertyId": "property-uuid",
  "landId": null,
  "title": "エアコンフィルター清掃",
  "description": "101号室のエアコンフィルターが汚れています",
  "status": "todo",
  "priority": "medium",
  "assignedTo": null,
  "dueDate": "2024-03-01T00:00:00.000Z"
}
```

**レスポンス (201):**
```json
{
  "id": "task-uuid",
  "propertyId": "property-uuid",
  "landId": null,
  "ownerId": "user-uuid",
  "title": "エアコンフィルター清掃",
  "description": "101号室のエアコンフィルターが汚れています",
  "status": "todo",
  "priority": "medium",
  "assignedTo": null,
  "dueDate": "2024-03-01T00:00:00.000Z",
  "isAiSuggested": false,
  "aiReason": null,
  "createdAt": "2024-02-20T10:00:00.000Z",
  "updatedAt": "2024-02-20T10:00:00.000Z"
}
```

---

### GET /my/properties/:id/tasks/ai-suggestions

AIタスク提案。

**レスポンス (200):**
```json
[
  {
    "title": "給水ポンプ定期点検",
    "description": "前回の点検から12ヶ月が経過しています。法定点検期限が近づいています。",
    "priority": "high",
    "isAiSuggested": true,
    "aiReason": "設備台帳から給水ポンプの最終点検日が2023年2月であることを確認。定期点検（年1回）が未実施の状態です。"
  },
  {
    "title": "201号室 入居者更新案内送付",
    "description": "201号室の契約終了日は2024年3月31日です。更新または退去の確認を行ってください。",
    "priority": "urgent",
    "isAiSuggested": true,
    "aiReason": "テナント情報から契約終了日が40日後に迫っています。一般的に2ヶ月前の通知が必要です。"
  }
]
```

---

## 業者 API / Vendors API

### GET /vendors

承認済み業者一覧（クライアントアプリ用）。

**クエリパラメータ:** `category`, `serviceArea`, `search`, `page`, `limit`

**レスポンス例 (1件):**
```json
{
  "id": "vendor-uuid",
  "name": "大阪水道設備株式会社",
  "category": "plumbing",
  "contactName": "中村 健一",
  "email": "info@osaka-suido.co.jp",
  "phone": "06-1234-5678",
  "address": "大阪府大阪市北区...",
  "description": "大阪・阪神エリアの水道設備工事専門業者。緊急対応可能。",
  "website": "https://osaka-suido.co.jp",
  "serviceAreas": ["大阪府", "兵庫県", "奈良県"],
  "rating": 4.5,
  "isApproved": true,
  "createdAt": "2024-01-10T10:00:00.000Z"
}
```

---

## SNS API

### GET /sns/posts

SNS 投稿一覧。

**クエリパラメータ:** `type` (general/advice/knowledge/event/case_study/official/tax_advice/vendor_info/announcement), `tags`, `page`, `limit`

**レスポンス例 (1件):**
```json
{
  "id": "post-uuid",
  "authorId": "user-uuid",
  "author": {
    "id": "user-uuid",
    "name": "田中 太郎",
    "profileImageUrl": "/uploads/profile.jpg",
    "role": "landlord"
  },
  "type": "advice",
  "title": "空室対策に効果的だったリノベーション事例",
  "content": "3年間空室だった2LDKを...",
  "tags": ["空室対策", "リノベーション", "大阪"],
  "imageUrls": ["/uploads/renovation-before.jpg", "/uploads/renovation-after.jpg"],
  "viewCount": 234,
  "likesCount": 18,
  "commentsCount": 5,
  "isLiked": false,
  "createdAt": "2024-02-15T10:00:00.000Z",
  "updatedAt": "2024-02-15T10:00:00.000Z"
}
```

---

## 資産価値 API / Asset Valuation API

### POST /my/valuation

資産価値の AI 評価を実行（計算に数秒かかる場合があります）。

**リクエスト:**
```json
{
  "landIds": ["land-uuid-1", "land-uuid-2"],
  "propertyIds": ["property-uuid-1"]
}
```

**レスポンス (200):**
```json
{
  "id": "valuation-uuid",
  "userId": "user-uuid",
  "landIds": ["land-uuid-1", "land-uuid-2"],
  "propertyIds": ["property-uuid-1"],
  "totalCurrentValue": 173000000,
  "aiPredictedValue": 195000000,
  "predictionYear": 2027,
  "breakdown": {
    "lands": [
      { "id": "land-uuid-1", "name": "豊中市の土地", "currentValue": 38000000 },
      { "id": "land-uuid-2", "name": "吹田市の土地", "currentValue": 50000000 }
    ],
    "properties": [
      { "id": "property-uuid-1", "name": "グリーンハイツ豊中", "currentValue": 85000000 }
    ]
  },
  "aiAnalysis": "大阪北部エリア（豊中市・吹田市）の地価は、大阪駅周辺の再開発効果と阪急電車の沿線人気により、今後3年間で約8〜12%の上昇が見込まれます。グリーンハイツ豊中は築20年が近づいているため、建物価値の減少をカバーするためのリノベーション投資を検討することをお勧めします。",
  "calculatedAt": "2024-02-20T16:00:00.000Z"
}
```

---

## 管理者 API / Admin API

### GET /admin/stats/system

システム統計（管理者ダッシュボード用）。

**認証:** 必須
**ロール:** admin, super_admin

**レスポンス (200):**
```json
{
  "totalUsers": 1250,
  "totalProperties": 3840,
  "totalLands": 890,
  "totalVendors": 145,
  "activeVendors": 128,
  "pendingVendors": 12,
  "totalContent": 89,
  "publishedContent": 72,
  "userGrowthRate": 12.5,
  "propertyGrowthRate": 8.3
}
```

---

### POST /admin/vendors/:id/approve

業者を承認する。

**認証:** 必須
**ロール:** admin, super_admin

**レスポンス (200):**
```json
{
  "id": "vendor-uuid",
  "name": "大阪水道設備株式会社",
  "isApproved": true,
  "status": "approved",
  "approvedAt": "2024-02-20T16:30:00.000Z"
}
```

---

### POST /admin/vendors/:id/reject

業者を却下する。

**リクエスト:**
```json
{
  "reason": "提出書類が不足しています。営業許可証のコピーをご提出ください。"
}
```

---

### POST /admin/vendors/bulk-approve

業者を一括承認する。

**リクエスト:**
```json
{
  "ids": ["vendor-uuid-1", "vendor-uuid-2", "vendor-uuid-3"]
}
```

**レスポンス (200):** `{ "success": true, "approvedCount": 3 }`

---

## Socket.io イベント仕様 / Socket.io Events Specification

### 接続 / Connection

```javascript
// クライアント接続設定
const socket = io('ws://localhost:3000/chat', {
  auth: { token: 'Bearer eyJhbGci...' },
  transports: ['websocket'],
})
```

### /chat Namespace - 接続

```javascript
// バックエンドポート: 3001
const socket = io('http://localhost:3001/chat', {
  transports: ['websocket', 'polling'],
})
```

### /chat Namespace - クライアント → サーバー

#### join_room

チャットルームに参加（ルームの Socket.io room に join）。

```javascript
socket.emit('join_room', 'chatroom-cuid')
```

---

#### leave_room

チャットルームから退出。

```javascript
socket.emit('leave_room', 'chatroom-cuid')
```

---

#### send_message

メッセージを送信する（DB保存 + 同ルーム全員にブロードキャスト）。

```javascript
socket.emit('send_message', {
  roomId: 'chatroom-cuid',
  content: 'メッセージ内容',
  senderId: 'user-cuid',
  senderName: '田中 一郎'
})
```

---

### /chat Namespace - サーバー → クライアント

#### new_message

新着メッセージ通知（同ルーム全員に broadcast）。

```javascript
socket.on('new_message', (message) => {
  // message の型:
  {
    id: 'msg-cuid',
    chatRoomId: 'chatroom-cuid',
    senderId: 'user-cuid',
    sender: { id: 'user-cuid', name: '田中 一郎' },
    content: 'メッセージ内容',
    createdAt: '2024-08-20T14:30:00.000Z'
  }
})
```

---

### /notification Namespace

```javascript
// 接続
const notifSocket = io('ws://localhost:3000/notification', {
  auth: { token: 'Bearer eyJhbGci...' },
})

// 通知受信
notifSocket.on('notification', (notification) => {
  // notification の型:
  {
    id: 'notification-uuid',
    userId: 'user-uuid',
    type: 'payment_due',  // 通知タイプ
    title: '家賃支払い期日のお知らせ',
    content: '101号室の2月分家賃の支払い期日が明日です。',
    isRead: false,
    relatedId: 'room-uuid',
    relatedType: 'room',
    createdAt: '2024-01-31T09:00:00.000Z'
  }
})
```

**通知タイプ一覧:**

| type | 説明 | relatedType |
|------|------|------------|
| `payment_due` | 家賃支払い期日 | room |
| `payment_received` | 家賃入金確認 | payment |
| `equipment_warning` | 設備警告 | equipment |
| `contract_expiring` | 契約期限間近 | contract |
| `task_due` | タスク期限間近 | task |
| `chat_message` | 新着チャットメッセージ | chat_room |
| `opportunity` | 新規ビジネスチャンス | opportunity |
| `system` | システム通知 | null |
| `vendor_approved` | 業者承認完了 | vendor |
