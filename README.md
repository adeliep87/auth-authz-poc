# auth-authz-poc

認証・認可PoCの実装準備用リポジトリ。現在は **仕様・調査受入条件のDRAFTレビューと正式固定の準備**です。

## レビューする2文書

- [Expense API仕様案](docs/poc-specification-draft.md): API判定順・応答、認証境界、業務認可、D1制約、承認項目A1〜A5。
- [追加調査の受入条件案](docs/investigation-acceptance-draft.md): 完全fixture、ケースIDと具体期待値、試験層、DB不変性、ログ、設定・生成物・再現証跡。

2026-10-17の改訂は、旧draft commit `b297e727d5087d71b26a6ea1f4eca4cf226f44c4` と最新学習記録を照合した補正です。元の決定・部署の明示承認は維持し、追加具体案A1〜A5は未承認としています。2文書を実装側の統合レビュー対象とし、同内容の別仕様書を増やしていません。

## 現在の状態

| 区分 | 状態 |
|---|---|
| 仕様・受入条件の文書レビュー/修正 | DRAFT改訂済み |
| 仕様の正式固定 | 対象commitに対する本人の明示承認待ち |
| Issue #1〜#7 | 既存のdecision/tracking。本文の最新決定同期は固定後 |
| 追加調査Issue | 未起票。固定後に重複を再確認し、#1のnative sub-issueとして作成する計画 |
| コード・DDL・migration・依存導入 | 未実施 |
| ビルド・ローカルD1/HTTP/Worker・ログ実測 | 全件NOT_RUN |
| 実装後の別セッション・別モデルによる独立レビュー | 未実施 |
| PR | 未作成 |
| Workers/Hono/D1/Vitestの採用 | Issue #3で所有者が後続判断。未決 |
| API本体への移行 | 未実施・別判断 |

理解確認の正答、説明用の仮定、DRAFT commitの存在を、仕様承認や実測成功に換算しません。学習記録・正式Session進捗は `adeliep87/my-projects/research/auth-learning/` で管理します。

## 後続順序

仕様・受入条件の固定 → 追加調査Issue → 独立ブランチでローカル実測 → 別セッション・別モデルの独立レビュー → PR → Issue #3で所有者の採用判断。

正式固定では仕様案§0に従い、2文書を含むcommit完全SHA、本人の明示承認、範囲・保留・未確認を記録します。文書保存への指示は、コード実装・正式採用・公開の承認と区別します。

調査報告完了、必須試験合格、技術採用、API本体への移行は別の判断です。配置は `services/expense-api/`、npm単独・package-lock管理。対象はローカル・架空データのみ。

実IdP、実Token暗号検証、BFF、Native、承認・更新API、クラウド公開、remote D1、`refs/tmp/`変更は今回の範囲外です。
