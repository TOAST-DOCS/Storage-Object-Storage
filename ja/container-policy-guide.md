<!-- machine_translated: true -->

{% include-markdown '../_object-storage-vars.md' %}

<!-- pre-align:aligned sig=5dd7d08822ff -->

<a id="storage-object-storage-container-policy-configuration-guide"></a>
## Storage > Object Storage > コンテナポリシー設定ガイド { #storage-object-storage-container-policy-configuration-guide }

この文書はコンテナポリシーを使用して、NHN Cloud Object Storage のコンテナに関連する設定を管理する方法について説明します。

<a id="container-policy"></a>
## コンテナポリシー { #container-policy }

コンテナポリシーを使用すると、コンテナ設定をJSON形式のポリシードキュメントとして統合して管理できます。

ポリシードキュメントは次のように機能別の最上位キーで構成され、各最上位キーの下位に詳細設定を定義します。

```json
{
  "lifecycle": { ... },
  "acl": { ... },
  "ip_acl": { ... },
  "cors": { ... },
  "lock": { ... }
}
```

各トップレベルキーが担当する機能は次のとおりです。

| 最上位キー | 機能 | 説明 |
|---|---|---|
| `lifecycle` | ライフサイクル | オブジェクトのライフサイクルルールを設定します。 |
| `acl` | アクセス制御 (ACL) | コンテナの読み取り、書き込み、照会のアクセス許可を設定します。 |
| `ip_acl` | IP アクセス制御 (IP ACL) | IP ベースのアクセス制御を設定します。 |
| `cors` | クロスオリジンリソース共有 (CORS) | 許可オリジンなどの CORS 設定を管理します。 |
| `lock` | オブジェクトロック | オブジェクトロック (WORM) のロック期間を設定します。 |
<a id="container-policy-api"></a>
## コンテナポリシーAPI { #container-policy-api }

<a id="get-container-policy"></a>
### ポリシー照会 { #get-container-policy }

コンテナに設定されたポリシードキュメントを照会します。

```
GET /v1/{Account}/{Container}?policy
X-Auth-Token: {token-id}
```

<a id="get-container-policy-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Account | URL | String | Y | ストレージアカウント |
| Container | URL | String | Y | コンテナ名 |
| policy | Query | String | Y | ポリシー照会のためのクエリパラメータ<br>値を指定しない場合は全体のポリシードキュメントを、機能別の最上位キーを指定した場合は該当機能のポリシーのみ照会します。 |
<a id="get-container-policy-response"></a>
#### レスポンス

成功時、HTTPステータスコード`200`とともにJSON形式のポリシードキュメントを返却します。設定されたポリシーがない場合は空のJSONオブジェクトを返却します。

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

`policy` クエリパラメータに値を指定しない場合、設定されたすべての機能のポリシーを1つのドキュメントとして返します。

<details>
  <summary>全ポリシードキュメント照会リクエストおよびレスポンス例</summary>

```
GET /v1/{Account}/{Container}?policy
X-Auth-Token: {token-id}
```

```json
{
  "lifecycle": {
    "default_rule": { "days": 10, "action": { "type": "delete" } },
    "rules": [
      {
        "name": "rule1",
        "condition": { "prefix": "logs/" },
        "days": 5,
        "action": { "type": "transfer", "destination": "archive-container" }
      }
    ]
  },
  "acl": {
    "read": { "public": true, "grantees": [ { "tenant": "project-a", "user": "user-1" } ] }
  },
  "ip_acl": {
    "whitelist": [ { "permission": "read", "cidr": "10.0.0.0/24" } ],
    "services": [ { "name": "service_gateway", "permission": "read" } ]
  },
  "cors": {
    "allow_origins": [ "https://example.com" ],
    "max_age": 3600,
    "expose_headers": [ "ETag" ]
  },
  "lock": { "days": 30 }
}
```

</details>

<br>

`policy` クエリパラメータに値を指定すると、該当機能のポリシーのみを返します。設定されていない機能はレスポンスに含まれません。

<details>
  <summary>特定機能の照会リクエストおよびレスポンス例</summary>

```
GET /v1/{Account}/{Container}?policy=acl
X-Auth-Token: {token-id}
```

```json
{
  "acl": {
    "read": {
      "public": true,
      "grantees": [ { "tenant": "project-a", "user": "user-1" } ]
    }
  }
}
```

</details>

<br>

<a id="set-container-policy"></a>
### ポリシー設定 { #set-container-policy }

リクエスト本文にJSONポリシードキュメントを含めてコンテナポリシーを設定します。

ポリシーを設定する際には、次のルールが適用されます。

* 最上位キーは `lifecycle`、`acl`、`ip_acl`、`cors`、`lock` のみ使用できます。スキーマに定義されていないキーやフィールドを含めると、リクエストが拒否されます。
* ポリシードキュメントに含めた最上位キーは、送信した内容で全体が上書きされます。含めなかった最上位キーの設定はそのまま維持されます。最上位キーを送信する際に一部のサブ項目のみを含めた場合、含めなかった項目は解除されるため、維持したい項目は必ず一緒に送信する必要があります。

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
Content-Type: application/json
```

<a id="set-container-policy-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Content-Type | Header | String | Y | `application/json` |
| Account | URL | String | Y | ストレージアカウント |
| Container | URL | String | Y | コンテナ名 |
| - | Body | JSON | Y | 設定するポリシードキュメント |
<a id="set-container-policy-response"></a>
#### レスポンス

成功時、HTTPステータスコード`204`を返却します。レスポンス本文はありません。

!!! tip "ヒント"
    同じリクエストでヘッダーとポリシードキュメントを併用した場合、ポリシードキュメントが優先して適用されます。
    ポリシードキュメントがスキーマに一致しない場合、HTTP ステータスコード `400` とともにエラーの位置と理由を含むメッセージを返します。

<br>

<a id="lifecycle"></a>
## ライフサイクル { #lifecycle }

コンテナポリシードキュメントの`lifecycle`キーを使用してライフサイクルルールを設定します。
ライフサイクルには2つの種類のルールがあります。

| ルールの種類 | 名前 | 説明 |
|---|---|---|
| `default_rule` | 基本ルール | 全ての条件ルールに合致しないオブジェクトに適用されるルールです。 |
| `rules` | 条件ルール | 設定した条件に合致するオブジェクトに適用されるルールです。基本ルールより優先して適用されます。 |

<br>

<a id="lifecycle-schema"></a>
### JSONポリシードキュメントスキーマ { #lifecycle-schema }

ライフサイクルポリシードキュメントの構造は次のとおりです。

```json
{
  "lifecycle": {
    "default_rule": {
      "days": integer,
      "action": {
        "type": "transfer" | "delete",
        "destination": string
      }
    },
    "rules": [
      {
        "name": string,
        "condition": {
          "prefix": string
        },
        "days": integer,
        "action": {
          "type": "transfer" | "delete",
          "destination": string
        }
      }
    ]
  }
}
```

<a id="lifecycle-schema-field-descriptions"></a>
#### フィールドの説明

| フィールド | 形式 | 必須 | 説明 | 備考 |
|---|---|---|---|---|
| `default_rule` | Object | N | デフォルトルール | |
| `default_rule.days` | Integer | Y | オブジェクトのライフサイクル | 日単位、最大 36,500 日 |
| `default_rule.action.type` | Enum | Y | 有効期限アクションのタイプ | `"transfer"` (移動) または `"delete"` (削除) |
| `default_rule.action.destination` | String | Conditional | 有効期限切れ時にオブジェクトを移動する対象コンテナ名 | `type` が `"transfer"` の場合は必須 |
| `rules` | Array | N | 条件ルールリスト | 最大30個 |
| `rules[*].name` | String | Y | ルール名 | コンテナ内で重複することはできません。 |
| `rules[*].condition.prefix` | String | Y | オブジェクト名のプレフィックス条件 | 空文字列は許可されません。 |
| `rules[*].days` | Integer | Y | オブジェクトのライフサイクル | 日単位、最大 36,500 日 |
| `rules[*].action.type` | Enum | Y | 有効期限アクションのタイプ | `"transfer"` (移動) または `"delete"` (削除) |
| `rules[*].action.destination` | String | Conditional | 有効期限切れ時にオブジェクトを移動する宛先コンテナ名 | `type` が `"transfer"` の場合は必須 |
<br>

<a id="lifecycle-apply"></a>
### ライフサイクルルールの適用 { #lifecycle-apply }

<a id="lifecycle-apply-rule-priority"></a>
#### ルールの優先順位

1つのオブジェクトには、1つのライフサイクルルールのみが適用されます。ルールは以下の優先順位に従って適用されます。

| 優先順位 | ルール | 説明 |
|---|---|---|
| 1 | 条件ルール (`rules`) | 条件ルールの順序に従って確認し、最初に合致したルールを適用します。 |
| 2 | 基本ルール (`default_rule`) | 合致する条件ルールがない場合は、基本ルールを適用します。 |
| 3 | 削除 | オブジェクトの有効期限切れ時に適用できるルールがない場合は、削除されます。 |

<a id="lifecycle-apply-lifecycle"></a>
#### ライフサイクル

ライフサイクルは**オブジェクトのアップロード時点**でコンテナのポリシーを照会して適用します。

* オブジェクトをアップロードする時点のライフサイクルルールを評価し、合致するルールのライフサイクル(`days`)値に基づいてオブジェクトの有効期限切れ日時を設定します。
* その後、コンテナのライフサイクルルールが変更されても、すでにアップロードされたオブジェクトの有効期限切れ日時は自動的に更新されません。

<a id="lifecycle-apply-expiration-action"></a>
#### 有効期限切れ時の動作

有効期限切れ時の動作は、**オブジェクトの有効期限切れ時点**でコンテナのポリシーを照会して適用します。

* オブジェクトの有効期限切れ時点にライフサイクルルールを再評価し、合致するルールの有効期限切れ時の動作(`action`)に従って移動(`transfer`)または削除(`delete`)を実行します。
* 有効期限切れ時点にどのルールにも合致せず、基本ルールもない場合、オブジェクトは削除されます。

<a id="lifecycle-apply-application-example"></a>
#### 適用例

以下のようなライフサイクルルールが設定されていると仮定します。

```json
{
  "lifecycle": {
    "default_rule": {
      "days": 10,
      "action": { "type": "delete" }
    },
    "rules": [
      {
        "name": "rule1",
        "condition": { "prefix": "logs/" },
        "days": 5,
        "action": { "type": "transfer", "destination": "archive-container" }
      }
    ]
  }
}
```

* **オブジェクト`image/test.jpg`のアップロード**
    * `logs/`プレフィックス条件に合致しないため、基本ルールを適用
    * ライフサイクル10日を設定
* **アップロード後、`rule1`の条件を変更**
    * 条件を`"condition": { "prefix": "image/" }`に変更
* **オブジェクト`image/test.jpg`のライフサイクルが期限切れ**
    * 有効期限切れ時点にルールを再評価：`rule1`の`image/`プレフィックス条件に合致
    * 有効期限切れ時の動作：`archive-container`に移動
<br>

<a id="acl"></a>

## アクセス制御 (ACL) { #acl }

コンテナポリシードキュメントの `acl` キーを使用して、コンテナのアクセス権限を設定します。アクセス制御は、読み取り、書き込み、照会の 3 つの権限で構成されます。

| 権限 | キー | 説明 |
|---|---|---|
| 読み取り | `read` | コンテナ情報とオブジェクト情報の照会およびダウンロードを許可します。 |
| 書き込み | `write` | オブジェクトのアップロード、削除などの変更リクエストを許可します。 |
| 照会 | `view` | コンテナのオブジェクト一覧の照会を許可します。 |
ポリシードキュメントの `read`、`write`、`view` は、それぞれコンテナの `X-Container-Read`、`X-Container-Write`、`X-Container-View` 属性に対応します。各権限の詳細については、[アクセスポリシー設定ガイド](acl-guide$[ file_suffix ]$/#role-based-access-api)を参照してください。

<br>

<a id="acl-schema"></a>
### JSON ポリシードキュメントのスキーマ { #acl-schema }

アクセス制御ポリシードキュメントの構造は次のとおりです。

```json
{
  "acl": {
    "read": {
      "public": boolean,
      "listing": boolean,
      "referrers": {
        "allow": [ string ],
        "deny": [ string ]
      },
      "grantees": [
        { "tenant": string, "user": string }
      ]
    },
    "write": {
      "grantees": [
        { "tenant": string, "user": string }
      ]
    },
    "view": {
      "grantees": [
        { "tenant": string, "user": string }
      ]
    }
  }
}
```

<a id="acl-schema-field-descriptions"></a>

#### フィールドの説明

| フィールド | 形式 | 必須 | 説明 | 備考 |
|---|---|---|---|---|
| `read` | Object | N | 読み取りアクセス許可 | `grantees`、`referrers`（`allow`、`deny`）、`public`、`listing` を合計して最大 100 個 |
| `read.public` | Boolean | N | すべてのユーザーへの読み取りを許可 | `.r:*` ポリシー要素に対応し、認証トークンなしでのアクセスを許可します。 |
| `read.listing` | Boolean | N | オブジェクト一覧照会を許可 | `.rlistings` ポリシー要素に対応し、読み取りアクセス許可を持つユーザーに一覧照会を許可します。単独では設定できません。 |
| `read.referrers.allow` | Array | N | アクセスを許可するリファラー (HTTP Referer) ドメインリスト | 各項目は `.r:<referrer>` ポリシー要素に対応します。空文字列やワイルドカード `*` は許可されず、`,` を含むまたは `-` で始まることはできません。 |
| `read.referrers.deny` | Array | N | アクセスをブロックするリファラー (HTTP Referer) ドメインのリスト | 各項目は `.r:-<referrer>` ポリシー要素に対応します。空文字列またはワイルドカード `*` は許可されず、`,` を含めたり `-` で始めることはできません。 |
| `read.grantees` | Array | N | 読み取り権限を付与するユーザーリスト | `<tenant>:<user>` 形式のロールベースのアクセスポリシー要素に対応します。 |
| `read.grantees[*].tenant` | String | Y | テナント（プロジェクト）ID | ワイルドカード `*` を使用できます。空文字列は許可されておらず、値に `,` や `:` を含めることはできません。 |
| `read.grantees[*].user` | String | Y | APIユーザーID | ワイルドカード `*` を使用できます。空文字列は許可されておらず、値に `,` を含めることはできません。 |
| `write` | Object | N | 書き込み権限 | `grantees` 最大 100 個 |
| `write.grantees` | Array | N | 書き込み権限を付与するユーザーのリスト | `grantees` の形式は `read.grantees` と同じです。 |
| `view` | Object | N | 閲覧権限 | `grantees` 最大100個 |
| `view.grantees` | Array | N | 閲覧権限を付与するユーザーのリスト | `grantees` の形式は `read.grantees` と同じです。 |
!!! tip "ヒント"
    `public`、`listing`、`referrers` に対応する要素については、アクセスポリシー設定ガイドの[その他のアクセスポリシー要素](acl-guide$[ file_suffix ]$/#common-access-elements)を、`grantees` に対応する要素については[ロールベースのアクセスポリシー要素](acl-guide$[ file_suffix ]$/#role-based-access-elements)を参照してください。

<a id="acl-schema-application-example"></a>
#### 適用例

```json
{
  "acl": {
    "read": {
      "public": true,
      "listing": true,
      "grantees": [ { "tenant": "project-a", "user": "user-1" } ]
    },
    "write": {
      "grantees": [ { "tenant": "project-a", "user": "user-1" } ]
    }
  }
}
```

`view` を含めていないため、照会権限は無効になります。

<br>

<a id="ip-acl"></a>

## IP アクセス制御 (IP ACL) { #ip-acl }

コンテナポリシードキュメントの `ip_acl` キーを使用して、IP ベースのアクセス制御を設定します。ホワイトリストとブラックリストは同時に使用できません。両方設定した場合は、ホワイトリストのみが適用されます。その他の動作については、アクセスポリシー設定ガイドの [IP ベースのアクセスポリシー](acl-guide$[ file_suffix ]$/#ip-based-access-policies) を参照してください。

<br>

<a id="ip-acl-schema"></a>
### JSON ポリシードキュメントのスキーマ { #ip-acl-schema }

IP アクセス制御ポリシードキュメントの構造は次のとおりです。

```json
{
  "ip_acl": {
    "whitelist": [
      { "permission": "read" | "write" | "full_control", "cidr": string }
    ],
    "blacklist": [
      { "permission": "read" | "write" | "full_control", "cidr": string }
    ],
    "services": [
      { "name": "service_gateway", "permission": "read" | "write" | "full_control" | "deny" }
    ]
  }
}
```

<a id="ip-acl-schema-field-descriptions"></a>
#### フィールドの説明

| フィールド | 形式 | 必須 | 説明 | 備考 |
|---|---|---|---|---|
| `whitelist` | Array | N | アクセスを許可する IP リスト | 最大 100 件 |
| `whitelist[*].permission` | Enum | Y | ルールを適用するアクション種別 | `"read"`, `"write"`, `"full_control"` |
| `whitelist[*].cidr` | String | Y | IPv4 アドレスまたは CIDR | 空の文字列は許可されません。 |
| `blacklist` | Array | N | アクセスをブロックする IP リスト | 最大 100 件 |
| `blacklist[*].permission` | Enum | Y | ルールを適用するオペレーションタイプ | `"read"`, `"write"`, `"full_control"` |
| `blacklist[*].cidr` | String | Y | IPv4アドレスまたはCIDR | 空の文字列は許可されません。 |
| `services` | Array | N | サービスごとのアクセス制御 | |
| `services[*].name` | Enum | Y | サービス名 | 現在は `service_gateway` のみサポートしています。 |
| `services[*].permission` | Enum | Y | 許可またはブロックする操作の種類 | `"read"`, `"write"`, `"full_control"`, `"deny"` |
!!! tip "ヒント"
    `cidr`には、IPv4 アドレスまたは CIDR を入力します。IP ベースのアクセスポリシーは IPv4 のみをサポートします。

<a id="ip-acl-schema-application-example"></a>
#### 適用例

```json
{
  "ip_acl": {
    "whitelist": [
      { "permission": "read", "cidr": "10.0.0.0/24" },
      { "permission": "full_control", "cidr": "192.168.0.1" }
    ],
    "services": [
      { "name": "service_gateway", "permission": "read" }
    ]
  }
}
```

<br>

<a id="cors"></a>

## Cross-Origin Resource Sharing (CORS) { #cors }

コンテナポリシードキュメントの `cors` キーを使用して、クロスオリジンリソース共有 (CORS) を設定します。CORS 設定および許可オリジンの形式の詳細については、API ガイドの[クロスオリジンリソース共有 (CORS)](api-guide$[ file_suffix ]$/#set-container-cors-policy) を参照してください。

<br>

<a id="cors-schema"></a>
### JSON ポリシードキュメントスキーマ { #cors-schema }

CORS ポリシードキュメントの構造は次のとおりです。

```json
{
  "cors": {
    "allow_origins": [ string ],
    "max_age": integer,
    "expose_headers": [ string ]
  }
}
```

<a id="cors-schema-field-descriptions"></a>
#### フィールドの説明

| フィールド | 形式 | 必須 | 説明 | 備考 |
|---|---|---|---|---|
| `allow_origins` | Array | N | 許可するオリジン (Origin) のリスト | 最大 100 件<br>各項目にスペースを含めることはできません。 |
| `max_age` | Integer | N | プリフライトレスポンスのキャッシュ時間 | 秒単位、0以上の整数 |
| `expose_headers` | Array | N | ブラウザに公開するレスポンスヘッダーのリスト | 最大 100 個<br>各項目に空白を含めることはできません。 |
<a id="cors-schema-application-example"></a>
#### 適用例

```json
{
  "cors": {
    "allow_origins": [ "https://example.com", "https://app.example.com" ],
    "max_age": 3600,
    "expose_headers": [ "ETag", "X-Timestamp" ]
  }
}
```

<br>

<a id="lock"></a>

## オブジェクトロック { #lock }

コンテナポリシードキュメントの `lock` キーを使用して、オブジェクトロック (WORM、Write-Once-Read-Many) のロック周期を設定します。オブジェクトロックの概念と制約については、APIガイドの[オブジェクトロック期間の変更](api-guide$[ file_suffix ]$/#set-container-object-lock-cycle)を参照してください。

<br>

<a id="lock-schema"></a>
### JSON ポリシードキュメントスキーマ { #lock-schema }

オブジェクトロックポリシードキュメントの構造は次のとおりです。

```json
{
  "lock": {
    "days": integer
  }
}
```

<a id="lock-schema-field-descriptions"></a>
#### フィールドの説明

| フィールド | 形式 | 必須 | 説明 | 備考 |
|---|---|---|---|---|
| `days` | Integer | Y | オブジェクトロック期間 | 日単位、0〜36,500日（最大100年） |
<a id="lock-schema-application-rules"></a>
#### 適用ルール

オブジェクトロックを設定する際は、次の点に注意してください。

* オブジェクトロックは、コンテナを作成するときにのみ有効化できます。オブジェクトロックが設定されていない既存のコンテナに `lock` を設定するリクエストは拒否されます。
* オブジェクトロックは無効化できません。`days` を `0` に設定すると、デフォルトのロック期間が 0 日になるだけで、オブジェクトロックは有効な状態のまま維持されます。

<a id="lock-schema-application-example"></a>
#### 適用例

```json
{
  "lock": {
    "days": 30
  }
}
```
s
