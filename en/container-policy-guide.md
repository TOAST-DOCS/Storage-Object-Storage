## Storage > Object Storage > Container Policy Configuration Guide

<a id="container-policy"></a>
### Container policy

Container policies allow you to manage container settings in an integrated manner using a JSON-format policy document.

A policy document is structured with top-level keys for each feature, as shown below. Detailed settings are defined under each top-level key.

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

> [Note]
> As of May 2026, only the **lifecycle** setting is supported. Support will be expanded to more features in the future.

<a id="container-policy-api"></a>
## Container policy API

<a id="get-container-policy"></a>
### Get policy

Retrieves the policy document configured for a container.

```
GET /v1/{Account}/{Container}?policy
X-Auth-Token: {token-id}
```

#### Request

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account |
| Container | URL | String | O | Container name |
| policy | Query | - | O | Query parameter for policy retrieval (used without a value) |

#### Response

On success, returns HTTP status code `200` along with the policy document in JSON format. If no policy is configured, an empty JSON object is returned.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

<details>
  <summary>Response example</summary>

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
### Set policy

Configures a container policy by including a JSON policy document in the request body.

The policy is overwritten per top-level key (e.g., lifecycle) included in the request. Settings for keys not included in the request are retained.

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
Content-Type: application/json
```

#### Request

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Content-Type | Header | String | O | `application/json` |
| Account | URL | String | O | Storage account |
| Container | URL | String | O | Container name |
| - | Body | JSON | O | Policy document to configure |

#### Response

On success, returns HTTP status code `204`. There is no response body.

> [Note]
> If a header and a policy document are used together in the same request, the policy document takes precedence.

<br/>

<a id="lifecycle"></a>
## Lifecycle

Lifecycle rules are configured using the `lifecycle` key in the container policy document.
There are two types of lifecycle rules.

| Rule type | Name | Description |
|---|---|---|
| `default_rule` | Default rule | A rule applied to objects that do not match any conditional rule. |
| `rules` | Conditional rule | A rule applied to objects that match the configured conditions. Takes precedence over the default rule. |

<br/>

<a id="lifecycle-schema"></a>
### JSON policy document schema

The structure of the lifecycle policy document is as follows.

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

#### Field descriptions

| Field | Type | Required | Description | Notes |
|---|---|---|---|---|
| `default_rule` | Object | - | Default rule | If omitted or set to an empty object (`{}`), the default rule is disabled. |
| `default_rule.days` | Integer | O | Object lifecycle | In days; maximum 36,500 days |
| `default_rule.action.type` | Enum | O | Expiration action type | `"transfer"` (move) or `"delete"` (delete) |
| `default_rule.action.destination` | String | - | Name of the target container to move the object to upon expiration | Required when `type` is `"transfer"` |
| `rules` | Array | - | List of conditional rules | If omitted or set to an empty array (`[]`), all conditional rules are deleted. |
| `rules[*].name` | String | O | Rule name | Must be unique within the container. |
| `rules[*].condition.prefix` | String | O | Prefix condition for the object name | Empty strings are not allowed. |
| `rules[*].days` | Integer | O | Object lifecycle | In days; maximum 36,500 days |
| `rules[*].action.type` | Enum | O | Expiration action type | `"transfer"` (move) or `"delete"` (delete) |
| `rules[*].action.destination` | String | - | Name of the target container to move the object to upon expiration | Required when `type` is `"transfer"` |

<br/>

<a id="lifecycle-apply"></a>
### Applying lifecycle rules

#### Rule priority

Only one lifecycle rule is applied to each object. Rules are applied according to the following priority order.

| Priority | Rule | Description |
|---|---|---|
| 1 | Conditional rule (`rules`) | Conditional rules are checked in order, and the first matching rule is applied. |
| 2 | Default rule (`default_rule`) | If no matching conditional rule exists, the default rule is applied. |
| 3 | Delete | If no applicable rule exists when the object expires, the object is deleted. |

#### Lifecycle

The lifecycle is determined by querying the container policy **at the time of object upload**.

* The lifecycle rules are evaluated at the time of upload, and the object expiration time is set based on the `days` value of the matching rule.
* If the lifecycle rules of the container are changed after upload, the expiration time of already uploaded objects is not automatically updated.

#### Expiration action

The expiration action is determined by querying the container policy **at the time of object expiration**.

* The lifecycle rules are re-evaluated at the time of expiration, and the object is moved (`transfer`) or deleted (`delete`) according to the expiration action (`action`) of the matching rule.
* If no rule matches and there is no default rule at the time of expiration, the object is deleted.

#### Application example

Assume the following lifecycle rules are configured.

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

* **Upload object `image/test.jpg`**
    * Does not match the `logs/` prefix condition, so the default rule is applied.
    * Lifecycle set to 10 days.
* **After upload, the condition for `rule1` is changed**
    * Condition is changed to `"condition": { "prefix": "image/" }`.
* **Object `image/test.jpg` lifecycle expires**
    * Rules are re-evaluated at expiration: matches the `image/` prefix condition of `rule1`.
    * Expiration action: move to `archive-container`.