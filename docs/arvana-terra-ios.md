# Arvana-Terra-iOS - 詳細仕様 / Detailed Specification

## 概要 / Overview

Arvana Terra の iPhone/iPad 向けネイティブ iOS アプリです。Swift/SwiftUI で構築され、MVVM アーキテクチャを採用しています。URLSession による REST API 通信と URLSessionWebSocketTask による Socket.io 互換の WebSocket 通信を実装しています。

Native iOS app for Arvana Terra. Built with Swift/SwiftUI using MVVM architecture. Implements REST API communication via URLSession and Socket.io-compatible WebSocket communication via URLSessionWebSocketTask.

---

## テクノロジースタック / Tech Stack

| 技術 | バージョン/詳細 | 用途 |
|------|----------------|------|
| Swift | 5.9 | プログラミング言語 |
| SwiftUI | iOS 17 SDK | UI フレームワーク |
| URLSession | Foundation | REST API HTTP クライアント |
| URLSessionWebSocketTask | Foundation | WebSocket / Socket.io |
| Combine | Foundation | リアクティブプログラミング |
| SwiftData / @State/@ObservableObject | Swift | 状態管理 |
| SF Symbols | 5.x | アイコンシステム |
| Min iOS | 17.0 | 最小対応 OS バージョン |
| Xcode | 15.x 以上 | 開発環境 |
| App Bundle ID | `jp.co.arvana.terra` | アプリ識別子 |

### サードパーティ依存ライブラリ

Arvana-Terra-iOS は **Swift Package Manager (SPM) のサードパーティ依存なし** で構築されています。すべての機能は Apple 標準フレームワーク（Foundation, SwiftUI, Combine）のみで実装します。

---

## アプリ設定 / App Configuration

```swift
// src/Config/AppConfig.swift
struct AppConfig {
    static let apiBaseURL = "http://localhost:3000/api/v1"
    static let wsURL = "ws://localhost:3000"
    static let appName = "Arvana Terra"
    static let appVersion = "1.0.0"
}
// 本番環境では Info.plist で上書き or BuildConfig で切り替え
```

---

## ディレクトリ構造 / Directory Structure

```
Arvana-Terra-iOS/
├── Arvana-Terra-iOS.xcodeproj/
└── Arvana-Terra-iOS/
    ├── ArvanaTerraiOSApp.swift        # @main エントリポイント
    ├── ContentView.swift              # ルートビュー（認証状態に応じた分岐）
    ├── Config/
    │   └── AppConfig.swift            # APIベースURL・カラー定義
    ├── Models/
    │   ├── User.swift                 # User, AuthResponse, UserRole
    │   ├── Land.swift                 # Land, LandStatus, CreateLandRequest
    │   ├── Property.swift             # Property, BuildingType, PropertyStatus
    │   ├── Room.swift                 # Room, RoomStatus, Tenant, Payment
    │   ├── Equipment.swift            # Equipment, EquipmentStatus, SmartDeviceData
    │   ├── Contract.swift             # Contract, ContractType, ContractStatus
    │   ├── Task.swift                 # Task, TaskStatus, TaskPriority
    │   ├── Employee.swift             # Employee
    │   ├── ChatRoom.swift             # ChatRoom, ChatMessage, ChatParticipant
    │   ├── Vendor.swift               # Vendor, VendorCategory
    │   ├── SnsPost.swift              # SnsPost, SnsComment, PostType
    │   ├── AssetValuation.swift       # AssetValuation, ValuationPrediction
    │   ├── BusinessOpportunity.swift  # BusinessOpportunity
    │   └── Notification.swift         # AppNotification, NotificationType
    ├── Services/                      # API サービス層
    │   ├── AuthService.swift
    │   ├── LandService.swift
    │   ├── PropertyService.swift
    │   ├── RoomService.swift
    │   ├── EquipmentService.swift
    │   ├── ContractService.swift
    │   ├── TaskService.swift
    │   ├── ChatService.swift
    │   ├── VendorService.swift
    │   ├── SnsService.swift
    │   └── WebSocketService.swift
    ├── ViewModels/                    # MVVM ViewModel 層
    │   ├── AuthViewModel.swift
    │   ├── LandListViewModel.swift
    │   ├── LandDetailViewModel.swift
    │   ├── PropertyListViewModel.swift
    │   ├── PropertyDetailViewModel.swift
    │   ├── RoomListViewModel.swift
    │   ├── ChatViewModel.swift
    │   ├── TaskViewModel.swift
    │   └── ...
    └── Views/                         # SwiftUI View 層
        ├── Auth/
        │   ├── LoginView.swift
        │   └── RegisterView.swift
        ├── Dashboard/
        │   └── DashboardView.swift
        ├── Land/
        │   ├── LandListView.swift
        │   ├── LandDetailView.swift
        │   ├── LandFormView.swift
        │   └── LandChatView.swift
        ├── Property/
        │   ├── PropertyListView.swift
        │   ├── PropertyDetailView.swift
        │   ├── RoomListView.swift
        │   ├── RoomDetailView.swift
        │   └── PropertyTaskView.swift
        ├── Equipment/
        │   ├── EquipmentListView.swift
        │   └── SmartDeviceView.swift
        ├── Chat/
        │   ├── ChatRoomListView.swift
        │   └── ChatDetailView.swift
        ├── SNS/
        │   ├── SnsFeedView.swift
        │   └── SnsPostDetailView.swift
        ├── Vendor/
        │   └── VendorListView.swift
        ├── Settings/
        │   └── SettingsView.swift
        └── Components/
            ├── StatusBadge.swift
            ├── PropertyCard.swift
            ├── LandCard.swift
            └── ...
```

---

## アーキテクチャ / Architecture (MVVM)

```
View (SwiftUI)
  │── observes ──► ViewModel (@ObservableObject / @Observable)
  │                    │── calls ──► Service (URLSession)
  │                    │                │── HTTP request ──► Backend API
  │                    │                │◄── Response ──────── Backend API
  │                    │◄── updates models
  │◄── re-renders

例:
LandListView
  └── @StateObject var viewModel = LandListViewModel()
        └── func fetchLands() → LandService.getMyLands()
              └── URLSession.shared.dataTask(...)
                    → GET /api/v1/my/lands
                    ← [Land]
              → viewModel.lands = [Land]
        └── @Published var lands: [Land] = []
        └── @Published var isLoading: Bool = false
```

---

## スクリーン一覧 / Screen List

### 認証フロー

| スクリーン | 説明 |
|-----------|------|
| スプラッシュ / Splash | アプリ起動時のローディング・認証チェック |
| ログイン / Login | メール・パスワード入力 |
| 新規登録 / Register | アカウント作成（role 選択含む） |

### メインタブ構成

```
TabView
├── ホーム / Home (house.fill)
├── 土地 / Lands (map.fill) ← landlord のみ
├── 物件 / Properties (building.2.fill)
├── チャット / Chat (bubble.left.and.bubble.right.fill)
├── SNS / Community (person.3.fill)
└── 設定 / Settings (gearshape.fill)
```

### ホーム / Dashboard

| スクリーン | 説明 |
|-----------|------|
| ダッシュボード | 資産サマリー・最新タスク・通知バッジ |

### 土地管理 / Land Management (landlord のみ)

| スクリーン | 説明 |
|-----------|------|
| 土地一覧 | スクロール可能なカードリスト・ステータスフィルタ |
| 土地詳細 | 土地情報・関連物件・チャット・タスクへのナビゲーション |
| 土地新規作成 | 土地情報入力フォーム |
| 土地編集 | 土地情報編集 |
| 土地チャット一覧 | チャットルーム一覧 |
| 土地タスク | タスク一覧・新規作成 |

### 物件管理 / Property Management

| スクリーン | 説明 |
|-----------|------|
| 物件一覧 | カードリスト・建物タイプ・ステータスフィルタ |
| 物件詳細 | 物件情報・入居率・月次収入 |
| 物件新規作成 | 物件情報入力フォーム |
| 部屋一覧 | フロア別部屋リスト・ステータス表示 |
| 部屋詳細 | 部屋情報・入居者情報・支払い履歴 |
| 設備一覧 | 設備カテゴリ別一覧・ステータス色分け |
| スマートデバイス | 水道・電気メーター読み取り値グラフ |
| 契約書一覧 | 物件に紐づく契約書 |
| 物件チャット | チャットルーム一覧 |
| 物件タスク | タスク一覧（AI提案タグ付き） |
| 従業員一覧 | 従業員リスト |

### チャット / Chat

| スクリーン | 説明 |
|-----------|------|
| チャット一覧 | 全チャットルーム一覧・未読バッジ |
| チャット詳細 | リアルタイムチャット・メッセージバブル |

### SNS

| スクリーン | 説明 |
|-----------|------|
| フィード | SNS 投稿フィード・カテゴリフィルタ |
| 投稿詳細 | 投稿・コメント・いいね |
| 投稿作成 | テキスト・画像投稿フォーム |

### 業者 / Vendors

| スクリーン | 説明 |
|-----------|------|
| 業者一覧 | 承認済み業者リスト・カテゴリフィルタ |
| 業者詳細 | 業者情報・連絡先 |

### 設定 / Settings

| スクリーン | 説明 |
|-----------|------|
| プロフィール | ユーザー情報・編集 |
| 通知設定 | 各通知のオン/オフ |
| アプリ情報 | バージョン・ライセンス |
| ログアウト | セッションクリア |

---

## API インテグレーション / API Integration

### URLSession による REST API 通信

```swift
// Services/AuthService.swift の例
class AuthService {
    private let baseURL = AppConfig.apiBaseURL

    func login(email: String, password: String) async throws -> AuthResponse {
        guard let url = URL(string: "\(baseURL)/auth/login") else {
            throw NetworkError.invalidURL
        }

        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(LoginRequest(email: email, password: password))

        let (data, response) = try await URLSession.shared.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse,
              200..<300 ~= httpResponse.statusCode else {
            throw NetworkError.serverError
        }

        return try JSONDecoder().decode(AuthResponse.self, from: data)
    }
}
```

### 認証トークン管理

```swift
// Keychain を使ったトークン保管
class TokenManager {
    static let shared = TokenManager()

    private let accessTokenKey = "arvana_terra_access_token"
    private let refreshTokenKey = "arvana_terra_refresh_token"

    var accessToken: String? {
        get { KeychainHelper.read(key: accessTokenKey) }
        set { KeychainHelper.save(key: accessTokenKey, value: newValue ?? "") }
    }

    var refreshToken: String? {
        get { KeychainHelper.read(key: refreshTokenKey) }
        set { KeychainHelper.save(key: refreshTokenKey, value: newValue ?? "") }
    }

    func clear() {
        KeychainHelper.delete(key: accessTokenKey)
        KeychainHelper.delete(key: refreshTokenKey)
    }
}
```

### 認証ヘッダーの付与

```swift
// 認証が必要なリクエストの共通処理
func authenticatedRequest(url: URL, method: String = "GET") throws -> URLRequest {
    guard let token = TokenManager.shared.accessToken else {
        throw NetworkError.unauthorized
    }
    var request = URLRequest(url: url)
    request.httpMethod = method
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
    return request
}
```

---

## Socket.io / WebSocket 実装

```swift
// Services/WebSocketService.swift
class WebSocketService: ObservableObject {
    private var webSocketTask: URLSessionWebSocketTask?
    private let wsURL = AppConfig.wsURL

    @Published var receivedMessages: [ChatMessage] = []
    @Published var isConnected = false

    func connect() {
        guard let token = TokenManager.shared.accessToken,
              let url = URL(string: "\(wsURL)/chat") else { return }

        // Socket.io ハンドシェイクパラメータ
        var components = URLComponents(url: url, resolvingAgainstBaseURL: false)!
        components.queryItems = [
            URLQueryItem(name: "EIO", value: "4"),
            URLQueryItem(name: "transport", value: "websocket"),
            URLQueryItem(name: "auth", value: token)
        ]

        let session = URLSession(configuration: .default)
        webSocketTask = session.webSocketTask(with: components.url!)
        webSocketTask?.resume()
        isConnected = true
        receiveMessages()
    }

    private func receiveMessages() {
        webSocketTask?.receive { [weak self] result in
            switch result {
            case .success(let message):
                switch message {
                case .string(let text):
                    self?.handleMessage(text)
                case .data(let data):
                    self?.handleData(data)
                @unknown default: break
                }
                self?.receiveMessages()
            case .failure:
                self?.isConnected = false
            }
        }
    }

    func sendMessage(chatRoomId: String, content: String) {
        let payload: [String: Any] = [
            "chatRoomId": chatRoomId,
            "content": content,
            "messageType": "text"
        ]
        if let data = try? JSONSerialization.data(withJSONObject: payload),
           let text = String(data: data, encoding: .utf8) {
            let message = URLSessionWebSocketTask.Message.string("42[\"send_message\",\(text)]")
            webSocketTask?.send(message) { _ in }
        }
    }

    func joinRoom(chatRoomId: String) {
        let message = URLSessionWebSocketTask.Message.string("42[\"join_chat\",\"\(chatRoomId)\"]")
        webSocketTask?.send(message) { _ in }
    }

    func disconnect() {
        webSocketTask?.cancel(with: .normalClosure, reason: nil)
        isConnected = false
    }
}
```

---

## デザインシステム / Design System

### カラーパレット

```swift
// Config/AppConfig.swift - Color Extension
extension Color {
    static let primaryNavy    = Color(hex: "#1B3A6B")  // メインカラー
    static let secondaryBlue  = Color(hex: "#2E5EAA")  // セカンダリ
    static let accentBlue     = Color(hex: "#4A90D9")  // アクセント
    static let textDark       = Color(hex: "#1A1A2E")  // メインテキスト
    static let textGray       = Color(hex: "#6B7280")  // サブテキスト
    static let successGreen   = Color(hex: "#059669")  // 成功・入居中
    static let warningOrange  = Color(hex: "#D97706")  // 警告
    static let errorRed       = Color(hex: "#DC2626")  // エラー
    static let backgroundGray = Color(hex: "#FAFAFA")  // 背景
    static let surfaceWhite   = Color(hex: "#FFFFFF")  // カード背景
    static let borderGray     = Color(hex: "#E5E7EB")  // ボーダー
}
```

### タイポグラフィ

```swift
// SwiftUI 標準フォントスケール
Text("タイトル").font(.title2).fontWeight(.bold).foregroundColor(.textDark)
Text("本文").font(.body).foregroundColor(.textDark)
Text("補足").font(.caption).foregroundColor(.textGray)
Text("見出し").font(.headline)
Text("小見出し").font(.subheadline)
```

### SF Symbols 使用例

```swift
// 主要 SF Symbols
"house.fill"                    // ホーム
"map.fill"                      // 土地
"building.2.fill"               // 物件
"person.2.fill"                 // 入居者
"wrench.and.screwdriver.fill"   // 設備
"doc.text.fill"                 // 契約書
"bubble.left.and.bubble.right.fill" // チャット
"checkmark.circle.fill"         // タスク完了
"exclamationmark.triangle.fill" // 警告
"chart.line.uptrend.xyaxis"     // 資産価値
"magnifyingglass"               // 検索
"plus.circle.fill"              // 追加
"pencil"                        // 編集
"trash.fill"                    // 削除
"bell.fill"                     // 通知
"gearshape.fill"                // 設定
```

---

## モデル定義例 / Model Definitions

```swift
// Models/User.swift
enum UserRole: String, Codable {
    case landlord = "landlord"
    case homeowner = "homeowner"
    case employer = "employer"
}

struct User: Codable, Identifiable {
    let id: String
    let email: String
    let name: String
    let role: UserRole
    let phone: String?
    let profileImageUrl: String?
    let bio: String?
    let createdAt: String
}

struct AuthResponse: Codable {
    let user: User
    let accessToken: String
    let refreshToken: String
}

// Models/Land.swift
enum LandStatus: String, Codable {
    case active = "active"
    case forSale = "for_sale"
    case sold = "sold"
}

struct Land: Codable, Identifiable {
    let id: String
    let ownerId: String
    let name: String
    let address: String
    let area: Double
    let zoning: String?
    let status: LandStatus
    let isPublic: Bool
    let thumbnailUrl: String?
    let purchasePrice: Double?
    let currentValue: Double?
    let tags: [String]
    let createdAt: String
    let updatedAt: String
}
```

---

## セットアップ要件 / Setup Requirements

| 要件 | バージョン |
|------|-----------|
| Xcode | 15.0 以上 |
| iOS Deployment Target | iOS 17.0 以上 |
| Swift | 5.9 以上 |
| macOS (開発用) | macOS 14 Sonoma 以上 |
| Apple Developer Account | 実機テスト・App Store 配布に必要 |

### Xcode でのセットアップ

```bash
# リポジトリをクローン後
open Arvana-Terra-iOS/Arvana-Terra-iOS.xcodeproj

# または
cd Arvana-Terra-iOS
open Arvana-Terra-iOS.xcodeproj

# バックエンドサーバーが localhost:3000 で起動していることを確認
# Config/AppConfig.swift の apiBaseURL を確認・変更
```

---

## ビルドとデプロイ / Build and Deployment

### 開発ビルド（シミュレーター）

```
1. Xcode でプロジェクトを開く
2. シミュレーター（iPhone 15 Pro など）を選択
3. Cmd + R でビルド・実行
```

### 実機デバッグ

```
1. Apple Developer Account でプロビジョニングプロファイルを設定
2. iPhone を USB 接続
3. Xcode でデバイスを選択
4. Cmd + R でビルド・転送
```

### App Store 配布

```
1. App Store Connect でアプリ登録
2. Xcode: Product → Archive
3. Organizer から "Distribute App" → App Store Connect
4. TestFlight でテスト後、審査提出
```

---

## テストアプローチ / Testing Approach

```swift
// Arvana-Terra-iOSTests/
// XCTest フレームワークを使用

// ユニットテスト例
class LandViewModelTests: XCTestCase {
    func testFetchLands_ReturnsLands() async throws {
        let mockService = MockLandService()
        let viewModel = LandListViewModel(service: mockService)
        await viewModel.fetchLands()
        XCTAssertFalse(viewModel.lands.isEmpty)
    }
}

// UIテスト
class ArvanaTerraUITests: XCTestCase {
    func testLoginFlow() {
        let app = XCUIApplication()
        app.launch()
        // ログインフォームに入力・送信してダッシュボードが表示されることを確認
    }
}
```

---

## ナビゲーションフロー / Navigation Flow

```
起動
  ↓
ContentView
  ├── 未認証 → LoginView
  │              ↓ ログイン成功
  └── 認証済み → MainTabView
                   ├── Tab 1: ダッシュボード (DashboardView)
                   │
                   ├── Tab 2: 土地 (LandListView)
                   │           ↓ 選択
                   │         LandDetailView
                   │           ├── 編集 → LandFormView
                   │           ├── チャット → ChatRoomListView → ChatDetailView
                   │           └── タスク → TaskListView
                   │
                   ├── Tab 3: 物件 (PropertyListView)
                   │           ↓ 選択
                   │         PropertyDetailView
                   │           ├── 部屋 → RoomListView → RoomDetailView
                   │           ├── 設備 → EquipmentListView
                   │           ├── チャット → ChatRoomListView → ChatDetailView
                   │           └── タスク → TaskListView
                   │
                   ├── Tab 4: チャット (ChatRoomListView)
                   │           ↓
                   │         ChatDetailView (リアルタイム)
                   │
                   ├── Tab 5: SNS (SnsFeedView)
                   │           ↓
                   │         SnsPostDetailView
                   │
                   └── Tab 6: 設定 (SettingsView)
```
