# Arvana Terra - 外部連携ガイド / Integration Guide

## 概要 / Overview

Arvana Terra は複数の外部システム・サービスと連携しています。このドキュメントでは各連携の詳細、認証フロー、データフロー、および実装上の注意点を説明します。

Arvana Terra integrates with multiple external systems and services. This document describes each integration in detail, including authentication flows, data flows, and implementation notes.

---

## Arvana Work 連携 / Arvana Work Integration

### 概要

Arvana Work（Arvana の雇用管理プラットフォーム）から連携した雇用者（employer ロール）が、Arvana Terra のチャット機能にアクセスできます。

Employers from Arvana Work (Arvana's employment management platform) can access the chat functionality in Arvana Terra.

### ユースケース

```
地主 A (landlord) が Arvana Work で管理しているスタッフ（管理人・清掃員等）を
Arvana Terra の物件チャットに参加させ、物件管理に関するリアルタイム連絡を行う。

Landlord A manages staff (property managers, cleaners, etc.) in Arvana Work.
They join the property chat in Arvana Terra for real-time property management communication.
```

### 連携フロー / Integration Flow

```
Arvana Work                     Arvana Terra                    Admin Panel
     │                               │                               │
     │                               │                               │
     │  [雇用者がArvana Terraに       │                               │
     │   アクセスするための           │                               │
     │   アカウント作成を依頼]         │                               │
     │                               │                               │
     │                               │◄── POST /auth/register ───────│
     │                               │    { role: 'employer' }       │
     │                               │── ユーザー作成 ──────────────►│(DB)
     │                               │                               │
     │                               │── アカウント情報を Arvana Work →│
     │                               │   に連携（メール通知等）        │
     │                               │                               │
雇用者がAdmin Panelにログイン ────────►│                               │
     │                               │                               │
     │                        ┌──────┴──────────────────────────────┤
     │                        │ employer ダッシュボード              │
     │                        │ - 担当物件一覧                       │
     │                        │ - 担当土地一覧                       │
     │                        │ - チャットルーム一覧                  │
     │                        └─────────────────────────────────────┘
     │                               │
     │                               │◄── GET /employer/properties ──│
     │                               │── 担当物件を返却 ─────────────►│
     │                               │                               │
     │                               │◄── GET /employer/properties/:id/chats
     │                               │── チャット一覧を返却 ──────────►│
     │                               │                               │
     │                               │◄── POST /chats/:id/participants
     │                               │    { userId: "landlord-uuid" } │
     │                               │── 地主をチャットに追加 ─────────►│(DB)
```

### 雇用者の権限 / Employer Permissions

| 機能 | 権限 |
|------|------|
| 担当物件の確認 | GET /employer/properties |
| 担当土地の確認 | GET /employer/lands |
| 物件チャットルームの閲覧 | GET /employer/properties/:id/chats |
| 物件チャットルームの作成 | POST /employer/properties/:id/chats |
| 物件チャットの更新（名前・説明） | PATCH /employer/properties/:id/chats/:chatId |
| 土地チャットルームの管理 | 同上 (lands) |
| チャット参加者の追加・削除 | POST/DELETE /chats/:id/participants |
| メッセージの閲覧（履歴） | GET /chats/:id/messages |
| メッセージの送信 | Socket.io send_message |
| Web/iOS/Androidの物件・土地管理機能 | 利用不可 |

### 実装上の注意点

```typescript
// employer ロールは Web クライアントではなく Admin パネルからのみアクセス
// Admin パネルの useRBAC フックでロール分岐

const { isAdmin, isEmployer } = useRBAC()

if (isEmployer) {
  // EmployerDashboard を表示
  // - 担当物件・土地のチャット管理機能
} else if (isAdmin) {
  // AdminDashboard を表示
  // - 業者管理・コンテンツ管理・全チャット閲覧
}
```

---

## JWT 共通認証パターン / Shared JWT Authentication Patterns

### トークン構造

```
Access Token Payload:
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "role": "landlord",  // landlord | homeowner | employer | admin | super_admin
  "iat": 1705300000,
  "exp": 1705300900    // 15分後
}
```

### アプリ別トークン保管場所

| アプリ | 保管場所 | キー名 |
|--------|--------|--------|
| Web Client | localStorage | `accessToken`, `refreshToken` |
| Admin Panel | localStorage | `arvana_admin_token`, `arvana_admin_refresh_token` |
| iOS | Keychain | `arvana_terra_access_token`, `arvana_terra_refresh_token` |
| Android | DataStore Preferences | `access_token`, `refresh_token` |

### トークンリフレッシュ戦略

```
Web / iOS / Android:
  アクセストークン期限切れ (401) を検知
  → POST /auth/refresh { refreshToken }
  → 新しい accessToken を取得・保管
  → 元のリクエストを再試行

Admin Panel:
  401 → localStorage をクリア → /login へリダイレクト
  (自動リフレッシュなし - セキュリティ優先)
```

### Socket.io 認証

```javascript
// 接続時に JWT を渡す
const socket = io('ws://host:3001/chat', {
  auth: { token: accessToken },  // "Bearer " プレフィックスは不要
  transports: ['websocket'],
})

// サーバー側での検証
const authMiddleware = (socket, next) => {
  const token = socket.handshake.auth.token
  const decoded = jwt.verify(token, process.env.JWT_SECRET)
  socket.user = decoded
  next()
}
```

---

## スマートデバイス連携 / Smart Device Integration

### 対応デバイス

| デバイスタイプ | 用途 | データ形式 |
|-------------|------|----------|
| water_meter | 水道メーター読み取り | m³ / 週次 |
| electric_meter | 電気メーター読み取り | kWh / 週次 |
| camera | 防犯カメラ監視 | ステータス (active/inactive/error) |
| sensor | 各種センサー | 任意単位 |

### SmartDeviceData モデル

```prisma
model SmartDeviceData {
  id           String       @id @default(uuid())
  propertyId   String
  roomId       String?
  deviceType   DeviceType   # water_meter / electric_meter / camera / sensor
  deviceId     String       # デバイス固有ID (ハードウェアシリアル等)
  location     String?      # 設置場所の説明 (例: "1F 共用廊下")
  readings     Json         # 週次読み取りデータ配列 (JSONB)
  cameraStatus CameraStatus? # カメラのみ: active/inactive/error
  lastUpdated  DateTime     @default(now())
}
```

### readings JSON 形式

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
  },
  {
    "date": "2024-02-26",
    "value": 131.8,
    "unit": "m3"
  }
]
```

### デバイスデータ登録フロー

```
スマートデバイス (IoT)
  │── HTTP POST → バックエンドAPI (デバイス認証)
  │   POST /api/v1/properties/:propertyId/smart-devices
  │   { deviceType: "water_meter", deviceId: "SN-12345", readings: [...] }
  │
バックエンド
  │── SmartDeviceData.readings に append
  │── lastUpdated を更新
  │── 閾値を超えた場合に Notification を作成
  │── Socket.io /notification で通知を push
  │
クライアントアプリ (Web / iOS / Android)
  └── グラフで週次推移を表示
```

### スマートデバイス API

```typescript
// GET /api/v1/properties/:propertyId/smart-devices
// 物件のスマートデバイス一覧を取得

// Web クライアントでの利用
const { data: devices } = useQuery({
  queryKey: ['smart-devices', propertyId],
  queryFn: () => smartDevicesApi.getSmartDevices(propertyId),
})

// Recharts でグラフ表示
<LineChart data={readings}>
  <Line dataKey="value" stroke="#4A90D9" />
</LineChart>
```

---

## MoneyForward 連携 / MoneyForward Integration

### 概要

MoneyForward は日本の主要なクラウド会計・給与計算サービスです。Arvana Terra は MoneyForward との連携により、家賃収入・修繕費等の会計データを同期します。

MoneyForward is Japan's leading cloud accounting and payroll service. Arvana Terra integrates with MoneyForward to sync rental income, repair costs, and other accounting data.

### 連携データ

| Arvana Terra | MoneyForward | 方向 |
|-------------|-------------|------|
| 家賃入金記録 (Payment) | 売上明細 | Terra → MoneyForward |
| 修繕費・設備購入 (任意入力) | 経費明細 | Terra → MoneyForward |
| 請求書 (Contract 関連) | 請求書管理 | 双方向 |

### 実装方針（将来）

```
現在: 手動データエクスポート (CSV)
将来: MoneyForward Open API v2 を使用した自動同期

連携フロー:
1. ユーザーが MoneyForward で OAuth 認証
2. Arvana Terra が MoneyForward API のアクセストークンを取得・保管
3. 家賃入金確認時に自動で MoneyForward に売上を記録
4. 毎月末にサマリーレポートを同期
```

---

## AI 連携 / AI Integration

### OpenAI GPT-4 の使用

Arvana Terra は OpenAI API (GPT-4) を以下の機能に使用しています。

```
使用モデル: gpt-4 (openai SDK v4.28.0)
APIキー: 環境変数 OPENAI_API_KEY
```

### 1. AIタスク提案 / AI Task Suggestions

```typescript
// POST /my/properties/:id/tasks/ai-suggestions の内部処理

async function generateAiTaskSuggestions(propertyId: string, userId: string) {
  // コンテキスト収集
  const property = await prisma.property.findUnique({
    where: { id: propertyId },
    include: {
      rooms: { include: { tenant: true } },
      equipment: true,
      tasks: { where: { status: { not: 'done' } } },
    }
  })

  // AIプロンプト構築
  const prompt = `
あなたは日本の不動産管理の専門家です。
以下の物件情報を分析して、次に必要なタスクを3〜5件提案してください。

物件情報:
- 名称: ${property.name}
- 建物タイプ: ${property.buildingType}
- 築年数: ${new Date().getFullYear() - property.builtYear} 年
- 総部屋数: ${property.totalRooms}
- 入居中: ${property.rooms.filter(r => r.status === 'occupied').length} 部屋
- 空室: ${property.rooms.filter(r => r.status === 'vacant').length} 部屋

設備状況:
${property.equipment.map(e => `- ${e.name}: ${e.status}, 最終点検: ${e.lastInspectionDate}`).join('\n')}

現在のタスク:
${property.tasks.map(t => `- [${t.priority}] ${t.title}`).join('\n')}

JSON形式で提案してください:
[{ "title": "...", "description": "...", "priority": "high/medium/low", "aiReason": "..." }]
  `

  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{ role: 'user', content: prompt }],
    response_format: { type: 'json_object' },
  })

  return JSON.parse(response.choices[0].message.content)
}
```

### 2. 資産価値AI評価 / AI Asset Valuation

```typescript
// POST /my/valuation の内部処理

async function calculateAiAssetValuation(
  userId: string,
  landIds: string[],
  propertyIds: string[]
) {
  const [lands, properties] = await Promise.all([
    prisma.land.findMany({ where: { id: { in: landIds } } }),
    prisma.property.findMany({
      where: { id: { in: propertyIds } },
      include: { rooms: { include: { tenant: true } } }
    })
  ])

  const prompt = `
あなたは日本の不動産市場の専門アナリストです。
以下の資産情報を分析して、現在の資産価値と3年後の予測値を算出してください。

土地情報:
${lands.map(l => `
- 名称: ${l.name}
- 住所: ${l.address}
- 面積: ${l.area}m²
- 用途地域: ${l.zoning}
- 購入価格: ${l.purchasePrice}円
- 現在の推定価値: ${l.currentValue}円
`).join('')}

物件情報:
${properties.map(p => `
- 名称: ${p.name}
- 住所: ${p.address}
- 建物タイプ: ${p.buildingType}
- 築年数: ${new Date().getFullYear() - p.builtYear}年
- 総部屋数: ${p.totalRooms}
- 入居率: ${Math.round(p.rooms.filter(r => r.status === 'occupied').length / p.totalRooms * 100)}%
- 現在の推定価値: ${p.currentValue}円
`).join('')}

以下のJSON形式で回答してください:
{
  "totalCurrentValue": 数値,
  "aiPredictedValue": 数値（3年後予測）,
  "predictionYear": ${new Date().getFullYear() + 3},
  "aiAnalysis": "詳細な分析コメント（日本語、200〜400字）"
}
  `

  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{ role: 'user', content: prompt }],
    response_format: { type: 'json_object' },
  })

  const result = JSON.parse(response.choices[0].message.content)

  // DB に保存
  return await prisma.assetValuation.create({
    data: {
      userId,
      landIds,
      propertyIds,
      totalCurrentValue: result.totalCurrentValue,
      aiPredictedValue: result.aiPredictedValue,
      predictionYear: result.predictionYear,
      aiAnalysis: result.aiAnalysis,
    }
  })
}
```

### 3. ビジネスチャンス分析 / Business Opportunity Analysis

```typescript
// ビジネスチャンスの AI 分析コメント生成
// BusinessOpportunity.aiAnalysis フィールドに保存

async function analyzeBusinessOpportunity(opportunity: BusinessOpportunity) {
  const prompt = `
以下の不動産ビジネスチャンスを分析してください。
タイプ: ${opportunity.type}
場所: ${opportunity.location} (${opportunity.region})
推定価値: ${opportunity.estimatedValue}円

このビジネスチャンスのリスク・メリット・おすすめアクションを日本語で簡潔に説明してください（150字以内）。
  `
  // ... openai API 呼び出し
}
```

### レート制限・コスト管理

```typescript
// AI機能へのアクセス制限（バックエンドで実装）
// - タスク提案: 1ユーザー1物件あたり1時間に最大3回
// - 資産価値評価: 1ユーザーあたり1日に最大5回
// - Redis でレート制限を管理

const rateLimitKey = `ai:valuation:${userId}:${today}`
const count = await redis.incr(rateLimitKey)
if (count === 1) await redis.expire(rateLimitKey, 86400)  // 24時間
if (count > 5) throw new AppError('AI評価の1日あたりの利用制限に達しました', 429)
```

---

## 業者登録ワークフロー / Vendor Registration Workflow

### 全体フロー

```
管理者 (Admin Panel)
  │
  ├── 業者情報を手動入力
  │   POST /admin/vendors { status: 'approved' }
  │   → isApproved = true で即時登録
  │
  └── または業者申請を受け付けて審査
      POST /admin/vendors { status: 'pending' }
      → 審査後に POST /admin/vendors/:id/approve

クライアントアプリ (Web / iOS / Android)
  │
  └── GET /vendors (isApproved=true のみ返却)
      → 地主・家主が業者を検索
      → POST /my/vendors/:id/connect で業者と連携
      → UserVendor テーブルに保存
      → 業者の連絡先情報にアクセス可能に
```

### 業者データ同期

```
クライアントアプリに表示される業者:
  WHERE isApproved = true

管理パネルに表示される業者:
  全ステータス (approved / pending / rejected / deleted)

業者の connectedUsersCount:
  UserVendor テーブルの COUNT(userId) WHERE vendorId = :id
```

### 業者と地主・家主の連携

```typescript
// POST /my/vendors/:vendorId/connect
// UserVendor テーブルにレコードを作成

const userVendor = await prisma.userVendor.create({
  data: {
    userId: currentUser.id,
    vendorId: vendorId,
    notes: '水漏れ修理で対応してもらった',
    connectedAt: new Date(),
  }
})

// DELETE /my/vendors/:vendorId/disconnect
await prisma.userVendor.delete({
  where: {
    userId_vendorId: {
      userId: currentUser.id,
      vendorId: vendorId,
    }
  }
})
```

---

## 通知フロー / Notification Flow

### 通知の種類と発生タイミング

```
Notification タイプ     | 発生タイミング              | 優先度
------------------------|---------------------------|-------
payment_due             | 支払い期日の3日前           | high
payment_received        | 支払いステータスが paid に更新 | medium
equipment_warning       | 設備ステータスが warning/broken に変更 | high
contract_expiring       | 契約期限の30日前           | medium
task_due                | タスク期限の1日前           | medium
chat_message            | 新着メッセージ（未読の場合） | low
opportunity             | 新規ビジネスチャンス追加    | low
system                  | システムメンテナンス等       | high
vendor_approved         | 業者承認完了               | medium
```

### 通知配信フロー

```
イベント発生（例: 設備ステータス更新）
  │
  ▼
バックエンドルートハンドラー
  │
  ▼
DB: Notification レコードを作成
  prisma.notification.create({
    data: {
      userId: property.ownerId,
      type: 'equipment_warning',
      title: '設備警告: エアコン',
      content: '101号室のエアコンが警告状態になりました',
      relatedId: equipmentId,
      relatedType: 'equipment',
    }
  })
  │
  ▼
Socket.io: /notification namespace で push 配信
  io.of('/notification')
    .to(`user:${userId}`)
    .emit('notification', notificationRecord)
  │
  ▼
クライアント受信
  - Web: useNotifications フックで受信 → Toast + バッジカウント更新
  - iOS: URLSessionWebSocketTask で受信 → バナー通知 (local notification)
  - Android: Socket.io-client で受信 → NotificationCompat.Builder で表示
```

### クライアント別通知実装

#### Web (Socket.io-client)

```typescript
// src/hooks/useNotifications.ts
const notifSocket = io(`${WS_URL}/notification`, {
  auth: { token: accessToken },
})

notifSocket.on('notification', (notification: Notification) => {
  addNotification(notification)  // Zustand store に追加
  toast.info(notification.title)  // トースト表示
  // バッジカウント +1
})
```

#### iOS (URLSessionWebSocketTask)

```swift
// 受信した通知を UNUserNotificationCenter でローカル通知として表示
let content = UNMutableNotificationContent()
content.title = notification.title
content.body = notification.content
content.sound = .default

let request = UNNotificationRequest(
  identifier: notification.id,
  content: content,
  trigger: nil  // 即座に表示
)
UNUserNotificationCenter.current().add(request)
```

#### Android (Notification)

```kotlin
// SocketManager で受信 → NotificationHelper で表示
fun showNotification(notification: AppNotification) {
    val builder = NotificationCompat.Builder(context, CHANNEL_ID)
        .setSmallIcon(R.drawable.ic_notification)
        .setContentTitle(notification.title)
        .setContentText(notification.content)
        .setPriority(NotificationCompat.PRIORITY_DEFAULT)
        .setAutoCancel(true)

    NotificationManagerCompat.from(context).notify(
        notification.id.hashCode(),
        builder.build()
    )
}
```

---

## 将来の連携予定 / Planned Future Integrations

### Google Maps / MapKit 連携

```
用途:
- 土地・物件の地図表示
- 近隣の地価情報表示
- 業者のサービスエリア地図表示

実装予定:
- Web: Google Maps JavaScript API
- iOS: MapKit (Apple Maps)
- Android: Google Maps SDK
```

### MoneyForward 自動連携

```
MoneyForward Open API v2 を使用した家賃収入の自動連携。
OAuth 2.0 フローでユーザー認証後、
月次の家賃収入データを自動同期。
```

### LINE 通知連携

```
LINE Messaging API を使用したプッシュ通知。
重要な通知（設備故障・支払い延滞等）を
LINE に転送するオプション。
```

### e-Tax / eLTAX 連携

```
不動産所得の確定申告データ生成。
収入・支出データを e-Tax 形式でエクスポート。
地主向けの税務サポート機能。
```
