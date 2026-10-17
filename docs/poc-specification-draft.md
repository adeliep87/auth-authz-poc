# Expense API PoC 仕様案（DRAFT・未承認）

根拠: my-projects/research/auth-learning/poc-preparation-2026-10-17.md、poc-preparation-next-session.md。元決定・部署の明示承認・レビュー補足・新提案を区別し、実装や実測の成功を主張しない。

## API
GET /expenses/{id}。IDは ^R-[0-9]{4,12}$、必要scopeは expenses:read。成功JSONは id,ownerId,amountMinor,currency,status の5項目のみ。処理順: 認証→scope→（非GETは405）→利用者確認→ID形式→現在のtenantで限定した申請取得→業務認可→200。401は未認証とBearer challenge、403はscope不足/無効・未登録利用者/非対応主体、400は不正ID、404は不存在/別tenant/閲覧対象外、503はD1依存障害。公開healthなし、未知route404、保護応答no-store。

レビュー補足案（未承認）: 一般例外500、405 Allow: GET、HEAD本文なし、エラーJSONは {"error":{"code":"…","requestId":"…"}}、同じサーバー生成requestIdをX-Request-Idと非秘密ログに使用し、入力requestIdを採用しない。具体的code案は AUTHENTICATION_REQUIRED、INSUFFICIENT_SCOPE、USER_NOT_ELIGIBLE、PRINCIPAL_NOT_SUPPORTED、INVALID_EXPENSE_ID、EXPENSE_NOT_FOUND、ROUTE_NOT_FOUND、METHOD_NOT_ALLOWED、DEPENDENCY_UNAVAILABLE、INTERNAL_ERROR。非GETは認証・scope後、D1前に終了。全route共通finalizer、未デコードIDの形式検証、取得行破損500は今回の具体化案であり固定前に確認する。

## 認証境界
テスト専用アダプター出力は issuer,subject,scopes,kind(user/service)。D1 usersの現在の user_id,tenant_id,role,department,active を要求ごとに取得し、認証データや任意ヘッダーで上書きしない。(issuer,subject)の組で内部利用者を解決し、subjectやemailだけで結合しない。通常Worker入口は常に認証を拒否し、fake Bearer/任意主体ヘッダーでも401。成功経路は隔離したテスト専用entrypointで注入し、通常bundleへfakeを混入させない。実Token暗号検証や実IdP接続は範囲外。

## 認可表
| 閲覧者 | 自分の申請（全3状態） | 他人の同部署PENDING | 他人の同部署APPROVED/REJECTED | 他人の別部署 |
|---|---|---|---|---|
| EMPLOYEE | 許可 | 拒否 | 拒否 | 拒否 |
| MANAGER | 許可 | 許可 | 拒否 | 拒否 |

同tenant・有効user・必要scopeが前提。別tenantは404。未知RoleやADMINへ追加権限を与えない。**明示承認済み部署ルール**: MANAGERの現在のusers.departmentと申請時保存のexpenses.departmentを比較。所有者の異動だけで既存申請部署を変更しない。自分の申請に同部署条件を追加しない。

## D1制約案
users/expensesの2表、生SQL。users UNIQUE(issuer,subject)、expenses PRIMARY KEY(tenant_id,id)、expenses(tenant_id,owner_id)からusers(tenant_id,user_id)への同tenant複合FK。usersの全体user_id PKと複合UNIQUEは具体化案。必須列NOT NULL、role EMPLOYEE/MANAGER、active 0/1、status PENDING/APPROVED/REJECTED、currency JPY、amount_minorは0〜9007199254740991の整数。FKのRESTRICT動作・SQLite保存型をlocal D1で未検証。tenant限定SELECT（他tenantの読出し防止）と所有者FK（不正な保存防止）は別々の受入条件。

## 範囲・工程
services/expense-api/、npm単独＋package-lock。Workers/Hono/D1/Vitestは評価候補であり採用未決。ローカル・架空データのみ。実IdP、実Token検証、BFF、Native、承認/更新API、クラウド公開、remote D1、refs/tmp/変更は除外。

仕様・受入条件の固定→追加調査Issue→独立ブランチでローカル実測→別セッション・別モデルの独立レビュー→PR→Issue #3で所有者の採用判断。HTTP理解確認Q1〜Q20は終了済み、正式Session番号は変更しない。