# Arvana-Terra-Android - 詳細仕様 / Detailed Specification

## 概要 / Overview

Arvana Terra の Android 向けネイティブアプリです。Kotlin/Jetpack Compose で構築され、MVVM + Repository パターンを採用しています。依存性注入に Hilt、HTTP 通信に Retrofit2 を使用します。チャット機能は 3 秒間隔の HTTP ポーリングで実装しており、socket.io-client への依存なしに動作します。

Native Android app for Arvana Terra. Built with Kotlin/Jetpack Compose using MVVM + Repository pattern. Uses Hilt for DI and Retrofit2 for HTTP. Chat feature uses 3-second HTTP polling (no socket.io-client dependency required).

---

## テクノロジースタック / Tech Stack

| パッケージ / ライブラリ | バージョン | 用途 |
|----------------------|-----------|------|
| Kotlin | 2.x | プログラミング言語 |
| Jetpack Compose | BOM 最新 | UI フレームワーク |
| Material3 | Compose BOM | デザインシステム |
| Hilt | 最新 | 依存性注入 (DI) |
| Retrofit2 | 最新 | REST API HTTP クライアント |
| OkHttp3 | 最新 | HTTP クライアント基盤 + ロギングインターセプター |
| Gson | 最新 | JSON シリアライズ/デシリアライズ |
| socket.io-client | 最新 | WebSocket / Socket.io（通知用） ※チャットは HTTP ポーリング |
| Coroutines | 最新 | 非同期処理 |
| ViewModel / LiveData | Lifecycle | 状態管理 |
| Navigation Compose | 最新 | 画面遷移 |
| DataStore Preferences | 最新 | 設定・トークン保存 |
| Coil | 最新 | 画像読み込み |
| KSP | 最新 | コード生成（Hilt用） |
| Min SDK | 34 (Android 14) | 最小対応 Android バージョン |
| Target SDK | 34 | ターゲット SDK |
| App ID | `jp.co.arvana.terra` | アプリ識別子 |

---

## ビルド設定 / Build Configuration

```kotlin
// app/build.gradle.kts
android {
    namespace = "jp.co.arvana.terra"
    compileSdk = 34

    defaultConfig {
        applicationId = "jp.co.arvana.terra"
        minSdk = 34
        targetSdk = 34
        versionCode = 1
        versionName = "1.0.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"))
        }
        debug {
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
        }
    }

    buildFeatures { compose = true }
    kotlinOptions { jvmTarget = "17" }
}
```

---

## ディレクトリ構造 / Directory Structure

```
Arvana-Terra-Android/
├── app/
│   ├── build.gradle.kts
│   └── src/main/java/jp/co/arvana/terra/
│       ├── ArvanaApplication.kt        # @HiltAndroidApp
│       ├── MainActivity.kt             # @AndroidEntryPoint + NavGraph
│       ├── data/
│       │   ├── api/
│       │   │   ├── ApiService.kt       # Retrofit インターフェース定義
│       │   │   └── models/            # API レスポンスモデル
│       │   │       ├── UserModels.kt
│       │   │       ├── LandModels.kt
│       │   │       ├── PropertyModels.kt
│       │   │       ├── RoomModels.kt
│       │   │       ├── EquipmentModels.kt
│       │   │       ├── ContractModels.kt
│       │   │       ├── TaskModels.kt
│       │   │       ├── VendorModels.kt
│       │   │       └── SnsModels.kt
│       │   ├── model/
│       │   │   └── Chat.kt             # ChatRoom, ChatMessage, ChatUserRef, CreateChatRoomRequest
│       │   ├── remote/
│       │   │   └── ChatApiService.kt   # チャット専用 Retrofit インターフェース（Hilt 提供）
│       │   ├── local/
│       │   │   └── TokenDataStore.kt  # DataStore Preferences
│       │   └── repository/
│       │       ├── AuthRepository.kt
│       │       ├── LandRepository.kt
│       │       ├── PropertyRepository.kt
│       │       ├── RoomRepository.kt
│       │       ├── EquipmentRepository.kt
│       │       ├── TaskRepository.kt
│       │       ├── ChatRepository.kt
│       │       └── VendorRepository.kt
│       ├── di/
│       │   ├── NetworkModule.kt       # Retrofit, OkHttp Hilt モジュール
│       │   ├── RepositoryModule.kt    # Repository Hilt モジュール
│       │   └── DataStoreModule.kt     # DataStore Hilt モジュール
│       ├── socket/
│       │   └── SocketManager.kt       # socket.io-client 管理
│       └── ui/
│           ├── navigation/
│           │   └── NavGraph.kt         # Compose Navigation グラフ
│           ├── theme/
│           │   ├── Color.kt            # カラーパレット
│           │   ├── Theme.kt            # Material3 テーマ
│           │   └── Type.kt             # タイポグラフィ
│           ├── auth/
│           │   ├── LoginScreen.kt
│           │   ├── RegisterScreen.kt
│           │   └── AuthViewModel.kt
│           ├── dashboard/
│           │   ├── DashboardScreen.kt
│           │   └── DashboardViewModel.kt
│           ├── land/
│           │   ├── LandListScreen.kt
│           │   ├── LandDetailScreen.kt
│           │   ├── LandFormScreen.kt
│           │   └── LandViewModel.kt
│           ├── property/
│           │   ├── PropertyListScreen.kt
│           │   ├── PropertyDetailScreen.kt
│           │   ├── RoomListScreen.kt
│           │   ├── RoomDetailScreen.kt
│           │   └── PropertyViewModel.kt
│           ├── equipment/
│           │   ├── EquipmentScreen.kt
│           │   └── EquipmentViewModel.kt
│           ├── chat/
│           │   ├── ChatListScreen.kt   # チャットルーム一覧・新規作成ダイアログ
│           │   ├── ChatRoomScreen.kt   # メッセージ表示・送信（3秒ポーリング）
│           │   └── ChatViewModel.kt    # @HiltViewModel、ポーリング管理
│           ├── sns/
│           │   ├── SnsFeedScreen.kt
│           │   ├── SnsDetailScreen.kt
│           │   └── SnsViewModel.kt
│           ├── vendor/
│           │   ├── VendorListScreen.kt
│           │   └── VendorViewModel.kt
│           ├── task/
│           │   ├── TaskListScreen.kt
│           │   └── TaskViewModel.kt
│           ├── settings/
│           │   └── SettingsScreen.kt
│           └── components/
│               ├── StatusBadge.kt
│               ├── PropertyCard.kt
│               ├── LandCard.kt
│               └── LoadingIndicator.kt
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

---

## アーキテクチャ / Architecture (MVVM + Repository)

```
UI Layer (Compose Screens)
  │── collectAsStateWithLifecycle ──► ViewModel (HiltViewModel)
  │                                      │── launch(coroutine) ──► Repository
  │                                      │                            │── ApiService (Retrofit)
  │                                      │                            │    └── HTTP → Backend
  │                                      │                            │── TokenDataStore
  │                                      │◄── Result<T>
  │◄── StateFlow<UiState>

例:
PropertyListScreen
  └── val state by viewModel.uiState.collectAsStateWithLifecycle()
        └── PropertyListViewModel (HiltViewModel)
              └── @Inject constructor(private val repo: PropertyRepository)
                    └── fun loadProperties() = viewModelScope.launch {
                          repo.getMyProperties().collect { result -> ... }
                        }
                          └── PropertyRepository
                                └── suspend fun getMyProperties() =
                                      apiService.getMyProperties()
                                        └── Retrofit: GET /api/v1/my/properties
```

---

## 依存性注入 (Hilt) / Dependency Injection

### NetworkModule

```kotlin
// di/NetworkModule.kt
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    private const val BASE_URL = "http://10.0.2.2:3000/api/v1/"
    // 実機テスト: "http://192.168.x.x:3000/api/v1/"
    // 本番: "https://api.arvana-terra.jp/api/v1/"

    @Provides
    @Singleton
    fun provideOkHttpClient(
        tokenDataStore: TokenDataStore
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor { chain ->
                val token = runBlocking { tokenDataStore.getAccessToken() }
                val request = if (token != null) {
                    chain.request().newBuilder()
                        .addHeader("Authorization", "Bearer $token")
                        .build()
                } else {
                    chain.request()
                }
                chain.proceed(request)
            }
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY  // Debug のみ
            })
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}
```

### RepositoryModule

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun bindAuthRepository(
        impl: AuthRepositoryImpl
    ): AuthRepository

    @Binds
    @Singleton
    abstract fun bindLandRepository(
        impl: LandRepositoryImpl
    ): LandRepository

    // ... 他の Repository も同様
}
```

---

## Retrofit API Service / API インターフェース

```kotlin
// data/api/ApiService.kt
interface ApiService {
    // 認証
    @POST("auth/login")
    suspend fun login(@Body request: LoginRequest): AuthResponse

    @POST("auth/register")
    suspend fun register(@Body request: RegisterRequest): AuthResponse

    @POST("auth/refresh")
    suspend fun refresh(@Body request: RefreshRequest): TokenResponse

    @GET("auth/me")
    suspend fun getMe(): User

    // 土地
    @GET("my/lands")
    suspend fun getMyLands(
        @Query("status") status: String? = null,
        @Query("page") page: Int = 1,
        @Query("limit") limit: Int = 20
    ): PaginatedResponse<Land>

    @POST("my/lands")
    suspend fun createLand(@Body request: CreateLandRequest): Land

    @GET("my/lands/{id}")
    suspend fun getLandById(@Path("id") id: String): Land

    @PATCH("my/lands/{id}")
    suspend fun updateLand(@Path("id") id: String, @Body request: UpdateLandRequest): Land

    @DELETE("my/lands/{id}")
    suspend fun deleteLand(@Path("id") id: String): Unit

    // 物件
    @GET("my/properties")
    suspend fun getMyProperties(
        @Query("page") page: Int = 1,
        @Query("limit") limit: Int = 20
    ): PaginatedResponse<Property>

    @GET("my/properties/{id}")
    suspend fun getPropertyById(@Path("id") id: String): Property

    @GET("my/properties/{id}/rooms")
    suspend fun getRooms(@Path("id") propertyId: String): List<Room>

    @GET("my/properties/{id}/equipment")
    suspend fun getEquipment(@Path("id") propertyId: String): List<Equipment>

    @GET("my/properties/{id}/tasks")
    suspend fun getPropertyTasks(
        @Path("id") propertyId: String,
        @Query("status") status: String? = null
    ): List<Task>

    // ※ チャット API は ChatApiService.kt（別 Retrofit インスタンス）で定義

    // タスク
    @POST("tasks")
    suspend fun createTask(@Body request: CreateTaskRequest): Task

    @PATCH("tasks/{id}")
    suspend fun updateTask(@Path("id") id: String, @Body request: UpdateTaskRequest): Task

    // 業者
    @GET("vendors")
    suspend fun getVendors(
        @Query("category") category: String? = null,
        @Query("page") page: Int = 1
    ): PaginatedResponse<Vendor>

    // SNS
    @GET("sns/posts")
    suspend fun getSnsPosts(
        @Query("type") type: String? = null,
        @Query("page") page: Int = 1
    ): PaginatedResponse<SnsPost>
}
```

---

## チャット実装（HTTP ポーリング）

チャット機能は socket.io-client を使用せず、3 秒間隔の HTTP ポーリングで実装しています。`ChatViewModel` が `viewModelScope` 内でループし、`ChatApiService` を通じて最新メッセージを取得します。

```kotlin
// data/remote/ChatApiService.kt（Hilt で提供）
interface ChatApiService {
    @GET("chats")
    suspend fun getChatRooms(
        @Query("type") type: String,
        @Query("targetId") targetId: String
    ): ApiResponse<List<ChatRoom>>

    @POST("chats")
    suspend fun createChatRoom(@Body request: CreateChatRoomRequest): ApiResponse<ChatRoom>

    @GET("chats/{id}")
    suspend fun getChatRoom(@Path("id") id: String): ApiResponse<ChatRoom>

    @GET("chats/{id}/messages")
    suspend fun getMessages(
        @Path("id") id: String,
        @Query("page") page: Int = 1
    ): ApiResponse<ChatMessagesResponse>

    @POST("chats/{id}/messages")
    suspend fun sendMessage(
        @Path("id") id: String,
        @Body request: SendMessageRequest
    ): ApiResponse<ChatMessage>
}

// ui/chat/ChatViewModel.kt
@HiltViewModel
class ChatViewModel @Inject constructor(
    private val chatApiService: ChatApiService
) : ViewModel() {

    private var pollingJob: Job? = null

    fun startPolling(roomId: String) {
        pollingJob = viewModelScope.launch {
            while (isActive) {
                loadMessages(roomId)
                delay(3000)
            }
        }
    }

    fun stopPolling() {
        pollingJob?.cancel()
    }

    override fun onCleared() {
        super.onCleared()
        stopPolling()
    }
}
```

### Socket.io（通知用）

プッシュ通知向けに `SocketManager` を保持していますが、チャットには使用しません。

```kotlin
// socket/SocketManager.kt（通知専用）
@Singleton
class SocketManager @Inject constructor(
    private val tokenDataStore: TokenDataStore
) {
    private var socket: Socket? = null

    fun connect() { /* /notification namespace に接続 */ }
    fun disconnect() { socket?.disconnect() }
}
```

---

## 画面一覧 / Screen List

### NavGraph

```kotlin
// ui/navigation/NavGraph.kt
sealed class Screen(val route: String) {
    object Login : Screen("login")
    object Register : Screen("register")
    object Dashboard : Screen("dashboard")

    // 土地
    object LandList : Screen("lands")
    object LandDetail : Screen("lands/{landId}")
    object LandForm : Screen("lands/form?landId={landId}")

    // 物件
    object PropertyList : Screen("properties")
    object PropertyDetail : Screen("properties/{propertyId}")
    object RoomList : Screen("properties/{propertyId}/rooms")
    object RoomDetail : Screen("properties/{propertyId}/rooms/{roomId}")
    object EquipmentList : Screen("properties/{propertyId}/equipment")
    object TaskList : Screen("properties/{propertyId}/tasks")

    // チャット（type/targetId/targetName で対象を指定、roomId/roomTitle でルーム遷移）
    object ChatList : Screen("chat-list/{type}/{targetId}/{targetName}")
    object ChatRoom : Screen("chat-room/{roomId}/{roomTitle}")

    // SNS
    object SnsFeed : Screen("sns")
    object SnsDetail : Screen("sns/{postId}")

    // 業者
    object VendorList : Screen("vendors")

    // 設定
    object Settings : Screen("settings")
}
```

### スクリーン一覧

| スクリーン | Route | 説明 |
|-----------|-------|------|
| ログイン | `login` | メール・パスワードログイン |
| 新規登録 | `register` | アカウント作成 |
| ダッシュボード | `dashboard` | 資産サマリー・タスク概要 |
| 土地一覧 | `lands` | 土地カードリスト・フィルタ |
| 土地詳細 | `lands/{id}` | 土地情報・関連物件 |
| 土地フォーム | `lands/form` | 作成・編集 |
| 物件一覧 | `properties` | 物件カードリスト |
| 物件詳細 | `properties/{id}` | 物件情報・ダッシュボード |
| 部屋一覧 | `properties/{id}/rooms` | フロア別部屋リスト |
| 部屋詳細 | `properties/{id}/rooms/{rid}` | 部屋・入居者・支払い |
| 設備一覧 | `properties/{id}/equipment` | カテゴリ別設備リスト |
| タスク一覧 | `properties/{id}/tasks` | タスク一覧・AI提案 |
| チャット一覧 | `chat-list/{type}/{targetId}/{targetName}` | 土地・物件・従業員別チャットルーム一覧 |
| チャット詳細 | `chat-room/{roomId}/{roomTitle}` | メッセージ表示・送信（3秒 HTTP ポーリング） |
| SNSフィード | `sns` | 投稿フィード |
| SNS詳細 | `sns/{id}` | 投稿詳細・コメント |
| 業者一覧 | `vendors` | 承認済み業者 |
| 設定 | `settings` | プロフィール・通知設定 |

---

## デザインシステム / Design System (Material3 + Custom)

### カラーパレット

```kotlin
// ui/theme/Color.kt
val PrimaryNavy = Color(0xFF1B3A6B)      // プライマリ
val SecondaryBlue = Color(0xFF2E5EAA)    // セカンダリ
val AccentBlue = Color(0xFF4A90D9)       // アクセント
val TextDark = Color(0xFF1A1A2E)         // 本文テキスト
val TextGray = Color(0xFF6B7280)         // サブテキスト
val SuccessGreen = Color(0xFF059669)     // 成功・入居中
val WarningOrange = Color(0xFFD97706)    // 警告
val ErrorRed = Color(0xFFDC2626)         // エラー
val BackgroundGray = Color(0xFFFAFAFA)   // ページ背景
val SurfaceWhite = Color(0xFFFFFFFF)     // カード背景
val BorderGray = Color(0xFFE5E7EB)       // ボーダー
```

### Material3 テーマ設定

```kotlin
// ui/theme/Theme.kt
private val LightColorScheme = lightColorScheme(
    primary = PrimaryNavy,
    onPrimary = Color.White,
    primaryContainer = AccentBlue.copy(alpha = 0.1f),
    secondary = SecondaryBlue,
    background = BackgroundGray,
    surface = SurfaceWhite,
    onSurface = TextDark,
    error = ErrorRed,
)

@Composable
fun ArvanaTerraTheme(
    content: @Composable () -> Unit
) {
    MaterialTheme(
        colorScheme = LightColorScheme,
        typography = ArvanaTerraTypography,
        content = content
    )
}
```

---

## DataStore によるトークン管理 / Token Management

```kotlin
// data/local/TokenDataStore.kt
@Singleton
class TokenDataStore @Inject constructor(
    @ApplicationContext context: Context
) {
    private val dataStore = context.createDataStore("arvana_terra_prefs")

    companion object {
        val ACCESS_TOKEN_KEY = stringPreferencesKey("access_token")
        val REFRESH_TOKEN_KEY = stringPreferencesKey("refresh_token")
    }

    suspend fun saveTokens(accessToken: String, refreshToken: String) {
        dataStore.edit { prefs ->
            prefs[ACCESS_TOKEN_KEY] = accessToken
            prefs[REFRESH_TOKEN_KEY] = refreshToken
        }
    }

    suspend fun getAccessToken(): String? {
        return dataStore.data.first()[ACCESS_TOKEN_KEY]
    }

    suspend fun getRefreshToken(): String? {
        return dataStore.data.first()[REFRESH_TOKEN_KEY]
    }

    suspend fun clearTokens() {
        dataStore.edit { it.clear() }
    }
}
```

---

## ViewModel の例 / ViewModel Example

```kotlin
// ui/land/LandViewModel.kt
@HiltViewModel
class LandViewModel @Inject constructor(
    private val repository: LandRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow(LandUiState())
    val uiState: StateFlow<LandUiState> = _uiState.asStateFlow()

    init {
        loadLands()
    }

    fun loadLands() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }
            try {
                val result = repository.getMyLands()
                _uiState.update { it.copy(lands = result.data, isLoading = false) }
            } catch (e: Exception) {
                _uiState.update { it.copy(error = e.message, isLoading = false) }
            }
        }
    }

    fun deleteLand(id: String) {
        viewModelScope.launch {
            try {
                repository.deleteLand(id)
                loadLands()  // 一覧を再取得
            } catch (e: Exception) {
                _uiState.update { it.copy(error = e.message) }
            }
        }
    }
}

data class LandUiState(
    val lands: List<Land> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)
```

---

## セットアップ要件 / Setup Requirements

| 要件 | バージョン/詳細 |
|------|---------------|
| Android Studio | Hedgehog (2023.1.1) 以上 |
| Kotlin | 2.x |
| Gradle | 8.x |
| Min SDK | 34 (Android 14) |
| JDK | 17 |

### Android Studio でのセットアップ

```bash
# リポジトリをクローン後
# Android Studio → "Open" → Arvana-Terra-Android フォルダを選択
# Gradle sync が自動実行される

# エミュレーター: API 34 (Android 14) の AVD を作成
# NetworkModule.kt の BASE_URL を確認
# エミュレーターから localhost へは 10.0.2.2 を使用
```

---

## ビルドとデプロイ / Build and Deployment

```bash
# Debug APK ビルド
./gradlew assembleDebug
# → app/build/outputs/apk/debug/app-debug.apk

# Release APK ビルド（署名設定後）
./gradlew assembleRelease

# Google Play 向け AAB ビルド
./gradlew bundleRelease
# → app/build/outputs/bundle/release/app-release.aab

# Google Play Console にアップロード
# 1. Internal Testing → Closed Testing (Beta) → Production
```

### Proguard 設定

```proguard
# proguard-rules.pro
# Retrofit
-keepattributes Signature
-keepattributes *Annotation*
-keep class retrofit2.** { *; }

# Gson
-keep class com.google.gson.** { *; }
-keep class jp.co.arvana.terra.data.api.models.** { *; }

# Socket.io
-keep class io.socket.** { *; }
```

---

## テストアプローチ / Testing Approach

```kotlin
// ユニットテスト (JUnit4)
// app/src/test/
class LandViewModelTest {
    @get:Rule val mainDispatcherRule = MainDispatcherRule()

    @Test
    fun `loadLands success updates uiState`() = runTest {
        val fakeRepo = FakeLandRepository()
        val viewModel = LandViewModel(fakeRepo)
        val state = viewModel.uiState.first()
        assertThat(state.lands).isNotEmpty()
        assertThat(state.isLoading).isFalse()
    }
}

// インストルメンテーションテスト (Compose UI Test)
// app/src/androidTest/
@Test
fun loginScreen_displaysEmailAndPasswordFields() {
    composeTestRule.setContent { LoginScreen(onLoginSuccess = {}) }
    composeTestRule.onNodeWithText("メールアドレス").assertIsDisplayed()
    composeTestRule.onNodeWithText("パスワード").assertIsDisplayed()
}
```

---

## ナビゲーションフロー / Navigation Flow

```
起動
  ↓
認証チェック (TokenDataStore)
  ├── 未認証 → LoginScreen
  │              ↓ ログイン成功
  └── 認証済み → メイン画面 (BottomNavigation)
                   ├── ホーム (DashboardScreen)
                   │
                   ├── 土地 (LandListScreen)
                   │    └── LandDetailScreen
                   │         ├── 編集 → LandFormScreen
                   │         └── チャット → ChatRoomScreen
                   │
                   ├── 物件 (PropertyListScreen)
                   │    └── PropertyDetailScreen
                   │         ├── 部屋 → RoomListScreen → RoomDetailScreen
                   │         ├── 設備 → EquipmentScreen
                   │         ├── タスク → TaskListScreen
                   │         └── チャット → ChatRoomScreen
                   │
                   ├── チャット (ChatListScreen)
                   │    └── ChatRoomScreen (3秒 HTTP ポーリング)
                   │
                   ├── SNS (SnsFeedScreen)
                   │    └── SnsDetailScreen
                   │
                   └── 設定 (SettingsScreen)
```
