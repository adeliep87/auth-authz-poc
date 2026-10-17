# auth-authz-poc

認証・認可の段階的PoC。現在は **DRAFT・仕様固定未承認** です。

- [PoC統合仕様案](docs/poc-specification-draft.md): GET Expense API、認証境界、認可表、D1制約、HTTP契約。
- [追加調査の受入条件案](docs/investigation-acceptance-draft.md): fixture、期待値、local D1、通常Workerとの比較、ログ、証跡。

元の決定と承認済み部署ルールは保持し、レビュー補足と新しい具体化案は未承認として区別します。Issue #1〜#7は別途承認状態を同期するまで従来のままです。コード・migration・試験・調査Issue・PRは未作成で、全受入ケースはNOT_RUNです。

順序: 仕様・受入条件の固定→追加調査Issue→独立ブランチのローカル実測→別セッション・別モデルの独立レビュー→PR→Issue #3で所有者が技術採用を判断。実IdP・実Token暗号検証・BFF・Native・承認/更新API・クラウド公開・remote D1・refs/tmp/変更は今回の対象外です。
