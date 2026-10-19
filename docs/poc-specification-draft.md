# Expense API PoC 仕様案（A1〜A5承認済み・文書一式の正式固定は未実施）

改訂: 2026-10-19（JST）。最終照合・表記同期の基点はA5承認記録済みhead `b046dffc79430d096f61a13902097af3454e4a6e`。初回レビューの基点 `b297e727d5087d71b26a6ea1f4eca4cf226f44c4` と2026-10-18補足の基点 `7ee8df448b9d8044cfe4716e5871b9fc14b29209` は履歴として保持する。本書と[調査受入条件案](investigation-acceptance-draft.md)を一組の正式固定候補とする。コード・migration・依存導入・実測は未実施、受入試験はすべて **NOT_RUN**。

## 0. 根拠・承認状態・固定方法

元のレビュー根拠（当時）は `adeliep87/my-projects` の `5ef2793fb1d46c37aa18f4a1d50303739551ce46` にある、[レビュー時の引き継ぎ](https://github.com/adeliep87/my-projects/blob/5ef2793fb1d46c37aa18f4a1d50303739551ce46/research/auth-learning/poc-preparation-next-session.md)、[元の準備記録](https://github.com/adeliep87/my-projects/blob/5ef2793fb1d46c37aa18f4a1d50303739551ce46/research/auth-learning/poc-preparation-2026-10-17.md) §2〜§4・§7、[境界確認記録](https://github.com/adeliep87/my-projects/blob/5ef2793fb1d46c37aa18f4a1d50303739551ce46/research/auth-learning/poc-preparation-boundary-check-2026-10-17.md)。当時の本人指示はこれらを維持し、文書レビューと補正を行うもの。現在のA1〜A5承認原文・対象・保存状態は、A5まで保存した学習側commit `d7d0ab8de5fae265c71ee8790d519a407d65b223` の[引き継ぎ§4.3〜§4.7](https://github.com/adeliep87/my-projects/blob/d7d0ab8de5fae265c71ee8790d519a407d65b223/research/auth-learning/poc-preparation-next-session.md)と本書の最新台帳を優先する。2026-10-19の指示は最終照合・現在表記の整理・固定対象の明確化であり、個別承認の再取得ではない。

| 区分 | 内容 | この改訂での扱い |
|---|---|---|
| 元の決定 | GET契約、判定順、非GETの早期405、HEAD本文なし、scope、返却5項目、業務認可、D1の3保証、ローカル範囲 | 維持。再選択しない |
| 本人の追加明示承認 | MANAGER現在部署と申請保存部署の比較、所有者の異動では既存申請部署を変更しない | 維持。自己申請に同部署条件を追加しない |
| 以前のレビュー補足 | 一般例外500、Allow、requestIdの配置と対応、通常条件の認証後経路 | 仕様案へ統合。理解確認の正答を正式承認にしない |
| 今回の具体化案 | route境界・未デコードID、共通finalizer、各code/ヘッダー、破損行の500、usersの具体的キー、fixture・受入詳細 | 下記台帳のとおりA1〜A5すべて承認済み |
| 調査時の実装選択 | 認証アダプターの引数・戻り値の外側表現（§4.1の承認済みA2-5の委任範囲）、DDLの記法、テスト専用entrypoint/auxiliary Worker、初期化・artifact検査・fault注入の方式、依存版 | 固定した要件の下で調査。未測定の方式や版を採用済みとしない |

### 個別承認の最新台帳（文書一式の正式固定とは区別）

| ID | 承認対象 | 参照 | 最新状態 |
|---|---|---|---|
| A1 | 未知routeと保護routeの境界、未デコードID、末尾slash・queryの扱い | §1 | 承認済み（§1.3） |
| A2 | エラーコードと応答JSON/headers、全route共通finalizer、主体/行破損/例外の分類、認証結果の必須型と表現の委任範囲 | §2〜§3・§4.1 | 承認済み（§4.1.1） |
| A3 | users全体PK・複合UNIQUE、複合FKのRESTRICT、保存型を含む具体的DB制約 | §5 | 承認済み（§5.3） |
| A4 | 完全なfixture、認可表との対応、独立した具体的HTTP/JSON/DB期待値 | 受入条件案§2〜§6 | 承認済み（本書§0.1） |
| A5 | L1〜L4の必須範囲、設定・artifact照合、障害注入、DB不変性、ログと再現性の合格条件 | 受入条件案§1・§6.3・§7〜§10 | 承認済み（§0.2） |

**A1〜A5は本人承認済み（2026-10-18 JST）。** A1〜A3は内容commit `18650f135f103ccfdf4574365dc25aed314f82f1` の各範囲に対する承認を§1.3・§4.1.1・§5.3に保存済み。A4は内容commit `89e94baeca3cfe65a8fa3c2bc4192cd80d89da2e` の受入条件案の入力・期待値への承認（§0.1）、A5は内容commit `e7488674bf5cf2a0775c8fad92cb5a66a890d2c4` の検証要件・合格条件への承認（§0.2）である。A1〜A5の個別承認は仕様／受入条件全体の正式固定、技術採用、API本体移行、実装・試験開始を意味しない。

正式固定時は、この2文書を含む**対象commitの完全SHA**、本人の明示承認原文または所在、承認日時（JST）、承認したA番号・節、除外/保留、残る未確認事項を記録する。承認は対象commitの内容に結び付け、承認記録を追記した次commitから元commitを参照する。未承認欄を自動で埋めない。承認後の意味変更は差分と影響ケースを示して再レビューする。

現在: **A1〜A5承認済み／仕様・受入条件全体の正式固定は未実施／試験NOT_RUN／技術採用未決**。過去の承認記録内の「A1のみ」「A1・A2のみ」「A4・A5未承認」等は各記録時点の履歴であり、最新の承認状態は本節の台帳と§0.2、内容照合と固定候補は§0.3を優先する。§0.1・§1.3・§4.1.1・§5.3の承認理由・原文・当時の状態は履歴として保持し、現在の未承認事項として読み替えない。

### 0.1 A4-1〜A4-5の本人承認・採用理由（2026-10-18 JST）

> 履歴注記：以下の原文・理由・状態はこの承認記録時点のもの。現在はA1〜A5すべて承認済みで、最新台帳は§0・§0.2を優先する。

**本人の明示承認原文**（本会話。改行を保持）：

> A4の判断は次のとおりです。
> A4-1 固定fixture（users 7件・expenses 14件）：承認
> A4-2 認可18ケースと境界ケース：承認
> A4-3 HTTP・JSON・DBの具体的期待値：承認
> A4-4 現在権限の変更シナリオ（CH01〜CH06）：承認
> A4-5 DB制約・tenant SQL・金額境界：承認
> 承認した項目と採用理由をGitHubに記録してください。A5は未承認のままにしてください。

**承認対象**：commit `89e94baeca3cfe65a8fa3c2bc4192cd80d89da2e` の[調査受入条件案](https://github.com/adeliep87/auth-authz-poc/blob/89e94baeca3cfe65a8fa3c2bc4192cd80d89da2e/docs/investigation-acceptance-draft.md) §2〜§6の固定fixture・ケース入力・具体的期待値・変更シナリオ、および§1の期待値を実装から独立させる方針。取得した同ファイルのblob SHAは `039d513c6d408624389ab84944359eb5431bc617`。内容基点 `18650f135f103ccfdf4574365dc25aed314f82f1` から直前headまでの差分はA1〜A3の仕様側承認記録のみで、受入条件案の内容は変わっていない。承認日をJSTで記録し、本人送信の分・秒は推測しない。

以下は直前の説明に対応する採用理由の整理であり、本人が独力で説明した学習答案へ換算せず、新しい仕様・ケース・期待値を追加しない。

| 判断 | 承認した範囲 | 採用理由・留意点 |
|---|---|---|
| A4-1 固定fixture | §2.1〜§2.2のusers 7行・expenses 14行（各全7列）、認証identity定数 | 少数の架空データで所有者・role・部署・状態・tenant・有効状態を区別する。同ID別tenantに異なるowner/金額を置き、同subject別issuerを別userへ対応付け、M-101の自己申請は現在部署と保存部署を異ならせて誤った条件追加を検出しやすくする。7/14は共通baselineの件数であり、性能試験の規模や追加境界fixtureを禁止する件数ではない。 |
| A4-2 認可18ケースと境界 | §3のAC01〜AC06各P/A/RとAU-role・AU-tenant | 2 role×3関係×3状態の基本18組を対応付け、EMPLOYEEの他人許可、MANAGERの別部署許可・非PENDING許可、自己申請への部署条件追加を検出する。未知roleやtenant不一致は追加の合成入力で検査し、18件だけで全セキュリティ性質を網羅したとはしない。通常D1の制約を外して不正fixtureを保存しない。 |
| A4-3 HTTP・JSON・DB期待値 | §2.3のJ01〜J10とerror期待値、§3〜§4および§6の対応する具体値・終了位置 | HTTP 200や型一致だけで別申請・誤金額・余分な項目を見逃さないよう、許可キー集合・型・値を独立定数で比較する。実装の認可関数や応答mapperから期待値を作らず、同じ誤りの共有を避ける。requestIdは固定仮値ではなく同一要求内の対応を検査する。不在・別tenant・閲覧拒否の404の外部契約をそろえ、GET前後の全users/expenses不変を期待する。応答時間まで含む完全な非識別性の保証には広げない。 |
| A4-4 現在権限の変更 | §5のCH01〜CH06。CH04-a/bは別々のbaselineから行う2シナリオ | 静的認可だけでは見えない古いrole/部署/active/tenantやsubject単独のキャッシュを検出する。変更シナリオ中は同じapp/Workerを維持し、再seed・app再生成で不具合を隠さない。CH06の明示したissuer/subject切替以外は同じ認証主体を維持する。CH04は所有者異動で申請保存部署を変えず、CH05は所有申請なしM-102を使ってFK RESTRICTと両立させる。変更完了後に開始する次要求での再取得が対象で、処理中要求の即時取消・同時変更snapshot保証ではない。 |
| A4-5 DB制約・tenant SQL・金額 | §6.1〜§6.2のmigration/SQL/DFケース、§4.2 H27の追加fixtureとJSON期待値 | DDLの存在、保存時のPK/FK/UNIQUE/RESTRICT/NOT NULL/値・保存型制約、取得時のtenant限定SQLを別々に検証し、一つの成功を他の保証の代わりにしない。不正INSERT/UPDATE等の拒否時は全業務値・件数不変、許可時は指定差分のみを期待する。金額0と9007199254740991のHTTP返却、負数・小数・上限超過等の保存拒否、数値文字列の適法な整数変換保存を承認済みA3に従って検査する。 |

**A5との境界**：今回承認したのは入力・事前状態・期待結果であり、L1〜L4の必須実行範囲、通常条件との同等性、artifactと実行物の対応、障害注入方式、ログ・再現性・観測証跡の合格条件を一括承認したものではない。受入条件案に併記されたこれらA5事項は**未承認のまま**。A2で承認済みのHTTP分類や共通finalizerの責務は変更しない。

§6.3の破損行・認証出力不正などの期待500と、正常DBの制約による保存拒否は別の境界で扱う。障害注入の方式や通常条件の認証後経路の成立性は未確認で、通常入口401とVitest条件下200だけで通常条件の認証後実行を確認済みにしない。

**実施状態**：A1〜A4承認済み、A5未承認。記録のみでfixture行・認可条件・HTTP/JSON/DB期待値は変更していない。全受入試験NOT_RUN。仕様・受入条件全体の正式固定、Issue反映・追加起票、コード/DDL実装、ローカル実測、実装後の独立レビュー、PR、技術採用、API本体移行は今回行わない。A4に意味変更が必要になった場合は差分と影響ケースを示して再承認する。

### 0.2 A5の承認と採用理由（2026-10-18 JST）

本人原文：「すべて承認 その理由をgithubに記載」。直前に説明したA5-1〜A5-5のすべてに対する明示承認。承認対象はcommit `e7488674bf5cf2a0775c8fad92cb5a66a890d2c4` の受入条件案§1・§6.3・§7〜§10にある検証要件と合格条件。理由は説明を基に整理したものであり、本人の独力回答として扱わない。

| 項目 | 採用理由 |
|---|---|
| A5-1 L1〜L4 | 認可単体、HTTPとlocal D1、通常Workerの拒否、通常条件に合わせた検証専用Workerの認証後処理を分離する。L2で200かつL3で401だけでは、通常条件で認証後処理が動くとは言えないため、L4代表5ケースまで必須とする。 |
| A5-2 実効条件・artifact・モック不在 | 実際に有効なruntime/flags/binding/依存解決・ビルド条件と実行した生成物を確認し、テスト専用条件だけでの成功や検査対象と実行物の不一致を防ぐ。通常入口に認証モック・faultが混入しないことを確認する。 |
| A5-3 障害とDB不変性 | 500/503を実際の依存境界から共通分類・応答処理に通し、結果を直接返すモックで検証を迂回しない。GETの前後で全業務行を観測し、観測不能なDB状態は合格扱いしない。 |
| A5-4 ログと合成秘密ラベル | 応答とのrequestId対応を保つ終端イベント1件を確認し、合成秘密ラベルがlogger・console・framework・runner出力に露出しないことも検査する。必要なログと非漏えいの両方を必須とする。 |
| A5-5 再現性・証跡・独立レビュー | code/spec commit、版・lockfile、実行コマンド、DB初期状態、期待値・実測、設定・生成物を対応付ける。未実行・未観測をPASSにせず、独立レビュー・PR・技術採用を別判断にする。 |

**最新承認状態：A1〜A5の個別判断すべて承認済み。** 本記録は検証要件の承認であり、仕様・受入条件一式の正式固定、Issue起票、実装、試験実施、技術採用を完了したものではない。全試験NOT_RUN。具体的な依存版・注入方式・検証用entrypoint等は後続調査で決める。

### 0.3 最終照合・固定候補の範囲（2026-10-19 JST）

開始時の指定ブランチheadは `b046dffc79430d096f61a13902097af3454e4a6e` で、引き継ぎ作成時から更新なし。A1〜A3の内容基点 `18650f135f103ccfdf4574365dc25aed314f82f1` からは5commit進み、変更は本書の承認台帳・理由の記録のみだった。§1〜§7は後から追加した承認理由節を除くと内容が一致する。

受入条件案は、A4内容commit `89e94baeca3cfe65a8fa3c2bc4192cd80d89da2e`、A5内容commit `e7488674bf5cf2a0775c8fad92cb5a66a890d2c4`、開始時headの3版でblob `039d513c6d408624389ab84944359eb5431bc617` が一致した。今回の改訂は現在状態・参照の同期と履歴注記に限り、業務ルール・fixture・ケースID・HTTP/JSON/DB期待値・受入基準は変更しない。

| 個別承認 | 承認対象の内容commit | 範囲・原文/理由の所在 |
|---|---|---|
| A1〜A3 | `18650f135f103ccfdf4574365dc25aed314f82f1` | A1=本書§1、A2=§2〜§3・§4.1、A3=§5.1〜§5.2。理由は§1.3・§4.1.1・§5.3。A1原文は上記学習側引き継ぎ§4.3、A2/A3原文は各理由節 |
| A4 | `89e94baeca3cfe65a8fa3c2bc4192cd80d89da2e` | 受入条件案§1の独立期待値方針と§2〜§6の入力・事前状態・期待値。原文/理由は本書§0.1 |
| A5 | `e7488674bf5cf2a0775c8fad92cb5a66a890d2c4` | 受入条件案§1・§6.3・§7〜§10の検証要件・合格条件。原文/理由は本書§0.2 |

個別承認日はすべて2026-10-18（JST）。本人送信の時・分・秒は推測せず、保存commit時刻と区別する。

**固定候補**は、本改訂を含むcommitの `docs/poc-specification-draft.md` と `docs/investigation-acceptance-draft.md` の2文書一式。本書§1〜§5の契約・§6の調査範囲/後続判断と、受入条件案§1〜§10を、既存の元の決定・A1〜A5承認の対応を保って固定する。承認理由と履歴は判断根拠として保持する。READMEは案内であり第三の仕様正本ではない。固定候補の完全SHAは保存後に再取得し、学習側引き継ぎへ記録して提示する。

意味変更が必要な不整合は今回の文書照合では見つからなかった。具体的依存版、DDL、認証アダプターの引数/未認証/成功外側の具体型、検証用entrypoint・fault方式、実効条件/生成物検査方法とそれらの実動作は後続調査の未確認事項。これらを文書固定前の新しい必須条件にはしない。

**2026-10-19 JSTの固定指示前の状態（履歴）。正式固定完了は§0.4を優先。** 対象SHA・範囲を提示した後、文書一式の正式固定とIssue反映の作業指示範囲を確認する。A1〜A5は再承認しない。固定を実施する次commitは対象内容commit、既存承認の原文または所在・承認日、固定作業の指示、範囲・除外・未確認を記録し、自分自身のSHAは埋め込まない。Issue反映・追加調査Issue・コード/DDL/依存・試験・実装後の独立レビュー・PR・採用・API本体移行は、この表記同期の完了に含めない。全試験NOT_RUN、対象外は§6を維持する。


### 0.4 仕様・受入条件一式の正式固定（2026-10-19 JST）

本人の今回の指示「進めて」により、対象内容commit `f3dfa197aabc6b013472c11c2d9041bcfc804275` にある既存の仕様案と受入条件案の2文書一式を正式固定する。これは2026-10-18 JSTのA1〜A5個別明示承認を取り直すものではない。対応する承認対象commit・本人原文と採用理由は§0.1〜§0.3・§1.3・§4.1.1・§5.3および学習側引き継ぎ§4.3〜§4.8を参照する。記録用commit自身のSHAは記載しない。

固定範囲：本書§1〜§5の契約と§6の調査・工程境界、受入条件案§1〜§10のfixture・ケースID・期待HTTP/JSON/DB・試験層・DB/ログ/設定/artifact観測と合格条件。§0の承認経緯・参考資料は判断根拠として保持する。READMEは案内であり第三の正本ではない。A1〜A5の個別承認日はいずれも2026-10-18 JST、文書一式の固定指示は2026-10-19 JST。本人の送信時分秒は推定しない。

除外：具体的依存版・DDL・認証アダプター外側型・検証専用entrypoint・fault注入/生成物照合方法は後続調査で決定する。実IdP、実Token暗号検証、BFF、Native、承認/更新API、クラウド公開、remote D1、refs/tmp/変更、技術採用、API本体移行を固定によって承認しない。

未確認：実装・依存導入・migration・ローカルD1/HTTP/Worker/ログ実測はいずれも未実施、全受入ケースNOT_RUN。L4同等性・通常bundleのmock/fault不在・障害/DB観測も未実測。調査報告、必須試験PASS、独立コードレビュー、PR、Issue #3の技術採用は別工程。意味変更が判明した場合は該当差分・理由・影響ケースだけを別途判断する。

## 1. 対象APIとroute境界

### 1.1 維持する決定

- GET `/expenses/{id}`。必要scopeの設定値は `expenses:read`。
- ID全体が `^R-[0-9]{4,12}$` に一致する。先頭0を許可し、文字列として扱う。
- 公開healthは作らない。未知routeは404。
- 保護routeの非GETは認証→scopeを通過した後に405。HEADもこの対象で、全最終応答の本文を空にする。
- 成功JSONは `id, ownerId, amountMinor, currency, status` の5項目だけ。

### 1.2 route具体化案（A1）

Workerへ届いたRequestのURLの**pathname**を基準に、`/expenses/`の後ろに空でない単一segmentだけがある形を保護routeとする。route識別ではIDの正規表現を使わず、ID形式の検査は利用者確認の後に行う。route識別自体は認証・D1を呼ばない。

segmentはpercent-decodeせずにID検査する。Frameworkがdecodeしたroute parameterをそのまま使う方式は本案に一致しない。percent encodingを別名として許可しない方針であり、URI一般の規則だとは主張しない。

| pathnameの例 | route区分 | 認証・scope・有効user通過後のGET |
|---|---|---|
| `/expenses/R-1001` | 保護route | tenant限定取得・認可へ |
| `/expenses/R-123`、`/expenses/abc` | 保護route | 400 INVALID_EXPENSE_ID |
| `/expenses/%52-1001`、`/expenses/R%2D1001` | 保護route | 400（decodeしない） |
| `/expenses/R-1001%2Fextra`、`/expenses/%ZZ` | 保護routeとしてRequestに到達した場合 | 400 |
| `/expenses`、`/expenses/`、`/expenses/R-1001/` | 未知route | 404 ROUTE_NOT_FOUND |
| `/expenses/R-1001/extra`、`/Expenses/R-1001`、`/health` | 未知route | 404 ROUTE_NOT_FOUND |

末尾slashのredirect・自動補正はしない。未知routeはmethod・認証情報の有無によらず404、HEADなら本文なし。queryはroute/ID/認証/権限の情報源にせず、このPoCでは無視する。`?tenant_id=T2`等でもtenantは変わらない。

この契約はRequestとしてWorkerへ到達した範囲を対象とする。URLのdot segment等がクライアントやruntimeで事前正規化される場合、送信文字列と実際のpathnameを区別して記録する。到達前の不正HTTP/URL拒否を、このAPIの400/404観測へ換算しない。

### 1.3 A1採用理由と承認記録（2026-10-18 JST）

> 履歴注記：以下の原文・理由・状態はこの承認記録時点のもの。現在はA1〜A5すべて承認済みで、最新台帳は§0・§0.2を優先する。

本人が直前の説明を受けA1の5項目を明示承認。承認対象は上記の内容commit `18650f135f103ccfdf4574365dc25aed314f82f1` の§1であり、以下は判断理由の記録であって契約の意味変更ではない。

| 項目 | 採用理由とトレードオフ |
|---|---|
| A1-1 単一segment | 定義したExpense APIだけを保護routeとし、曖昧な前方一致で未知の子pathを取り込まない。将来の履歴等は別routeとして明示的に追加できる。 |
| A1-2 percent-decodeしないID | `R-1001` と `%52-1001` 等の別表記を許容せず、クライアント／runtime／framework／アプリ間の二重decodeや解釈差、ログ・テストの表記揺れを減らす。decode自体が一般に危険または禁止という主張ではなく、ASCIIの所定IDだけを扱う本PoCの簡素化判断。Workerへ実際に届くpathnameの表現は後続調査で確認する。 |
| A1-3 末尾slash非正規化 | `/expenses/R-1001` を唯一の規定形とし、redirect・自動補正の差を試験契約に持ち込まない。末尾slashを同一視するAPIも成立するが今回は採用しない。 |
| A1-4 query無視 | `tenant_id`、role、department等の利用者が書き換え可能なqueryを認可根拠にせず、認証アダプターの主体とD1の現在user／申請データを正本とする。将来の検索・ページングqueryの利用を全面禁止する意味ではない。 |
| A1-5 未知route404と保護route内の不正ID400の分離 | route未定義と入力形式不正を区別して失敗原因・テスト期待値を明瞭にする。保護routeでは既決の認証・scope等を優先するため、未認証かつ不正IDは401、認証等を通過した不正IDは400。判定順の細部・error.code・共通headersは未承認A2と整合を取って固定する。 |

承認状態: **A1のみ承認**。A2〜A5および仕様／受入条件全体の正式固定は未承認。実装可能性・URL正規化の実測・試験は**NOT_RUN**。A1に意味上の変更が必要になった場合は差分を示して再承認する。

## 2. 判定順とエラー分類

以下は元の順序を具体化した固定候補（A2）。開始時にサーバーでrequestIdを生成し、各終了結果を§3の共通処理へ渡す。後続段階は実行しない。

| 順 | 判定 | 終了する条件 | status / error.code |
|---:|---|---|---|
| 0 | route | §1の保護routeでない | 404 / ROUTE_NOT_FOUND |
| 1 | 認証アダプター | 未認証 | 401 / AUTHENTICATION_REQUIRED |
| 1 | 認証結果の構造 | 結果種別・成功結果の必須型・kindが不正 | 500 / INTERNAL_ERROR |
| 2 | scope | 必要scopeの完全一致要素なし | 403 / INSUFFICIENT_SCOPE |
| 3 | 元のHTTP method | GET以外 | 405 / METHOD_NOT_ALLOWED |
| 4 | 主体種別 | kind=service | 403 / PRINCIPAL_NOT_SUPPORTED |
| 5 | usersを(issuer,subject)で取得 | D1可用性障害 | 503 / DEPENDENCY_UNAVAILABLE |
| 5 | usersの結果 | 0行（未登録） | 403 / USER_NOT_ELIGIBLE |
| 5 | 取得行の整合性 | 不正な型/必須値/role、複数行等 | 500 / INTERNAL_ERROR |
| 5 | 正常なuser行の資格 | active=0 | 403 / USER_NOT_ELIGIBLE |
| 6 | 未デコードID | 形式不正 | 400 / INVALID_EXPENSE_ID |
| 7 | expensesを現在tenantとidで限定取得 | D1可用性障害 | 503 / DEPENDENCY_UNAVAILABLE |
| 7 | expensesの結果 | 0行 | 404 / EXPENSE_NOT_FOUND |
| 7 | 取得行の整合性 | 不正な型/状態/金額/tenant/id等 | 500 / INTERNAL_ERROR |
| 8 | 業務認可 | §4の明示許可に該当しない | 404 / EXPENSE_NOT_FOUND |
| 9 | 許可項目の応答生成 | 許可 | 200、成功JSON |
| 各段階 | 一般例外 | プログラム不具合・予期しない内部例外 | 500 / INTERNAL_ERROR |

- `scopes`は文字列配列。完全一致で照合し、`expenses:read-extra`や大小文字違いを許可しない。
- 非GETは主体種別・users・ID・expenses・業務認可より前に終了する。serviceのPOSTでも認証/scope通過後は405。HEADがFrameworkによりGET handlerへ自動dispatchされても、**元のmethod**を使いこの順序を守る。
- 段階1で認証アダプターの結果種別と成功結果の構造を検証する。正規の未認証結果は401、不正な結果・成功結果の必須フィールド型不正・未定義kindは内部契約違反として500にする案。正規の成功結果だけを段階2のscopeへ渡す。正規のservice主体はscope通過後の非GETなら段階3で405、GETなら段階4で403になる。
- D1可用性障害は、対象版の公式仕様と実測で分類根拠を定める。SQL構文ミス、欠落table/migration、binding設定ミス、型変換ミス、制約違反を「D1に関係する」だけで503にしない。分類不能の例外は500として残す案。例外messageの文字列だけを外部code決定やログ項目に流用しない。
- 取得行破損500は新しい具体化案。未知RoleはL1の不正入力ではdeny、D1へのINSERT/UPDATEでは制約拒否、取得アダプターの破損行注入では500とし、試験層を混ぜない。
- 取得したuserの構造を検証してからactiveを評価する。expensesのtenant/idが取得条件と一致しない異常も500として扱い、別tenant行を利用しない。正常な「別tenantにしか存在しない」はSELECT結果0行による404。
- 1回の正常GETにつきアプリ側のusers取得1回・expenses取得1回を基本とする。アプリ側で無断の再試行・権限キャッシュ・所有者現在部署の追加取得を導入しない。D1内部の再試行はアプリ取得回数と分離し、対象版で記録する。呼出し回数と未到達段階を受入条件で確認する。
- 同一要求中の並行権限変更を即時取消する保証は置かない。「変更の完了後に始めた次要求で再取得」を今回の鮮度契約とする。

## 3. 応答契約と共通finalizer（A2）

### 3.1 JSON

成功例:

```json
{"id":"R-1001","ownerId":"E-301","amountMinor":12000,"currency":"JPY","status":"PENDING"}
```

error例（`<server-request-id>`は要求ごとの値を表す記法）:

```json
{"error":{"code":"EXPENSE_NOT_FOUND","requestId":"<server-request-id>"}}
```

JSONはキー順を問わず、値・型・許可キー集合を厳密に比較する。成功にtenant/department/role/active/issuer/subject/requestIdを追加しない。errorにmessage、SQL、stack、例外オブジェクト、対象のowner/部署/状態を追加しない。

不存在・別tenantのみ存在・業務閲覧対象外は同じ404、code、JSON形、共通ヘッダー。requestIdの値は異なってよい。応答時間を含む完全な非識別性はこの契約の保証ではない。

### 3.2 ヘッダーの具体化案

| 応答 | アプリが設定するヘッダー |
|---|---|
| 全routeの全最終応答 | `Cache-Control: no-store`、`X-Request-Id: <server-request-id>` |
| JSONを返す応答 | `Content-Type: application/json; charset=UTF-8`（media type/charsetは大小文字を正規化して検査） |
| 401 | `WWW-Authenticate: Bearer realm="expense-api"` |
| scope不足403 | `WWW-Authenticate: Bearer realm="expense-api", error="insufficient_scope", scope="expenses:read"` |
| 405 | `Allow: GET` |
| HEAD | その終了結果のヘッダーを維持し、本文を0 bytesにする。JSON表現のContent-Typeは維持する案 |

401ではこのPoCがToken暗号検証をしないため、期限切れ・署名不正等を判定したような `invalid_token` / `error_description` を付けない。Bearerのrealm具体値とscope不足403のchallengeは今回の提案。その他の403、404、400、500、503には本案では追加challengeを付けない。Allowは405で付け、他の終了結果に自動流用しない。

requestIdは入力 `X-Request-Id` を採用せずサーバーで生成する。同一要求のheader/error JSON/終端ログで同じ値、異なる要求で異なる値とする。形式は非空の安全な不透明IDとし、認証や認可の根拠にはしない。成功JSONの5項目には入れない。

### 3.3 全route共通finalizer

成功、早期拒否、未知route、捕捉した例外に対して、共通ヘッダー・HEAD本文除去・必要な終端イベントを一か所の共通処理で適用する案。具体的な関数名・middleware配置は調査時に選ぶ。Honoの通常エラー処理やHEAD自動処理がこの契約を迂回しないことを試験する。

アプリ側ではContent-Lengthを固定しない。runtimeが付与する転送関連ヘッダーの完全な集合は契約に固定せず、HEADで実本文が送信されないことを検査する。

Worker呼出し前の拒否、プロセス停止等でアプリがResponseを作れない障害は、共通finalizerで処理済みと扱わない。HEADの503本文除去は合成finalizer試験で確認し、D1前に405で終了する通常HEADのHTTP到達実績とは分ける。

## 4. 認証境界と業務認可

### 4.1 認証と現在権限の正本（既決）

| 境界 | 情報 | 用途 |
|---|---|---|
| テスト専用認証アダプター | issuer、subject、scopes、kind（user/service） | 外部主体識別・scope・主体種別 |
| 要求ごとのD1 users取得 | user_id、tenant_id、role、department、active | 内部利用者・現在のtenant/業務権限 |
| tenant限定取得したexpenses | tenant_id、id、owner_id、department、amount_minor、currency、status | 対象・申請時部署・状態・返却項目 |

**認証結果型と実装表現の委任（A2-5・2026-10-18承認済み）:** 成功結果の必須型は `issuer: string`、`subject: string`、`scopes: string[]`、`kind: "user" | "service"` とする。正規の未認証結果と成功結果を区別できる一つの戻り値型を設ける。アダプターの引数、未認証を表す値またはtag、成功結果の外側の構造は、調査時に一つの具体的な型として定義・記録する実装選択とする。認証出力から採用する情報は上表の4項目に限り、現在の業務権限を付加情報から採用しない。

元の準備記録§3.4の「具体的な入出力の型は仕様案作成時に定義する」に対し、本補足は**必須フィールドの型をここで示し、引数・外側表現の決定を調査へ委任する承認済みの具体化**である。受入fixtureのUNA・成功identity・AUTH-shapeをその具体型へ対応付け、調査コードのREADMEから型定義・fixtureを参照できるようにする。正規の未認証結果は401、結果種別不正・成功結果の必須フィールド欠落/型不正・未定義kindは§2の500とし、不正出力を未認証へ丸めない。既存のHTTP期待値・認可条件・ケース数は変更しない。

外部IDは(issuer,subject)の組で完全一致照合する。subject単独・email・任意のtenant/role/userヘッダーで結合/上書きしない。初回は**1内部利用者につき1 tenant・1外部ID**。異なるissuerの同subjectを別利用者へ対応付けるfixtureは、同一人物の複数IdP統合を実装する意味ではない。

通常Workerは保護routeで常に認証拒否。Authorizationなし、偽Bearer、任意の主体/tenant/role headerを提示しても401。通常入口へモック切替を追加しない。成功は隔離した検証用入口で認証アダプターを注入し、通常と共通の取得・認可・応答処理を通す。通常bundleへのモック・fault混入を受入条件で検査する。

### 4.1.1 A2-1〜A2-5の本人承認と理由（2026-10-18 JST）

> 履歴注記：以下の原文・理由・状態はこの承認記録時点のもの。現在はA1〜A5すべて承認済みで、最新台帳は§0・§0.2を優先する。

**承認原文**：「A2の判断は次の通りです。A2-1 判定順：承認／A2-2 エラー分類：承認／A2-3 JSON・headers：承認／A2-4 共通finalizer：承認／A2-5 認証アダプター：承認。承認した項目は理由とともにGitHubに記録してください。A3〜A5は未承認のままにしてください。」

**承認対象**：内容commit `18650f135f103ccfdf4574365dc25aed314f82f1` の§2（判定順・エラー分類）、§3（JSON/headers・finalizer）、§4.1（認証アダプターの必須型と実装表現の委任）。A1の理由・承認のみを追加した直前のcommit `4f85159d61d00f7e7e1fea62fef5f2754c350303` から、A2の契約内容は変更していない。以下は既存契約の採用理由であり、新しいHTTP要件を追加しない。

| 項目 | 採用理由・境界 |
|---|---|
| A2-1 判定順 | 認証・scopeを先に判定し、未認証時に申請取得やID詳細判定へ進まない。非GETは認証・scope後の405でD1を呼ばずに終了させる。早期終了と決定的なエラー優先順位を保ち、未知route404はA1で先に判定する。 |
| A2-2 HTTP statusとerror.code | 403等の失敗理由を機械可読に分ける。不存在・別tenant・閲覧拒否を同じ404/EXPENSE_NOT_FOUNDへ寄せて情報露出を抑える。D1可用性障害503とSQL不備・内部契約違反・一般例外500を区別し、未知障害を安易に503へ分類しない。 |
| A2-3 JSON/headers | 成功は許可5項目のみ、失敗は安定したerror.codeとrequestIdの限定JSONにして内部情報を返さない。全応答のno-store・X-Request-Id、401/不足scopeのBearer challenge、405のAllow、HEAD本文0などのHTTP契約を一定にする。同一要求のレスポンス・終端ログのrequestIdを一致させる。 |
| A2-4 共通finalizer | 200・各拒否・未知route・捕捉例外に共通ヘッダー、HEAD本文除去、終端ログを一貫適用し、分岐ごとの適用漏れを防ぐ。具体的middleware配置やframework APIは調査で選ぶ。Workerへ到達前やResponse不能時まで保証しない。 |
| A2-5 認証アダプター | 成功時必須のissuer:string、subject:string、scopes:string[]、kind:user/serviceを固定し、正規未認証401と不正アダプター結果500を混同しない。引数・未認証tag/値・成功結果の外側表現は調査時に具体型として定義・記録し、fixture・受入ケースに対応付ける。元準備§3.4の「具体的な入出力型を仕様作成時に定義」からこの限定範囲を調査へ委任する変更も、今回A2-5として明示承認された。 |

**状態**：A1・A2のみ承認。A3〜A5、仕様・受入条件全体の正式固定、追加調査Issue、実装、試験、技術採用は未承認／未実施。全試験NOT_RUN。後でA2内容に意味上の変更が必要な場合は差分と影響ケースを示して再承認する。

### 4.2 認可表（既決）

有効user・必要scope・同tenant・妥当な行が前提。

| 閲覧者 | 自分 PENDING | 自分 APPROVED | 自分 REJECTED | 他人・同部署 PENDING | 他人・同部署 APPROVED/REJECTED | 他人・別部署 全状態 |
|---|---|---|---|---|---|---|
| EMPLOYEE | 許可 | 許可 | 許可 | 拒否 | 拒否 | 拒否 |
| MANAGER | 許可 | 許可 | 許可 | 許可 | 拒否 | 拒否 |

同tenantの確認を、所有者判定より前に必要条件として置く。未知Role/ADMINに権限を追加しない。service主体はroleではなくkindで拒否する。

**明示承認済み部署ルール:** MANAGERの現在の `users.department` と、申請時に保存された `expenses.department` を比較する。所有者の異動だけでは既存申請部署を変更しない。自分の申請には部署一致条件を付けない。

現在のrole/department/active/tenantを要求ごとに取得し、変更完了後の次要求で反映する。参照中ownerのtenant変更は§5のFK案により禁止され得るため、tenant鮮度は所有申請のないviewerで試す。

## 5. D1モデルと保証

users/expensesの2表、生SQL。ORM、承認履歴、version、複数外部ID対応表は今回追加しない。列は以下ですべてNOT NULL。

| 表 | 必須列 |
|---|---|
| users | user_id、tenant_id、issuer、subject、role、department、active |
| expenses | tenant_id、id、owner_id、department、amount_minor、currency、status |

### 5.1 別々に検証する3保証（既決）

| 保証 | 手段 | 防ぐもの |
|---|---|---|
| 読み出し時tenant分離 | expensesのWHERE条件に現在の `tenant_id` と `id` をbind | 他tenant行を取得して後から隠す実装 |
| 保存時の同tenant所有者 | expenses(tenant_id,owner_id) → users(tenant_id,user_id) 複合FK | T1の申請がT2所有者を参照する不整合 |
| tenant内申請ID一意 | expenses PRIMARY KEY(tenant_id,id) | 同tenant・同idの複数行 |

別tenantで同じidが存在することは許可する。PK/FKがあってもtenant限定SELECTを省略しない。

### 5.2 具体的キーと制約の提案（A3）

- users: `PRIMARY KEY(user_id)`（全体で一意）、`UNIQUE(issuer,subject)`、`UNIQUE(tenant_id,user_id)`。
- expenses: `PRIMARY KEY(tenant_id,id)`、上記の同tenant複合FK。
- FKは `ON UPDATE RESTRICT / ON DELETE RESTRICT`。参照中usersのuser_id/tenant_id変更や削除を拒否し、自動cascadeで申請を移動/削除しない。非キーのrole/department/active変更は許可。
- issuer/subjectおよび識別子は大小文字を勝手に正規化せず、case-sensitiveな比較/一意性を用いる案。issuerのネットワーク接続やdiscoveryはしない。
- roleはEMPLOYEE/MANAGER、activeは保存値0/1、statusはPENDING/APPROVED/REJECTED、currencyはJPY。
- amount_minorは保存型が整数、範囲0〜9007199254740991。activeも保存型が整数で0/1。INTEGER宣言だけで保証したとしない。SQLiteの型affinityによる変換後の保存型・値を検査する。整数へ変換される数値文字列まで一律拒否するという未決要件は追加しない。
- 複合PKの列にも明示NOT NULL。その他必須列のNULLも拒否する。
- APIのID正規表現をDBにも重複実装することは必須にしない。DB由来行のID/型/値は§2で検証する。文字列長等の新しい業務制約は今回追加しない。
- users全体PKを選ぶ場合でも、複合FKの親側列組に対応する複合UNIQUEを設ける。これは3保証の置き換えではなく、SQLiteでその関係を実現するための具体化案。

DDL構文、型変換・範囲境界・FK enforcement・RESTRICTの実動作はlocal D1で**未検証**。通常制約を外して未知Roleや破損行をfixtureへ保存しない。

### 5.3 A3-1〜A3-5の本人承認・採用理由（2026-10-18 JST）

> 履歴注記：以下の原文・理由・状態はこの承認記録時点のもの。現在はA1〜A5すべて承認済みで、最新台帳は§0・§0.2を優先する。

**本人の明示承認原文**：「A3の判断は次のとおりです。A3-1 usersのPK・UNIQUE：承認／A3-2 expensesの複合PK・複合FK：承認／A3-3 FKのRESTRICT：承認／A3-4 role・active・status・金額等の値制約：承認／A3-5 保存型・NOT NULL：承認。承認した項目は採用理由とともにGitHubに記録してください。A4・A5は未承認のままにしてください。」

**対象内容**：commit `18650f135f103ccfdf4574365dc25aed314f82f1` §5.1〜§5.2。後続のA1承認記録commit `4f85159d61d00f7e7e1fea62fef5f2754c350303`、A2承認記録commit `71a8b244b9cfc4838f9e4dec89537554049a21f9` は§5のDB契約を変更していない。以下は§5の採用理由であり、実測に基づくDDL承認や追加要件ではない。

| 判断 | 採用理由・留意点 |
|---|---|
| A3-1 usersのPKとUNIQUE | `user_id` の全体一意性、`(issuer,subject)` の外部主体一意性を独立して保証。`UNIQUE(tenant_id,user_id)` は全体PKと論理的に冗長でも、SQLiteの複合FKが参照する親キーの列組に対応する一意制約として設ける。 |
| A3-2 expensesの複合PKと複合FK | `PRIMARY KEY(tenant_id,id)` は同tenant内の申請ID重複を拒否し、異なるtenantでの同IDを許す。`(tenant_id,owner_id)→users(tenant_id,user_id)` は別tenantのownerを保存時に拒否する。これらは取得時のtenant限定SELECT／API業務認可を代替しない。 |
| A3-3 FKのRESTRICT | 参照中の親userのキー変更・削除を拒否し、CASCADEで申請が意図せず移動／削除されることを防止。role・department・active等の非キー変更は可能で、異動・退職等の将来業務仕様を新規決定したわけではない。 |
| A3-4 値の制約 | 許容role=EMPLOYEE/MANAGER、active=0/1、status=PENDING/APPROVED/REJECTED、currency=JPY、amount_minorは非負のJS安全整数範囲とし、APIを通らない書込みでも業務上無効なデータを拒否する。ADMIN権限や追加通貨は作らない。 |
| A3-5 保存型とNOT NULL | SQLiteのINTEGER宣言と実保存型を混同せず、active／amount_minorの格納型・値を検査する。複合PK列を含む必須列に明示NOT NULLを適用する。SQLiteの型変換で適法な整数として保存される数値文字列を入力文字列だっただけで一律拒否する追加要件は置かない。 |

**状態**：A1〜A3は承認済み、**A4・A5は未承認**。仕様／受入条件全体の正式固定・実装・試験・技術採用は未実施。D1実機におけるDDL・FK enforcement・RESTRICT・CHECK・型変換の実動作は**NOT_RUN**であり、正式採用判定ではない。承認範囲に意味変更があれば対象差分を示して再承認する。

## 6. 調査範囲と後続工程

配置は `services/expense-api/`、npm単独＋package-lock。手書き相当200行は分割目安で、必要な試験を削らない。既存ハーネスの全面移植をしない。

Workers/Hono/D1/Vitestは評価候補。Node/npm/TypeScript/Wrangler/lintを含め、Issueに残る版は調査時に公式metadata・engines・peer・使用APIと再照合する。peer無視による成功扱いをしない。

**仕様・受入条件の固定 → 追加調査Issue → 独立ブランチでローカル実測 → 別セッション・別モデルの独立レビュー → PR → Issue #3で所有者の採用判断。**

追加調査Issueは固定時点で未起票。固定後に存在を再照合し、未起票なら#1のnative sub-issueとして作成、#3から参照する。予定番号を推測しない。文書全文をIssueへ二重管理せず、決定状態・理由・残課題と固定commitを参照する。

| 既存Issue | 固定後の同期内容 |
|---|---|
| #1 | 現在の段階、固定commit、調査子Issueと後続順序 |
| #2 | 配置/npm/lockfile/分割・独立レビュー方針。版は評価結果参照 |
| #3 | 評価条件と調査参照。技術採用は所有者判断までopen/未決 |
| #4 | 本書§1〜§3の契約参照と承認範囲 |
| #5 | 本書§4の境界、通常/L4の隔離・保証範囲 |
| #6 | 認可表・部署の既決・D1具体案とfixture |
| #7 | 受入条件の固定commit、必須ケース・NOT_RUN・未確認 |

2026-10-19 JSTの再取得で、Issue #1〜#7は2026-10-17作成時の未決案のまま、全open・全コメント0件。追加調査Issueなし、PRは全状態0件、#1のnative子Issueは#2〜#7の6件である。旧選択肢は本人の後続決定を取り消さない。この改訂ではIssueの完了チェック・採用状態を更新していない。

調査報告完了、必須試験PASS、技術採用、API本体への移行は別判定。仕様レビューとこの会話内の分担チェックを、実装後の別セッション・別モデルによる独立コードレビューへ換算しない。

対象はローカル・架空データのみ。実IdP、実Token暗号検証、BFF、Native、承認/更新API、クラウド公開、remote D1、`refs/tmp/`変更は範囲外。D1は--localを明示しremote bindingを使わず、起動はloopback。fixtureの管理用DB変更を更新APIへ広げない。

## 7. 参照資料と確認範囲

以下は仕様レビューの参考。製品の採用・実装・実測の証拠ではない。具体的なバージョンでの使用経路は調査時に照合する。

- [RFC 9110 §9.3.2、§10.2.1、§15.5.2/6](https://www.rfc-editor.org/rfc/rfc9110.html): HEADの本文禁止、405のAllow、401のchallenge。HEADを今回の保護routeで405とするのは既存のPoC契約。
- [RFC 6750 §3〜§3.1](https://www.rfc-editor.org/rfc/rfc6750.html): Bearer challengeとscope不足。内部業務エラーcodeは本PoCの提案でありRFCのcodeそのものではない。
- [SQLite Foreign Keys](https://www.sqlite.org/foreignkeys.html): 複合外部キーと親側一意キー、参照更新/削除の動作。
- [SQLite Datatypes](https://www.sqlite.org/datatype3.html): 型affinityと保存型。local D1の実測を代替しない。
- [Cloudflare D1 Foreign keys](https://developers.cloudflare.com/d1/sql-api/foreign-keys/): D1におけるFKの扱い。
