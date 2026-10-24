# RP-app-REA — Zeta / SynClub UI リバースエンジニアリング

日本向けAIロールプレイアプリ **Zeta（zeta）** と **SynClub（シンクラブ）** のUIを、公開Webビルド・App Store素材・実機キャプチャから分解した記録です。画面のスクリーンショット93点、デザイントークン、ルート/コンポーネント目録、実装スタックの分析、再構築ブループリントを収録しています。

> 解析対象は未認証で到達できる公開面のみ。ログイン回避・認証情報の窃取・非公開APIの叩き込みは行っていません。UGC本文は公開用に一部レダクト/選定しています。

![ギャラリー](docs/assets/gallery-preview.png)

---

## 30秒サマリー

| | **Zeta** | **SynClub** |
|---|---|---|
| ポジション | 「日本のNo.1 AIチャット」。推しとの会話・夢小説・なりきり | 「やさしく寄り添う AIキャラチャット・ノベル」。会話＋創作基盤 |
| コンテンツ単位 | **プロット（物語）**。1プロットに最大10キャラ、設定集、イントロ、スタイル | **AIキャラクター（エージェント）**。単体キャラ＋タグ＋説明 |
| Web戦略 | アプリと同じUIを**440px固定シェル**で再現。ゲストチャット可 | **「ブラウザ版」**として別プロダクト化。発見＋クリエイタースタジオ |
| ビジュアル | 常時ダーク `#151516` × 紫 `#6728FF`。没入・夜・推し活 | ライト `#FFFFFF` × コーラル `#FF7870`。編集・雑誌・手描き |
| ナビゲーション | 上部セグメント＋**下部タブバー**（5タブ） | モバイルは**巨大カテゴリグリッド**、デスクトップは左レール＋サイドバー |
| チャット | フラットな吹き出し。ナレーション＋キャラ＋ユーザー＋`*`アクション | 背景イラスト＋色付き吹き出し＋音声メッセージ。ノベル/劇場型 |
| 課金 | **ピース**従量＋**zeta pass**＋広告除去 | **コイン**従量＋**サブスク**＋アプリ内課金（18+） |
| スタック | Next.js (App Router, Turbopack) / React / Tailwind v4 / Zustand / Sentry / Vercel | Nuxt 3 / Vue 3 / Element Plus / Pinia / APISIX / Stripe |

**一番おもしろい差分**：Zetaは「物語の世界に入る」ためのUI、SynClubは「世界とキャラを編集する」ためのUIに振り切っています。同じAIキャラChatでも、情報設計の起点が *プロット* か *キャラクター* かで、ホーム・詳細・チャット・課金のすべてが変わります。

---

## 収録物

```
RP-app-REA/
├── index.html                 # 93画面のギャラリー（フィルタ/検索/ライトボックス）
├── data/
│   ├── screens.json           # 各画面のメタデータ + SHA-256
│   ├── tokens.json            # キュレーション済みデザイントークン
│   ├── routes.json            # 観測/ゲート/バンドルのルート目録
│   ├── components.json        # React/Vueのコンポーネント目録
│   ├── features.json          # 27項目の機能比較マトリクス
│   └── app-metadata.json      # ストア/HTTP/バンドルのメタデータ
├── docs/
│   ├── 01-zeta-teardown.md
│   ├── 02-synclub-teardown.md
│   ├── 03-comparison.md
│   ├── 04-design-system.md
│   ├── 05-implementation.md
│   ├── 06-rebuild-blueprint.md
│   ├── 07-evidence-index.md
│   └── 08-method-and-limitations.md
├── evidence/
│   ├── screenshots/{zeta,synclub}/{web,mobile}, appstore/{zeta,synclub}
│   ├── dom/structures/*.json  # テキストを除いたDOM構造アウトライン
│   ├── tokens/*-observed.json # 実測した色/フォント/角丸の出現頻度
│   └── rea/README.md          # REA Evidence IDと実行条件
└── analysis/design-tokens.css # 両社のトークンをCSS変数化
```

ギャラリーは `index.html` をブラウザで開くか、GitHub Pages でそのまま閲覧できます。

---

## 主要な発見

### 1. ZetaのWebは「アプリの移植」、SynClubのWebは「別プロダクト」

Zetaは1280pxのデスクトップでも中央440pxのモバイルシェルを維持します（`desktop_zeta_dark_feed.png`）。上下のナビゲーション、カード幅、フォントサイズまでアプリと同一で、Webはアプリの代替チャネルです。

SynClubはモバイル（0–500px）とデスクトップ（769px+）でレイアウトが別物になります。デスクトップでは左アイコンレール＋サイドバー＋4カラムグリッド（`desktop_synclub_light_feed.png`）。さらに `/` はキャラチャットではなく、動画・漫画・画像・音声を作る**クリエイタースタジオ**です（`synclub_s35_desktop_creator_home.png`）。

### 2. Zetaは「プロット」、SynClubは「キャラクター」が原子

Zetaの詳細画面は、ヒーロー→紹介→プロット（キャラ定義）→イントロ→コメント→関連、という**物語の台本**の順序です。作者は `【描写方針】` `【絶対原則】` のようなプロンプトをそのまま公開しています（`zeta_z40_profile_scroll_3600.png`）。UIは「読む→入る」の導線です。

SynClubのフィードはキャラクターカードの集合で、カテゴリは「学園・青春」「ヤンデレ」「背徳感」など**感情・関係性のタグ**。チャットは背景イラストと音声を含む**劇場**として設計されています（`synclub_s40_shared_chat_redacted.png`）。UIは「探す→演じる」の導線です。

### 3. 課金は「時間」と「創作」に課金する

Zetaは**ピース**で「日記を読む」「キャラのスマホを覗く」「メッセンジャーで話す」「スナップショットを生成する」「有料モデルを使う」を都度課金します。加えて**zeta pass**（7日/サブスク）と広告除去。物語の**寄り道**が課金ポイントです。

SynClubは**コイン**で「1コインで続行」「サブスクに登録」を選ばせ、画像生成・ボイス・18+コンテンツを従量/購読で販売します。WebのCSSには `subscription-plan` `coin-package` `card-info-dialog` `unpaid-order-dialog` など**決済ダイアログだけでも10種類以上**が含まれています（`data/components.json`）。創作の**回数**が課金ポイントです。

### 4. 季節テーマとイベントをコンポーネントで差し替える

Zetaは `HalloweenNavigationBar.tsx` `HalloweenTabBarIcon.tsx` `HalloweenHomeDecoration.tsx` `HalloweenStyles` `VeritasIntroPage.tsx` という専用コンポーネントを持ち、ハロウィンでは下部ナビのアイコン・背景装飾・タブ色を丸ごと差し替えます（`zeta_z02_home.png` のカボチャ、`zeta_z23_creator_center.png` のマスコット）。イベントはブランドコラボ（PixAI、Veritas）まで含めて**UIのスキン**として実装されています。

### 5. アクセシビリティは両社とも「これから」

Zetaのアイコンボタンは日本語UIなのに `aria-label="Go back"` `"Send"` `"Insert asterisk"` と英語です。SynClubのカテゴリタブは `<div class="tab-item">` で、キーボードフォーカスもリンクセマンティクスもありません。カードは `<section>` + クリックハンドラで、URLを持たないため新しいタブで開けません。どちらも**見た目優先でセマンティクスが後回し**になっています。

---

## ギャラリーの見方

`index.html` を開くと、アプリ（Zeta/SynClub）とサーフェス（Web/iOS/App Store）でフィルタできます。カードをクリックすると原寸、URL、SHA-256、寸法が出ます。

- **Zeta Web**：ホーム、ランキング、検索、プロット詳細の全セクション、ゲストチャット、ログイン、クリエイターセンター、AIプロットアシスタント、イベント、404
- **SynClub Web**：ランディング、キャラフィード、カテゴリ、発見（漫画/ゲーム/日常/モーション）、クリエイタースタジオ、画像/ボイス、共有チャット、404、デスクトップ/モバイル
- **Zeta iOS**：実機キャプチャ（発見、設定、漫画、プロット詳細、イントロ、コメント、ペルソナ選択）
- **App Store**：両社のストア素材（Zeta 10点 / SynClub 9点）

---

## 方法と証拠

- **REA（Reverse Engineering Anything）** で生産JSバンドルを静的解析
  - Zeta: 109ファイル / 5.88MB / 1,702,775 ASTノード / 119 findings / parse failure 0
  - SynClub: 38ファイル / 4.59MB / 997,030 ASTノード / 2,727 findings / parse failure 0
  - Evidence IDは [`evidence/rea/README.md`](evidence/rea/README.md) に記録
- **実画面キャプチャ**：Playwright + システムChrome、viewport 390×844 / DPR2 / ja-JP / Asia-Tokyo。2026-10-24〜25に取得
- **デザイントークン**：本番CSSのユーティリティ定義と、実際の computed style を突き合わせて二重検証
- **モバイル**：App Store lookup API（2026-10-25時点の最新版）＋ユーザー提供のiOS実機キャプチャ
- **インフラ**：HTTPレスポンスヘッダ、PWAマニフェスト、robots.txt、sitemap

詳細は [`docs/08-method-and-limitations.md`](docs/08-method-and-limitations.md)。

### REA分析の限界

REAのセマンティック解析は結果バジェットの制約で**部分的**です（多くのチャンクで「静的ファクトは保持、セマンティックグラフは未完了」）。本レポートのコンポーネント名・トークン・ルートの主張は、REAのセマンティックグラフではなく、**本番CSS/DOM/バンドルの静的文字列**と**実画面**で裏取りしています。この区別は各章で明記します。

---

## 注意

- 本リポジトリは**UI構造の分析**を目的とし、各社のロゴ・キャラクター画像・文章の権利は各社に帰属します。
- キャラクター説明やチャット本文などのUGCは、公開用に**一部を選定・モザイク処理**しています。Zetaの実機キャプチャは提供者の同意のもとで収録しました。
- 認証回避は行っていません。`/subscription` や `/chat` など未ログインでリダイレクトされる画面は、**バンドル/CSS内の存在表明**にとどめ、推測と明記しています。
- 2026-10-25時点のUIです。両社とも更新頻度が高いため、現在の画面とは異なる可能性があります。

---

## ライセンス

分析テキスト・構造化データは CC BY 4.0。スクリーンショットは各権利者に帰属し、本リポジトリのライセンス対象外です。

---

*Built with REA (Reverse Engineering Anything) — static evidence, runtime observation, and explicit unknowns.*



