# Arvana Terra - システムアーキテクチャ / System Architecture

## 概要 / Overview

Arvana Terra は、マイクロサービス的な責任分離を取りつつも、単一のバックエンドAPIサーバーを中心としたモノリシックAPIアーキテクチャを採用しています。スケーラビリティよりも開発速度・保守性を優先した設計です。

Arvana Terra uses a monolithic API architecture centered on a single backend API server, with clear separation of concerns across client applications. The design prioritizes development velocity and maintainability over extreme scalability.

---

## コンポーネント構成 / Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Arvana Terra - System Components                    │
│                                                                          │
│  Client Layer                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                                                                  │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┌────────┐  │    │
│  │  │  Web Client  │  │  Admin Panel │  │   iOS    │  │Android │  │    │
│  │  │  React/Vite  │  │  React/Vite  │  │  SwiftUI │  │Compose │  │    │
│  │  │  :5174       │  │  Docker:3001 │  │  iOS 17+ │  │API 34+ │  │    │
│  │  └──────┬───────┘  └──────┬───────┘  └────┬─────┘  └───┬────┘  │    │
│  │         │                 │                │             │        │    │
│  └─────────┼─────────────────┼────────────────┼─────────────┼────────┘   │
│            │                 │                │             │             │
│  ┌─────────▼─────────────────▼────────────────▼─────────────▼────────┐   │
│  │                     REST API (HTTP/HTTPS)                          │   │
│  │                     Base URL: /api/v1                              │   │
│  └──────────────────────────────┬────────────────────────────────────┘   │
│                                 │                                         │
│  Application Layer              │                                         │
│  ┌──────────────────────────────▼────────────────────────────────────┐   │
│  │                    arvana-terra-backend                            │   │
│  │                    Node.js / TypeScript                            │   │
│  │                    Express.js 4.18                                 │   │
│  │                    Port: 3000                                      │   │
│  │                                                                    │   │
│  │  ┌────────────────────────────────────────────────────────────┐   │   │
│  │  │  Route Layer                                               │   │   │
│  │  │  /auth  /my/lands  /my/properties  /tasks  /chats         │   │   │
│  │  │  /employees  /vendors  /sns  /admin  /employer             │   │   │
│  │  └───────────────────────────┬────────────────────────────────┘   │   │
│  │                              │                                     │   │
│  │  ┌────────────────────────────────────────────────────────────┐   │   │
│  │  │  Middleware Layer                                          │   │   │
│  │  │  authenticate() → requireRoles() → validation → handler   │   │   │
│  │  └───────────────────────────┬────────────────────────────────┘   │   │
│  │                              │                                     │   │
│  │  ┌────────────────────────────────────────────────────────────┐   │   │
│  │  │  Service Layer                                             │   │   │
│  │  │  AuthService  LandService  PropertyService  RoomService    │   │   │
│  │  │  EquipmentService  (business logic here)                   │   │   │
│  │  └───────────────────────────┬────────────────────────────────┘   │   │
│  │                              │                                     │   │
│  │  ┌────────────────────────────────────────────────────────────┐   │   │
│  │  │  Socket.io Layer                                           │   │   │
│  │  │  Namespace /chat          Namespace /notification          │   │   │
│  │  └───────────────────────────┬────────────────────────────────┘   │   │
│  │                              │                                     │   │
│  └──────────────────────────────┼────────────────────────────────────┘   │
│                                 │                                         │
│  Data Layer                     │                                         │
│  ┌──────────────────────────────▼────────────────────────────────────┐   │
│  │                                                                    │   │
│  │  ┌─────────────────────┐         ┌──────────────────────────┐    │   │
│  │  │    PostgreSQL 16     │         │        Redis 7            │    │   │
│  │  │  (via Prisma ORM)   │         │  Session / Cache /        │    │   │
│  │  │                     │         │  Rate Limiting            │    │   │
│  │  │  Tables:            │         └──────────────────────────┘    │   │
│  │  │  - users            │                                          │   │
│  │  │  - lands            │         ┌──────────────────────────┐    │   │
│  │  │  - properties       │         │     File Storage          │    │   │
│  │  │  - rooms            │         │  ./uploads (multer)       │    │   │
│  │  │  - tenants          │         │  images, PDFs, docs       │    │   │
│  │  │  - payments         │         └──────────────────────────┘    │   │
│  │  │  - equipment        │                                          │   │
│  │  │  - contracts        │         ┌──────────────────────────┐    │   │
│  │  │  - chat_rooms       │         │      OpenAI API           │    │   │
│  │  │  - chat_messages    │         │  GPT-4 for:               │    │   │
│  │  │  - tasks            │         │  - Task suggestions       │    │   │
│  │  │  - employees        │         │  - Asset valuation        │    │   │
│  │  │  - vendors          │         │  - Business analysis      │    │   │
│  │  │  - sns_posts        │         └──────────────────────────┘    │   │
│  │  │  - notifications    │                                          │   │
│  │  │  - ...              │                                          │   │
│  │  └─────────────────────┘                                          │   │
│  └───────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## データフロー / Data Flow Diagrams

### 認証フロー / Authentication Flow

```
Client                          Backend                      Database
  │                                │                              │
  │── POST /auth/login ──────────►│                              │
  │   { email, password }          │── findUser(email) ──────────►│
  │                                │◄── User record ─────────────│
  │                                │── bcrypt.compare() ─────────►│(local)
  │                                │── jwt.sign(payload) ─────────►│(local)
  │                                │── refreshToken = uuid() ─────►│(local)
  │                                │── INSERT refresh_tokens ─────►│
  │◄── { accessToken,              │◄── OK ──────────────────────│
  │      refreshToken, user } ─────│                              │
  │                                │                              │
  │                                │                              │
  │── GET /my/lands ──────────────►│                              │
  │   Authorization: Bearer <token>│── jwt.verify() ─────────────►│(local)
  │                                │── prisma.land.findMany() ────►│
  │◄── [ land1, land2, ... ] ──────│◄── Land[] ──────────────────│
```

### Socket.io チャットフロー / Socket.io Chat Flow

```
Client A           Backend Socket.io           Client B
   │                      │                        │
   │── connect ──────────►│ /chat namespace         │
   │   auth: { token }    │── jwt.verify() ──►      │
   │                      │                         │
   │── join_chat ─────────►│                        │
   │   { chatRoomId }     │── DB check participant ─►│(DB)
   │                      │── socket.join(room) ───  │
   │◄── user_joined ───── │                         │
   │                      │                         │
   │── send_message ──────►│                        │
   │   { chatRoomId,      │── DB check participant  │
   │     content }        │── prisma.chatMessage.create()
   │                      │── chatRoom.update()     │
   │                      │                         │
   │◄── new_message ───── │── new_message ─────────►│
   │   (to all in room)   │   (broadcast to room)   │
```

### ファイルアップロードフロー / File Upload Flow

```
Client                          Backend                    Filesystem
  │                                │                            │
  │── POST /my/properties ────────►│                            │
  │   Content-Type: multipart/form │                            │
  │   thumbnail: <file>            │── multer.diskStorage ─────►│
  │   images: <files[]>            │   ./uploads/<uuid>.ext     │
  │                                │── prisma.property.create() ►│(DB)
  │◄── { property with URLs } ─────│   thumbnailUrl: /uploads/..│
```

---

## 技術選定の理由 / Technology Decisions

### バックエンド: Node.js + TypeScript + Express.js

**理由:**
- チーム全体でJavaScript/TypeScriptの知識を共有でき、フロントエンドとの型共有が容易
- Express.js はシンプルで学習コストが低く、小〜中規模APIに最適
- TypeScript によりコンパイル時の型チェックでバグを早期発見
- Node.js のイベントループが WebSocket (Socket.io) と相性が良い

### ORM: Prisma

**理由:**
- TypeScript ファーストで型安全なDBアクセス
- スキーマファイル (schema.prisma) が単一の真実のソース
- マイグレーション管理が容易
- PostgreSQL・MySQL・SQLite 等への切り替えが可能

### データベース: PostgreSQL

**理由:**
- JSONB 型サポート（readings, parties, breakdown 等の非構造化データ）
- 配列型サポート（imageUrls, tags, serviceAreas 等）
- ACID 準拠の強力なトランザクション
- Prisma との相性が良い

### キャッシュ: Redis

**理由:**
- セッション管理・レート制限のためのインメモリストア
- 将来的なセッション共有（水平スケーリング時）に対応
- ioredis による TypeScript サポート

### フロントエンド: React + Vite + TypeScript

**理由:**
- React エコシステムの豊富なライブラリ
- Vite による高速HMR（開発体験の向上）
- Tailwind CSS によるユーティリティファーストのスタイリング

### 状態管理: Zustand + TanStack Query

**理由:**
- Zustand: 軽量でシンプルなグローバル状態管理（認証・チャット・通知）
- TanStack Query: サーバー状態のキャッシュ・同期・更新を宣言的に管理
- Redux より学習コストが低く、ボイラープレートが少ない

### モバイル: ネイティブ (Swift/Kotlin)

**理由:**
- 最高のユーザー体験とパフォーマンス
- プラットフォーム固有のUI/UXガイドラインへの完全準拠
- Siri・ウィジェット等のOS機能へのアクセス

---

## APIデザイン原則 / API Design Principles

### RESTful 設計

```
GET    /api/v1/my/lands            → 自分の土地一覧
POST   /api/v1/my/lands            → 土地の新規作成
GET    /api/v1/my/lands/:id        → 土地の詳細取得
PATCH  /api/v1/my/lands/:id        → 土地の更新（部分更新）
DELETE /api/v1/my/lands/:id        → 土地の削除

GET    /api/v1/lands               → 公開土地一覧（認証不要）
GET    /api/v1/lands/:id           → 公開土地詳細（認証不要）
```

### ネストリソース

```
GET  /my/properties/:id/rooms              → 物件の部屋一覧
GET  /my/properties/:id/rooms/:roomId      → 部屋詳細
PATCH /my/properties/:id/rooms/:roomId     → 部屋更新

GET  /my/properties/:id/equipment          → 設備一覧
GET  /my/properties/:id/equipment/floor/:n → フロア別設備

GET  /my/properties/:id/contracts          → 契約書一覧
POST /my/properties/:id/contracts          → 契約書作成
```

### レスポンスフォーマット

```typescript
// 成功時
{
  "success": true,
  "data": { ... }
}

// ページネーション付き
{
  "data": [...],
  "total": 100,
  "page": 1,
  "limit": 20,
  "totalPages": 5
}

// エラー時
{
  "success": false,
  "error": "エラーメッセージ"
}
```

### 認証

```
すべての /my/*, /tasks/*, /chats/*, /employees/*, /vendors/*, /sns/*
エンドポイントは認証必須。

Authorization: Bearer <accessToken>

/auth/login, /auth/register はパブリック。
/lands, /properties (GET) はパブリック（公開物件のみ）。
```

---

## リアルタイム通信アーキテクチャ / Real-time Communication Architecture

### Socket.io Namespaces

```
ws://host:3000
  ├── /chat         チャットメッセージのリアルタイム配信
  └── /notification プッシュ通知のリアルタイム配信
```

### /chat Namespace

```
Connection: { auth: { token: "<JWT>" } }

Rooms (socket.join):
  - "user:<userId>"       → 個人ルーム（ターゲット通知用）
  - "chat:<chatRoomId>"   → チャットルーム

Client Events (emit):
  join_chat  { chatRoomId: string }
  leave_chat { chatRoomId: string }
  send_message { chatRoomId: string, content: string, messageType?: string }
  typing { chatRoomId: string }

Server Events (on):
  user_joined  { chatRoomId: string, userId: string }
  user_left    { chatRoomId: string, userId: string }
  new_message  ChatMessage (DB record with sender info)
  user_typing  { userId: string, chatRoomId: string }
  error        { message: string }
```

### /notification Namespace

```
Connection: { auth: { token: "<JWT>" } }

Rooms (socket.join):
  - "user:<userId>"   → 個人通知ルーム

Server Events (emit):
  notification  Notification (DB record)
```

---

## データベース設計概要 / Database Design Overview

### エンティティ関係 / Entity Relationships

```
User (1) ──── (N) Land
User (1) ──── (N) Property
Land (1) ──── (N) Property      ← 土地の上に建物
Property (1) ── (N) Room
Room (1) ───── (1) Tenant       ← 1部屋に1入居者
Room (1) ───── (N) Payment
Property (1) ── (N) Equipment
Property (1) ── (N) SmartDeviceData
Property (1) ── (N) Contract
Land (1) ────── (N) Contract
Property (1) ── (N) ChatRoom
Land (1) ────── (N) ChatRoom
ChatRoom (1) ── (N) ChatParticipant
ChatRoom (1) ── (N) ChatMessage
Property (1) ── (N) Task
Land (1) ────── (N) Task
User (1) ───── (N) Employee
Vendor (M) ─── (N) User         ← UserVendor junction
User (1) ───── (N) SnsPost
SnsPost (1) ── (N) SnsLike
SnsPost (1) ── (N) SnsComment
SnsPost (1) ── (1) SnsEvent
User (1) ───── (1) UserPreference
User (1) ───── (N) Notification
User (1) ───── (N) AssetValuation
User (1) ───── (N) BusinessOpportunity
```

### 主要テーブルの設計ポイント

| テーブル | 設計ポイント |
|----------|-------------|
| `users` | ソフトデリート (`isActive` フラグ) |
| `lands` | `imageUrls String[]`, `tags String[]` → PostgreSQL配列型 |
| `properties` | `landId` は nullable（土地なし物件も可） |
| `rooms` | `roomId` が `tenant` テーブルで UNIQUE → 1部屋1入居者 |
| `smart_device_data` | `readings Json` → 週次データをJSONBで格納 |
| `contracts` | `parties Json` → 当事者リストをJSONBで格納 |
| `chat_messages` | `readBy String[]` → 既読ユーザーIDをPostgreSQL配列で管理 |
| `tasks` | `isAiSuggested Boolean` → AI提案フラグ |
| `vendors` | `isApproved Boolean` → 管理者承認フラグ |
| `user_vendors` | UNIQUE(userId, vendorId) → 重複連携防止 |

---

## セキュリティアーキテクチャ / Security Architecture

### 認証・認可レイヤー

```typescript
// リクエスト処理フロー
Request
  → Helmet (セキュリティヘッダー)
  → CORS (オリジン制限)
  → Rate Limiting (express-rate-limit)
  → authenticate() (JWT検証)
  → requireRoles() (RBAC)
  → Route Handler
  → Response
```

### JWT 設計

```
Access Token:
  - Payload: { userId, email, role }
  - 有効期限: 15分 (JWT_EXPIRES_IN 環境変数で変更可能)
  - 署名: HS256 (JWT_SECRET)

Refresh Token:
  - UUIDv4 (ランダム文字列)
  - DB の refresh_tokens テーブルに保存
  - 有効期限: 30日
  - 使い捨て（リフレッシュ時に旧トークン削除）
```

### パスワードセキュリティ

```
- bcryptjs によるハッシュ化（SALT_ROUNDS = 12）
- 平文パスワードは一切DBに保存しない
- レスポンスにパスワードフィールドを含めない（Prismaの select で除外）
```

### ファイルアップロードセキュリティ

```
- MIMEタイプ検証（ホワイトリスト方式）
- ファイルサイズ制限（デフォルト10MB）
- ファイル名をUUID+タイムスタンプで再生成（パストラバーサル防止）
- 画像: jpeg, png, webp, gif のみ
- ドキュメント: pdf, doc, docx 追加
```

---

## デプロイメントアーキテクチャ / Deployment Architecture

### 開発環境 / Development

```
┌─────────────────────────────────────────────┐
│  Developer Machine                          │
│                                             │
│  node dev (backend) → :3000                 │
│  vite dev (web)     → :5174                 │
│  vite dev (admin)   → :3001                 │
│  psql               → :5432                 │
│  redis-cli          → :6379                 │
└─────────────────────────────────────────────┘
```

### 本番環境 / Production (推奨構成)

```
Internet
    │
    ▼
┌───────────────────┐
│   Nginx / CDN     │  静的ファイル配信
│   SSL Termination │  HTTPS → HTTP
└─────┬─────────────┘
      │
      ├── /          → arvana-terra-web (React SPA)
      ├── /admin      → arvana-terra-admin (Docker + Nginx)
      └── /api/*      → arvana-terra-backend (:3000)
                            │
                    ┌───────┴────────┐
                    │                │
              PostgreSQL 16     Redis 7
              (RDS / managed)  (ElastiCache)
```

### Docker 構成（管理パネル）

```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### Nginx 設定（SPA ルーティング）

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:3000;
    }
}
```
