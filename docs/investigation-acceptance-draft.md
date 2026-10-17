# PoC追加調査の受入条件案（DRAFT・全件NOT_RUN）

[仕様案](poc-specification-draft.md)に対応。期待値は実装と独立に定義し、実測・PASS・正式採用を先取りしない。

## 4層
L1 認可単体: 所有者・Role・部署・状態・未知Roleのallow/deny。L2 fake認証HTTP＋実migration適用済みlocal D1: 判定順・HTTP・現在権限・tenant・DB不変性。L3 通常entrypoint/bundle: 保護route401、fakeヘッダー無効、未知route404。L4 隔離した検証専用entrypoint/bundle: 通常とruntime、compatibility_date、実効flags、bindings、module resolution、bundle条件を比較し、共通コードの200・代表拒否404・D1依存障害503を実測。L2の200とL3の401だけではL4は確認できない。通常bundleにfakeが含まれないことを依存graph/入力/生成物で確認。生成物hashと実行物を対応付ける。全層とも実Token暗号検証・実IdP・クラウドを保証しない。

## 架空fixtureとHTTP期待値
T1: E-301（EMPLOYEE/SALES/active1）、E-302（EMPLOYEE/DEV/active1）、M-101（MANAGER/SALES/active1）、E-399（active0）。T2: E-901（EMPLOYEE/SALES/active1）。issuer Aのsubject same-sub→E-301、issuer Bの同じsubject→E-901。

T1/R-1001=E-301/SALES/PENDING/12000JPY、T1/R-1003=E-301/SALES/APPROVED/20000JPY、T1/R-1004=E-301/SALES/REJECTED/500JPY、T1/R-1005=E-302/DEV/PENDING/60000JPY、T2/R-1001=E-901/SALES/PENDING/99000JPY、T2/R-9001=E-901/SALES/PENDING/91000JPY。別状態の自分の申請やMANAGERの自分の申請も追加fixtureで全3状態確認。

| ID | 入力 | 期待 |
|---|---|---|
| H01 | E-301 GET R-1001 | 200、5項目のみ、ownerId E-301、amountMinor 12000 |
| H02 | E-901 GET R-1001 | 200、ownerId E-901、amountMinor 99000（同ID別tenant） |
| H03 | M-101 GET R-1001 | 200（他人同部署PENDING） |
| H04 | E-302 GET R-1001 | 共通404 EXPENSE_NOT_FOUND |
| H05 | M-101 GET R-1003/1004/1005 | 各404（状態/部署） |
| H06 | E-301 GET R-9001 | 404（別tenantにだけ存在） |
| H07 | 未認証＋不正ID | 401、後続検査なし |
| H08 | scope不足＋所有者 | 403、後続検査なし |
| H09 | active0＋不正ID | 403、ID検査なし |
| H10 | 有効user＋不正ID | 400。数字3桁/13桁も400、4桁/12桁の不存在IDは404 |
| H11 | service主体 GET | 403、users未取得 |
| H12 | scope通過済みPOST/HEAD | 405 Allow: GET、D1未取得。HEAD本文なし |
| H13 | 未認証HEAD/scope不足HEAD | 401/403、本文なし |
| H14 | GET/HEAD /health・未知route | 404、HEAD本文なし |
| H15 | users取得D1障害＋不正ID | 503、ID検査なし |
| H16 | expenses取得D1障害 | 503 |
| H17 | 予期しない内部例外 | 500、内部詳細なし |
| H18 | 通常Workerへ偽Bearer/主体header | すべて401 |
| H19 | L4で許可・拒否・障害 | 200/404/503、設定差分・artifact対応を記録 |

全HTTPケースでstatus/code/JSON/header/no-store/requestId/後続呼出し回数を比較。外部404は不存在と対象外で同じ形式。GET前後のusers/expenses業務データ不変を比較。障害で観測不能ならNOT_OBSERVEDとしPASSにしない。HEADの合成503本文除去はfinalizer単体とし、HTTPで到達したとは主張しない。

## 現在権限の変更
同じアプリ・同じfake identityで要求1→D1変更完了→要求2（途中再初期化なし）。M-101 role MANAGER→EMPLOYEE: R-1001は200→404。M-101 department SALES→DEV: R-1001は200→404、R-1005は200。M-101 active1→0: 200→403。所有者E-301の部署SALES→DEV: M-101のR-1001閲覧は200維持、expenses.departmentはSALESのまま。E-301自身も200維持。

## local D1制約
空DB migration、再適用、別空DB再構築を確認。INSERT/UPDATEでuser PK、(issuer,subject)一意、expense(tenant_id,id)PK、同tenant所有者FK（有効/不存在/他tenant）、role、active、status、currency、NOT NULL、金額0/9007199254740991/負数/小数/上限超過/非数値を検証。参照中userの削除・キー変更RESTRICTも確認。SQLのtenant限定取得と複合FKは別試験。未知RoleのためにD1制約を外さず、認可単体で不正Role拒否を検証する。

## ログ・再現性
正常/拒否/例外のraw stdout/stderrなど導入済み経路で、必要な非秘密終端イベント（requestId/operation/stage/result/httpStatus）が存在し、合成秘密ラベル（Authorization/Cookie/body/query/入力X-Request-Id/例外message）が出力されないことを同時確認。ログ0件は不合格。Node/npm/依存/engines/peer/lockfile、lint/typecheck/test/build/loopback起動、local D1 --local、migration、実効flags、mock/fault境界、case ID、spec/code commit、artifact hash、期待値、実測、PASS/FAIL/BLOCKED/NOT_RUN、未確認、非秘密証跡を記録。実IdP、実Token検証、BFF、Native、公開、remote D1は対象外。

固定→調査Issue→独立ブランチ実測→独立レビュー→PR→Issue #3で所有者判断。