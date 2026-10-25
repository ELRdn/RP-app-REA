# 実装リバースエンジニアリング

この章は「UIがどう作られているか」の話です。対象は**本番の公開ビルド**で、認証不要で取得できるHTML/CSS/JSと、ブラウザが実際に読み込んだリソースだけです。REAの静的解析結果と、本番CSS/DOMの実測を突き合わせています。

---

## 1. 調査対象と証拠

| | Zeta | SynClub |
|---|---|---|
| 本番HTML | `https://zeta-ai.io/ja`（410,115 bytes） | `https://www.synclubaichat.com/`（418,122 bytes） |
| JS | 109ファイル / 5,881,016 bytes | 38ファイル / 4,591,357 bytes |
| CSS | 3ファイル / 188,052 bytes | 3ファイル / 452,490 bytes |
| REA | 1,702,775 ASTノード / 119 findings / parse failure 0 | 997,030 ASTノード / 2,727 findings / parse failure 0 |
| REA Evidence | `ev_cf67c9…`（親）/ `ev_6b7956…`（summary view） | `ev_5a12c6…`（summary view） |

REAのセマンティックグラフは**結果バジェットの制約で部分的**です。以下の主張は、REAのセマンティック辺ではなく、**バンドル内の静的文字列・クラス名・DOM属性・HTTPヘッダ**で裏取りしています。この区別は重要です（`docs/08`参照）。

---

## 2. Zeta — Next.js + React + Tailwind

### 2.1 スタックの同定

| 証拠 | 結論 |
|---|---|
| `X-Powered-By: Next.js`、`Vary: rsc, next-router-state-tree` | Next.js App Router（RSC） |
| `_next/static/chunks/turbopack-*.js` | Turbopackビルド |
| `data-sentry-component` / `data-sentry-source-file` | Sentry SDKが本番Reactツリーに属性を注入 |
| `@layer properties`、`--tw-*`、`lab()` 出力 | Tailwind CSS v4 |
| `Pretendard JP Variable` | 可変フォント |
| `zustand` 8ファイル | 状態管理 |
| `swr` 1ファイル | データ取得 |
| `rive` 12ファイル / `lottie` 2ファイル | リッチアニメーション |
| `firebase` 4ファイル | Push/認証? |
| `amplitude` 2 / `gtag` 1 / `clarity` 1 / `MetaPixel` 19 | 分析 |
| `pagead2.googlesyndication.com` / `pubads_impl.js` | 広告 |

### 2.2 コンポーネントアーキテクチャ

Sentry属性から**93個のユニークなコンポーネント名**と**84個のソースファイル**が復元できました。これは本番ビルドが**ソースファイル名を保持している**（minifyしても属性は残る）ためです。

| レイヤ | コンポーネント |
|---|---|
| デザインシステム | `Button`, `Input`, `Select`, `Modal`, `Toaster`, `Link`, `WrappedImage`, `WrappedDiv`, `WrappedCarousel`, `Rule`, `Section`, `Cta`, `OpenBadge`, `TabIcon`, `CountTag`, `Hashtags`, `LinearGradient` |
| レイアウト | `LocaleLayout`, `Header`, `HeaderRight`, `NavigationBar`, `HalloweenNavigationBar`, `FeedTabNavigator`, `BottomActionArea`, `GoBackHeader`, `KeyboardAvoidingView`, `CompositionFocusBridge` |
| フィード | `Home`, `HomeContents`, `FrameProfileCard`, `FrameProfileCardCols`, `ColumnWrapper`, `PlotCoverMulti`, `HomeSpecialCurationBanner`, `CreatorSearchSection`, `GenrePlotRanking` |
| 詳細 | `PlotProfile`, `PlotDetail`, `PlotBasic`, `PlotIntro`, `PlotCharacter`, `PlotProfileCollapseButton`, `PlotCreatorTopRow`, `PlotCommentSection`, `RelatedPlotSection`, `PlotLikePill`, `ChatProfileTag`, `PlotDate`, `CreatorSearchCard` |
| チャット | `ChatBubbleContainer`, `BubbleContainer`, `NarratorBubble`, `StaticIntroMessage`, `BottomActionArea`, `CompositionFocusBridge` |
| 季節 | `HalloweenHomeDecoration`, `HalloweenStyles`, `HalloweenTabBarIcon`, `HalloweenNavigationBar`, `VeritasIntroPage` |
| 基盤 | `JsonLdRenderer`, `JsonLdSupplier`, `MetaPixel`, `AppDownloadBanner`, `Toaster`, `LogRawButton`, `ShakeStage` |

出現回数の上位は `Link.tsx`（1,455）、`WrappedImage`（378）、`CountTag`（149）、`FrameProfileCard`（143）、`Hashtags`（142）、`Button`（122）。**LinkとImageのラッパー**が全画面を支えており、計測・最適化・多言語を一箇所に集約する思想が読み取れます。

`CompositionFocusBridge` と `KeyboardAvoidingView` は、モバイルWebで入力欄とキーボードのフォーカスを制御するための部品。チャット体験の品質に直結しています。

### 2.3 ルーティングとi18n

バンドルから復元したルート：

```
/auth/{line,google,apple,kakao,brandpay,sso,get-token,authorize}
/plots/create
/plots/assistant
/creator-center
/discover
/rooms
```

i18nは**1,667個のメッセージ三つ組**（ko / ja / en）。キーは韓国語で、開発が韓国語ファースト、日本語がローカライズであることが分かります。

```js
로어북: { messages: { ko: "설정집", ja: "設定集", en: "Sourcebook" } }
인트로:  { messages: { ko: "인트로", ja: "イントロ", en: "Intro" } }
피스:    { messages: { ko: "...", ja: "ピース", en: "Pieces" } }
```

この辞書から、UIには出ていない**創作オブジェクト**（設定集、イントロ、スタイル、トークプロフィール、スナップショット、ピース、zeta pass）の存在と制約が分かります。

### 2.4 レンダリング / SEO

- プロット詳細は**SSR**（HTMLにタイトル・紹介・キャラ名が含まれる）
- `hreflang` で ja/ko/en、`Link` ヘッダでpreload
- `JsonLdSupplier` でJSON-LD
- `Cache-Control: no-store`（動的）
- 画像は `image.zeta-ai.io` のNext Image最適化（`?w=128&q=75&f=webp`）

SEOは「プロット詳細ページ」に集中投資しています。ホームやランキングはクライアント寄りで、検索エンジンには詳細ページを食わせる戦略です。

### 2.5 認証 / セッション

ゲストでも以下のCookieが発行されます。

- `TOKEN`（JWT, `ano: true`, HS384）
- `REFRESH_TOKEN`
- `TIMEZONE`
- `LANGUAGE`
- `DEVICE_ID`
- `NEXT_LOCALE`

すべて `Secure; HttpOnly; SameSite=Lax`。ゲストJWTに `ano: true` が入っており、**匿名ユーザーを一人前のアカウントとして扱う**設計です。これがWebゲストチャットを可能にしています。

### 2.6 広告 / 計測

- Google AdSense / Publisher Tag（`pubads_impl.js`, `pagead2.googlesyndication.com`）
- GA4（`analytics.google.com/g/collect`）
- Microsoft Clarity（`clarity.ms`）
- Sentry（`sentry.luda.ai`）
- Meta Pixel
- Amplitude（バンドル）

`z01_app_download_modal` の `data-clarity-mask="true"` は、個人情報をClarityの記録から隠す配慮です。

---

## 3. SynClub — Nuxt 3 + Vue + Element Plus

### 3.1 スタックの同定

| 証拠 | 結論 |
|---|---|
| `X-Powered-By: Nuxt`、`__nuxt`、`data-v-*` | Nuxt 3 / Vue 3 SFC |
| `--el-color-primary`、`el-dialog`、`el-input` | Element Plus |
| `pinia` 2ファイル | Pinia |
| `Poppins` / `Poppins-Regular.ttf` | Webフォント |
| `iconify--svg` | Iconify |
| `stripe` 2ファイル | Stripe |
| `amplitude` 3 / `gtag` 2 | 分析 |
| `WebSocket` 1ファイル | リアルタイム |
| `Server: APISIX/3.8.0` | APISIXゲートウェイ |
| `X-RateLimit-Limit: 100` / 60s | レート制限 |

### 3.2 ルーティング

`BqdEWTjq.js` のVue Router定義から：

```js
{ name: "intro-next",   path: "/intro-next" }
{ name: "landing-page", path: "/landing-page" }
{ name: "login",        path: "/login" }
{ name: "payment-error",path: "/payment-error" }
{ name: "payment-success", path: "/payment-success" }
{ name: "shared-chat",  path: "/shared-chat" }
{ name: "path-fallback-home", path: "/:slug(.*)*" }
{ name: "ocAgentEntry", path: "/", middleware: ["root-layout"] }
```

`ocAgentEntry` がルート `/` の**クリエイタースタジオ**です。`/:slug(.*)*` のフォールバックは、存在しないURLをホームに逃がすのではなく**404ページ**に落とすための仕組みです。

### 3.3 BEM + Element Plus

本番CSSには**31のBEMブロック**と、Element Plusの355KBのCSSが含まれます。主なブロック：

| 領域 | ブロック |
|---|---|
| 発見 | `conversation-list`, `contacts-list`, `content-list`, `card-item`, `introduce-card`, `instroduce-card` |
| ナビ | `app-container`, `mobile-top-bar`, `tab-item`, `label-title__tabs-item`, `icon-item` |
| 課金 | `subscription-page`, `subscription-plan(s)`, `coin-package(s)`, `card-info-dialog`, `order-confirm-dialog`, `payment-method-dialog`, `payment-rules-dialog`, `payment-error-dialog`, `unpaid-order-dialog`, `cancel-subscription-dialog`, `consumption-details-dialog` |
| 認証 | `login-wrap`, `user-dialog-content`, `verification-code`, `region-wrap`, `is-agree`, `sign-in` |
| システム | `message-box`, `feedback-dialog`, `age-confirm`, `app-benefits-tooltip`, `error-page`, `scan-code` |

**決済系だけで10種類以上**のダイアログがあります。Webの主目的が「創作ツールの購読/コイン販売」であることがCSSから読み取れます。

### 3.4 remスケーリング

```css
html { font-size: 3.8647vw; }  /* 390px → 15.0725px */
```

すべてのサイズが `rem`。これにより、どの端末でも**比率が一定**になります。ただし、デスクトップでは文字が大きくなり、固定幅のクリエイター画面と衝突します（`synclub_s26_image_creator_entry.png`）。

### 3.5 SSR / SEO

- Nuxt SSR（HTMLにカテゴリ・キャラ名・説明が含まれる）
- sitemap index **209件**、`/discover/{type}/{id}` を大量に生成
- JSON-LD：`Organization`（HiClub株式会社）、`Brand`、`SoftwareApplication`、`WebSite`
- `X-Robots-Tag: index, follow, max-image-preview:large`
- 共有チャットは `wap.synclubaichat.com` の**静的HTML**（`sc-chat-sharing-v2.html`）で、アプリ外でも軽量に表示

SEOは**発見投稿のロングテール**に振っています。キャラチャット本体はアプリなので、Webは「検索から来て、アプリへ送る」漏斗です。

### 3.6 ネットワーク / API

観測した公開エンドポイント：

| アプリ | エンドポイント | 用途 |
|---|---|---|
| Zeta | `GET https://api.zeta-ai.io/v1/open-stores/ZETA_STORE_GLOBAL/products` | ストア商品（ピース/pass） |
| SynClub | `POST https://api.synclubaichat.com/aichat/h5/resourceManage/getResource` | フィード/リソース取得 |
| SynClub | `POST https://zeta-568626cc1fcb4a.pingpong.us/` | 計測 |
| 共通 | `analytics.google.com/g/collect` | GA4 |

ZetaのAPIホストが `api.zeta-ai.io`、SynClubが `api.synclubaichat.com`。どちらも**公開フロントから直接叩ける**構成で、BFF/GraphQLは確認できませんでした（Zetaのバンドルに `graphql` の語はあるが、実際の通信はREST）。

---

## 4. パフォーマンス

| 指標 | Zeta | SynClub |
|---|---|---|
| JSファイル数 | 109 | 38 |
| JS合計（非圧縮） | 5.88 MB | 4.59 MB |
| CSS合計 | 188 KB | 452 KB |
| HTML | 410 KB | 418–452 KB |
| フォント | Pretendard JP Variable（woff2, preload） | Poppins 3ウェイト（ttf） |
| 画像 | Next Image（webp, 幅指定） | CDN（png/jpg/mp4） |
| SSR | RSC + 一部クライアント | Nuxt SSR |
| キャッシュ | `no-store` | ヘッダなし（APISIX） |

**Zetaの課題**：HTML 410KB + JS 5.88MB。Turbopackのチャンクが細かく分割され、109本のscriptリクエストになります。HTTP/2/3なら多重化できますが、モバイル回線では立ち上がりが重い。`Link` と `Image` のラッパーでprefetch/最適化しているのが救いです。

**SynClubの課題**：Element PlusのCSS 355KBが常に付いてきます。実際に使っているのは `el-dialog` `el-input` `el-checkbox` など一部なのに、フルセットを読み込んでいます。Poppinsは**ttf**で、woff2に比べて2–3倍重い。`font-display` の指定も確認できませんでした。

**共通**：どちらもSSRで初期HTMLにコンテンツを載せ、LCPは画像。Zetaは `image.zeta-ai.io` のNext Imageで最適化、SynClubは `newcdn.synclubaichat.com` の生PNG/JPG/MP4。SynClubのフィードは動画サムネイルが重く、スクロール時の読み込みが課題です。

---

## 5. セキュリティ / プライバシー

| 項目 | Zeta | SynClub |
|---|---|---|
| HSTS | `max-age=63072000` ✅ | なし ❌ |
| CSP | なし ❌ | なし ❌ |
| X-Frame-Options | なし ❌ | なし ❌ |
| Cookie | `Secure; HttpOnly; SameSite=Lax` ✅ | 不明 |
| レート制限 | 不明 | 100 req/60s ✅ |
| `X-Powered-By` | `Next.js` ⚠️ | `Nuxt` ⚠️ |
| サードパーティ | GA, Clarity, Sentry, Meta, Ads | GA, GTM, Amplitude, pingpong |

どちらも**CSP未設定**で、広告・分析・Sentryなど多数のサードパーティスクリプトを許可しています。XSSが入った場合の影響範囲が広い。ZetaはHSTSとCookie属性が適切、SynClubはAPISIXのレート制限が強い一方、HSTSがありません。

`X-Powered-By` はフレームワークを開示するだけで実害は小さいですが、攻撃者の偵察を助けます。

---

## 6. アクセシビリティ実装

| 項目 | Zeta | SynClub |
|---|---|---|
| ランドマーク | `<header>`, `<nav>`, `<main>` | `div` 中心 |
| 見出し | `h1`–`h3` あり | `h1` なしのページあり |
| ボタン | `<button>` + `aria-label`（英語） | `div` + クリック |
| リンク | `<a href>` ✅ | `<section>` + JS ❌ |
| フォーカス | 紫リング | なし |
| 画像alt | あり（`alt="zeta"` 等） | あり |
| 言語 | `<html lang="ja">` | `<html lang="ja">` |
| モーション | `prefers-reduced-motion` 対応は不明 | 不明 |

Zetaは**セマンティクスは概ね正しい**が、ラベルが英語。SynClubは**見た目優先**で、キーボード操作とURL共有に弱い。どちらも「AIアプリの成長期」らしい実装です。

---

## 7. MiraiMind / CrushOn の実装（追記）

### 7.1 MiraiMind — Vue 3 + Ant Design Vue

| 証拠 | 結論 |
|---|---|
| `data-v-app`, `__vitePreload` | Vue 3 + Vite |
| `ant-layout` `ant-btn` `ant-input` `ant-image` | Ant Design Vue |
| `pinia` | Pinia |
| `socket.io` / `wss://cosmos-im-ws-hw-ap.hellogroup.com` | socket.ioによるIM |
| `RTCPeerConnection` / TTS | 音声通話・読み上げ |
| `rive` | Riveアニメーション |
| `appsflyer` / `gtag` / `adsbygoogle` | 広告・計測 |
| Astro 5.17.1（公式サイト） | マーケティングは静的Astro |

- JS 25ファイル / 4.39MB、CSS 281KB
- REA：22ファイル / 4.47MB / 1,203,611 ASTノード / 518 findings
- 匿名セッションを自動発行し、**5リクエスト**までチャット可能（ランタイムログ `requestLimit: 5`）
- デスクトップWebは**空**。モバイル専用

### 7.2 CrushOn.AI — Next.js + Mantine

| 証拠 | 結論 |
|---|---|
| `Minified React error` / `_next` | Next.js + React |
| `mantine-*` / `rc-dialog-*` | Mantine + rc-dialog |
| utility classes / `--tw-` | Tailwind |
| `*-store` localStorage | Zustand |
| `stripe` | Stripe決済 |
| `sentry` / `amplitude` / `sensorsdata` | 監視・分析 |
| `/pricing` `/store` `/account` `/coin` | 課金・アカウント |

- JS 106ファイル / 7.65MB、CSS 636KB、**2,100モジュール**
- REA：106ファイル / 7.67MB / 1,954,108 ASTノード / **10,221 findings**
- 7段階のサブスク、Coin/Diamonds、18+ゲート
- マーケティング（`chat.crushon.ai`）とアプリ（`crushon.ai`）で**別ビルド**

### 7.3 4製品の実装比較

| | Zeta | SynClub | MiraiMind | CrushOn |
|---|---|---|---|---|
| フレームワーク | Next.js | Nuxt 3 | Vue 3 + Vite | Next.js |
| UI | Tailwind v4 | Element Plus | Ant Design Vue | Mantine + Tailwind |
| 状態 | Zustand / SWR | Pinia | Pinia | Zustand / SWR |
| リアルタイム | REST中心 | WebSocket | socket.io | REST中心 |
| 計測 | Sentry/GA/Clarity/Meta/Amplitude | GA/GTM/Amplitude | GTM/GA/AppsFlyer/TikTok/Meta | Sentry/Amplitude/GTM/Sensors |
| JS（非圧縮） | 5.88MB / 109 | 4.59MB / 38 | 4.39MB / 25 | 7.65MB / 106 |
| CSS | 188KB | 452KB | 281KB | 636KB |

**観察**：4社とも「チャットUI」なのに、実装の重心が違います。ZetaはSEOとSSR、SynClubは管理画面（Element Plus）と決済、MiraiMindはリアルタイムIM、CrushOnはSEOページ量産と課金。UIの見た目以上に、**バックエンドの都合がフロントに漏れている**のが面白いところです。

---

## 8. 不明な点

- ZetaのReact/Next.jsの正確なバージョン（バンドルからは特定できず）
- SynClubのWebチャットAPIの詳細（未ログインで叩けない）
- 両社のABテスト基盤（Amplitude/GA以外）
- Zetaの`Firebase`の用途（PushかRemote Configか）
- SynClubの`WebSocket`の用途（チャットか通知か）
- 動画/CDNの配信基盤（CloudFrontか自前か）
- 課金の実際の価格（ログイン後のみ）

これらは「未知」として残します。推測で埋めません。

