# Arvana Terra

> 地主・家主のオールインワン資産・業務管理プラットフォーム
> All-in-One Property & Land Management Platform for Japanese Landlords and Homeowners

---

## Arvana Terra について / About Arvana Terra

Arvana Terra は、日本の地主（土地所有者）および家主（建物所有者）を対象とした、包括的な資産・業務管理プラットフォームです。土地・物件の管理から入居者対応、設備管理、契約書管理、AI を活用した資産価値評価まで、地主・家主の日常業務をすべてカバーします。

Arvana Terra is a comprehensive asset and operations management platform designed for Japanese landowners (地主) and property owners (家主). It covers everything from land and property management to tenant communication, equipment monitoring, contract management, and AI-powered asset valuation.

### 対象ユーザー / Target Users

| ロール | 説明 |
|--------|------|
| **地主 (landlord)** | 土地を所有・管理するユーザー。土地管理機能と物件管理機能の両方を利用可能 |
| **家主 (homeowner)** | 建物・物件を所有・管理するユーザー。物件管理機能に特化 |
| **雇用者 (employer)** | Arvana Work から連携した雇用者。担当物件・土地のチャット参加権限を持つ |
| **管理者 (admin)** | 業者登録管理・公式コンテンツ投稿を行うプラットフォーム管理者 |
| **スーパー管理者 (super_admin)** | システム全体の完全なアクセス権を持つ最上位管理者 |

### コンセプト / Concept

```
「地主・家主の仕事を、シンプルに、スマートに。」
Making the work of landlords and property owners simple and smart.
```

Arvana Terra は単なる管理ツールではありません。AI によるタスク提案・資産価値予測、スマートデバイス（水道メーター・電気メーター・カメラ）との連携、チャット機能による関係者間のリアルタイム連絡、そして Arvana Work との統合によって、地主・家主の事業を包括的にサポートします。

---

## アーキテクチャ概要 / Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Arvana Terra Platform                        │
│                                                                      │
│  ┌──────────────────┐                ┌──────────────────────────┐   │
│  │  arvana-terra-   │  REST API      │    arvana-terra-admin    │   │
│  │      web         │◄──────────────►│   (Admin Panel)          │   │
│  │ (React/Vite)     │                │   React/TypeScript       │   │
│  │  :5174           │                │   Docker + Nginx :3001   │   │
│  └──────────────────┘                └──────────────────────────┘   │
│           │                                      │                   │
│           │ REST API + Socket.io                 │ REST API          │
│           ▼                                      ▼                   │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                  arvana-terra-backend                        │    │
│  │              Node.js / TypeScript / Express.js               │    │
│  │                      Port: 3000                              │    │
│  │                                                              │    │
│  │   ┌──────────────┐  ┌──────────┐  ┌──────────────────────┐ │    │
│  │   │  PostgreSQL   │  │  Redis   │  │    Socket.io         │ │    │
│  │   │  (Prisma ORM) │  │(Session/ │  │  /chat namespace     │ │    │
│  │   │              │  │ Cache)   │  │  /notification ns    │ │    │
│  │   └──────────────┘  └──────────┘  └──────────────────────┘ │    │
│  └─────────────────────────────────────────────────────────────┘    │
│           │                                      │                   │
│           │ REST API + Socket.io                 │ REST API          │
│           ▼                                      ▼                   │
│  ┌──────────────────┐                ┌──────────────────────────┐   │
│  │  Arvana-Terra-   │                │    Arvana-Terra-         │   │
│  │      iOS         │                │      Android             │   │
│  │  Swift/SwiftUI   │                │  Kotlin/Jetpack Compose  │   │
│  │  iOS 17+         │                │  Min API 34              │   │
│  └──────────────────┘                └──────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

External Integrations:
  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
  │ Arvana Work  │  │   OpenAI     │  │  Smart Devices   │
  │ (雇用者連携) │  │  (AI機能)    │  │ (水道/電気/カメラ)|
  └──────────────┘  └──────────────┘  └──────────────────┘
```

---

## 各アプリの概要 / App Overview

### 1. arvana-terra-backend

**目的:** 全クライアントアプリの中核となるAPIサーバー
**Purpose:** Central API server and hub for all client applications

| 項目 | 詳細 |
|------|------|
| Runtime | Node.js 20+ |
| Language | TypeScript 5.3 |
| Framework | Express.js 4.18 |
| ORM | Prisma 5.9 + PostgreSQL 16 |
| Cache/Session | Redis 7 (ioredis 5.3) |
| Real-time | Socket.io 4.7 |
| Auth | JWT (jsonwebtoken 9) + bcryptjs |
| AI | OpenAI SDK 4.28 |
| Port | `3000` |
| API Prefix | `/api/v1` |

**詳細ドキュメント:** [docs/arvana-terra-backend.md](docs/arvana-terra-backend.md)

---

### 2. arvana-terra-web

**目的:** 地主・家主向けのメインWebクライアント
**Purpose:** Primary web client for landlords and homeowners

| 項目 | 詳細 |
|------|------|
| Language | TypeScript 5.7 |
| Framework | React 18.3 + Vite 5.4 |
| Styling | Tailwind CSS 3.4 |
| State Management | Zustand 4.5 + TanStack Query 5.62 |
| Routing | React Router DOM 6.28 |
| HTTP Client | Axios 1.7 |
| Real-time | socket.io-client 4.8 |
| Charts | Recharts 2.15 |
| Forms | React Hook Form 7.54 + Zod 3.24 |
| Port (dev) | `5174` |

**詳細ドキュメント:** [docs/arvana-terra-web.md](docs/arvana-terra-web.md)

---

### 3. arvana-terra-admin

**目的:** プラットフォーム管理者・雇用者向けの管理パネル
**Purpose:** Admin panel for platform administrators and employers (from Arvana Work)

| 項目 | 詳細 |
|------|------|
| Language | TypeScript 5.3 |
| Framework | React 18.2 + Vite 5.1 |
| Styling | Tailwind CSS 3.4 |
| State Management | Zustand 4.5 + TanStack Query 5.17 |
| HTTP Client | Axios 1.6 |
| Deployment | Docker + Nginx |
| Port (dev) | `3001` |
| Admin Roles | `admin` / `super_admin` / `employer` |

**詳細ドキュメント:** [docs/arvana-terra-admin.md](docs/arvana-terra-admin.md)

---

### 4. Arvana-Terra-iOS

**目的:** iPhone/iPad向けのネイティブiOSアプリ
**Purpose:** Native iOS app for iPhone and iPad

| 項目 | 詳細 |
|------|------|
| Language | Swift 5.9 |
| UI Framework | SwiftUI |
| Architecture | MVVM |
| HTTP Client | URLSession |
| WebSocket | URLSessionWebSocketTask |
| Min iOS | iOS 17.0 |
| Xcode | 15+ |
| App ID | `jp.co.arvana.terra` |

**詳細ドキュメント:** [docs/arvana-terra-ios.md](docs/arvana-terra-ios.md)

---

### 5. Arvana-Terra-Android

**目的:** Android向けのネイティブアプリ
**Purpose:** Native Android app

| 項目 | 詳細 |
|------|------|
| Language | Kotlin 2.x |
| UI Framework | Jetpack Compose |
| Architecture | MVVM + Repository |
| DI | Hilt |
| HTTP Client | Retrofit2 + OkHttp |
| WebSocket | socket.io-client (Android) |
| Min API | 34 (Android 14) |
| Target API | 34 |
| App ID | `jp.co.arvana.terra` |

**詳細ドキュメント:** [docs/arvana-terra-android.md](docs/arvana-terra-android.md)

---

## アプリ間連携 / App Integration

### 共通バックエンド / Shared Backend

すべてのクライアントアプリ（web, iOS, Android, admin）は同一のバックエンドAPIに接続します。認証はJWT（アクセストークン15分 + リフレッシュトークン30日）で統一されています。

All client apps (web, iOS, Android, admin) connect to the same backend API. Authentication is unified via JWT (access token: 15 minutes, refresh token: 30 days).

```
Client App  →  POST /api/v1/auth/login
            ←  { accessToken, refreshToken, user }

Subsequent  →  Authorization: Bearer <accessToken>
requests    ←  API Response

Token       →  POST /api/v1/auth/refresh { refreshToken }
refresh     ←  { accessToken }
```

### Socket.io リアルタイム通信 / Real-time Communication

```
Namespace: /chat
  - join_chat(chatRoomId)     → チャットルームに参加
  - leave_chat(chatRoomId)    → チャットルームから退出
  - send_message(data)        → メッセージ送信
  - typing(data)              → タイピング中通知
  Events:
  - new_message               → 新着メッセージ受信
  - user_joined               → ユーザー参加通知
  - user_left                 → ユーザー退出通知
  - user_typing               → タイピング中通知

Namespace: /notification
  - (auto-join personal room)
  Events:
  - notification              → プッシュ通知受信
```

### Arvana Work 連携 / Arvana Work Integration

Arvana Work（雇用管理プラットフォーム）の雇用者（employer）は、担当する物件・土地のチャットルームに参加できます。管理パネル（arvana-terra-admin）の employer ロールからアクセスし、プロパティ・土地のチャット管理を行います。

Employers from Arvana Work can join chat rooms for properties and lands they are assigned to. They access this through the `employer` role in the admin panel, with read access to message history and the ability to create/manage chat rooms for their assigned properties.

### 業者管理の連携 / Vendor Management Integration

```
Admin Panel (管理者)
    ↓ 業者を登録・承認
Backend DB (vendors table, isApproved=true)
    ↓ 承認済み業者のみ
Web/iOS/Android (クライアントアプリ)
    → 地主・家主が承認済み業者を参照・連携
```

---

## ユーザーロールと権限 / User Roles & Permissions

| 機能 | landlord | homeowner | employer | admin | super_admin |
|------|:--------:|:---------:|:--------:|:-----:|:-----------:|
| 土地管理 (Land management) | ✓ | - | - | - | ✓ |
| 物件管理 (Property management) | ✓ | ✓ | - | - | ✓ |
| 部屋・入居者管理 (Room/Tenant) | ✓ | ✓ | - | - | ✓ |
| 設備管理 (Equipment) | ✓ | ✓ | - | - | ✓ |
| 契約書管理 (Contracts) | ✓ | ✓ | - | - | ✓ |
| タスク管理 (Tasks) | ✓ | ✓ | - | - | ✓ |
| 従業員管理 (Employees) | ✓ | ✓ | - | - | ✓ |
| チャット参加 (Chat) | ✓ | ✓ | ✓(担当のみ) | ✓ | ✓ |
| SNS投稿 (SNS posts) | ✓ | ✓ | - | ✓(official) | ✓ |
| 資産価値AI分析 (Asset AI) | ✓ | ✓ | - | - | ✓ |
| ビジネスチャンス (Opportunities) | ✓ | ✓ | - | - | ✓ |
| 業者閲覧 (View vendors) | ✓ | ✓ | - | ✓ | ✓ |
| 業者管理 (Manage vendors) | - | - | - | ✓ | ✓ |
| 公式コンテンツ投稿 (Official content) | - | - | - | ✓ | ✓ |
| ユーザー管理 (User management) | - | - | - | - | ✓ |
| 全システムアクセス (Full system) | - | - | - | - | ✓ |

---

## 主要機能一覧 / Main Features

### 土地管理 / Land Management (地主のみ)
- 土地の登録・編集・削除
- 土地ステータス管理（active / for_sale / sold）
- 土地画像・サムネイル管理
- 土地関連契約書の管理
- 土地チャットルーム（関係者とのチャット）
- 土地関連タスクの管理

### 物件管理 / Property Management
- 物件の登録・編集・削除
- 物件種別（アパート、一戸建て、商業施設、倉庫等）
- 物件ステータス管理（active / for_sale / sold / under_renovation）
- 物件画像・サムネイル管理
- 物件ダッシュボード（入居率、月次収入）

### 部屋・入居者管理 / Room & Tenant Management
- 部屋の登録・一覧表示（フロア別）
- 部屋ステータス（occupied / vacant / maintenance）
- 入居者情報管理（氏名・連絡先・入居日・退去日）
- 家賃支払い記録（paid / pending / late / overdue）
- 支払い履歴・一覧

### 設備管理 / Equipment Management
- 設備の登録・一覧（照明・ドア・空調・配管・電気・エレベーター・カメラ等）
- 設備ステータス管理（good / warning / broken / replaced）
- 点検日・保証期限管理
- スマートデバイスデータ（水道メーター・電気メーター・カメラ）読み取り
- フロア別設備マップ表示

### 契約書管理 / Contract Management
- NDA・賃貸借・売買・管理契約書の作成・管理
- 契約書テンプレートの利用
- 契約ステータス（draft / active / expired / terminated）
- 契約当事者管理
- ファイルアップロード

### チャット / Chat & Communication
- 物件・土地・従業員チャットルーム
- Socket.io によるリアルタイムメッセージ配信
- テキスト・画像・ファイル送信
- 既読管理
- チャット参加者管理（admin による参加者追加・削除）

### タスク管理 / Task Management
- タスクの作成・進捗管理（todo / in_progress / done / cancelled）
- 優先度設定（low / medium / high / urgent）
- タスクの担当者割り当て
- AIによるタスク提案（物件情報・設備状態を分析）
- 期限管理

### 従業員管理 / Employee Management
- 従業員情報の登録・管理
- マイナンバー管理（オーナーのみ閲覧可能）
- 雇用形態・部署管理

### 資産価値・分析 / Asset Valuation & Analytics
- AI による資産価値評価・将来予測
- 月次収入・支出グラフ
- 入居率推移
- タスク完了率分析

### SNS / Community SNS
- 地主・家主コミュニティの投稿・いいね・コメント
- 投稿カテゴリ（general / advice / knowledge / event / case_study / official / tax_advice / vendor_info / announcement）
- 公式コンテンツ（admin が投稿）
- イベント機能（参加登録）

### ビジネスチャンス / Business Opportunities
- 土地購入・売却・賃貸・開発機会の管理
- AI分析コメント付き
- ブックマーク機能

### 業者管理 / Vendor Management
- 承認済み業者の閲覧・連携
- 業者カテゴリ（水道/電気/清掃/リノベーション等）
- サービスエリア検索

### 通知 / Notifications
- リアルタイムプッシュ通知（Socket.io）
- 支払い期日・設備警告・契約期限・タスク期限・チャット新着

---

## セットアップ / Setup Guide

### 前提条件 / Prerequisites

| ツール | バージョン | 用途 |
|--------|-----------|------|
| Node.js | 20.x 以上 | バックエンド・Webフロントエンド |
| PostgreSQL | 16.x 以上 | メインデータベース |
| Redis | 7.x 以上 | セッション・キャッシュ |
| Xcode | 15.x 以上 | iOS アプリ開発 |
| Android Studio | Hedgehog 以上 | Android アプリ開発 |
| Docker | 24.x 以上 | Admin パネルのデプロイ |

### リポジトリのクローン / Clone Repository

```bash
git clone https://github.com/arvana/arvana-terra.git
cd arvana-terra
```

### 起動順序 / Startup Order

**重要:** バックエンドを最初に起動してください。

```
1. arvana-terra-backend  (必須 - 最初に起動)
2. arvana-terra-web      (開発時)
3. arvana-terra-admin    (開発時)
4. Arvana-Terra-iOS      (Xcode から実行)
5. Arvana-Terra-Android  (Android Studio から実行)
```

### バックエンドのセットアップ / Backend Setup

```bash
cd arvana-terra-backend

# 依存関係インストール
npm install

# 環境変数設定
cp .env.example .env
# .env を編集: DATABASE_URL, JWT_SECRET, REDIS_URL 等を設定

# データベースセットアップ
npm run prisma:migrate    # マイグレーション実行
npm run prisma:seed       # シードデータ投入（任意）

# 開発サーバー起動
npm run dev
# → http://localhost:3000
```

### Webクライアントのセットアップ / Web Client Setup

```bash
cd arvana-terra-web

# 依存関係インストール
npm install

# 環境変数設定（任意 - デフォルトで localhost:3000 に接続）
cp .env.example .env.local

# 開発サーバー起動
npm run dev
# → http://localhost:5174
```

### 管理パネルのセットアップ / Admin Panel Setup

```bash
cd arvana-terra-admin

# 開発モード
npm install
npm run dev
# → http://localhost:3001

# Docker でのデプロイ
docker build -t arvana-terra-admin .
docker run -p 3001:80 arvana-terra-admin
```

### クイックスタート (Docker Compose) / Quick Start

```bash
# プロジェクトルートで実行
docker-compose up -d

# サービス確認
docker-compose ps
```

---

## 開発環境 / Development Environment

### 必須ツール / Required Tools

```bash
# Node.js バージョン確認
node --version   # v20.x 以上

# PostgreSQL 確認
psql --version   # 16.x 以上

# Redis 確認
redis-cli ping   # PONG

# iOS 開発
xcode-select --version   # Xcode 15 以上

# Android 開発
# Android Studio Hedgehog (2023.1.1) 以上
```

### 推奨 IDE 設定 / Recommended IDE Config

**VS Code Extensions:**
- ESLint
- Prettier
- Prisma
- Tailwind CSS IntelliSense
- TypeScript Vue Plugin (Volar)

**Xcode:**
- Swift Package Manager でサードパーティ依存なし（標準ライブラリのみ）

**Android Studio:**
- Kotlin plugin 最新版
- Jetpack Compose plugin

---

## 環境変数 / Environment Variables

### バックエンド (.env)

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/arvana_terra"

# Redis
REDIS_URL="redis://localhost:6379"

# JWT
JWT_SECRET="your-super-secret-key-min-32-chars"
JWT_EXPIRES_IN="15m"

# Server
PORT=3000
NODE_ENV=development

# CORS
ALLOWED_ORIGINS="http://localhost:5174,http://localhost:3001"

# File Upload
UPLOAD_DIR="./uploads"
MAX_FILE_SIZE="10485760"  # 10MB

# OpenAI
OPENAI_API_KEY="sk-..."
```

### Webクライアント (.env.local)

```env
VITE_API_BASE_URL=http://localhost:3000/api/v1
VITE_WS_URL=ws://localhost:3000
```

### 管理パネル (.env.local)

```env
VITE_API_BASE_URL=http://localhost:3000/api
```

---

## ドキュメント / Documentation

| ドキュメント | 内容 |
|-------------|------|
| [docs/architecture.md](docs/architecture.md) | システムアーキテクチャ詳細 |
| [docs/arvana-terra-backend.md](docs/arvana-terra-backend.md) | バックエンド詳細仕様 |
| [docs/arvana-terra-web.md](docs/arvana-terra-web.md) | Webクライアント詳細仕様 |
| [docs/arvana-terra-admin.md](docs/arvana-terra-admin.md) | 管理パネル詳細仕様 |
| [docs/arvana-terra-ios.md](docs/arvana-terra-ios.md) | iOSアプリ詳細仕様 |
| [docs/arvana-terra-android.md](docs/arvana-terra-android.md) | Androidアプリ詳細仕様 |
| [docs/api-spec.md](docs/api-spec.md) | API仕様書 |
| [docs/design-system.md](docs/design-system.md) | デザインシステム |
| [docs/integration-guide.md](docs/integration-guide.md) | 外部連携ガイド |

---

## ライセンス / License

Copyright © 2024 Arvana, Inc. All rights reserved.

---

## 関連プロジェクト / Related Projects

- **[Arvana Work](../Arvana-Work/)** - 雇用管理プラットフォーム（Arvana Terra と連携）
