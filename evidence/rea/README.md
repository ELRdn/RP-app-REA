# REA Evidence

REA（Reverse Engineering Anything）で実行した分析の記録。入力は**本番配信されているJavaScriptバンドル**で、実行はしていません（静的解析）。

## 実行一覧

| Evidence ID | 操作 | 入力 | SHA-256 | 結果 |
|---|---|---|---|---|
| `ev_cf67c92998dbb0780870791e3654ad8c62e8ed43ee268915c685a58cb9cee939` | `analyze_javascript_application` | `js/zeta`（109ファイル / 5,881,016 bytes） | `26bdb2d78b0944d433c7e7ff57966f480cefcb67ebb9dd39d0d8bb3713bff259` | 1,702,775 ASTノード / 119 findings / 0 parse failures / semantic partial |
| `ev_6b7956f45903b4abf84a84525af3fc9fe469172230dac34f5e2678802f452a64` | `inspect_analysis_view`（summary） | 上記の親Evidence | — | 統計・カバレッジ・制約のサマリー |
| `ev_5a12c697c01da527370c8c07eb28982ac7ef73f183e51adaeca5688593157ffd` | `analyze_javascript_application` | `js/synclub`（38ファイル / 4,591,357 bytes） | — | 997,030 ASTノード / 2,727 findings / 0 parse failures / semantic partial |
| `ev_5d2ca88b51398d6641a60679adc90729425eb62489fb9287189ac5d7eed9a24f` | `analyze_javascript_application` | `js2/miraimind`（22ファイル / 4,467,831 bytes） | — | 1,203,611 ASTノード / 518 findings / 0 parse failures |
| `ev_fb8d749c4ec5e8b7736082ff6e7e9dff219729342fcf8e8757f6a90f44328333` | `inspect_analysis_view`（summary） | 上記の親Evidence | — | 統計・カバレッジ |
| `ev_2dd124526919fcdaf3ab78dc02faa9d553d00b64a29249af680bdd10eaa6f6cd` | `analyze_javascript_application` | `js2/crushon`（106ファイル / 7,674,553 bytes） | — | 1,954,108 ASTノード / 10,221 findings / 2,100 modules / 0 parse failures |
| `ev_9361af851e189c0dc70dcacc6f89ad068569d9edf4e275aef681454613552e7b` | `inspect_analysis_view`（summary） | 上記の親Evidence | — | 統計・カバレッジ |
| `ev_234da7ab8f3eeeed4e04d5d4c96cedd2bda59efa0dfdd084079ee37b5ac708f1` | `analyze_javascript_application` | `oz/js-bundle`（128 JS / 8,892,541 bytes） | `a4e5b22f05d8806779fe6ce9e8f0f51a70bf371584ef2d14c7f821a645478806` | 2,252,274 ASTノード / 9,640 findings / 2,220 modules / 0 parse failures / semantic partial（result budget 0） |
| `ev_df0dabf07f461a2d9a04dbdf22316aa18eab5249ecc57acd4421014a4021a390` | `inspect_analysis_view`（summary） | 上記の親Evidence | — | 128ファイル / 128 parsed / 132 assets / 9.1MB |

## 実行条件

- `format`: `directory`
- `detail`: `summary`
- `max_heap_mb`: 2048
- `analysis_timeout_ms`: 600000
- `integrity_policy`: デフォルト（`fail`）

## カバレッジと制約

Zetaのセマンティック解析は、結果バジェット（`javascript_analysis_parent_result_budget_bytes`）の制約により、多くのチャンクで**静的ファクトのみ保持**となりました。SynClubも一部のチャンクで同様です。OzChatはさらに制約が強く、セマンティック結果バジェットが **0 bytes** だったため、アプリケーショングラフ（2,220 modules / 9,640 findings）は保持されていますが、セマンティック関係グラフは**部分・省略**です。

したがって本レポートでは、REAのセマンティックグラフ（呼び出し関係・データフロー）を**直接の根拠にしていません**。UIの主張は次の証拠で裏取りしています。

1. 本番CSSのユーティリティ定義（色・角丸・タイポ）
2. 実画面の `getComputedStyle` 実測
3. DOMの `data-sentry-component` / `data-v-*` / BEMクラス
4. バンドル内の静的文字列（i18n、ルート、課金文言）

## 証拠バンドルについて

`export_evidence_bundle` を実行したところ **192,851,393 bytes**（16 records）になり、GitHubの1ファイル100MB制限を超えるため、リポジトリには含めていません。必要な場合は、同じ入力ディレクトリ（ハッシュで固定）に対して再実行すれば再生成できます。

## 入力の再取得

`js/zeta` と `js/synclub` は、公開ページが実際に読み込んだ `.js` をPlaywrightのレスポンスイベントから保存したものです。ページのビルドが変わるとハッシュも変わります。
