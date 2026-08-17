# arvana-terra-admin - 詳細仕様 / Detailed Specification

## 概要 / Overview

Arvana Terra のプラットフォーム管理パネルです。2つの異なるロール（`admin`/`super_admin` と `employer`）がアクセスし、それぞれ異なる機能を利用します。Docker + Nginx によるコンテナデプロイに対応しています。

The platform administration panel for Arvana Terra. Two distinct roles (`admin`/`super_admin` and `employer`) access this panel with different feature sets. Supports containerized deployment with Docker + Nginx.

---

## テクノロジースタック / Tech Stack

| パッケージ | バージョン | 用途 |
|-----------|-----------|------|
| React | 18.2.0 | UI フレームワーク |
| TypeScript | 5.3.3 | 言語 |
| Vite | 5.1.0 | ビルドツール |
| Tailwind CSS | 3.4.1 | スタイリング |
| React Router DOM | 6.22.0 | ルーティング |
| Zustand | 4.5.0 | グローバル状態管理 |
| TanStack Query | 5.17.19 | サーバー状態管理 |
| Axios | 1.6.7 | HTTP クライアント |
| socket.io-client | 4.7.4 | WebSocket（チャット機能） |
| React Hook Form | 7.50.1 | フォーム管理 |
| @hookform/resolvers | 3.3.4 | Zod バリデーション連携 |
| Zod | 3.22.4 | バリデーション |
| Recharts | 2.10.3 | グラフ・統計表示 |
| @heroicons/react | 2.1.1 | アイコン |

---

## ユーザーロール詳細 / User Roles in Detail

### admin / super_admin（プラットフォーム管理者）

Arvana Terra プラットフォームを運営する管理者。以下の機能にアクセスできます：

| 機能 | admin | super_admin |
|------|:-----:|:-----------:|
| ダッシュボード（システム統計） | ✓ | ✓ |
| 業者管理（登録・承認・却下） | ✓ | ✓ |
| 公式コンテンツ管理（投稿・編集・削除） | ✓ | ✓ |
| チャット管理（全チャットルーム閲覧） | ✓ | ✓ |
| ユーザー管理 | - | ✓ |
| システム設定 | - | ✓ |

### employer（雇用者 - Arvana Work 連携）

Arvana Work から連携した雇用者。担当する物件・土地のチャット管理が主な用途です：

| 機能 | 説明 |
|------|------|
| 雇用者ダッシュボード | 担当物件・土地数・チャットアクティビティ概要 |
| 担当物件チャット | 自分が担当する物件のチャットルームの作成・管理・参加 |
| 担当土地チャット | 自分が担当する土地のチャットルームの作成・管理・参加 |
| 参加者管理 | チャット参加者の追加・削除 |
| メッセージ閲覧 | チャット履歴の閲覧（読み取り専用） |

---

## ディレクトリ構造 / Directory Structure

```
arvana-terra-admin/
├── src/
│   ├── api/
│   │   ├── client.ts          # Axios インスタンス（ベースURL: /api）
│   │   ├── auth.ts            # 管理者認証 API
│   │   ├── vendors.ts         # 業者管理 API
│   │   ├── content.ts         # コンテンツ管理 API
│   │   ├── chats.ts           # チャット管理・雇用者 API
│   │   ├── stats.ts           # 統計 API・雇用者ダッシュボード
│   │   └── users.ts           # ユーザー管理 API
│   ├── components/
│   │   └── ui/
│   │       ├── Badge.tsx      # ステータスバッジ
│   │       ├── Button.tsx     # ボタン
│   │       ├── Card.tsx       # カード
│   │       ├── Input.tsx      # 入力フィールド
│   │       └── Modal.tsx      # モーダル
│   ├── hooks/
│   │   ├── useAuth.ts         # 認証フック
│   │   └── useRBAC.ts         # ロール判定フック
│   ├── store/
│   │   └── auth.store.ts      # 認証状態（Zustand + persist）
│   └── types/
│       └── index.ts           # 全型定義
├── nginx/
│   └── nginx.conf             # Nginx 設定（SPA ルーティング）
├── Dockerfile                 # Multi-stage Docker ビルド
├── vite.config.ts
└── package.json
```

---

## 型定義 / Type Definitions

### 管理者認証

```typescript
type UserRole = 'arvana_admin' | 'super_admin' | 'employer'

interface AuthUser {
  id: string
  name: string
  email: string
  role: UserRole
  avatar?: string
  createdAt: string
}

interface AuthResponse {
  token: string
  refreshToken: string
  user: AuthUser
}
```

### 業者

```typescript
type VendorStatus = 'approved' | 'pending' | 'rejected' | 'deleted'
type VendorCategory =
  | 'plumbing'      // 水道・配管
  | 'electrical'    // 電気工事
  | 'cleaning'      // 清掃・クリーニング
  | 'renovation'    // リノベーション
  | 'landscaping'   // 造園・外構
  | 'roofing'       // 屋根工事
  | 'painting'      // 塗装
  | 'inspection'    // 建物検査
  | 'pest_control'  // 害虫・害獣駆除
  | 'moving'        // 引越し
  | 'other'

interface Vendor {
  id: string
  name: string
  category: VendorCategory
  contactName: string
  email: string
  phone: string
  address: string
  serviceAreas: string[]       // 対応都道府県
  description?: string
  website?: string
  status: VendorStatus
  connectedUsersCount: number  // 連携中のユーザー数
  createdAt: string
  updatedAt: string
  approvedAt?: string
  rejectedAt?: string
  rejectionReason?: string
}
```

### コンテンツ

```typescript
type ContentType = 'announcement' | 'case_study' | 'official' | 'tax_advice'
type ContentStatus = 'published' | 'draft' | 'archived'

interface ContentPost {
  id: string
  type: ContentType
  title: string
  content: string
  tags: string[]
  imageUrl?: string
  status: ContentStatus
  authorId: string
  authorName: string
  viewCount: number
  createdAt: string
  updatedAt: string
  publishedAt?: string
}
```

### 統計

```typescript
interface SystemStats {
  totalUsers: number
  totalProperties: number
  totalLands: number
  totalVendors: number
  activeVendors: number
  pendingVendors: number
  totalContent: number
  publishedContent: number
  userGrowthRate: number
  propertyGrowthRate: number
}

interface RegistrationTrend {
  month: string       // "2024-01"
  users: number
  properties: number
  lands: number
}

interface ActivityMetric {
  month: string
  chatMessages: number
  newVendors: number
  contentViews: number
}
```

---

## 管理画面一覧 / Admin Screens

### 共通

| 画面 | URL | 説明 |
|------|-----|------|
| ログイン | `/login` | 管理者・雇用者ログイン |
| ダッシュボード | `/` または `/dashboard` | ロールに応じたダッシュボード |
| プロフィール | `/profile` | アカウント情報表示 |

### admin / super_admin 専用画面

| 画面 | URL | 説明 |
|------|-----|------|
| システムダッシュボード | `/dashboard` | KPI・グラフ・システム統計 |
| 業者一覧 | `/vendors` | 全業者の一覧（ステータスフィルタ・検索） |
| 業者詳細 | `/vendors/:id` | 業者情報詳細 |
| 業者新規登録 | `/vendors/new` | 新規業者の登録フォーム |
| 業者編集 | `/vendors/:id/edit` | 業者情報の編集 |
| コンテンツ一覧 | `/content` | 公式コンテンツ一覧（ドラフト・公開・アーカイブ） |
| コンテンツ新規作成 | `/content/new` | コンテンツ作成フォーム |
| コンテンツ編集 | `/content/:id/edit` | コンテンツ編集 |
| チャット管理 | `/chats` | 全チャットルームの閲覧 |
| チャット詳細 | `/chats/:id` | チャット内容・参加者管理 |

### employer 専用画面

| 画面 | URL | 説明 |
|------|-----|------|
| 雇用者ダッシュボード | `/dashboard` | 担当物件・土地・チャット概要 |
| 担当物件一覧 | `/employer/properties` | 担当物件の一覧 |
| 物件チャット一覧 | `/employer/properties/:id/chats` | 物件のチャットルーム |
| 担当土地一覧 | `/employer/lands` | 担当土地の一覧 |
| 土地チャット一覧 | `/employer/lands/:id/chats` | 土地のチャットルーム |
| チャット詳細 | `/chats/:id` | リアルタイムチャット・参加者管理 |

---

## 業者管理ワークフロー / Vendor Management Workflow

```
管理者 (admin)                    システム                    クライアントアプリ
     │                              │                               │
     │── 業者情報を入力 ──────────►│                               │
     │   POST /admin/vendors         │                               │
     │   { status: 'approved' }     │── vendors テーブルに保存      │
     │                              │   isApproved = true           │
     │◄── 業者作成完了 ────────────│                               │
     │                              │                               │
     │── 業者一覧確認 ────────────►│── GET /admin/vendors          │
     │   (全ステータス表示)          │                               │
     │                              │                               │
     │── 承認待ち業者を承認 ────────►│                               │
     │   POST /admin/vendors/:id/approve                            │
     │                              │── isApproved = true に更新    │
     │◄── 承認完了 ────────────────│                               │
     │                              │                               │
     │                              │◄── GET /vendors ─────────────│
     │                              │   isApproved=true のみ返却   │
     │                              │── 承認済み業者一覧 ─────────►│
```

### 業者ステータス遷移

```
[登録]
  ↓
pending (承認待ち)
  ├── approve → approved (承認済み) → クライアントアプリに表示
  ├── reject  → rejected (却下) → 非表示
  └── [管理者が直接作成] → approved (即時承認)

approved
  ├── delete → deleted
  └── reject → rejected

rejected
  └── approve → approved
```

---

## チャット参加者管理ワークフロー / Chat Participant Management

### 雇用者がチャット参加者を管理する手順

```typescript
// 1. ユーザー検索
GET /users/search?q=田中
// → [{ id, name, email }]

// 2. 参加者として追加
POST /chats/:chatId/participants
{ userId: "target-user-uuid" }
// → ChatParticipant { id, userId, userName, role: 'member', joinedAt }

// 3. 参加者一覧確認
GET /chats/:chatId/participants
// → ChatParticipant[]

// 4. 参加者の削除
DELETE /chats/:chatId/participants/:participantId
```

---

## 公式コンテンツ管理 / Official Content Management

管理者は SNS フィードに公式コンテンツを投稿できます。これらの投稿は通常のユーザー投稿と区別して表示されます。

Administrators can post official content to the SNS feed. These posts are displayed distinctly from regular user posts.

```typescript
// コンテンツタイプ
type ContentType =
  | 'announcement'  // お知らせ - プラットフォームの重要なお知らせ
  | 'case_study'    // ケーススタディ - 成功事例の紹介
  | 'official'      // 公式情報 - 法規制・制度情報
  | 'tax_advice'    // 税務アドバイス - 地主・家主向け税務情報

// 公開フロー
draft → (edit) → draft
draft → publishPost() → published
published → unpublishPost() → draft (修正して再公開)
published → archivePost() → archived
```

---

## Axios クライアント設定 / Axios Client Configuration

```typescript
// src/api/client.ts
const BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api'

// Request Interceptor
// localStorage の 'arvana_admin_token' を Authorization ヘッダーに付与

// Response Interceptor
// 401 → localStorage をクリア → /login にリダイレクト
// (admin パネルはトークン自動リフレッシュなし - 再ログインを求める)
```

**Note:** Web クライアント (`arvana-terra-web`) のデフォルト API URL は `localhost:3000`、Admin パネルのデフォルトは `localhost:8000` です（本番では同一バックエンドを指します）。

---

## 状態管理 / State Management

### auth.store.ts

```typescript
// Zustand + persist (localStorage)
// key: 'arvana_admin_auth'

interface AuthState {
  user: AuthUser | null
  token: string | null
  refreshToken: string | null
  isAuthenticated: boolean

  setAuth: (user, token, refreshToken) => void  // ログイン
  clearAuth: () => void                           // ログアウト
  updateUser: (updates) => void                   // ユーザー情報更新
}

// ロール判定ヘルパー
const isAdminRole = (role: UserRole) => role === 'arvana_admin' || role === 'super_admin'
const isEmployerRole = (role: UserRole) => role === 'employer'
```

### useRBAC フック

```typescript
// src/hooks/useRBAC.ts
export function useRBAC() {
  const { user } = useAuthStore()

  return {
    isAdmin: isAdminRole(user?.role),
    isSuperAdmin: user?.role === 'super_admin',
    isEmployer: isEmployerRole(user?.role),
    canManageVendors: isAdminRole(user?.role),
    canManageContent: isAdminRole(user?.role),
    canViewAllChats: isAdminRole(user?.role),
    canViewEmployerFeatures: isEmployerRole(user?.role),
  }
}
```

---

## システムダッシュボード / System Dashboard (admin)

管理者ダッシュボードで表示される統計・グラフ：

```typescript
// KPI カード
totalUsers          総ユーザー数
totalProperties     総物件数
totalLands          総土地数
activeVendors       承認済み業者数
pendingVendors      承認待ち業者数
publishedContent    公開済みコンテンツ数
userGrowthRate      月次ユーザー増加率 (%)
propertyGrowthRate  月次物件増加率 (%)

// グラフ (Recharts)
RegistrationTrend   ユーザー・物件・土地の月次登録推移 (12ヶ月)
ActivityMetric      チャットメッセージ数・新規業者数・コンテンツ閲覧数 (12ヶ月)
```

---

## 雇用者ダッシュボード / Employer Dashboard

```typescript
interface EmployerDashboard {
  assignedPropertiesCount: number    // 担当物件数
  assignedLandsCount: number         // 担当土地数
  totalChatRooms: number             // 総チャットルーム数
  recentChatActivity: [              // 最近のチャットアクティビティ
    {
      chatRoomId: string
      chatRoomName: string
      targetName: string             // 物件名/土地名
      targetType: 'property' | 'land'
      lastMessage: string
      lastMessageAt: string
    }
  ]
}
```

---

## 環境変数 / Environment Variables

```env
# .env.local (開発時)
VITE_API_BASE_URL=http://localhost:3000/api

# .env.production
VITE_API_BASE_URL=https://api.arvana-terra.jp/api
```

---

## Docker デプロイ設定 / Docker Deployment Configuration

### Dockerfile

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
ARG VITE_API_BASE_URL
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### nginx/nginx.conf

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip 圧縮
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    # SPA ルーティング: 全リクエストを index.html にフォールバック
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API プロキシ（本番では不要・直接バックエンドに向ける）
    location /api {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # キャッシュ設定
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### Docker コマンド

```bash
# ビルド
docker build \
  --build-arg VITE_API_BASE_URL=https://api.arvana-terra.jp/api \
  -t arvana-terra-admin:latest \
  .

# 起動
docker run -d \
  --name arvana-terra-admin \
  -p 3001:80 \
  arvana-terra-admin:latest

# Docker Compose
version: '3.8'
services:
  admin:
    build:
      context: ./arvana-terra-admin
      args:
        VITE_API_BASE_URL: ${API_URL:-http://localhost:3000/api}
    ports:
      - "3001:80"
    depends_on:
      - backend
    restart: unless-stopped
```

---

## 開発セットアップ / Development Setup

```bash
cd arvana-terra-admin

# 依存関係インストール
npm install

# 環境変数設定
echo "VITE_API_BASE_URL=http://localhost:3000/api" > .env.local

# 開発サーバー起動
npm run dev
# → http://localhost:3001

# 型チェック
npm run type-check

# リント
npm run lint
```

---

## サービスエリア定数 / Service Area Constants

```typescript
// src/types/index.ts
export const SERVICE_AREAS = [
  '大阪府', '奈良県', '東京都', '神奈川県',
  '埼玉県', '千葉県', '京都府', '兵庫県',
  '愛知県', '福岡県', '北海道', '宮城県',
  '広島県', '静岡県',
]

// 業者カテゴリの日本語ラベル
export const VENDOR_CATEGORY_LABELS: Record<VendorCategory, string> = {
  plumbing: '水道・配管',
  electrical: '電気工事',
  cleaning: '清掃・クリーニング',
  renovation: 'リノベーション',
  landscaping: '造園・外構',
  roofing: '屋根工事',
  painting: '塗装',
  inspection: '建物検査',
  pest_control: '害虫・害獣駆除',
  moving: '引越し',
  other: 'その他',
}
```
