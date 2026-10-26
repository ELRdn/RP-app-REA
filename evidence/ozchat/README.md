# OzChat — evidence pack

対象：**オズチャット -Oz Chat-**（株式会社Trippy）
観測日：2026-10-26
公開面：`https://0z.chat/`（マーケティング）／`https://app.0z.chat/`（Webアプリ）／App Store

## 何を撮ったか

| ディレクトリ | 内容 | 枚数 |
|---|---|---|
| `screenshots/ozchat/app/` | 未認証で到達できるWebアプリ画面（ログイン、メール認証、FAQ、規約、サポート、共有カード、イベント、エラー） | 26 |
| `screenshots/ozchat/marketing/` | 公式サイト、Wiki、マガジン、診断、プレスキット、404（desktop + mobile） | 47 |
| `screenshots/ozchat/characters/` | 公式キャラクター個別ページ | 23 |
| `screenshots/ozchat/appstore/` | App Store公式スクリーンショット | 8 |
| `tokens/` | 実画面の `getComputedStyle` 集計（色・フォント・サイズ・角丸） | 5 |

合計 **104枚**（`data/screens.json` にSHA-256付きで登録）。

## 認証境界

Webアプリは**ログイン必須**です。`/`、`/room`、`/notifications`、`/profile/edit`、`/app/*` などはすべて
`/auth/signin?redirect_to=...` へリダイレクトされます。ゲスト用のデモ、チャットプレビュー、無料枠はありません。

認証後のホーム／チャット／ストア／設定は**未認証では到達できない**ため、実画面キャプチャはありません。
この範囲は次の公開資料で補完しています。

- App Storeの公式スクリーンショット8点（ホーム、カード、チャット、親愛度、ストーリー）
- 公式Wiki（チャット、メモリー、親愛度、レベル、シチュエーションカード）
- 公式マガジン（AIチャットの仕組み、記憶、安全性）
- 公開バンドル内のUI文字列とCSSトークン

**認証回避は行っていません。**

## 撮影条件

- Playwright + システムChrome（headful）
- Desktop: 1440×900 / DPR1 / ja-JP / Asia-Tokyo
- Mobile: 390×844 / DPR2 / iPhone UA / hasTouch
- 小型端末の確認のみ 320×568
- アニメーションは撮影用に `animation-duration: .001s` へ固定（レイアウトは変えない）
- 遅延読み込み画像は `loading=eager` に変更してから撮影

## コンテンツの扱い

- 共有カードのキャラクター画像は**ブラー**（`oz_a14` / `oz_a15` / `oz_a15b`）
- ハロウィンイベントは画像を**ブラー**（`oz_a18`）。カード一覧の全長ショットは公開していません
- ビキニコレクション（`/events/bikini-collection-2026`）は**撮影・収録していません**
- App Store素材は公式の一般公開素材で、成人向けの露出はありません
- UGC本文・チャット履歴・ユーザー名は収録していません

## 生データ

- `ux-raw.json` — マーケティング→ログインのハンドオフ、タップ領域、文字サイズ、LCP/CLS、通信量
- `routes.txt` — Next.js build manifestの55ルート
- `sitemap-urls.txt` — 公式サイトの889 URL（7言語）
- `signin-showcase.json` — ログイン画面の公開カードAPI（12件）
- `itunes.json` — App Store lookup（バージョン、評価、17+、スクリーンショットURL）
- `js-bundle-index.json` — 解析した132アセットのURLとサイズ
- `tokens/*.json` — 5画面のデザイントークン集計

## 制約

- 認証後のUIとチャット応答は未観測
- 購入・サブスク・MP消費は未実行
- 共有カードのCTAはログインへ遷移するところまで確認
- イベントページのカード画像は内容を確認せずブラーしている
- 年齢確認はWebのログイン画面には見当たらない（App Storeは17+）
