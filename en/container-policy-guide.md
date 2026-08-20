<!-- machine_translated: true -->

{% include-markdown '../_object-storage-vars.md' %}

<!-- pre-align:aligned sig=5dd7d08822ff -->

<a id="storage-object-storage-container-policy-configuration-guide"></a>
## Storage > Object Storage > Container Policy Configuration Guide { #storage-object-storage-container-policy-configuration-guide }

This document describes how to manage container-related settings in NHN Cloud Object Storage using container policies.

<a id="container-policy"></a>
## Container Policy { #container-policy }

Container policies allow you to manage container settings in an integrated manner using a JSON-format policy document.

A policy document is structured with top-level keys for each feature, as shown below. Detailed settings are defined under each top-level key.

```json
{
  "lifecycle": { ... },
  "acl": { ... },
  "ip_acl": { ... },
  "cors": { ... },
  "lock": { ... }
}
```

The functions of each top-level key are as follows.

| Top-level key | Function | Description |
|---|---|---|
| `lifecycle` | Life cycle | Sets the life cycle rules of objects. |
| `acl` | Access control (ACLs) | Sets the read, write, and view access permissions for the container. |
| `ip_acl` | IP access control (IP ACL) | Sets IP-based access control. |
| `cors` | Cross-Origin Resource Sharing (CORS) | Manages CORS settings such as allowed origins. |
| `lock` | Object lock | Sets the object lock (WORM) lock cycle. |
<a id="container-policy-api"></a>
## Container Policy API { #container-policy-api }

<a id="get-container-policy"></a>
### Get policy { #get-container-policy }

Retrieves the policy document configured for a container.

```
GET /v1/{Account}/{Container}?policy
X-Auth-Token: {token-id}
```

<a id="get-container-policy-request"></a>
#### Request

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage Account |
| Container | URL | String | Y | Container name |
| policy | Query | String | Y | Query parameter for retrieving a policy<br>If no value is specified, the entire policy document is retrieved. If a top-level key for a specific feature is specified, only the policy for that feature is retrieved. |
<a id="get-container-policy-response"></a>
#### Response

On success, returns HTTP status code `200` along with the policy document in JSON format. If no policy is configured, an empty JSON object is returned.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

If no value is specified for the `policy` query parameter, the policies of all configured features are returned as a single document.

<details>
  <summary>Request and Response Example for Retrieving All Policy Documents</summary>

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

If you specify a value in the `policy` query parameter, only the policy for that feature is returned. Features that are not configured are not included in the response.

<details>
  <summary>Request and response examples for querying specific features</summary>

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
### Set policy { #set-container-policy }

Configures a container policy by including a JSON policy document in the request body.

The following rules apply when you set a policy.

* Only `lifecycle`, `acl`, `ip_acl`, `cors`, and `lock` can be used as top-level keys. If you include keys or fields that are not defined in the schema, the request will be rejected.
* Top-level keys included in the policy document will overwrite the entire existing configuration with the sent content, while the settings of top-level keys not included will remain unchanged. If you send a top-level key with only some sub-items, any sub-items that are not included will be removed, so you must always include all items that you want to retain.

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
Content-Type: application/json
```

<a id="set-container-policy-request"></a>
#### Request

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Content-Type | Header | String | Y | `application/json` |
| Account | URL | String | Y | Storage Account |
| Container | URL | String | Y | Container name |
| - | Body | JSON | Y | Policy document to set |
<a id="set-container-policy-response"></a>
#### Response

On success, returns HTTP status code `204`. There is no response body.

!!! tip "Tip"
    If you use both a header and a policy document in the same request, the policy document takes precedence.
    If the policy document does not conform to the schema, it returns HTTP status code `400` along with a message containing the error location and reason.

<br>

<a id="lifecycle"></a>
## Lifecycle { #lifecycle }

Lifecycle rules are configured using the `lifecycle` key in the container policy document.
There are two types of lifecycle rules.

| Rule type | Name | Description |
|---|---|---|
| `default_rule` | Default rule | A rule applied to objects that do not match any conditional rule. |
| `rules` | Conditional rule | A rule applied to objects that match the configured conditions. Takes precedence over the default rule. |

<br>

<a id="lifecycle-schema"></a>
### JSON policy document schema { #lifecycle-schema }

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

<a id="lifecycle-schema-field-descriptions"></a>
#### Field descriptions

| Field | Type | Required | Description | Notes |
|---|---|---|---|---|
| `default_rule` | Object | N | Default rule | |
| `default_rule.days` | Integer | Y | Object life cycle | In days, up to 36,500 days |
| `default_rule.action.type` | Enum | Y | Expiration behavior type | `"transfer"` (move) or `"delete"` (delete) |
| `default_rule.action.destination` | String | Conditional | Name of the target container to move the object to when it expires | Required when `type` is `"transfer"` |
| `rules` | Array | N | List of condition rules | Max. 30 items |
| `rules[*].name` | String | Y | Rule name | Cannot be duplicated within the container. |
| `rules[*].condition.prefix` | String | Y | Prefix condition for the object name | Empty strings are not allowed. |
| `rules[*].days` | Integer | Y | Object life cycle | In days, up to 36,500 days |
| `rules[*].action.type` | Enum | Y | Expiration behavior type | `"transfer"` (transfer) or `"delete"` (delete) |
| `rules[*].action.destination` | String | Conditional | Target container name to move the object to when it expires | Required when `type` is `"transfer"` |
<br>

<a id="lifecycle-apply"></a>
### Applying lifecycle rules { #lifecycle-apply }

<a id="lifecycle-apply-rule-priority"></a>
#### Rule priority

Only one lifecycle rule is applied to each object. Rules are applied according to the following priority order.

| Priority | Rule | Description |
|---|---|---|
| 1 | Conditional rule (`rules`) | Conditional rules are checked in order, and the first matching rule is applied. |
| 2 | Default rule (`default_rule`) | If no matching conditional rule exists, the default rule is applied. |
| 3 | Delete | If no applicable rule exists when the object expires, the object is deleted. |

<a id="lifecycle-apply-lifecycle"></a>
#### Lifecycle

The lifecycle is determined by querying the container policy **at the time of object upload**.

* The lifecycle rules are evaluated at the time of upload, and the object expiration time is set based on the `days` value of the matching rule.
* If the lifecycle rules of the container are changed after upload, the expiration time of already uploaded objects is not automatically updated.

<a id="lifecycle-apply-expiration-action"></a>
#### Expiration action

The expiration action is determined by querying the container policy **at the time of object expiration**.

* The lifecycle rules are re-evaluated at the time of expiration, and the object is moved (`transfer`) or deleted (`delete`) according to the expiration action (`action`) of the matching rule.
* If no rule matches and there is no default rule at the time of expiration, the object is deleted.

<a id="lifecycle-apply-application-example"></a>
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
    * Expiration action: move to `archive-container`.<br>

<a id="acl"></a>
## Access control (ACLs) { #acl }

Use the `acl` key of the container policy document to configure container access permissions. Access control consists of three types of permissions: read, write, and get.

| Permission | Key | Description |
|---|---|---|
| Read | `read` | Allows querying container information and object information, and downloading objects. |
| Write | `write` | Allows change requests such as uploading and deleting objects. |
| View | `view` | Allows listing objects in the container. |
The `read`, `write`, and `view` fields in the policy document correspond to the `X-Container-Read`, `X-Container-Write`, and `X-Container-View` properties of the container, respectively. For more details on each permission, refer to the [Access Policy Configuration Guide](acl-guide$[ file_suffix ]$/#role-based-access-api).

<br>

<a id="acl-schema"></a>
### JSON Policy Document Schema { #acl-schema }

The structure of the access control policy document is as follows.

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
#### Field Description

| Field | Format | Required | Description | Note |
|---|---|---|---|---|
| `read` | Object | N | Read permission | Maximum 100 items combined across `grantees`, `referrers` (`allow`, `deny`), `public`, and `listing` |
| `read.public` | Boolean | N | Allow read for all users | Corresponds to the `.r:*` policy element, and allows access without an authentication token. |
| `read.listing` | Boolean | N | Allow object list query | Corresponds to the `.rlistings` policy element and allows users with read permission to query the object list. This policy element cannot be set alone. |
| `read.referrers.allow` | Array | N | List of referer (HTTP Referer) domains to allow access | Each item corresponds to a `.r:<referrer>` policy element. Empty strings and the wildcard `*` are not allowed, and values cannot contain `,` or start with `-`. |
| `read.referrers.deny` | Array | N | List of referer (HTTP Referer) domains to block access | Each item corresponds to the `.r:-<referrer>` policy element. Empty strings and the wildcard `*` are not allowed, and values cannot contain `,` or start with `-`. |
| `read.grantees` | Array | N | List of users to grant read permission | Corresponds to role-based access policy elements in the format `<tenant>:<user>`. |
| `read.grantees[*].tenant` | String | Y | Tenant (project) ID | You can use the wildcard `*`. Empty strings are not allowed, and the value cannot contain `,` or `:`. |
| `read.grantees[*].user` | String | Y | API User ID | You can use the wildcard `*`. Empty strings are not allowed, and the value cannot contain `,`. |
| `write` | Object | N | Write permission | Max. 100 items for `grantees` |
| `write.grantees` | Array | N | List of users to grant write permission to | The format of `grantees` is the same as `read.grantees`. |
| `view` | Object | N | View permission | Max. 100 items for `grantees` |
| `view.grantees` | Array | N | List of users to grant view permission to | The format of `grantees` is the same as `read.grantees`. |
!!! tip "Tip"
    For elements corresponding to `public`, `listing`, and `referrers`, see [Other Access Policy Elements](acl-guide$[ file_suffix ]$/#common-access-elements) in the Access Policy Configuration Guide. For elements corresponding to `grantees`, see [Role-Based Access Policy Elements](acl-guide$[ file_suffix ]$/#role-based-access-elements).

<a id="acl-schema-application-example"></a>
#### Application Example

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

Since `view` was not included, the view permission will be revoked.

<br>

<a id="ip-acl"></a>
## IP Access Control (IP ACL) { #ip-acl }

Use the `ip_acl` key in the container policy document to configure IP-based access control. A whitelist and a blacklist cannot be used at the same time; if both are set, only the whitelist is applied. For other details on how this works, see [IP-based access policies](acl-guide$[ file_suffix ]$/#ip-based-access-policies) in the Access Policy Configuration Guide.

<br>

<a id="ip-acl-schema"></a>
### JSON Policy Document Schema { #ip-acl-schema }

The structure of the IP access control policy document is as follows.

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
#### Field Description

| Field | Format | Required | Description | Note |
|---|---|---|---|---|
| `whitelist` | Array | N | List of IP addresses to allow access | Max. 100 items |
| `whitelist[*].permission` | Enum | Y | Type of task to apply the rule to | `"read"`, `"write"`, `"full_control"` |
| `whitelist[*].cidr` | String | Y | IPv4 address or CIDR | Empty strings are not allowed. |
| `blacklist` | Array | N | List of IPs to block access | Max. 100 items |
| `blacklist[*].permission` | Enum | Y | Type of task to apply the rule to | `"read"`, `"write"`, `"full_control"` |
| `blacklist[*].cidr` | String | Y | IPv4 address or CIDR | Empty strings are not allowed. |
| `services` | Array | N | Access control per service | |
| `services[*].name` | Enum | Y | Service Name | Currently, only `service_gateway` is supported. |
| `services[*].permission` | Enum | Y | Type of task to allow or deny | `"read"`, `"write"`, `"full_control"`, `"deny"` |
!!! tip "Tip"
    Enter an IPv4 address or CIDR in `cidr`. IP-based access policies support only IPv4.

<a id="ip-acl-schema-application-example"></a>
#### Application Example

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

Set Cross-Origin Resource Sharing (CORS) using the `cors` key in the container policy document. For more information about CORS settings and allowed origin formats, see [Cross-Origin Resource Sharing (CORS)](api-guide$[ file_suffix ]$/#set-container-cors-policy) in the API guide.

<br>

<a id="cors-schema"></a>
### JSON Policy Document Schema { #cors-schema }

The structure of the CORS policy document is as follows.

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
#### Field Description

| Field | Format | Required | Description | Note |
|---|---|---|---|---|
| `allow_origins` | Array | N | List of origins to allow | Max. 100 items<br>Cannot include a blank space in each entry. |
| `max_age` | Integer | N | Preflight response cache time | In seconds, an integer of 0 or greater |
| `expose_headers` | Array | N | List of response headers to expose to the browser | Max. 100 items<br>Cannot include a blank space in each item. |
<a id="cors-schema-application-example"></a>
#### Application Example

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
## Object Lock { #lock }

Use the `lock` key in the container policy document to set the lock cycle for object lock (WORM, Write-Once-Read-Many). For concepts and constraints related to object lock, see [Change Object Lock Cycle](api-guide$[ file_suffix ]$/#set-container-object-lock-cycle) in the API guide.

<br>

<a id="lock-schema"></a>
### JSON Policy Document Schema { #lock-schema }

The structure of the Object Lock policy document is as follows.

```json
{
  "lock": {
    "days": integer
  }
}
```

<a id="lock-schema-field-descriptions"></a>
#### Field Description

| Field | Format | Required | Description | Note |
|---|---|---|---|---|
| `days` | Integer | Y | Object lock cycle | In days, from 0 to 36,500 days (max. 100 years) |
<a id="lock-schema-application-rules"></a>
#### Application Rules

When configuring Object Lock, note the following:

* Object Lock can only be enabled when creating a container. Requests to set `lock` on an existing container that does not have Object Lock configured will be rejected.
* Object Lock cannot be disabled. Setting `days` to `0` only sets the default lock period to 0 days; Object Lock remains active.

<a id="lock-schema-application-example"></a>
#### Application Example

```json
{
  "lock": {
    "days": 30
  }
}
```
s