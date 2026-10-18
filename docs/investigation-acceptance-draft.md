# PoC追加調査の受入条件案（A4/A5承認済み・正式固定未実施・全件NOT_RUN）

改訂: 2026-10-19（JST）。現在状態と参照だけを同期し、fixture・ケース・期待値・合格条件は維持。[仕様案](poc-specification-draft.md)の固定候補に対応する。旧版 `b297e727d5087d71b26a6ea1f4eca4cf226f44c4` のH01〜H19を追跡可能にし、不足を具体化した。**文書レビューのみ。コード・DDL・migration・試験実行・実測は未実施。**

本書のfixture・期待値はA4、試験範囲・観測・ログ・再現性はA5として2026-10-18に明示承認済み（仕様案§0.1・§0.2）。route/応答/DB制約は承認済みA1〜A3に従う。最新台帳・承認内容の接続と固定候補は仕様案§0・§0.3を参照する。文書一式の正式固定・Issue反映は未実施で、個別承認は取り直さない。理解確認Q3のbundle成功・通常401・Vitest200は設問上の仮定であり、本書の実測結果ではない。

## 1. 試験層・状態・独立期待値

| 層 | 実行対象 | この層で確認すること | この層だけでは確認できないこと |
|---|---|---|---|
| L1 認可単体 | HTTP/D1非依存の共通認可関数 | 認可表、tenant前提、未知Roleのallow/deny | HTTP、SQL/D1、通常bundle、実認証 |
| L2 モック認証HTTP＋local D1 | 実migration適用DBと共通HTTP/取得/認可/応答 | runnerの実効条件でHTTP・現在権限・tenant・DB不変性 | 通常条件での認証後実行、実Token/実IdP |
| L3 通常入口 | 通常entrypointのartifactをローカル起動 | 未認証/偽Bearer/主体header拒否401、未知route404 | 認証後のD1・認可・200 |
| L4 通常条件の検証専用Worker | 別entrypoint、同じ共通コード、事前生成artifact | §7の同等条件で200・代表404・D1依存503 | 通常bundle自体の200、実認証、クラウド |
| 限定fault/component | 取得結果/依存境界/finalizerの明示した差替え | 破損行500、例外分類、HEAD合成応答 | 実D1へその不正行を保存できること、実クラウド障害 |

このPoC固有のL1〜L4は、上位ロードマップの「実Token/実IdPを含む四層」と番号の意味が異なる。実Token/実IdP層は今回未実施のまま。

L2の200＋L3の401でL4を合格にしない。L2でworkerdを使っているだけでもL4の設定・module resolution・生成物対応を代替しない。通常スキーマを破損させたHTTP試験をL2通常D1結合の合格にしない。

| 記録欄 | 値と意味 |
|---|---|
| case status | NOT_RUN=未実行、PASS=必須観測がすべて一致、FAIL=期待違反を観測、BLOCKED=前提/計測不能で判定未完 |
| 個別観測 | HTTP、DB、log、設定同等性、artifact対応を分離。取得できない観測はNOT_OBSERVED |
| 現在 | 全ケースNOT_RUN。実測欄・artifact/code commit欄は未取得であり、仮の値を入れない |

期待値は以下の固定表/定数から作り、認可関数や応答mapperを呼んで生成しない。型/schemaの一致と具体的値の一致を両方確認する。表のパラメータ化ケースは指定した全値を展開し、個別case IDと結果を残す。aliasの重複実行は必須にせず、同じ観測へ複数の要求IDを対応付けてよい。

全HTTPケースには仕様案§3のheaders/JSON/HEAD/requestId、本書§8のDB不変性、本書§9の終端ログを適用する。ただしD1等を使わないL1/componentにはHTTP/DB観測を要求しない。観測範囲は実行層ごとに記録する。

## 2. 完全な架空fixtureと独立JSON期待値（A4）

### 2.1 users：7行・全7列

A=`https://issuer-a.example.invalid`、B=`https://issuer-b.example.invalid`。DBでは別名A/Bではなくこの文字列を保存する。実IdPへの接続はない。

| user_id | tenant_id | issuer | subject | role | department | active |
|---|---|---|---|---|---|---:|
| E-301 | T1 | A | same-sub | EMPLOYEE | SALES | 1 |
| E-302 | T1 | A | e302 | EMPLOYEE | DEV | 1 |
| E-303 | T1 | A | e303 | EMPLOYEE | SALES | 1 |
| M-101 | T1 | A | m101 | MANAGER | SALES | 1 |
| M-102 | T1 | A | m102 | MANAGER | SALES | 1 |
| E-399 | T1 | A | e399 | EMPLOYEE | SALES | 0 |
| E-901 | T2 | B | same-sub | EMPLOYEE | SALES | 1 |

「identity=E-301」等はその行のissuer/subject、`scopes=["expenses:read"]`、`kind="user"`を認証アダプターから注入する略記。user_id/tenant/role/department/activeを認証から注入する意味ではない。

追加identity定数: UNA=正規の未認証結果、SVC=`{issuer:A,subject:"service-fixture",scopes:["expenses:read"],kind:"service"}`、ABS=`{issuer:A,subject:"absent-user",scopes:["expenses:read"],kind:"user"}`。scope不足は対応identityのscopesだけを指定値へ変更する。

UNA・成功identity・SVC/ABSは仕様上の入力定数。認証アダプターの引数・戻り値の外側表現は仕様案§4.1の承認済みA2-5の委任範囲に従って調査時に定義し、これらの定数と§6.3のAUTH-shapeを同じ具体型へ対応付けて記録する。外側表現の選択で401/500の分類や認証から採用する4項目を変えない。2026-10-18の補足はこの委任範囲の明示であり、fixture行・ケース・期待値の追加変更ではない。

M-102は所有申請なし。tenant変更でowner-FKを破壊しないためのviewer。E-303はEMPLOYEEの他人・同部署拒否用。

### 2.2 expenses：14行・全7列

| tenant_id | id | owner_id | department | amount_minor | currency | status |
|---|---|---|---|---:|---|---|
| T1 | R-1001 | E-301 | SALES | 12000 | JPY | PENDING |
| T1 | R-1003 | E-301 | SALES | 20000 | JPY | APPROVED |
| T1 | R-1004 | E-301 | SALES | 500 | JPY | REJECTED |
| T1 | R-1005 | E-302 | DEV | 60000 | JPY | PENDING |
| T1 | R-1006 | E-302 | DEV | 60001 | JPY | APPROVED |
| T1 | R-1007 | E-302 | DEV | 60002 | JPY | REJECTED |
| T1 | R-1101 | M-101 | DEV | 7001 | JPY | PENDING |
| T1 | R-1102 | M-101 | DEV | 7002 | JPY | APPROVED |
| T1 | R-1103 | M-101 | DEV | 7003 | JPY | REJECTED |
| T1 | R-1201 | E-303 | SALES | 8001 | JPY | PENDING |
| T1 | R-1202 | E-303 | SALES | 8002 | JPY | APPROVED |
| T1 | R-1203 | E-303 | SALES | 8003 | JPY | REJECTED |
| T2 | R-1001 | E-901 | SALES | 99000 | JPY | PENDING |
| T2 | R-9001 | E-901 | SALES | 91000 | JPY | PENDING |

M-101の自己申請は保存部署DEV、現在部署SALES。3状態とも許可となることで、自己申請へ同部署条件を誤追加した実装を検出する。所有者現在部署と申請保存部署が異なることは不正fixtureではない。

### 2.3 成功JSON定数

| JSON ID | 期待JSON（このキー集合・型・値だけ） |
|---|---|
| J01 | `{"id":"R-1001","ownerId":"E-301","amountMinor":12000,"currency":"JPY","status":"PENDING"}` |
| J02 | `{"id":"R-1003","ownerId":"E-301","amountMinor":20000,"currency":"JPY","status":"APPROVED"}` |
| J03 | `{"id":"R-1004","ownerId":"E-301","amountMinor":500,"currency":"JPY","status":"REJECTED"}` |
| J04 | `{"id":"R-1005","ownerId":"E-302","amountMinor":60000,"currency":"JPY","status":"PENDING"}` |
| J05 | `{"id":"R-1101","ownerId":"M-101","amountMinor":7001,"currency":"JPY","status":"PENDING"}` |
| J06 | `{"id":"R-1102","ownerId":"M-101","amountMinor":7002,"currency":"JPY","status":"APPROVED"}` |
| J07 | `{"id":"R-1103","ownerId":"M-101","amountMinor":7003,"currency":"JPY","status":"REJECTED"}` |
| J08 | `{"id":"R-1201","ownerId":"E-303","amountMinor":8001,"currency":"JPY","status":"PENDING"}` |
| J09 | `{"id":"R-1001","ownerId":"E-901","amountMinor":99000,"currency":"JPY","status":"PENDING"}` |
| J10 | `{"id":"R-9001","ownerId":"E-901","amountMinor":91000,"currency":"JPY","status":"PENDING"}` |

error期待値E(code,rid)=`{"error":{"code":code,"requestId":rid}}`。ridは固定の仮文字列ではなく、同じ応答のX-Request-Id・終端ログとの一致を検査する。E404はE(EXPENSE_NOT_FOUND,rid)。HEADではこのJSONを送信せず0 bytes。

## 3. 認可表との対応：18ケース＋境界

各ACはL1のallow/denyとL2のHTTP＋実local D1の両方で確認。共通fixtureは§2、methodはGET、pathは `/expenses/`＋対象ID。全GETでDB不変。

| case ID | viewer | 対象ID | 関係/状態 | L1 | L2 |
|---|---|---|---|---|---|
| AC01-P | E-301 | R-1001 | 自己PENDING | allow | 200/J01 |
| AC01-A | E-301 | R-1003 | 自己APPROVED | allow | 200/J02 |
| AC01-R | E-301 | R-1004 | 自己REJECTED | allow | 200/J03 |
| AC02-P | E-301 | R-1201 | 他人同部署PENDING | deny | 404/E404 |
| AC02-A | E-301 | R-1202 | 他人同部署APPROVED | deny | 404/E404 |
| AC02-R | E-301 | R-1203 | 他人同部署REJECTED | deny | 404/E404 |
| AC03-P | E-301 | R-1005 | 他人別部署PENDING | deny | 404/E404 |
| AC03-A | E-301 | R-1006 | 他人別部署APPROVED | deny | 404/E404 |
| AC03-R | E-301 | R-1007 | 他人別部署REJECTED | deny | 404/E404 |
| AC04-P | M-101 | R-1101 | 自己・部署不一致PENDING | allow | 200/J05 |
| AC04-A | M-101 | R-1102 | 自己・部署不一致APPROVED | allow | 200/J06 |
| AC04-R | M-101 | R-1103 | 自己・部署不一致REJECTED | allow | 200/J07 |
| AC05-P | M-101 | R-1001 | 他人同部署PENDING | allow | 200/J01 |
| AC05-A | M-101 | R-1003 | 他人同部署APPROVED | deny | 404/E404 |
| AC05-R | M-101 | R-1004 | 他人同部署REJECTED | deny | 404/E404 |
| AC06-P | M-101 | R-1005 | 他人別部署PENDING | deny | 404/E404 |
| AC06-A | M-101 | R-1006 | 他人別部署APPROVED | deny | 404/E404 |
| AC06-R | M-101 | R-1007 | 他人別部署REJECTED | deny | 404/E404 |

L1追加境界（通常D1へ不正行は保存しない）:

- AU-role-{ADMIN,UNKNOWN}-{self,other}: M-101のroleだけ置換。selfはR-1101、otherはR-1001。4ケースともdeny。未知Roleがowner分岐で許可されないことも確認。
- AU-tenant-self: E-301に対し、R-1001の対象tenantだけT2へ置換した合成入力。owner_idが一致してもdeny。
- AU-tenant-manager: M-101にT2/R-1001を合成入力。部署とPENDINGが一致してもdeny。

上記は認可関数の入力境界の試験。HTTPではユーザー状態を先に確認して403、別tenantはSELECTで0行404、未知Roleの取得行注入は§6.3の500。結果の分類が異なることは矛盾ではない。

## 4. HTTP・判定順・tenant・通常入口

U/E/Aはアプリ側のusersLookup/expensesLookup/authorize呼出し回数。fixture/snapshotのSQLやD1内部動作は含めない。実D1処理を委譲する観測ラッパー等で確認し、通常のL2成功経路をDBモックへ変えない。認証・scope・ID等の「未到達」も該当ケースで検査する。

### 4.1 旧H01〜H19の保持と展開

| ID | 層・入力/事前状態 | 期待HTTP/JSON・終了位置 |
|---|---|---|
| H01 | L2 E-301 GET R-1001 | AC01-Pと同じ200/J01、U/E/A=1/1/1 |
| H02 | L2 E-901 GET R-1001 | 200/J09、1/1/1。同ID別tenant・同subject別issuer |
| H03 | L2 M-101 GET R-1001 | AC05-Pと同じ200/J01、1/1/1 |
| H04 | L2 E-302 GET R-1001 | 404/E404、1/1/1 |
| H05-a/b/c | L2 M-101 GET R-1003 / R-1004 / R-1005 | AC05-A/AC05-R/AC06-P、各404/E404、1/1/1 |
| H06 | L2 E-301 GET R-9001 | 404/E404、1/1/0。T2の行を読んでから隠さない |
| H07 | L2 UNA GET R-123 | 401/AUTHENTICATION_REQUIRED、0/0/0。scope/ID検査なし |
| H08-a/b/c | L2 E-301、scopes=[] / ["expenses:read-extra"] / ["Expenses:read"]、GET R-1001 | 各403/INSUFFICIENT_SCOPE、0/0/0。後続なし |
| H09 | L2 E-399 GET R-123 | 403/USER_NOT_ELIGIBLE、1/0/0。ID検査なし |
| H10-a/b | L2 E-301 GET R-123 / R-1234567890123 | 各400/INVALID_EXPENSE_ID、1/0/0 |
| H10-c/d | L2 E-301 GET R-1234 / R-123456789012 | 各404/E404、1/1/0。4桁/12桁とも形式は有効、fixtureには不存在 |
| H11 | L2 SVC GET R-1001 | 403/PRINCIPAL_NOT_SUPPORTED、0/0/0。users未取得 |
| H12-{POST,PUT,PATCH,DELETE,OPTIONS,HEAD} | L2 E-301、各method、R-1001 | 各405/METHOD_NOT_ALLOWED、Allow: GET、0/0/0。HEADは本文0 |
| H12-bad-id | L2 E-301 POST R-123 | 405、0/0/0。IDを先に検査しない |
| H12-service | L2 SVC POST R-1001 | 405、0/0/0。kind確認よりmethodが先 |
| H12-inactive | L2 E-399 HEAD R-1001 | 405、本文0、0/0/0。users未取得 |
| H12-d1 | L2 E-301 HEAD R-1001、users faultを設定 | 405、本文0、fault到達0、0/0/0 |
| H13-a/b | L2 UNA HEAD R-123 / E-301 scopes=[] HEAD R-123 | 401/AUTHENTICATION_REQUIRED、403/INSUFFICIENT_SCOPE。各本文0、0/0/0 |
| H13-post-a/b | 上記H13のmethodだけPOST | 401/403でmethod判定より前に終了。error JSONあり |
| H14-{GET,HEAD,POST}-{health,unknown} | L2各method、/healthまたは/unknown、UNA | 404/ROUTE_NOT_FOUND、認証・U/E/Aすべて0。HEAD本文0 |
| H15 | 限定fault E-301 GET R-123、usersの可用性障害 | 503/DEPENDENCY_UNAVAILABLE、1/0/0。ID検査なし |
| H16 | 限定fault E-301 GET R-1001、usersは実D1、expensesの可用性障害 | 503/DEPENDENCY_UNAVAILABLE、1/1/0 |
| H17 | 限定fault E-301 GET R-1001、users境界で一般Error | 500/INTERNAL_ERROR、1/0/0、内部詳細なし |
| H18 | L3、通常入口の偽入力群（§4.3） | 保護routeはすべて401。未知routeは404 |
| H19 | L4-01〜L4-05（§7.3） | 200/200/404/503/503を個別に判定 |

code略記はすべて§2.3のE(code,rid)を表す。HEAD以外はJSON、HEADは本文0。同時にno-store、requestId、Allow/challengeの適用・非適用を仕様案§3に従い検査する。

### 4.2 追加の具体ケース

| ID | 入力・準備 | 期待 |
|---|---|---|
| H20-a/b | ABS GET R-1001 / R-123 | 各403/USER_NOT_ELIGIBLE、U/E/A=1/0/0、ID検査なし |
| H21-a | E-302＋X-Role=MANAGER、X-Department=SALES、X-User-Id=E-301、GET R-1001 | 404/E404、D1の現在値を上書きしない |
| H21-b | E-301＋X-Tenant-Id=T2、GET R-9001 | 404/E404。1/1/0 |
| H21-c | E-399＋X-Active=1、GET R-1001 | 403/USER_NOT_ELIGIBLE。1/0/0 |
| H21-d | E-301 GET /expenses/R-9001?tenant_id=T2&role=MANAGER | 404/E404。queryでtenant/roleを変更しない |
| H22 | E-301 GET R-9999（全tenantで不存在） | 404/E404。H04/H06と同じstatus/code/JSON形・ヘッダー方針 |
| H23-{01..05} | E-301 GETでID=%52-1001 / R%2D1001 / R-1001%2Fextra / %ZZ / R-1001%0A | 各400/INVALID_EXPENSE_ID、1/0/0（Request到達時） |
| H24-{01..05} | E-301 GET。順に `/expenses`、`/expenses/`、`/expenses/R-1001/`、`/expenses/R-1001/extra`、`/Expenses/R-1001` | 各404/ROUTE_NOT_FOUND、認証・U/E/Aすべて0 |
| H25-a/b | E-301 GET R-1001を2回、各入力X-Request-Idに合成ラベル | 各200/J01。サーバーID同士は異なり、入力を採用せず、header/終端logが一致 |
| H25-c | E-301 GET R-9999、入力X-Request-Idに合成ラベル | 404/E404。header/error JSON/終端logのrid一致 |
| H26 | E-301 scopes=["other","expenses:read"] GET R-1001 | 200/J01。必要scopeが完全一致の一要素なら許可 |
| H27-0/max | 下記金額追加fixture、E-301 GET R-8000 / R-8001 | 下記の正確な200 JSON。amountMinorはnumber・安全整数 |

H27の追加行は、当該ケースの開始前だけに準備する完全なfixture:

- `T1,R-8000,E-301,SALES,0,JPY,PENDING`
- `T1,R-8001,E-301,SALES,9007199254740991,JPY,PENDING`

期待JSON:

- H27-0: `{"id":"R-8000","ownerId":"E-301","amountMinor":0,"currency":"JPY","status":"PENDING"}`
- H27-max: `{"id":"R-8001","ownerId":"E-301","amountMinor":9007199254740991,"currency":"JPY","status":"PENDING"}`

GET直前は追加行を含む状態、GET後はその全業務データと一致。case後にbaselineへ戻す。

### 4.3 通常入口L3のH18展開

各入力群でGET/HEAD/POST `/expenses/R-1001` を実行する（H18-{none,bearer,headers,mixed}-{GET,HEAD,POST}）。

- none: Authorizationも主体headerもなし。
- bearer: `Authorization: Bearer SYNTH_AUTH_H18`。
- headers: `X-User-Id:E-301, X-Tenant-Id:T1, X-Role:MANAGER, X-Department:SALES, X-Active:1`。
- mixed: bearer＋headers。

すべて401、challenge、no-store、サーバーrequestId。HEAD本文0、その他はE(AUTHENTICATION_REQUIRED,rid)。U/E/A=0/0/0。保護routeの不正ID `R-123` でも同じ（H18-invalid）。`/health`と`/unknown`のGET/HEADは404/ROUTE_NOT_FOUND、認証未実行（H18-route-*）。

通常artifactの生成成功・401観測と、L4の認証後処理成功は別の証跡にする。

## 5. 現在権限の変更反映（L2・A4）

各CH開始前にbaselineへ戻す。シナリオ途中は同じapp/Worker・同じidentityを維持し、DB再seed・app再生成・module cache破棄をしない（CH06の主体切替のみ明示例外）。変更完了を待って次要求を始める。これは試験側の管理用SQLであり、更新API追加ではない。

| ID | 変更 | 前→後の具体的期待 |
|---|---|---|
| CH01 | M-101.role MANAGER→EMPLOYEE | R-1001:200/J01→404/E404。自己R-1101:200/J05維持 |
| CH02 | M-101.department SALES→DEV | R-1001:200/J01→404/E404。R-1005:404/E404→200/J04。自己R-1101/1102/1103は200/J05/J06/J07維持 |
| CH03 | M-101.active 1→0 | R-1001:200/J01→403/USER_NOT_ELIGIBLE。自己R-1101:200/J05→同403 |
| CH04-a | 所有者E-301.department SALES→DEV。同じviewer=M-101 | R-1001は200/J01維持。E-301所有3行のexpenses.departmentはSALESのまま、全expenses不変 |
| CH04-b | 独立baselineからE-301.department SALES→DEV。同じviewer=E-301 | 自己R-1001/1003/1004は全200/J01/J02/J03維持。E-301所有3行のexpenses.departmentはSALESのまま、全expenses不変 |
| CH05 | 所有申請なしM-102.tenant_id T1→T2 | R-1001:200/J01→200/J09。R-1201:200/J08→404/E404。R-9001:404/E404→200/J10 |
| CH06 | DB変更なし。同appでA/same-sub→B/same-sub→A/same-subへ切替 | R-1001:200/J01→200/J09→200/J01 |

各GET直前・直後は全users/expenses不変。CH01〜CH05の各シナリオ全体で許す差分は指定users1行の1列だけ。CH04-a/bは別々のbaselineから実施し、各シナリオ内でidentityを維持する。CH06は全体差分なし。CH04-aは所有者の現在部署を認可時に引き直していないことも確認する。

CH05は、申請を所有するuserのtenant変更をFKのRESTRICT案で拒否する試験と両立する。要求の途中で同時更新された権限まで一貫したsnapshotや即時取消を保証したとは記録しない。

## 6. local D1制約・tenant SQL・限定fault

### 6.1 migrationとSQL

- DB-M01: 新しい空local DBへ実migrationを適用し、仕様案§5の表・キー・制約が存在する。users/expensesは0行。
- DB-M02: migration管理コマンドを同じDBへ再実行し、適用済みmigrationを二重実行せず、schema/業務値が変わらない。生DDLを無条件に2回実行する試験と混同しない。
- DB-M03: 別の空local DBへ同じmigrationと§2のfixtureを適用し、同じ7 users/14 expensesを再構築できる。
- SQL-T01: 実際のexpenses取得SQLに現在tenantとidのbindがあり、H01/H02/H06を別々に実行する。H01はT1/E-301/12000、H02はT2/E-901/99000、H06はT1で0行。SQL文字列連結やtenantなしSELECTを合格にしない。
- SQL-I01: usersの条件がissuerとsubjectの両方をbindし、CH06で別人解決できる。subject単独検索/キャッシュを許可しない。
- DDL確認、FK作動、PK重複拒否、tenant限定SQLは別の観測欄。作成成功や1つの拒否結果だけで全部を合格にしない。

### 6.2 制約ケース

各ケースの初期状態はbaseline。正常INSERT素材:
Unew=`E-390,T1,A,e390,EMPLOYEE,SALES,1`（usersの列順）、
Xnew=`T1,R-7000,E-301,SALES,123,JPY,PENDING`（expensesの列順）。

INSERTでは指定した列以外をUnew/Xnewの全値で埋める。UPDATEでは指定した対象行の指定列だけを変更し、その他の列は対象行の現在値を保持する。UPDATEの通常対象はusers=M-102、expenses=T1/R-1001。特記した対象が優先。各パラメータ・INSERT(I)/UPDATE(U)は独立caseに展開し、例 `DF12-b-I-minus1` のように操作と値を識別する。失敗は例外/失敗結果だけでなく**全業務値・件数不変**。成功は指定行/列だけの差分（同値UPDATEなら値差分なし）で、他は不変。

| ID | 具体的操作 | 期待DB状態 |
|---|---|---|
| DF01 | Xnew INSERT | 1行追加、上記7列と保存型/値一致、users不変 |
| DF02-a | Xnewのid=R-1001でINSERT | T1/id複合PK重複で拒否・全不変 |
| DF02-b | R-7000をT1/E-301/123とT2/E-901/456で各INSERT。他列SALES/JPY/PENDING | 2行共存・両成功。同IDの全体一意へ強化しない |
| DF02-c | T1/R-1003.idをR-1001へUPDATE | 重複拒否・全不変 |
| DF03-a/b | Xnew.owner_idをE-901 / E-999にして各INSERT | 他tenant / 不存在owner、各FK拒否・全不変 |
| DF03-c1/c2 | T1/R-1001.owner_idをE-901 / E-999へ各UPDATE | 各FK拒否・全不変 |
| DF03-d | T1/R-1001.owner_idをE-303へUPDATE | 同tenantなので成功、当該owner_idだけ変更 |
| DF04-a | Unewのuser_id=E-301,tenant_id=T2,subject=new-userでINSERT | 全体user PK案では拒否・全不変 |
| DF04-b | 非所有者M-102.user_idをE-301へUPDATE | PK拒否・全不変 |
| DF05-a/b | Unewのissuer/subjectをA/same-subとしてINSERT / M-102のissuer/subjectをA/same-subへUPDATE | 各UNIQUE拒否・全不変 |
| DF05-c | A/same-sub=E-301とB/same-sub=E-901をbaselineとして保存 | 両方保存・別利用者解決 |
| DF06-a/b/c | 参照中E-301をDELETE / user_idをE-311へUPDATE / tenant_idをT2へUPDATE | 各RESTRICT拒否・全不変 |
| DF06-d | 非所有者M-102.tenant_idをT2へUPDATE | 成功、当該1列だけ変更 |
| DF06-e | 非所有者M-102をDELETE | 成功、そのusers1行だけ削除、expenses不変 |
| DF07-{table}-{column}-{I,U} | users全7列/expenses全7列を1列ずつNULLにする。IはUnew/Xnew、UはM-102/T1-R-1001 | 全28ケース拒否・全不変。PK列にもNOT NULL |
| DF08-{value}-{I,U} | role=EMPLOYEE,MANAGER,ADMIN,UNKNOWN,空文字 | 前2値成功、後3値拒否・全不変 |
| DF09-{value}-{I,U} | active=0,1,-1,2,0.5,"abc" | 0/1だけ成功、他は拒否・全不変 |
| DF10-{value}-{I,U} | status=PENDING,APPROVED,REJECTED,DRAFT,空文字 | 前3値成功、後2値拒否・全不変 |
| DF11-{value}-{I,U} | currency=JPY,USD,jpy,空文字 | JPYだけ成功、他は拒否・全不変 |
| DF12-a-{0,max}-{I,U} | amount_minor=0 / 9007199254740991 | 成功、保存型integer・値一致 |
| DF12-b-{minus1,fraction,over,nonnumber}-{I,U} | amount_minor=-1 / 0.5 / 9007199254740992 / "abc" | 各拒否・全不変 |
| DF12-c-{I,U} | amount_minorにSQL bound TEXTの"12000" | 保存値契約案ではinteger 12000への変換保存なら成功。文字列のまま保存は不可 |
| DF13 | users PK/両UNIQUE、expenses複合PK/FK、列順/照合規則、NOT NULL/CHECK/FK actionのmetadata照合 | 承認された具体要件に一致。FK enforcementをDF03/06でも確認 |

NULL拒否はDF07へ集約する。型affinityの変換とJSONのnumber型を分け、H27で安全整数のHTTP返却を確認する。主キー/UNIQUE/FK等が重なって拒否した場合、英語エラー文字列だけで制約一つの効果へ断定しない。users全体PKがある場合の複合UNIQUEはmetadataと有効な複合FK作動で確認する。

### 6.3 fault / 取得行破損 / HEAD finalizer

fault/componentのcase IDには使用層・差替箇所を記録。通常D1のCHECK/FKを外さない。Responseを直接503/500にするモックは受入不可。

| ID | 注入境界と具体値 | 期待 |
|---|---|---|
| FT-U503 | users依存境界で、根拠を定めた可用性障害をthrow | 共通分類→503、U/E/A=1/0/0 |
| FT-E503 | usersは実local D1、expenses依存境界だけ可用性障害 | 共通分類→503、1/1/0 |
| FT-500 | users依存境界で一般Error（合成ラベルをmessage/causeへ） | 500/INTERNAL_ERROR、1/0/0、秘密なし |
| ROW-U-{role,active,missing,multiple} | usersの正常結果をrole=ADMIN / active=2 / user_id欠落 / 2行へ各置換 | 各500/INTERNAL_ERROR、expenses/認可未実行 |
| ROW-E-{status,negative,fraction,over,missing,tenant,id} | 正常R-1001取得結果をstatus=DRAFT / amount_minor=-1 / amount_minor=0.5 / amount_minor=9007199254740992 / owner_id欠落 / tenant_id=T2 / id=R-1003へ各置換 | 各500/INTERNAL_ERROR、認可/成功JSON未実行 |
| AUTH-shape-{kind,issuer,scopes} | E-301の認証出力をkind=robot / issuer欠落 / scopesを文字列へ各置換し、GET R-1001 | 段階1で500/INTERNAL_ERROR、scope以降未実行、U/E/A=0/0/0 |
| AUTH-shape-post | E-301の認証出力のkind=robot、scopes=[]、POST R-1001 | 構造不正がscope/methodより先のため500、U/E/A=0/0/0 |
| FIN-HEAD-{200,400,401,403,404,405,500,503} | 各statusの合成ResponseをHEAD用finalizerへ渡すcomponent試験 | 各本文0、共通headers、該当するAllow/challengeを保持 |

FT-U503/H15とFT-E503/H16は同一実装観測を共有してよい。D1エラー表現と可用性分類の根拠を対象版の一次資料/取得コードへ結び付ける。根拠のある故障注入方式を決められなければBLOCKED。SQL構文ミス・table欠落を503用に代用しない。実D1/クラウド障害を発生させたとは呼ばない。

FIN-HEADの503等は合成試験であり、HTTP HEADでD1障害へ到達した実績ではない。HTTPで到達するHEADはH12/H13/H14/H18で別に確認する。

## 7. L4：通常条件・生成物・実行物（A5）

### 7.1 L3/L4実効条件の照合

| 項目 | 必須の一致・許容差分 |
|---|---|
| source | 同一code commit、共通HTTP/認可/DB/応答/finalizer/loggerの同一ソース |
| runtime | 実runtimeの版/識別子一致。「workerd」という名前だけで済ませない |
| compatibility_date | 明示した同一値 |
| flags | runner/plugin/生成設定による追加・上書きを含む実効値を比較 |
| D1 bindings | 同じbinding名・API型・schema/migration。別のlocal DB実体/保存先は許可。remoteなし |
| module resolution | export conditions、alias、polyfill、transform等を比較。L4共通コードにL2用global mockやVite変換を混入させない |
| build | 固定版bundler/lockfile、define/NODE_ENV、target/minify/module rules等の実効条件が同等 |
| launch | 実行entrypoint、artifact、実際に読んだ設定、コマンド、loopback bindを記録 |
| 許容する試験差分 | 認証注入、Worker名/port/local DB保存先、観測。障害ケースだけ依存境界faultを許可し別記 |

説明のない差分・観測できない実効値が残る場合、同等性は未確認。L4をPASSにしない。必要なら固定版runner/pluginの設定解決実装を読む。公式資料に記載がないことから追加flagがないと推定しない。

entrypoint/auxiliary Workerの具体方式は調査で選ぶ。通常共通コードをコピーして別実装する方法や、通常入口に認証/fault切替header・route・環境スイッチを追加する方法は採らない。

### 7.2 artifactとmock不在の対応

通常・検証用ごとに、code commit、entrypoint、config/lockfile識別、build command、全出力ファイル一覧とSHA-256、実行に指定されたファイルを対応付ける。追加module/chunk/存在するsource mapも検査する。

起動時に再bundle/transformされるなら、前段のhashだけでは実行対応の証拠にならない。実行された最終生成物を識別するか、事前生成物を再加工せず実行する方式を用い、根拠を記録する。通常と検証用は別artifactであり、同じhashは要求しない。

通常bundleにfake/faultがないことを、次の組合せで確認する。

1. 通常entrypointからの到達依存graph。
2. bundler入力・追加module一覧。
3. 全出力ファイル群の検査。
4. L3の偽入力拒否。

文字列検索だけで不在を証明しない。devDependencyがlockfileにあることと、通常artifactへ組み込まれることを分ける。

### 7.3 H19を展開したL4必須ケース

全ケースは本書§2のbaseline、仕様案§3のheadersと本書§9のログを使用し、設定同等性とartifact対応の証跡へ結び付ける。

| case | 入力/障害 | 期待HTTP/JSON・後続 |
|---|---|---|
| L4-01 | E-301 GET R-1001 | 200/J01、実D1と共通認可を実行 |
| L4-02 | M-101 GET R-1001 | 200/J01、他人同部署PENDINGの経路を実行 |
| L4-03 | E-302 GET R-1001 | 404/E404、認可拒否 |
| L4-04 | E-301 GET R-1001、users可用性fault | 503/DEPENDENCY_UNAVAILABLE、expenses/認可なし |
| L4-05 | E-301 GET R-1001、usersは実D1、expenses可用性fault | 503/DEPENDENCY_UNAVAILABLE、認可なし |

5ケースともGET前後DB不変性と必要ログを確認する。障害で観測不能なら§8に従う。L2の全組合せをL4へ全面複製することは求めない。L4成功から実Token暗号検証、実IdP、remote D1、クラウドの動作や可用性・性能を保証しない。

## 8. GET前後のDB不変性・初期化

独立ケースは migration/fixture準備 → 全業務表の直前snapshot → 要求 → 応答とアプリ処理の完了 → **後片付け前**の同DB snapshot → 比較 → 初期化/片付け。待機対象のアプリ非同期処理があるなら完了を待つ。GETから業務書込みを起動しない。

比較対象はusers/expensesの全行・全7列・件数（別tenantも含む）、順は安定したキー順。D1内部metadataやmigration管理表は別扱い。対象申請1行や件数だけの比較では不足。比較実体はlocalに留め、必要な非秘密の件数/hash/一致判定を証跡にする。

権限変更CHは、各GETの直前・直後を比較する。シナリオ途中でapp再生成・fixture復元をしない。変更1行1列は試験側の意図した差分として別欄にする。

障害で直後snapshotが取得できなければNOT_OBSERVED。fault解除後、再seed/DB再作成をせず同じDBを追観測できた場合は、時点・解除条件を明記して比較する。観測不能のままならDB不変性はPASSにせず、HTTP503との結果を分ける。

snapshot等値が保証するのは観測前後の状態一致。途中に一度も書込みがなかったという主張には、GET側SQLが読取だけであるコード/SQL確認と観測も組み合わせる。L2で後続呼出回数を、L3/L4で対象構成の実際の到達経路を確認する。

ケース間のstorage isolationは対象版で確認する。test fileごとの分離があっても、同じfile内の各caseが自動resetされるとは仮定しない。並列caseが同一DBを壊し合わないよう分離し、CH内は同一状態を継続する。

## 9. 必要ログと合成秘密ラベル（A5）

### 9.1 終端イベント案

アプリが応答を確定した要求ごとに `api_request_finished` を1件出す案。これはクライアント受領、DB commit、監査配送完了の証明ではない。

許可フィールドは `event, requestId, operation, stage, result, httpStatus`。operationは保護routeなら固定値 `expense.read`、未知routeなら `route.unmatched`。生URL/path/queryを操作名にしない。case IDとridの対応は試験側証跡に保持する。不要な利用者IDや対象詳細をログへ追加しない。

| 終了条件 | stage | result | httpStatus |
|---|---|---|---:|
| 成功 | response | success | 200 |
| 未認証 | authentication | denied | 401 |
| scope不足 | scope | denied | 403 |
| 非GET | method | denied | 405 |
| service | principal | denied | 403 |
| 未登録/無効user | user_eligibility | denied | 403 |
| 不正ID | id | invalid | 400 |
| tenant限定取得0行 | expense_lookup | not_found | 404 |
| 業務閲覧拒否 | authorization | denied | 404 |
| 未知route | route | not_found | 404 |
| users可用性障害 | user_lookup | unavailable | 503 |
| expenses可用性障害 | expense_lookup | unavailable | 503 |
| 一般例外/破損行/不正adapter出力 | internal | error | 500 |

HEADでも同じ終端段階を記録し、headerのridで対応する。ログ0件や二重終端イベントはFAIL。raw capture不能ならNOT_OBSERVED/BLOCKED。イベント名・enum・1件ルールはA5として2026-10-18に明示承認済み（仕様案§0.2）。文書一式の正式固定と実測は未実施。

### 9.2 合成ラベルの具体試験

ラベル形式は `SYNTH_<case>_<location>`。有効Tokenや実利用者情報は用いない。Authorization/Cookie/query/入力X-Request-Id/body/exception.message/causeへ場所別の異なるラベルを置く。テスト名や失敗メッセージに入力値を自動表示しない。

| ID | 層・入力 | 期待 |
|---|---|---|
| LG01 | L2/L4の成功E-301 GET R-1001。Authorization/Cookie/query/X-Request-Idにラベル | 200/J01、必要終端1件。全観測出力・応答にラベルなし、入力rid不採用 |
| LG02 | L3の偽Bearer GET R-1001＋Cookie/query/ridラベル | 401、安全なauthentication終端1件。生requestを出さない |
| LG03-a/b/c | L2。a=H08-a(E-301 scopes=[] GET R-1001)、b=H04(E-302 GET R-1001)、c=H14-GET-unknown(UNA GET /unknown)。それぞれ同じ入力位置にラベル | 403/404/404、対応stageの終端あり、ラベルなし |
| LG04 | L2認証/scope通過POST R-1001、bodyにSYNTH_LG04_BODY、他位置にもラベル | 405、D1未実行。bodyを出力しない |
| LG05-u/e/internal | E-301 GET R-1001。users/expensesはL2限定faultおよびL4-04/05、一般ErrorはFT-500。message/causeに別ラベル | 503/503/500。安全な終端あり、L4を含むconsole/stack/causeにもラベルなし |
| LG06 | L2 E-301 POST /expenses/R-1001、認証/scope通過。本文へ合成ラベル＋改行＋JSONに似た文字列、queryへpercent-encoded改行、入力ridへ送信可能な安全文字列 | 405。余分なログ行/項目にならず、終端1件、ラベルなし |

GET/HEADへbodyを付けたHTTPケースは作らない。合成ラベル入りのresponse/row由来値を追加する場合は、正規の5項目契約を変えず、限定fault試験と明示する。導入していない出力経路の成功を主張しない。

検査対象は、実際に使用するアプリlogger、Worker console、Frameworkの例外処理、Wrangler/Miniflare/Vitestによるstdout/stderr転送等。独自loggerだけを見ない。捕捉開始/終了、出力経路、設定、イベント件数、ラベル探索結果を記録する。

mask後の画面だけではなく、対象経路のraw観測から判定する。canary入り生出力はGitHub/通常ログへ転載せず、合否・該当経路・非秘密の要約を残す。runner失敗出力やclient verbose dumpが入力ラベルを漏らした場合、その経路を都合よく除外しない。出力方式を直すかFAIL/BLOCKEDとして残す。

「ラベル不在」と「必要イベント存在」を同時に要求する。使用していないCollector/Exporter/クラウド送信は対象外。すべての秘密漏えい不存在を証明したとはせず、測定した版・入力・経路を明記する。

## 10. 再現手順・証跡・後続判断

現時点で以下のスクリプトやコマンドは**未実装/未実行**。調査用コードのREADMEへ、調査で確定した実コマンド・作業directory・exit codeを記録する。架空の成功出力を書かない。

1. 承認済み仕様commit/範囲、調査code commit/branch、日時、OS/arch、Node/npm、直接/推移依存、engines/peer、lockfileを記録。公式metadataで再照合して評価版を固定する。
2. 固定lockfileの `npm ci` 等による再構築を実行し、peer無視や `@latest` を確定再現手順にしない。
3. local DBの保存先分離、空DB migration、再適用、baseline seed、別空DB再構築を確認。D1操作は--local明示、remote bindingなし、起動loopback。
4. 実装時に用意したlint/typecheck、L1、L2、DDL制約、通常bundle生成/L3、検証bundle生成/L4、fault、DB、ログを順に実行。各実コマンドの結果と未実行理由を記録。
5. 各caseに入力、全事前状態/fixture版、期待HTTP/JSON/DB、実測、呼出し・ログ、spec/code commit、実効設定、artifact hash、mock/faultの範囲、PASS/FAIL/BLOCKED/NOT_RUNを対応付ける。
6. 別の空local DBから同じ条件を再現し、fixture初期化や一回限りの手作業に依存する箇所を残す。
7. 別セッション・別モデルへ対象code commitと一次仕様/証跡を渡して独立レビュー。実装側/レビュー側の識別、指摘、対応、残課題、修正後の再確認を記録してからPRへ進む。

ローカルDB、.wrangler、.env/.dev.vars、Token/Cookie/Authorization、秘密鍵、Terraform State、実利用者データをGit/Issueへ含めない。合成ラベルの漏えい生出力も転載しない。`refs/tmp/`は変更しない。

| 判断 | 完了条件 |
|---|---|
| 仕様/受入条件の固定 | 対象commitと本人の明示承認、範囲・残課題の記録 |
| 調査報告 | 実施/未実施・根拠・再現手順・未確認を記録。未合格があっても報告はできる |
| 必須試験合格 | 対象caseと必須DB/log/設定/artifact観測がPASS |
| 技術採用 | 調査と独立レビューを踏まえてIssue #3で所有者が判断 |
| API本体への移行 | 採用判断とは分けて範囲を決め、コード再利用もレビュー |

**固定 → 追加調査Issue（未起票なら#1のnative sub-issue、#3参照） → 独立ブランチのローカル実測 → 別セッション・別モデルの独立レビュー → PR → Issue #3で所有者判断。**

現在はA1〜A5の個別承認を終え、2文書一式の正式固定対象を提示して固定作業・Issue反映の指示範囲を確認する段階。全体PoCや実IdP/BFF/Native、補助ツール導入を調査開始の追加必須ゲートにしない。仕様の正式固定、Issue反映・追加起票、コード実装は未実施であり、表記同期から自動実行しない。

### 一次資料（仕様レビューで参照・実測の代替ではない）

- [Cloudflare Vitest isolation/concurrency](https://developers.cloudflare.com/workers/testing/vitest-integration/isolation-and-concurrency/): テスト分離とrunner側の実行条件。対象版で実効値を確認。
- [Wrangler bundling](https://developers.cloudflare.com/workers/wrangler/bundling/): module/生成設定/ビルド条件と実行物の照合。
- [D1 local development](https://developers.cloudflare.com/d1/best-practices/local-development/): local/remoteと永続状態。
- [D1 debug](https://developers.cloudflare.com/d1/observability/debug-d1/): 可用性とSQL/型等のエラー分類の検討。
- [Hono App](https://hono.dev/docs/api/hono): 例外処理の適用範囲と出力。
- [Workers console](https://developers.cloudflare.com/workers/runtime-apis/console/): ローカル出力経路。
