## Storage > Object Storage > コンテナポリシー設定ガイド

<a id="container-policy"></a>
### コンテナポリシー

コンテナポリシーを使用すると、コンテナ設定をJSON形式のポリシードキュメントとして統合して管理できます。

ポリシードキュメントは次のように機能別の最上位キーで構成され、各最上位キーの下位に詳細設定を定義します。

```json
{
  "lifecycle": {
    ...
  },
  "lock": {
    ...
  },
  "ip_acl": {
    ...
  }
}
```

> [参考]
> 2026年5月時点では**ライフサイクル**設定のみをサポートしており、今後はより多くの機能に拡大適用される予定です。

<a id="container-policy-api"></a>
## コンテナポリシーAPI

<a id="get-container-policy"></a>
### ポリシー照会

コンテナに設定されたポリシードキュメントを照会します。

```
GET /v1/{Account}/{Container}?policy
X-Auth-Token: {token-id}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ストレージアカウント |
| Container | URL | String | O | コンテナ名 |
| policy | Query | - | O | ポリシー照会のためのクエリパラメータ (値なしで使用) |

#### レスポンス

成功時、HTTPステータスコード`200`とともにJSON形式のポリシードキュメントを返却します。設定されたポリシーがない場合は空のJSONオブジェクトを返却します。

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

<details>
  <summary>レスポンスの例</summary>

```json
{
  "lifecycle": {
    "default_rule" : {
      "action": {
        "type": "transfer",
        "destination": "destination-con"
      },
      "days": 1
    },
    "rules": [
      {
        "name": "rule1",
        "condition": {
          "prefix": "temp/"
        },
        "days": 2,
        "action": {
          "type": "delete"
        }
      }
    ]
  }
}
```

</details>

<br/>

<a id="set-container-policy"></a>
### ポリシー設定

リクエスト本文にJSONポリシードキュメントを含めてコンテナポリシーを設定します。

ポリシーはリクエストに含まれる最上位キー(例: lifecycle)単位で上書きされ、含まれていないキーの設定は維持されます。

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
Content-Type: application/json
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Content-Type | Header | String | O | `application/json`|
| Account | URL | String | O | ストレージアカウント |
| Container | URL | String | O | コンテナ名 |
| - | Body | JSON | O | 設定するポリシードキュメント |

#### レスポンス

成功時、HTTPステータスコード`204`を返却します。レスポンス本文はありません。

> [参考]
> 同じリクエストでヘッダとポリシードキュメントを一緒に使用すると、ポリシードキュメントが優先して適用されます。

<br/>

<a id="lifecycle"></a>
## ライフサイクル

コンテナポリシードキュメントの`lifecycle`キーを使用してライフサイクルルールを設定します。
ライフサイクルには2つの種類のルールがあります。

| ルールの種類 | 名前 | 説明 |
|---|---|---|
| `default_rule` | 基本ルール | 全ての条件ルールに合致しないオブジェクトに適用されるルールです。 |
| `rules` | 条件ルール | 設定した条件に合致するオブジェクトに適用されるルールです。基本ルールより優先して適用されます。 |

<br/>

<a id="lifecycle-schema"></a>
### JSONポリシードキュメントスキーマ

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

#### フィールドの説明

| フィールド | 形式 | 必須 | 説明 | 備考 |
|---|---|---|---|---|
| `default_rule` | Object | - | 基本ルール | 省略したり空のオブジェクト(`{}`)に設定すると、基本ルールが解除されます。 |
| `default_rule.days` | Integer | O | オブジェクトのライフサイクル | 日単位、最大36,500日 |
| `default_rule.action.type` | Enum | O | 有効期限切れ時の動作タイプ | `"transfer"` (移動) または `"delete"` (削除) |
| `default_rule.action.destination` | String | - | 有効期限切れ時にオブジェクトを移動する対象コンテナ名 | `type`が`"transfer"`の場合は必須 |
| `rules` | Array | - | 条件ルール一覧 | 省略したり空の配列(`[]`)に設定すると、条件ルールが全て削除されます。 |
| `rules[*].name` | String | O | ルール名 | コンテナ内で重複することはできません。 |
| `rules[*].condition.prefix` | String | O | オブジェクト名のプレフィックス条件 | 空の文字列は許可されません。 |
| `rules[*].days` | Integer | O | オブジェクトのライフサイクル | 日単位、最大36,500日 |
| `rules[*].action.type` | Enum | O | 有効期限切れ時の動作タイプ | `"transfer"` (移動) または `"delete"` (削除) |
| `rules[*].action.destination` | String | - | 有効期限切れ時にオブジェクトを移動する対象コンテナ名 | `type`が`"transfer"`の場合は必須 |

<br/>

<a id="lifecycle-apply"></a>
### ライフサイクルルールの適用

#### ルールの優先順位

1つのオブジェクトには、1つのライフサイクルルールのみが適用されます。ルールは以下の優先順位に従って適用されます。

| 優先順位 | ルール | 説明 |
|---|---|---|
| 1 | 条件ルール (`rules`) | 条件ルールの順序に従って確認し、最初に合致したルールを適用します。 |
| 2 | 基本ルール (`default_rule`) | 合致する条件ルールがない場合は、基本ルールを適用します。 |
| 3 | 削除 | オブジェクトの有効期限切れ時に適用できるルールがない場合は、削除されます。 |

#### ライフサイクル

ライフサイクルは**オブジェクトのアップロード時点**でコンテナのポリシーを照会して適用します。

* オブジェクトをアップロードする時点のライフサイクルルールを評価し、合致するルールのライフサイクル(`days`)値に基づいてオブジェクトの有効期限切れ日時を設定します。
* その後、コンテナのライフサイクルルールが変更されても、すでにアップロードされたオブジェクトの有効期限切れ日時は自動的に更新されません。

#### 有効期限切れ時の動作

有効期限切れ時の動作は、**オブジェクトの有効期限切れ時点**でコンテナのポリシーを照会して適用します。

* オブジェクトの有効期限切れ時点にライフサイクルルールを再評価し、合致するルールの有効期限切れ時の動作(`action`)に従って移動(`transfer`)または削除(`delete`)を実行します。
* 有効期限切れ時点にどのルールにも合致せず、基本ルールもない場合、オブジェクトは削除されます。

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
