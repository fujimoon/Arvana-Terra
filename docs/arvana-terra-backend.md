# arvana-terra-backend - 詳細仕様 / Detailed Specification

## 概要 / Overview

Arvana Terra のバックエンドAPIサーバーです。Node.js/TypeScript + Express.js で構築され、PostgreSQL（Prisma ORM）・Redis・Socket.io を使用します。すべてのクライアントアプリ（web, iOS, Android, admin）が接続する中核サーバーです。

The backend API server for Arvana Terra. Built with Node.js/TypeScript + Express.js, using PostgreSQL (Prisma ORM), Redis, and Socket.io. It is the central server to which all client apps connect.

---

## テクノロジースタック / Tech Stack

| パッケージ | バージョン | 用途 |
|-----------|-----------|------|
| Node.js | 20.x | ランタイム |
| TypeScript | 5.3.3 | 言語 |
| Express.js | 4.18.2 | HTTP フレームワーク |
| Prisma | 5.9.1 | ORM |
| PostgreSQL | 16.x | メインデータベース |
| ioredis | 5.3.2 | Redis クライアント |
| Socket.io | 4.7.4 | WebSocket サーバー |
| jsonwebtoken | 9.0.2 | JWT 認証 |
| bcryptjs | 2.4.3 | パスワードハッシュ化 |
| multer | 1.4.5-lts.1 | ファイルアップロード |
| openai | 4.28.0 | OpenAI API クライアント |
| zod | 3.22.4 | バリデーション |
| helmet | 7.1.0 | セキュリティヘッダー |
| express-rate-limit | 7.1.5 | レート制限 |
| cors | 2.8.5 | CORS |
| morgan | 1.10.0 | HTTPロギング |
| winston | 3.11.0 | アプリケーションロギング |

---

## ディレクトリ構造 / Directory Structure

```
arvana-terra-backend/
├── prisma/
│   └── schema.prisma          # データベーススキーマ定義
├── src/
│   ├── index.ts               # エントリポイント
│   ├── lib/
│   │   ├── prisma.ts          # Prisma クライアントシングルトン
│   │   ├── redis.ts           # Redis クライアントシングルトン
│   │   └── logger.ts          # Winston ロガー
│   ├── middleware/
│   │   ├── auth.ts            # JWT 認証ミドルウェア
│   │   ├── rbac.ts            # ロールベースアクセス制御
│   │   ├── error.ts           # エラーハンドリング
│   │   └── upload.ts          # Multer ファイルアップロード
│   ├── routes/                # API ルート定義
│   ├── services/
│   │   ├── auth.service.ts    # 認証サービス
│   │   ├── land.service.ts    # 土地管理サービス
│   │   ├── property.service.ts # 物件管理サービス
│   │   ├── room.service.ts    # 部屋管理サービス
│   │   └── equipment.service.ts # 設備管理サービス
│   ├── socket/
│   │   ├── index.ts           # Socket.io 初期化・イベントハンドラー
│   │   └── instance.ts        # Socket.io インスタンスのシングルトン
│   ├── types/
│   │   └── index.ts           # 共通型定義
│   └── utils/
│       ├── asyncHandler.ts    # 非同期エラーハンドラー
│       └── crypto.ts          # 暗号化ユーティリティ
├── uploads/                   # アップロードファイルの保存先
├── logs/                      # アプリケーションログ
├── package.json
└── tsconfig.json
```

---

## データベーススキーマ / Database Schema

### Enums

```prisma
enum UserRole {
  landlord      # 地主
  homeowner     # 家主
  employer      # 雇用者（Arvana Work 連携）
  admin         # 管理者
  super_admin   # スーパー管理者
}

enum LandStatus    { active, for_sale, sold }
enum PropertyStatus { active, for_sale, sold, under_renovation }
enum BuildingType   { apartment, house, commercial, warehouse, other }
enum RoomType       { studio, one_k, one_ldk, two_ldk, three_ldk, four_ldk, office, shop, other }
enum RoomStatus     { occupied, vacant, maintenance }
enum PaymentRecordStatus { paid, pending, late, overdue }
enum TenantPaymentStatus { current, late, defaulted }
enum EquipmentCategory  { lighting, door, hvac, plumbing, electrical, elevator, camera, other }
enum EquipmentStatus    { good, warning, broken, replaced }
enum DeviceType         { water_meter, electric_meter, camera, sensor }
enum ContractType       { nda, rental, purchase, management, other }
enum ContractStatus     { draft, active, expired, terminated }
enum ChatRoomType       { property, land, employee, direct }
enum ChatParticipantRole { owner, member }
enum MessageType        { text, image, file }
enum TaskStatus         { todo, in_progress, done, cancelled }
enum TaskPriority       { low, medium, high, urgent }
enum VendorCategory     { glass, electric, plumbing, construction, cleaning, security, other }
enum OpportunityType    { land_purchase, sale, rental, development, other }
enum SnsPostType        { general, advice, knowledge, event, case_study, official, tax_advice, vendor_info, announcement }
```

### モデル定義 / Model Definitions

#### User

```prisma
model User {
  id              String    @id @default(uuid())
  email           String    @unique
  password        String    # bcryptjs ハッシュ (SALT_ROUNDS=12)
  name            String
  role            UserRole
  phone           String?
  address         String?
  profileImageUrl String?
  bio             String?   @db.Text
  isActive        Boolean   @default(true)  # ソフトデリートフラグ
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt

  # リレーション（省略）
}
```

#### Land (土地)

```prisma
model Land {
  id            String     @id @default(uuid())
  ownerId       String     # User.id (地主のみ)
  name          String     # 土地名称
  address       String     # 住所
  area          Float      # 面積 (m2)
  zoning        String?    # 用途地域
  description   String?    @db.Text
  status        LandStatus @default(active)
  isPublic      Boolean    @default(false)  # 公開/非公開
  thumbnailUrl  String?
  imageUrls     String[]   @default([])    # PostgreSQL 配列型
  purchasePrice Float?     # 取得価格
  currentValue  Float?     # 現在価値
  purchaseDate  DateTime?  # 取得日
  notes         String?    @db.Text
  tags          String[]   @default([])
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
}
```

#### Property (物件・建物)

```prisma
model Property {
  id            String         @id @default(uuid())
  ownerId       String         # User.id
  landId        String?        # nullable: 土地なし物件も可
  name          String
  address       String
  buildingType  BuildingType   # apartment / house / commercial / warehouse / other
  floors        Int?           # 階数
  totalRooms    Int?           # 総部屋数
  builtYear     Int?           # 建築年
  area          Float          # 延床面積 (m2)
  description   String?        @db.Text
  status        PropertyStatus @default(active)
  isPublic      Boolean        @default(false)
  thumbnailUrl  String?
  imageUrls     String[]       @default([])
  purchasePrice Float?
  currentValue  Float?
  purchaseDate  DateTime?
  notes         String?        @db.Text
  tags          String[]       @default([])
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
}
```

#### Room (部屋)

```prisma
model Room {
  id          String     @id @default(uuid())
  propertyId  String
  roomNumber  String     # 部屋番号 (例: "101", "A-201")
  floor       Int?       # 階数
  type        RoomType   # studio / 1K / 1LDK / 2LDK / 3LDK / 4LDK / office / shop / other
  area        Float?     # 専有面積 (m2)
  rentPrice   Float?     # 賃料
  status      RoomStatus @default(vacant)
  notes       String?    @db.Text
  memo        String?    @db.Text  # オーナー用メモ
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  tenant    Tenant?      # UNIQUE: 1部屋1入居者
}
```

#### Tenant (入居者)

```prisma
model Tenant {
  id              String              @id @default(uuid())
  roomId          String              @unique  # 1部屋1入居者
  propertyId      String
  name            String
  email           String?
  phone           String?
  moveInDate      DateTime
  moveOutDate     DateTime?
  contractEndDate DateTime?
  rentAmount      Float
  depositAmount   Float?
  paymentStatus   TenantPaymentStatus @default(current)
  notes           String?             @db.Text
}
```

#### Payment (支払い記録)

```prisma
model Payment {
  id         String              @id @default(uuid())
  tenantId   String?
  roomId     String
  propertyId String
  amount     Float
  dueDate    DateTime
  paidDate   DateTime?
  status     PaymentRecordStatus @default(pending)
  notes      String?             @db.Text
}
```

#### Equipment (設備)

```prisma
model Equipment {
  id                  String            @id @default(uuid())
  propertyId          String
  roomId              String?           # nullable: 共用設備はnull
  name                String
  category            EquipmentCategory
  floor               Int?
  location            String?           # 設置場所の説明
  manufacturer        String?
  model               String?
  serialNumber        String?
  installDate         DateTime?
  warrantyExpiry      DateTime?
  status              EquipmentStatus   @default(good)
  lastInspectionDate  DateTime?
  nextInspectionDate  DateTime?
  repairCostEstimate  Float?
  notes               String?           @db.Text
}
```

#### SmartDeviceData (スマートデバイスデータ)

```prisma
model SmartDeviceData {
  id           String       @id @default(uuid())
  propertyId   String
  roomId       String?
  deviceType   DeviceType   # water_meter / electric_meter / camera / sensor
  deviceId     String       # デバイス固有ID
  location     String?
  readings     Json         @default("[]")  # 週次データ JSON 配列
  cameraStatus CameraStatus?               # カメラの場合のみ
  lastUpdated  DateTime     @default(now())
}
```

#### Contract (契約書)

```prisma
model Contract {
  id         String         @id @default(uuid())
  propertyId String?
  landId     String?
  type       ContractType   # nda / rental / purchase / management / other
  title      String
  content    String         @db.Text  # 契約書本文
  templateId String?
  parties    Json           @default("[]")  # 当事者リスト (JSON)
  signedDate DateTime?
  expiryDate DateTime?
  status     ContractStatus @default(draft)
  fileUrl    String?        # アップロードされたPDFのURL
}
```

#### ChatRoom / ChatMessage

```prisma
enum ChatRoomType {
  land       # 土地チャット
  property   # 物件チャット
  employee   # 従業員チャット
}

model ChatRoom {
  id          String       @id @default(cuid())
  type        ChatRoomType
  title       String       # トピック名
  description String?      @db.Text
  landId      String?      # type=land の場合
  propertyId  String?      # type=property の場合
  employeeId  String?      # type=employee の場合
  createdById String       # 作成者 User.id
  messages    ChatMessage[]
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
}

model ChatMessage {
  id         String   @id @default(cuid())
  chatRoomId String
  senderId   String   # 送信者 User.id
  content    String   @db.Text
  createdAt  DateTime @default(now())
}
```

**リレーション:**
- `Land.chatRooms`: 土地に紐づくチャットルーム一覧
- `Property.chatRooms`: 物件に紐づくチャットルーム一覧
- `Employee.chatRooms`: 従業員に紐づくチャットルーム一覧
- `User.chatRoomsCreated`: ユーザーが作成したチャットルーム（`@relation("ChatRoomsCreated")`）
- `User.messagesSent`: ユーザーが送信したメッセージ（`@relation("MessagesSent")`）

#### Task (業務タスク)

```prisma
model Task {
  id            String       @id @default(uuid())
  propertyId    String?
  landId        String?
  ownerId       String
  title         String
  description   String?      @db.Text
  status        TaskStatus   @default(todo)
  priority      TaskPriority @default(medium)
  assignedTo    String?      # 担当者 User.id
  dueDate       DateTime?
  isAiSuggested Boolean      @default(false)  # AI 提案タスクフラグ
  aiReason      String?      @db.Text         # AI 提案理由
}
```

#### Vendor (業者)

```prisma
model Vendor {
  id           String         @id @default(uuid())
  name         String
  category     VendorCategory # glass / electric / plumbing / construction / cleaning / security / other
  contactName  String?
  email        String?
  phone        String?
  address      String?
  description  String?        @db.Text
  website      String?
  serviceAreas String[]       @default([])  # サービスエリア (都道府県)
  rating       Float?
  isApproved   Boolean        @default(false)  # 管理者承認フラグ
  registeredBy String?        # 登録した管理者のID
}
```

#### AssetValuation (資産価値)

```prisma
model AssetValuation {
  id                String   @id @default(uuid())
  userId            String
  landIds           String[] @default([])    # 評価対象の土地IDリスト
  propertyIds       String[] @default([])    # 評価対象の物件IDリスト
  totalCurrentValue Float?   # 現在の総資産価値
  aiPredictedValue  Float?   # AI予測価値
  predictionYear    Int?     # 予測対象年
  breakdown         Json?    # 内訳 (JSON)
  aiAnalysis        String?  @db.Text        # AI分析コメント
  calculatedAt      DateTime @default(now())
}
```

---

## APIエンドポイント / API Endpoints

### Base URL: `http://localhost:3001/api/v1`

### 認証 / Authentication

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/auth/register` | - | ユーザー登録 |
| POST | `/auth/login` | - | ログイン |
| POST | `/auth/logout` | Required | ログアウト |
| POST | `/auth/refresh` | - | トークンリフレッシュ |
| GET | `/auth/me` | Required | 自分のプロフィール取得 |
| PATCH | `/auth/me` | Required | 自分のプロフィール更新 |

**POST /auth/register**
```json
// Request
{
  "email": "landlord@example.com",
  "password": "securepass123",
  "name": "田中 太郎",
  "role": "landlord",
  "phone": "090-1234-5678"
}

// Response 201
{
  "user": { "id": "uuid", "email": "...", "name": "...", "role": "landlord" },
  "accessToken": "eyJ...",
  "refreshToken": "uuid-v4"
}
```

**POST /auth/login**
```json
// Request
{ "email": "landlord@example.com", "password": "securepass123" }

// Response 200
{
  "user": { "id": "uuid", "email": "...", "name": "...", "role": "landlord" },
  "accessToken": "eyJ...",
  "refreshToken": "uuid-v4"
}
```

---

### 土地 / Lands

| Method | Path | Auth | Roles | Description |
|--------|------|------|-------|-------------|
| GET | `/lands` | - | - | 公開土地一覧 |
| GET | `/lands/:id` | - | - | 公開土地詳細 |
| GET | `/my/lands` | Required | landlord, admin, super_admin | 自分の土地一覧 |
| POST | `/my/lands` | Required | landlord, admin, super_admin | 土地の新規作成 |
| GET | `/my/lands/:id` | Required | landlord, admin, super_admin | 自分の土地詳細 |
| PATCH | `/my/lands/:id` | Required | landlord, admin, super_admin | 土地の更新 |
| DELETE | `/my/lands/:id` | Required | landlord, admin, super_admin | 土地の削除 |
| GET | `/my/lands/:id/chats` | Required | landlord, admin, super_admin | 土地のチャットルーム一覧 |
| POST | `/my/lands/:id/chats` | Required | landlord, admin, super_admin | 土地チャットルーム作成 |
| GET | `/my/lands/:id/tasks` | Required | landlord, admin, super_admin | 土地タスク一覧 |
| POST | `/my/lands/upload` | Required | landlord, admin, super_admin | 土地画像アップロード |

**POST /my/lands**
```json
// Request
{
  "name": "大阪府豊中市の土地",
  "address": "大阪府豊中市庄内西町1-1-1",
  "area": 250.5,
  "zoning": "第1種住居地域",
  "description": "静かな住宅街にある土地",
  "status": "active",
  "isPublic": false,
  "purchasePrice": 35000000,
  "currentValue": 38000000,
  "tags": ["住宅地", "大阪府", "豊中市"]
}

// Response 201
{
  "id": "uuid",
  "ownerId": "user-uuid",
  "name": "大阪府豊中市の土地",
  "address": "...",
  "area": 250.5,
  ...
  "createdAt": "2024-01-15T10:00:00.000Z"
}
```

---

### 物件 / Properties

| Method | Path | Auth | Roles | Description |
|--------|------|------|-------|-------------|
| GET | `/properties` | - | - | 公開物件一覧 |
| GET | `/properties/:id` | - | - | 公開物件詳細 |
| GET | `/my/properties` | Required | All | 自分の物件一覧 |
| POST | `/my/properties` | Required | landlord, homeowner, admin, super_admin | 物件作成 |
| GET | `/my/properties/:id` | Required | All | 物件詳細 |
| PATCH | `/my/properties/:id` | Required | landlord, homeowner, admin, super_admin | 物件更新 |
| DELETE | `/my/properties/:id` | Required | landlord, homeowner, admin, super_admin | 物件削除 |
| GET | `/my/properties/:id/stats` | Required | All | 物件ダッシュボード統計 |
| GET | `/my/properties/:id/chats` | Required | All | 物件チャットルーム一覧 |
| POST | `/my/properties/:id/chats` | Required | landlord, homeowner, admin, super_admin | チャットルーム作成 |
| GET | `/my/properties/:id/tasks` | Required | All | 物件タスク一覧 |
| GET | `/my/properties/:id/tasks/ai-suggestions` | Required | All | AIタスク提案 |

---

### 部屋 / Rooms

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/my/properties/:id/rooms` | Required | 部屋一覧（フィルタ可） |
| GET | `/my/properties/:id/rooms/:roomId` | Required | 部屋詳細（入居者・設備含む） |
| PATCH | `/my/properties/:id/rooms/:roomId` | Required | 部屋情報更新 |
| GET | `/my/properties/:id/rooms/:roomId/payments` | Required | 支払い履歴 |
| PATCH | `/my/properties/:id/rooms/:roomId/payments/:payId` | Required | 支払い記録更新 |

---

### 設備 / Equipment

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/my/properties/:id/equipment` | Required | 設備一覧 |
| GET | `/my/properties/:id/equipment/floor/:n` | Required | フロア別設備 |
| GET | `/my/properties/:id/equipment/rooms/:roomId` | Required | 部屋別設備 |
| GET | `/my/properties/:id/equipment/:eqId` | Required | 設備詳細 |
| PATCH | `/my/properties/:id/equipment/:eqId` | Required | 設備更新 |
| GET | `/my/properties/:id/equipment/:devId/readings` | Required | スマートデバイスデータ |

---

### 契約書 / Contracts

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/my/properties/:id/contracts` | Required | 契約書一覧 |
| POST | `/my/properties/:id/contracts` | Required | 契約書作成 |
| GET | `/my/properties/:id/contracts/:cId` | Required | 契約書詳細 |
| PATCH | `/my/properties/:id/contracts/:cId` | Required | 契約書更新 |
| DELETE | `/my/properties/:id/contracts/:cId` | Required | 契約書削除 |
| GET | `/contracts/templates` | Required | テンプレート一覧 |
| GET | `/contracts/templates/:id` | Required | テンプレート詳細 |

---

### タスク / Tasks

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/tasks` | Required | タスク作成 |
| PATCH | `/tasks/:id` | Required | タスク更新 |
| DELETE | `/tasks/:id` | Required | タスク削除 |
| GET | `/my/properties/:id/tasks` | Required | 物件タスク一覧 |
| GET | `/my/lands/:id/tasks` | Required | 土地タスク一覧 |
| GET | `/my/properties/:id/tasks/ai-suggestions` | Required | AIタスク提案 |

---

### チャット / Chats

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/chats?type={type}&targetId={id}` | Required | チャットルーム一覧（type: land/property/employee） |
| POST | `/chats` | Required | チャットルーム作成 |
| GET | `/chats/:id` | Required | チャットルーム詳細 |
| GET | `/chats/:id/messages` | Required | メッセージ一覧（ページネーション: page, limit） |
| POST | `/chats/:id/messages` | Required | メッセージ送信（HTTP フォールバック） |

---

### 従業員 / Employees

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/my/properties/:id/employees` | Required | 従業員一覧 |
| POST | `/my/properties/:id/employees` | Required | 従業員登録 |
| GET | `/my/properties/:id/employees/:eId` | Required | 従業員詳細 |
| PATCH | `/my/properties/:id/employees/:eId` | Required | 従業員更新 |
| DELETE | `/my/properties/:id/employees/:eId` | Required | 従業員削除 |

---

### 業者 / Vendors

| Method | Path | Auth | Roles | Description |
|--------|------|------|-------|-------------|
| GET | `/vendors` | Required | All | 承認済み業者一覧 |
| GET | `/vendors/:id` | Required | All | 業者詳細 |
| POST | `/my/vendors/:id/connect` | Required | landlord, homeowner | 業者と連携 |
| DELETE | `/my/vendors/:id/disconnect` | Required | landlord, homeowner | 業者連携解除 |

---

### SNS

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/sns/posts` | Required | 投稿一覧 |
| POST | `/sns/posts` | Required | 投稿作成 |
| GET | `/sns/posts/:id` | Required | 投稿詳細 |
| PATCH | `/sns/posts/:id` | Required | 投稿更新（自分の投稿のみ） |
| DELETE | `/sns/posts/:id` | Required | 投稿削除 |
| POST | `/sns/posts/:id/like` | Required | いいね |
| DELETE | `/sns/posts/:id/like` | Required | いいね取り消し |
| GET | `/sns/posts/:id/comments` | Required | コメント一覧 |
| POST | `/sns/posts/:id/comments` | Required | コメント投稿 |
| DELETE | `/sns/posts/:id/comments/:cId` | Required | コメント削除 |

---

### 資産価値 / Asset Valuation

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/my/valuation` | Required | 最新の資産価値評価 |
| POST | `/my/valuation` | Required | 資産価値評価の再計算（AI） |

---

### ビジネスチャンス / Opportunities

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/my/opportunities` | Required | ビジネスチャンス一覧 |
| POST | `/my/opportunities` | Required | 作成 |
| GET | `/my/opportunities/:id` | Required | 詳細 |
| PATCH | `/my/opportunities/:id` | Required | 更新 |
| DELETE | `/my/opportunities/:id` | Required | 削除 |

---

### 入金管理 / Payments

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/payments/property/:propertyId` | Required | 物件の入金一覧 |
| POST | `/payments` | Required | 入金記録作成 |
| PATCH | `/payments/:id` | Required | 入金記録更新 |
| DELETE | `/payments/:id` | Required | 入金記録削除 |

---

### スマートデバイス / Smart Devices

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/properties/:propertyId/smart-devices` | Required | デバイス一覧 |
| POST | `/properties/:propertyId/smart-devices` | Required | デバイス登録 |
| PATCH | `/properties/:propertyId/smart-devices/:id` | Required | デバイス更新・readings追加 |
| DELETE | `/properties/:propertyId/smart-devices/:id` | Required | デバイス削除 |

---

### 通知 / Notifications

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/notifications` | Required | 通知一覧 |
| PATCH | `/notifications/:id/read` | Required | 既読にする |
| PATCH | `/notifications/read-all` | Required | 全て既読にする |

---

### 管理者 / Admin

| Method | Path | Auth | Roles | Description |
|--------|------|------|-------|-------------|
| GET | `/admin/stats/system` | Required | admin, super_admin | システム統計 |
| GET | `/admin/stats/registration-trends` | Required | admin, super_admin | 登録トレンド |
| GET | `/admin/stats/activity` | Required | admin, super_admin | アクティビティ指標 |
| GET | `/admin/vendors` | Required | admin, super_admin | 業者一覧（全ステータス） |
| POST | `/admin/vendors` | Required | admin, super_admin | 業者登録 |
| GET | `/admin/vendors/:id` | Required | admin, super_admin | 業者詳細 |
| PATCH | `/admin/vendors/:id` | Required | admin, super_admin | 業者更新 |
| POST | `/admin/vendors/:id/approve` | Required | admin, super_admin | 業者承認 |
| POST | `/admin/vendors/:id/reject` | Required | admin, super_admin | 業者却下 |
| DELETE | `/admin/vendors/:id` | Required | admin, super_admin | 業者削除 |
| POST | `/admin/vendors/bulk-approve` | Required | admin, super_admin | 一括承認 |
| POST | `/admin/vendors/bulk-reject` | Required | admin, super_admin | 一括却下 |
| GET | `/admin/content` | Required | admin, super_admin | コンテンツ一覧 |
| POST | `/admin/content` | Required | admin, super_admin | コンテンツ作成 |
| PATCH | `/admin/content/:id` | Required | admin, super_admin | コンテンツ更新 |
| POST | `/admin/content/:id/publish` | Required | admin, super_admin | コンテンツ公開 |
| DELETE | `/admin/content/:id` | Required | admin, super_admin | コンテンツ削除 |
| GET | `/admin/chats` | Required | admin, super_admin | 全チャット一覧 |

---

### 雇用者 / Employer

| Method | Path | Auth | Roles | Description |
|--------|------|------|-------|-------------|
| GET | `/employer/dashboard` | Required | employer | ダッシュボード |
| GET | `/employer/properties` | Required | employer | 担当物件一覧 |
| GET | `/employer/properties/:id/chats` | Required | employer | 物件チャット一覧 |
| POST | `/employer/properties/:id/chats` | Required | employer | 物件チャット作成 |
| GET | `/employer/lands` | Required | employer | 担当土地一覧 |
| GET | `/employer/lands/:id/chats` | Required | employer | 土地チャット一覧 |

---

## Socket.io イベント / Socket.io Events

### /chat Namespace

Socket.io 名前空間: `/chat`（URL: `http://localhost:3001/chat`）

各チャットルームは `room:{chatRoomId}` という Socket.io room として管理されます。

**クライアント送信イベント (Client → Server)**

| Event | Payload | Description |
|-------|---------|-------------|
| `join_room` | `roomId: string` | チャットルームに参加（`socket.join('room:{id}')`） |
| `leave_room` | `roomId: string` | チャットルームを退出 |
| `send_message` | `{ roomId, content, senderId, senderName }` | メッセージ送信（DB保存 + broadcast） |

**サーバー送信イベント (Server → Client)**

| Event | Payload | Trigger |
|-------|---------|---------|
| `new_message` | `{ id, chatRoomId, senderId, sender: {id, name}, content, createdAt }` | 新着メッセージ |

### /notification Namespace

**サーバー送信イベント**

| Event | Payload | Trigger |
|-------|---------|---------|
| `notification` | Notification DB record | 通知発生時 |

---

## 認証フロー / Authentication Flow

```typescript
// JWT Access Token Payload
interface AuthPayload {
  userId: string;   // User UUID
  email: string;
  role: UserRole;   // landlord | homeowner | employer | admin | super_admin
}

// Token Lifetime
Access Token:  15 minutes (JWT_EXPIRES_IN env var)
Refresh Token: 30 days (stored in DB)

// Refresh Token Rotation
// 古いリフレッシュトークンは使用時に削除される（セキュリティ向上）
```

---

## RBAC - ロールベースアクセス制御 / Role-Based Access Control

```typescript
// src/middleware/rbac.ts
export const requireRoles = (...roles: UserRole[]) => middleware;
export const requireAdmin = requireRoles('admin', 'super_admin');
export const requireSuperAdmin = requireRoles('super_admin');
export const requireLandlordOrHomeowner = requireRoles('landlord', 'homeowner', 'admin', 'super_admin');
```

---

## ファイルアップロード / File Upload

**エンドポイント例:**
```
POST /my/lands/upload          thumbnail (single image)
POST /my/properties/upload     thumbnail + images (multiple)
POST /chats/:id/messages/file  file (PDF, DOCX, images)
```

**制限:**
- 画像: JPEG, PNG, WebP, GIF
- ドキュメント: PDF, DOC, DOCX
- 最大ファイルサイズ: 10MB (MAX_FILE_SIZE 環境変数で変更可能)
- 複数画像: 最大10枚

---

## AI機能 / AI Features

### タスク提案 (AI Task Suggestions)

```typescript
// GET /my/properties/:id/tasks/ai-suggestions
// 物件の設備状態・過去タスク・入居率を分析して
// 次に必要なタスクをAIが提案する

// OpenAI GPT-4 を使用
// レスポンス例
[
  {
    "title": "エアコンフィルター清掃",
    "description": "101号室のエアコンが最後の点検から6ヶ月経過しています",
    "priority": "medium",
    "isAiSuggested": true,
    "aiReason": "前回点検から180日経過。夏季前の定期メンテナンスを推奨します。"
  }
]
```

### 資産価値AI評価 (AI Asset Valuation)

```typescript
// POST /my/valuation
// 所有する土地・物件の情報を分析して資産価値を評価
// 将来の価値予測（1年後・3年後・5年後）も生成

// レスポンス例
{
  "totalCurrentValue": 120000000,
  "aiPredictedValue": 135000000,
  "predictionYear": 2027,
  "breakdown": {
    "lands": [{ "id": "...", "name": "...", "value": 50000000 }],
    "properties": [{ "id": "...", "name": "...", "value": 70000000 }]
  },
  "aiAnalysis": "大阪北部エリアの地価は安定した上昇傾向にあります..."
}
```

---

## 環境変数 / Environment Variables

```env
# データベース
DATABASE_URL="postgresql://user:password@localhost:5432/arvana_terra"

# Redis
REDIS_URL="redis://localhost:6379"

# JWT
JWT_SECRET="minimum-32-character-secret-key-here"
JWT_EXPIRES_IN="15m"

# サーバー設定
PORT=3001
NODE_ENV=development

# CORS (カンマ区切りで複数オリジン指定可能)
ALLOWED_ORIGINS="http://localhost:5174,http://localhost:3002"

# ファイルアップロード
UPLOAD_DIR="./uploads"
MAX_FILE_SIZE="10485760"  # 10MB in bytes

# OpenAI
OPENAI_API_KEY="sk-..."

# ログレベル (error, warn, info, debug)
LOG_LEVEL="info"
```

---

## 開発環境セットアップ / Development Setup

```bash
# リポジトリ内の arvana-terra-backend ディレクトリへ移動
cd arvana-terra-backend

# 依存関係インストール
npm install

# 環境変数設定
cp .env.example .env
# .env を編集して各種設定を行う

# PostgreSQL データベース作成
createdb arvana_terra

# Prisma マイグレーション実行
npm run prisma:migrate
# → "Migration 'init' applied successfully."

# Prisma クライアント生成
npm run prisma:generate

# (任意) シードデータ投入
npm run prisma:seed

# 開発サーバー起動 (ts-node-dev によるホットリロード)
npm run dev
# → Server running on port 3001
# → Database connected
# → Redis connected
# → Socket.io initialized

# Prisma Studio (DB閲覧ツール)
npm run prisma:studio
# → http://localhost:5555
```

---

## 本番デプロイ / Production Deployment

```bash
# TypeScript コンパイル
npm run build
# → dist/ ディレクトリに JavaScript ファイルが生成される

# DB マイグレーション (本番)
npm run prisma:migrate:prod
# → prisma migrate deploy (マイグレーションファイルを適用)

# 本番サーバー起動
npm start
# → node dist/index.js

# PM2 を使用する場合
pm2 start dist/index.js --name arvana-terra-backend
pm2 save
pm2 startup
```

---

## ロギング / Logging

```typescript
// Winston を使用した構造化ロギング
// ./logs/error.log  → エラーログ
// ./logs/combined.log → 全ログ
// コンソール出力 → 開発時

// リクエストログ (Morgan)
// [HTTP] GET /api/v1/my/lands 200 45ms

// アプリケーションログ (Winston)
// [INFO] Server running on port 3000
// [INFO] User uuid connected to /chat
// [ERROR] Database connection failed: ...
```
