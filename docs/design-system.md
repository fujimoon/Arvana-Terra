# Arvana Terra - デザインシステム / Design System

## デザイン哲学 / Design Philosophy

### コンセプト: AXIS Magazine Aesthetic

Arvana Terra のデザインは、日本の高品質ライフスタイル誌「AXIS」のような美的感覚を基盤としています。白を基調とした余白の多いレイアウト、抑えたカラーパレット、シャープなタイポグラフィ、そして情報の明快な階層構造を特徴とします。

Arvana Terra's design is rooted in the aesthetic of Japan's premium lifestyle and design magazine "AXIS." Key characteristics include whitespace-rich layouts, a restrained color palette, sharp typography, and clear information hierarchy.

### デザイン原則 / Design Principles

1. **明快さ (Clarity):** 地主・家主は複雑な資産情報を扱う。情報の優先度を明確にし、必要な情報に素早くアクセスできる設計
2. **信頼性 (Trust):** 財産・契約・テナント情報を扱うプラットフォームとして、プロフェッショナルで信頼できる印象を与える
3. **効率性 (Efficiency):** 繰り返し行う操作（物件確認・支払い確認・タスク更新）を最小のタップ・クリックで完了できる
4. **一貫性 (Consistency):** Web・iOS・Android で同じビジュアル言語を使用。プラットフォームを跨いでも迷わない UI

---

## カラーパレット / Color Palette

### プライマリカラー / Primary Colors

| 名称 | HEX | RGB | 用途 |
|------|-----|-----|------|
| Primary Navy | `#1B3A6B` | rgb(27, 58, 107) | メインボタン・ナビゲーション・見出し |
| Secondary Blue | `#2E5EAA` | rgb(46, 94, 170) | サブボタン・リンク・強調テキスト |
| Accent Blue | `#4A90D9` | rgb(74, 144, 217) | フォーカスリング・バッジ・ハイライト |

### セマンティックカラー / Semantic Colors

| 名称 | HEX | RGB | 用途 |
|------|-----|-----|------|
| Success Green | `#059669` | rgb(5, 150, 105) | 成功・入居中(occupied)・支払い済み・good |
| Warning Orange | `#D97706` | rgb(217, 119, 6) | 警告・延滞(late)・要注意 |
| Error Red | `#DC2626` | rgb(220, 38, 38) | エラー・破損(broken)・過延滞 |
| Info Blue | `#3B82F6` | rgb(59, 130, 246) | 情報・お知らせ |

### ニュートラルカラー / Neutral Colors

| 名称 | HEX | RGB | 用途 |
|------|-----|-----|------|
| Text Dark | `#1A1A2E` | rgb(26, 26, 46) | 本文テキスト・見出し |
| Text Gray | `#6B7280` | rgb(107, 114, 128) | 補足テキスト・プレースホルダー |
| Background Gray | `#FAFAFA` | rgb(250, 250, 250) | ページ背景 |
| Surface White | `#FFFFFF` | rgb(255, 255, 255) | カード・モーダル背景 |
| Border Gray | `#E5E7EB` | rgb(229, 231, 235) | ボーダー・セパレーター |
| Light Gray | `#F3F4F6` | rgb(243, 244, 246) | テーブル交互行・無効状態 |
| Medium Gray | `#9CA3AF` | rgb(156, 163, 175) | 無効ボタン・アイコン |

### ステータスカラー対応表 / Status Color Mapping

#### 土地・物件ステータス

| ステータス | カラー | バッジ背景 | バッジテキスト |
|-----------|--------|-----------|--------------|
| active | Success Green | `#D1FAE5` | `#059669` |
| for_sale | Accent Blue | `#DBEAFE` | `#2563EB` |
| sold | Medium Gray | `#F3F4F6` | `#6B7280` |
| under_renovation | Warning Orange | `#FEF3C7` | `#D97706` |

#### 部屋ステータス

| ステータス | カラー | 日本語 |
|-----------|--------|--------|
| occupied | Success Green | 入居中 |
| vacant | Accent Blue | 空室 |
| maintenance | Warning Orange | 修繕中 |

#### 設備ステータス

| ステータス | カラー | 日本語 |
|-----------|--------|--------|
| good | Success Green | 正常 |
| warning | Warning Orange | 要注意 |
| broken | Error Red | 故障中 |
| replaced | Medium Gray | 交換済 |

#### 支払いステータス

| ステータス | カラー | 日本語 |
|-----------|--------|--------|
| paid | Success Green | 支払済 |
| pending | Accent Blue | 未払い |
| late | Warning Orange | 延滞 |
| overdue | Error Red | 過延滞 |

---

## タイポグラフィ / Typography

### フォントファミリー

| プラットフォーム | フォント | 理由 |
|----------------|--------|------|
| Web | system-ui → -apple-system → Segoe UI | プラットフォーム最適フォント |
| iOS | San Francisco (SF Pro) | Apple 標準 |
| Android | Google Sans / Roboto | Material3 標準 |

### タイポグラフィスケール

| レベル | サイズ | ウェイト | 用途 | Tailwind/SwiftUI |
|--------|-------|----------|------|-----------------|
| Display | 36px / 40px | 700 Bold | ランディングページ大見出し | `text-4xl font-bold` |
| H1 | 30px | 700 Bold | ページタイトル | `text-3xl font-bold` |
| H2 | 24px | 600 SemiBold | セクション見出し | `text-2xl font-semibold` |
| H3 | 20px | 600 SemiBold | カード見出し | `text-xl font-semibold` |
| H4 | 18px | 600 SemiBold | サブセクション | `text-lg font-semibold` |
| Body Large | 16px | 400 Regular | 本文・フォームラベル | `text-base` |
| Body | 14px | 400 Regular | 一般テキスト | `text-sm` |
| Body Small | 12px | 400 Regular | 補足・キャプション | `text-xs` |
| Label | 14px | 500 Medium | ボタン・バッジ | `text-sm font-medium` |
| Overline | 12px | 600 SemiBold uppercase | セクションラベル | `text-xs font-semibold uppercase` |

---

## スペーシングシステム / Spacing System

Tailwind CSS の 4px ベースグリッドシステムを採用しています。

| トークン | サイズ | 用途 |
|---------|-------|------|
| `space-1` | 4px | アイコンとテキストの間隔 |
| `space-2` | 8px | インラインアイテム間 |
| `space-3` | 12px | フォームフィールド間 |
| `space-4` | 16px | カード内パディング |
| `space-5` | 20px | セクション間 |
| `space-6` | 24px | カード間・グリッドギャップ |
| `space-8` | 32px | セクション見出しマージン |
| `space-10` | 40px | ページセクション間 |
| `space-12` | 48px | 大きなセクション間 |
| `space-16` | 64px | ページトップマージン |

### コンポーネント別スペーシング

```css
/* カード */
.card { padding: 24px; border-radius: 12px; }

/* ボタン */
.btn-primary { padding: 10px 20px; }
.btn-sm { padding: 6px 12px; }
.btn-lg { padding: 14px 28px; }

/* フォームフィールド */
.input { padding: 10px 14px; margin-bottom: 16px; }

/* ページレイアウト */
.page-container { max-width: 1280px; padding: 0 24px; }
.sidebar { width: 256px; }
.content-area { padding: 24px; }
```

---

## コンポーネント仕様 / Component Specifications

### ボタン / Buttons

```
Primary Button:
  背景: Primary Navy (#1B3A6B)
  テキスト: White
  ホバー: Secondary Blue (#2E5EAA)
  無効: Medium Gray (#9CA3AF)
  角丸: 8px (rounded-lg)
  パディング: 10px 20px

Secondary Button:
  背景: White
  ボーダー: Primary Navy (#1B3A6B) 1px
  テキスト: Primary Navy
  ホバー: Background Gray (#FAFAFA)

Danger Button:
  背景: Error Red (#DC2626)
  テキスト: White
  ホバー: #B91C1C

Ghost Button:
  背景: Transparent
  テキスト: Primary Navy
  ホバー: Background Gray (#FAFAFA)
```

### カード / Cards

```
Standard Card:
  背景: Surface White (#FFFFFF)
  ボーダー: Border Gray (#E5E7EB) 1px
  角丸: 12px (rounded-xl)
  シャドウ: shadow-sm (0 1px 2px rgba(0,0,0,0.05))
  ホバー: shadow-md (0 4px 6px rgba(0,0,0,0.07))
  パディング: 24px (p-6)

Property/Land Card:
  サムネイル画像: 上部 (aspect-ratio: 16/9)
  コンテンツ: 下部パディング 16px
  ステータスバッジ: 画像右上に絶対配置

Interactive Card (クリック可能):
  cursor: pointer
  トランジション: transition-shadow duration-200
```

### バッジ / Badges

```
ステータスバッジ (丸角ピル形):
  角丸: 9999px (rounded-full)
  パディング: 2px 10px
  フォント: 12px, font-medium

例:
  occupied: bg-green-100 text-green-700
  vacant: bg-blue-100 text-blue-700
  maintenance: bg-orange-100 text-orange-700
  broken: bg-red-100 text-red-700
  good: bg-emerald-100 text-emerald-700

カテゴリバッジ:
  bg-gray-100 text-gray-700 (ニュートラル)
```

### フォームインプット / Form Inputs

```
テキスト入力:
  ボーダー: Border Gray 1px
  角丸: 8px
  パディング: 10px 14px
  フォーカス: Accent Blue のアウトライン (2px)
  エラー: Error Red のボーダー + エラーメッセージ

セレクトボックス:
  同上 + ドロップダウンアイコン

ラベル:
  text-sm font-medium text-gray-700
  マージン下: 4px

エラーメッセージ:
  text-xs text-red-600
  マージン上: 4px
```

### テーブル / Tables

```
テーブルヘッダー:
  背景: Light Gray (#F3F4F6)
  テキスト: text-xs font-semibold uppercase text-gray-500
  パディング: 12px 16px

テーブル行:
  奇数行: Surface White
  偶数行: Background Gray (#FAFAFA)
  ホバー: #F0F4FF (薄いブルー)
  パディング: 16px

ボーダー:
  行間: Border Gray 1px
```

### モーダル / Modals

```
オーバーレイ: rgba(0,0,0,0.5)
モーダルコンテナ:
  背景: Surface White
  角丸: 16px
  最大幅: 480px (sm), 640px (md), 800px (lg)
  パディング: 24px
  シャドウ: shadow-xl

ヘッダー: H3 (font-semibold) + 閉じるボタン (×)
フッター: ボタン群 (右寄せ)
  スタック順: Cancel → Confirm
```

---

## モーションデザイン / Motion Design

### アニメーション原則

- **目的を持ったアニメーション:** 状態変化を伝えるために使用。装飾的なアニメーションは避ける
- **速度:** 素早く (`150ms`-`300ms`)。ユーザーを待たせない
- **イージング:** `ease-in-out` を基本。入場は `ease-out`、退場は `ease-in`

### 標準トランジション

```css
/* ホバー効果 */
.transition-standard {
  transition: all 150ms ease-in-out;
}

/* モーダル・ドロップダウン開閉 */
.transition-modal {
  transition: opacity 200ms ease-in-out, transform 200ms ease-in-out;
}

/* ページ遷移 */
.page-enter { opacity: 0; transform: translateY(8px); }
.page-enter-active { opacity: 1; transform: translateY(0); transition: all 300ms ease-out; }

/* ローディングスピナー */
.spinner { animation: spin 1s linear infinite; }
```

### Tailwind CSS トランジション

```html
<!-- ボタンホバー -->
<button class="transition-colors duration-150 ease-in-out hover:bg-secondary-blue">

<!-- カードホバー -->
<div class="transition-shadow duration-200 hover:shadow-md">

<!-- フェードイン -->
<div class="animate-fade-in opacity-0 animate-[fadeIn_0.3s_ease-out_forwards]">
```

---

## アイコンシステム / Icon System

### Web (Heroicons)

Heroicons 2.x を使用。Outline と Solid の2バリアント。

```typescript
// 主要アイコン使用例
import {
  HomeIcon,         // ホーム
  MapIcon,          // 土地
  BuildingOffice2Icon, // 物件
  UserGroupIcon,    // 入居者
  WrenchScrewdriverIcon, // 設備
  DocumentTextIcon, // 契約書
  ChatBubbleLeftRightIcon, // チャット
  CheckCircleIcon,  // 完了タスク
  ExclamationTriangleIcon, // 警告
  ChartLineIcon,    // 分析
  MagnifyingGlassIcon, // 検索
  PlusCircleIcon,   // 追加
  PencilIcon,       // 編集
  TrashIcon,        // 削除
  BellIcon,         // 通知
  Cog6ToothIcon,    // 設定
} from '@heroicons/react/24/outline'

// アクティブ状態は Solid バリアント
import { HomeIcon as HomeIconSolid } from '@heroicons/react/24/solid'
```

### iOS (SF Symbols)

```swift
// 主要 SF Symbols
"house.fill"                     // ホーム
"map.fill"                       // 土地
"building.2.fill"                // 物件
"person.fill"                    // ユーザー
"person.2.fill"                  // 入居者
"wrench.and.screwdriver.fill"    // 設備
"doc.text.fill"                  // 契約書
"bubble.left.and.bubble.right.fill" // チャット
"checkmark.circle.fill"          // 完了
"exclamationmark.triangle.fill"  // 警告
"chart.line.uptrend.xyaxis.circle.fill" // 分析
"magnifyingglass"                // 検索
"plus.circle.fill"               // 追加
"pencil.circle.fill"             // 編集
"trash.fill"                     // 削除
"bell.fill"                      // 通知
"gearshape.fill"                 // 設定
"yensign.circle.fill"            // 支払い (¥)
"leaf.fill"                      // 土地/自然
"key.fill"                       // 鍵/入居
```

### Android (Material Icons)

```kotlin
// Compose Icons (Material3 Icons Extended)
Icons.Filled.Home
Icons.Filled.Map
Icons.Filled.Business
Icons.Filled.Person
Icons.Filled.Group
Icons.Filled.Build
Icons.Filled.Description
Icons.Filled.Chat
Icons.Filled.CheckCircle
Icons.Filled.Warning
Icons.Filled.TrendingUp
Icons.Filled.Search
Icons.Filled.AddCircle
Icons.Filled.Edit
Icons.Filled.Delete
Icons.Filled.Notifications
Icons.Filled.Settings
```

---

## プラットフォーム適応 / Platform Adaptation Notes

### Web

- **レスポンシブデザイン:** モバイル (sm: 640px)、タブレット (md: 768px, lg: 1024px)、デスクトップ (xl: 1280px)
- **サイドバーナビゲーション:** デスクトップでは常時表示、モバイルではハンバーガーメニュー
- **グリッドレイアウト:** 物件・土地カードは `grid-cols-1 md:grid-cols-2 xl:grid-cols-3`
- **ホバーエフェクト:** デスクトップのみ (マウスホバー)

### iOS

- **HIG 準拠:** Apple Human Interface Guidelines に従ったUIパターン
- **Safe Area:** ノッチ・Dynamic Island を考慮した safeAreaInsets 対応
- **ナビゲーション:** NavigationStack + TabView の組み合わせ
- **スワイプジェスチャー:** 戻るジェスチャー・リストの削除スワイプ
- **Dynamic Type:** アクセシビリティのためのフォントスケール対応

### Android

- **Material3 ガイドライン:** Material You デザインシステムに準拠
- **Window Insets:** システムバー (ステータスバー・ナビゲーションバー) の考慮
- **ボトムナビゲーション:** 主要 5〜6 セクションへのアクセス
- **Adaptive Layout:** コンパクト・ミディアム・エクスパンデッド の3ウィンドウサイズクラス対応
- **Edge-to-Edge:** `enableEdgeToEdge()` によるフルスクリーン UI

---

## アクセシビリティ / Accessibility

### Web
- すべてのインタラクティブ要素に `aria-label` または可視ラベル
- カラーコントラスト比 4.5:1 以上 (WCAG 2.1 AA 準拠)
- キーボードナビゲーション対応 (`Tab` キー・`Enter` キー)
- フォームのエラーメッセージは `role="alert"`

### iOS
- VoiceOver ラベル: `.accessibilityLabel("土地詳細")`
- 動的フォントサイズ: `@ScaledMetric` / `dynamicTypeSize`
- コントラスト比: iOS ダークモード対応

### Android
- TalkBack サポート: `contentDescription` の適切な設定
- 最小タッチターゲット: 48dp × 48dp (Material Design ガイドライン)
- `Modifier.semantics { contentDescription = "..." }` の使用
