# arvana-terra-web - 詳細仕様 / Detailed Specification

## 概要 / Overview

地主・家主向けのメインWebクライアントアプリケーションです。React/TypeScript + Vite で構築され、Tailwind CSS によるスタイリング、Zustand + TanStack Query による状態管理、Socket.io によるリアルタイム通信を実装しています。

The primary web client application for landlords and homeowners. Built with React/TypeScript + Vite, styled with Tailwind CSS, state management via Zustand + TanStack Query, and real-time communication via Socket.io.

---

## テクノロジースタック / Tech Stack

| パッケージ | バージョン | 用途 |
|-----------|-----------|------|
| React | 18.3.1 | UI フレームワーク |
| TypeScript | 5.7.2 | 言語 |
| Vite | 5.4.11 | ビルドツール・開発サーバー |
| Tailwind CSS | 3.4.17 | ユーティリティファーストCSS |
| React Router DOM | 6.28.1 | クライアントサイドルーティング |
| Zustand | 4.5.5 | グローバル状態管理 |
| TanStack Query | 5.62.7 | サーバー状態管理・キャッシュ |
| Axios | 1.7.9 | HTTP クライアント |
| socket.io-client | 4.8.1 | WebSocket クライアント |
| React Hook Form | 7.54.2 | フォーム管理 |
| Zod | 3.24.1 | バリデーションスキーマ |
| Recharts | 2.15.0 | グラフ・チャート |
| @heroicons/react | 2.2.0 | アイコン |
| date-fns | 3.6.0 | 日付ユーティリティ |

---

## ディレクトリ構造 / Directory Structure

```
arvana-terra-web/
├── src/
│   ├── main.tsx               # エントリポイント
│   ├── App.tsx                # ルーティング定義・プロバイダー設定
│   ├── api/
│   │   ├── client.ts          # Axios インスタンス設定（インターセプター含む）
│   │   ├── auth.ts            # 認証 API
│   │   ├── lands.ts           # 土地 API
│   │   ├── properties.ts      # 物件 API
│   │   ├── rooms.ts           # 部屋・支払い API
│   │   ├── equipment.ts       # 設備・スマートデバイス API
│   │   ├── contracts.ts       # 契約書 API
│   │   ├── tasks.ts           # タスク API
│   │   ├── employees.ts       # 従業員 API
│   │   └── chats.ts           # チャット API
│   ├── components/
│   │   ├── ui/                # 汎用UIコンポーネント
│   │   ├── layout/            # レイアウトコンポーネント
│   │   ├── land/              # 土地関連コンポーネント
│   │   ├── property/          # 物件関連コンポーネント
│   │   ├── equipment/         # 設備関連コンポーネント
│   │   └── chat/              # チャット関連コンポーネント
│   ├── hooks/
│   │   ├── useAuth.ts         # 認証フック
│   │   ├── useLands.ts        # 土地データフック
│   │   ├── useProperties.ts   # 物件データフック
│   │   ├── useSocket.ts       # Socket.io フック
│   │   ├── useChat.ts         # チャットフック
│   │   └── useNotifications.ts # 通知フック
│   ├── pages/
│   │   ├── dashboard/         # ダッシュボード
│   │   ├── lands/             # 土地管理ページ
│   │   ├── properties/        # 物件管理ページ
│   │   ├── rooms/             # 部屋管理ページ
│   │   ├── employees/         # 従業員管理ページ
│   │   ├── chat/              # チャットページ
│   │   │   ├── ChatListPage.tsx   # チャットルーム一覧（トピック一覧）
│   │   │   └── ChatRoomPage.tsx   # チャットルーム（リアルタイムメッセージ画面）
│   │   ├── schedule/          # スケジュール管理ページ
│   │   ├── settings/          # 設定ページ
│   │   └── public/            # 公開ページ（認証不要）
│   ├── hooks/
│   │   └── useChat.ts         # Socket.io チャットフック（join/send/receive）
│   ├── store/
│   │   ├── auth.store.ts      # 認証グローバル状態
│   │   ├── chat.store.ts      # チャット状態
│   │   └── notification.store.ts # 通知状態
│   ├── types/
│   │   └── index.ts           # 全型定義
│   └── utils/
│       ├── format.ts          # 数値・日付フォーマット
│       ├── constants.ts       # 定数
│       └── index.ts           # ユーティリティ関数
├── public/
├── index.html
├── vite.config.ts
├── tailwind.config.ts
└── package.json
```

---

## ページ・スクリーン一覧 / Page & Screen List

### 公開ページ（認証不要）/ Public Pages

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/` | ランディングページ | プラットフォーム紹介 |
| `/login` | ログイン | メール・パスワードでのログイン |
| `/register` | 新規登録 | アカウント作成（landlord/homeowner/employer選択） |
| `/lands` | 公開土地一覧 | isPublic=true の土地一覧（フィルタ・検索可能） |
| `/lands/:id` | 公開土地詳細 | 土地の詳細情報表示 |
| `/properties` | 公開物件一覧 | isPublic=true の物件一覧 |
| `/properties/:id` | 公開物件詳細 | 物件の詳細情報表示 |

### 認証後ページ（要ログイン）/ Authenticated Pages

#### ダッシュボード / Dashboard

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/dashboard` | ダッシュボード | 所有資産サマリー・最新通知・タスク概要 |

#### 土地管理 / Land Management (landlord のみ)

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/my/lands` | 土地一覧 | 自分の土地一覧（フィルタ・ステータス管理） |
| `/my/lands/new` | 土地新規作成 | 土地情報入力フォーム |
| `/my/lands/:id` | 土地詳細 | 土地情報・サイドバーにチャットリンク |
| `/my/lands/:id/sale-request` | 売出し申請 | 売出し申請フォーム |
| `/my/lands/:id/chat?type=land&name=...` | 土地チャット一覧 | トピック一覧・新規作成（`ChatListPage`） |

#### 物件管理 / Property Management

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/my/properties` | 物件一覧 | 自分の物件一覧（フィルタ・検索） |
| `/my/properties/new` | 物件新規作成 | 物件情報入力フォーム |
| `/my/properties/:id` | 物件詳細 | 物件情報・サイドバーにチャットリンク |
| `/my/properties/:id/sale-request` | 売出し申請 | 売出し申請フォーム |
| `/my/properties/:id/chat?type=property&name=...` | 物件チャット一覧 | トピック一覧・新規作成（`ChatListPage`） |
| `/properties/:propertyId/rooms` | 部屋一覧 | 物件内の部屋一覧 |
| `/properties/:propertyId/rooms/new` | 部屋新規作成 | 部屋情報入力フォーム |
| `/properties/:propertyId/rooms/:roomId` | 部屋詳細 | 部屋情報・入居者・支払い履歴 |

#### チャット / Chat

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/my/lands/:id/chat` | 土地チャット一覧 | `?type=land&name=...` で絞り込み |
| `/my/properties/:id/chat` | 物件チャット一覧 | `?type=property&name=...` で絞り込み |
| `/my/employees/:id/chat` | 従業員チャット一覧 | `?type=employee&name=...` で絞り込み |
| `/my/chat/:roomId` | チャットルーム | リアルタイムメッセージ画面（Socket.io） |

#### タスク管理 / Task Management

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/tasks` | タスク一覧（全体） | 全資産のタスク一覧・カンバン表示 |

#### 分析・可視化 / Analytics & Visualization

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/visualization` | 資産分析 | 月次収入・支出グラフ・入居率推移 |
| `/valuation` | 資産価値評価 | AI による資産価値・将来予測 |

#### コミュニティ / Community

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/sns` | SNS フィード | 地主・家主コミュニティの投稿フィード |
| `/sns/new` | 投稿作成 | 新規SNS投稿 |
| `/sns/:id` | 投稿詳細 | 投稿詳細・コメント |

#### ビジネスチャンス / Business Opportunities

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/opportunities` | ビジネスチャンス | 土地購入・売却・開発機会一覧 |
| `/opportunities/:id` | チャンス詳細 | 詳細・AI分析コメント |

#### 業者 / Vendors

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/vendors` | 業者一覧 | 承認済み業者の検索・閲覧 |
| `/vendors/:id` | 業者詳細 | 業者の詳細情報・連絡先 |

#### 設定 / Settings

| URL | ページ名 | 説明 |
|-----|---------|------|
| `/settings` | アカウント設定 | プロフィール・通知設定 |
| `/settings/profile` | プロフィール編集 | 名前・電話番号・住所・自己紹介 |
| `/settings/notifications` | 通知設定 | 通知のオン/オフ設定 |
| `/settings/preferences` | 地域設定 | 関心地域の設定 |

---

## コンポーネントアーキテクチャ / Component Architecture

```
src/components/
├── ui/                          # 汎用UIコンポーネント
│   ├── Button.tsx               # ボタン（primary/secondary/danger）
│   ├── Input.tsx                # テキスト入力
│   ├── Select.tsx               # セレクトボックス
│   ├── Modal.tsx                # モーダルダイアログ
│   ├── Card.tsx                 # カードコンテナ
│   ├── Badge.tsx                # ステータスバッジ
│   ├── Spinner.tsx              # ローディングスピナー
│   ├── Pagination.tsx           # ページネーション
│   ├── Table.tsx                # テーブルコンポーネント
│   ├── Toast.tsx                # トースト通知
│   └── EmptyState.tsx           # 空状態表示
│
├── layout/                      # レイアウト
│   ├── AppLayout.tsx            # メインレイアウト（サイドバー含む）
│   ├── Sidebar.tsx              # サイドナビゲーション
│   ├── Header.tsx               # ヘッダー（通知・プロフィール）
│   └── AuthLayout.tsx           # 認証ページレイアウト
│
├── land/                        # 土地関連
│   ├── LandCard.tsx             # 土地カード（一覧表示用）
│   ├── LandForm.tsx             # 土地作成・編集フォーム
│   ├── LandStatusBadge.tsx      # ステータスバッジ
│   └── LandMap.tsx              # 地図表示
│
├── property/                    # 物件関連
│   ├── PropertyCard.tsx         # 物件カード
│   ├── PropertyForm.tsx         # 物件フォーム
│   ├── RoomGrid.tsx             # 部屋グリッド表示
│   ├── RoomCard.tsx             # 部屋カード
│   ├── TenantForm.tsx           # 入居者フォーム
│   └── PaymentTable.tsx         # 支払い一覧テーブル
│
├── equipment/                   # 設備関連
│   ├── EquipmentList.tsx        # 設備一覧
│   ├── EquipmentStatusBadge.tsx # 設備ステータスバッジ
│   ├── SmartDeviceChart.tsx     # スマートデバイスグラフ
│   └── FloorMap.tsx             # フロアマップ
│
└── chat/                        # チャット関連
    ├── ChatRoomList.tsx          # チャットルーム一覧
    ├── ChatMessages.tsx          # メッセージ一覧
    ├── MessageBubble.tsx         # メッセージバブル
    ├── ChatInput.tsx             # メッセージ入力欄
    └── TypingIndicator.tsx       # タイピング中インジケーター
```

---

## 状態管理 / State Management

### Zustand Stores (グローバル状態)

**auth.store.ts** - 認証状態

```typescript
interface AuthState {
  user: User | null
  tokens: AuthTokens | null    // { accessToken, refreshToken }
  isAuthenticated: boolean

  setAuth: (user: User, tokens: AuthTokens) => void
  clearAuth: () => void
  updateUser: (user: Partial<User>) => void
}
// localStorage に永続化 (persist middleware)
// key: "arvana_terra_auth"
```

**chat.store.ts** - チャット状態

```typescript
interface ChatState {
  messages: Record<string, ChatMessage[]>  // topicId → messages
  unreadCounts: Record<string, number>

  addMessage: (topicId: string, message: ChatMessage) => void
  setMessages: (topicId: string, messages: ChatMessage[]) => void
  markAsRead: (topicId: string) => void
}
```

**notification.store.ts** - 通知状態

```typescript
interface NotificationState {
  notifications: Notification[]
  unreadCount: number

  addNotification: (notification: Notification) => void
  markAsRead: (id: string) => void
  markAllAsRead: () => void
}
```

### TanStack Query (サーバー状態)

```typescript
// キャッシュ戦略例
// 土地一覧: staleTime 5分、gcTime 10分
const { data: lands } = useQuery({
  queryKey: ['my-lands', filters],
  queryFn: () => landsApi.getMyLands(filters),
  staleTime: 5 * 60 * 1000,
})

// 物件詳細: staleTime 2分（頻繁に更新される可能性）
const { data: property } = useQuery({
  queryKey: ['my-property', id],
  queryFn: () => propertiesApi.getMyPropertyById(id),
  staleTime: 2 * 60 * 1000,
})

// ミューテーション例
const updateLand = useMutation({
  mutationFn: ({ id, data }) => landsApi.updateLand(id, data),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['my-lands'] })
    queryClient.invalidateQueries({ queryKey: ['my-land', id] })
    toast.success('土地情報を更新しました')
  }
})
```

---

## API インテグレーション / API Integration

### Axios クライアント設定

```typescript
// src/api/client.ts
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api/v1'

// インターセプター:
// Request: localStorage から accessToken を取得して Authorization ヘッダーに付与
// Response: 401 エラー時に refreshToken を使ってトークンを自動リフレッシュ
//           リフレッシュ失敗時は localStorage をクリアして /login にリダイレクト
```

### API モジュール構成

```typescript
// src/api/lands.ts
export const landsApi = {
  getPublicLands: (filters?) => GET /lands
  getPublicLandById: (id) => GET /lands/:id
  getMyLands: (filters?) => GET /my/lands
  getMyLandById: (id) => GET /my/lands/:id
  createLand: (data) => POST /my/lands
  updateLand: (id, data) => PATCH /my/lands/:id
  deleteLand: (id) => DELETE /my/lands/:id
}

// src/api/properties.ts
export const propertiesApi = {
  getPublicProperties: (filters?) => GET /properties
  getMyProperties: (filters?) => GET /my/properties
  createProperty: (data) => POST /my/properties
  updateProperty: (id, data) => PATCH /my/properties/:id
  deleteProperty: (id) => DELETE /my/properties/:id
  getDashboardStats: (id) => GET /my/properties/:id/stats
}
```

---

## Socket.io インテグレーション / Socket.io Integration

```typescript
// src/hooks/useSocket.ts
const WS_URL = import.meta.env.VITE_WS_URL || 'ws://localhost:3000'

export function useSocket() {
  // Socket.io 接続設定
  const socket = io(WS_URL, {
    auth: { token: accessToken },
    transports: ['websocket'],
    reconnection: true,
    reconnectionDelay: 1000,
    reconnectionAttempts: 5,
  })

  // イベントリスナー
  socket.on('chat:message', (message) => addMessage(message.topicId, message))
  socket.on('notification', (notification) => addNotification(notification))

  // チャットへの参加・退出
  const joinRoom = (topicId) => socket.emit('chat:join', { topicId })
  const leaveRoom = (topicId) => socket.emit('chat:leave', { topicId })
  const sendMessage = (topicId, content) => socket.emit('chat:send', { topicId, content })

  return { socket, isConnected, joinRoom, leaveRoom, sendMessage }
}

// src/hooks/useChat.ts - 特定チャットルーム用フック
export function useChatSocket(topicId: string) {
  const { joinRoom, leaveRoom, sendMessage } = useSocket()

  useEffect(() => {
    joinRoom(topicId)
    return () => leaveRoom(topicId)
  }, [topicId])

  return { sendMessage: (content) => sendMessage(topicId, content) }
}
```

---

## ルーティング / Routing

```typescript
// src/App.tsx (概念的な構成)
<BrowserRouter>
  <Routes>
    {/* 公開ルート */}
    <Route path="/" element={<LandingPage />} />
    <Route path="/login" element={<LoginPage />} />
    <Route path="/register" element={<RegisterPage />} />
    <Route path="/lands" element={<PublicLandsPage />} />
    <Route path="/lands/:id" element={<PublicLandDetailPage />} />
    <Route path="/properties" element={<PublicPropertiesPage />} />
    <Route path="/properties/:id" element={<PublicPropertyDetailPage />} />

    {/* 認証必須ルート */}
    <Route element={<PrivateRoute />}>
      <Route element={<AppLayout />}>
        <Route path="/dashboard" element={<DashboardPage />} />

        {/* 土地管理 */}
        <Route path="/my/lands" element={<MyLandsPage />} />
        <Route path="/my/lands/new" element={<LandNewPage />} />
        <Route path="/my/lands/:id" element={<LandDetailPage />} />
        <Route path="/my/lands/:id/edit" element={<LandEditPage />} />
        <Route path="/my/lands/:id/chats" element={<LandChatsPage />} />
        <Route path="/my/lands/:id/chats/:chatId" element={<ChatDetailPage />} />
        <Route path="/my/lands/:id/tasks" element={<LandTasksPage />} />
        <Route path="/my/lands/:id/contracts" element={<LandContractsPage />} />

        {/* 物件管理 */}
        <Route path="/my/properties" element={<MyPropertiesPage />} />
        <Route path="/my/properties/new" element={<PropertyNewPage />} />
        <Route path="/my/properties/:id" element={<PropertyDetailPage />} />
        <Route path="/my/properties/:id/edit" element={<PropertyEditPage />} />
        <Route path="/my/properties/:id/rooms" element={<RoomsPage />} />
        <Route path="/my/properties/:id/rooms/:roomId" element={<RoomDetailPage />} />
        <Route path="/my/properties/:id/equipment" element={<EquipmentPage />} />
        <Route path="/my/properties/:id/equipment/floor/:floor" element={<FloorEquipmentPage />} />
        <Route path="/my/properties/:id/contracts" element={<PropertyContractsPage />} />
        <Route path="/my/properties/:id/contracts/:cId" element={<ContractDetailPage />} />
        <Route path="/my/properties/:id/chats" element={<PropertyChatsPage />} />
        <Route path="/my/properties/:id/chats/:chatId" element={<ChatDetailPage />} />
        <Route path="/my/properties/:id/tasks" element={<PropertyTasksPage />} />
        <Route path="/my/properties/:id/employees" element={<EmployeesPage />} />

        {/* その他 */}
        <Route path="/tasks" element={<AllTasksPage />} />
        <Route path="/visualization" element={<VisualizationPage />} />
        <Route path="/valuation" element={<ValuationPage />} />
        <Route path="/sns" element={<SnsPage />} />
        <Route path="/sns/:id" element={<SnsDetailPage />} />
        <Route path="/opportunities" element={<OpportunitiesPage />} />
        <Route path="/vendors" element={<VendorsPage />} />
        <Route path="/settings" element={<SettingsPage />} />
      </Route>
    </Route>
  </Routes>
</BrowserRouter>
```

---

## デザインシステム実装 / Design System Implementation

### カラーパレット (Tailwind CSS カスタム設定)

```typescript
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      'primary-navy': '#1B3A6B',      // メインナビ・ボタン
      'secondary-blue': '#2E5EAA',    // サブ要素
      'accent-blue': '#4A90D9',       // リンク・強調
      'text-dark': '#1A1A2E',         // 本文テキスト
      'text-gray': '#6B7280',         // 補足テキスト
      'success': '#059669',           // 成功・occupied
      'warning': '#D97706',           // 警告・late
      'danger': '#DC2626',            // エラー・broken
      'background': '#FAFAFA',        // ページ背景
      'surface': '#FFFFFF',           // カード背景
      'border': '#E5E7EB',            # ボーダー
    }
  }
}
```

### コンポーネント設計原則

- **Button:** `primary` (ネイビー塗り)、`secondary` (ネイビー枠)、`danger` (赤塗り)
- **Badge:** ステータスに応じた色分け（occupied=green, vacant=gray, maintenance=orange）
- **Card:** 白背景・角丸・薄シャドウ、hover 時に境界色を変更
- **Input:** フォーカス時に `accent-blue` のリング表示
- **Table:** ゼブラストライプ（奇数行: white, 偶数行: backgroundGray）

---

## 環境変数 / Environment Variables

```env
# .env.local (開発時)
VITE_API_BASE_URL=http://localhost:3000/api/v1
VITE_WS_URL=ws://localhost:3000

# .env.production
VITE_API_BASE_URL=https://api.arvana-terra.jp/api/v1
VITE_WS_URL=wss://api.arvana-terra.jp
```

---

## 開発セットアップ / Development Setup

```bash
cd arvana-terra-web

# 依存関係インストール
npm install

# 環境変数設定
cp .env.example .env.local
# VITE_API_BASE_URL と VITE_WS_URL を確認・編集

# 開発サーバー起動
npm run dev
# → http://localhost:5174 で起動（Vite HMR 有効）

# 型チェック
npx tsc --noEmit

# リント
npm run lint
```

---

## ビルドとデプロイ / Build and Deployment

```bash
# プロダクションビルド
npm run build
# → dist/ にバンドルファイルが生成される
# → TypeScript コンパイル → Vite バンドル

# ビルドプレビュー
npm run preview
# → http://localhost:4173

# 静的ファイルのデプロイ例（Nginx）
server {
    listen 80;
    root /var/www/arvana-terra-web;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
        # SPA ルーティングのため index.html にフォールバック
    }
}
```

---

## パフォーマンス最適化 / Performance Optimization

- **コード分割:** React.lazy + Suspense によるルートレベルのコード分割
- **TanStack Query キャッシュ:** 適切な staleTime 設定でAPIリクエストを削減
- **Vite バンドル最適化:** ESM、Tree Shaking、Rollup による最適化
- **画像:** WebP 推奨、適切な width/height 属性設定
- **Recharts:** 必要なコンポーネントのみインポート（named import）

---

## テスト方針 / Testing Approach

- **ユニットテスト:** Vitest + React Testing Library（カスタムフック・ユーティリティ関数）
- **E2Eテスト:** Playwright（主要ユーザーフロー）
- **型テスト:** TypeScript strict モードで型安全性を保証
- **手動テスト:** 実機・ブラウザでの動作確認

```bash
# テスト実行（設定後）
npm run test
npm run test:e2e
```
