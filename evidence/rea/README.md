# REA Evidence

REA（Reverse Engineering Anything）で実行した分析の記録。入力は**本番配信されているJavaScriptバンドル**で、実行はしていません（静的解析）。

## 実行一覧

| Evidence ID | 操作 | 入力 | SHA-256 | 結果 |
|---|---|---|---|---|
| `ev_cf67c92998dbb0780870791e3654ad8c62e8ed43ee268915c685a58cb9cee939` | `analyze_javascript_application` | `js/zeta`（109ファイル / 5,881,016 bytes） | `26bdb2d78b0944d433c7e7ff57966f480cefcb67ebb9dd39d0d8bb3713bff259` | 1,702,775 ASTノード / 119 findings / 0 parse failures / semantic partial |
| `ev_6b7956f45903b4abf84a84525af3fc9fe469172230dac34f5e2678802f452a64` | `inspect_analysis_view`（summary） | 上記の親Evidence | — | 統計・カバレッジ・制約のサマリー |
| `ev_5a12c697c01da527370c8c07eb28982ac7ef73f183e51adaeca5688593157ffd` | `analyze_javascript_application` | `js/synclub`（38ファイル / 4,591,357 bytes） | — | 997,030 ASTノード / 2,727 findings / 0 parse failures / semantic partial |

## 実行条件

- `format`: `directory`
- `detail`: `summary`
- `max_heap_mb`: 2048
- `analysis_timeout_ms`: 600000
- `integrity_policy`: デフォルト（`fail`）

## カバレッジと制約

Zetaのセマンティック解析は、結果バジェット（`javascript_analysis_parent_result_budget_bytes`）の制約により、多くのチャンクで**静的ファクトのみ保持**となりました。SynClubも一部のチャンクで同様です。

したがって本レポートでは、REAのセマンティックグラフ（呼び出し関係・データフロー）を**直接の根拠にしていません**。UIの主張は次の証拠で裏取りしています。

1. 本番CSSのユーティリティ定義（色・角丸・タイポ）
2. 実画面の `getComputedStyle` 実測
3. DOMの `data-sentry-component` / `data-v-*` / BEMクラス
4. バンドル内の静的文字列（i18n、ルート、課金文言）

## 証拠バンドルについて

`export_evidence_bundle` を実行したところ **192,851,393 bytes**（16 records）になり、GitHubの1ファイル100MB制限を超えるため、リポジトリには含めていません。必要な場合は、同じ入力ディレクトリ（ハッシュで固定）に対して再実行すれば再生成できます。

## 入力の再取得

`js/zeta` と `js/synclub` は、公開ページが実際に読み込んだ `.js` をPlaywrightのレスポンスイベントから保存したものです。ページのビルドが変わるとハッシュも変わります。
