<!-- pre-align:aligned sig=90b39f34080f -->

<a id="storage-object-storage-container-policy-configuration-guide"></a>
## Storage > Object Storage > 컨테이너 정책 설정 가이드 { #storage-object-storage-container-policy-configuration-guide }

이 문서는 컨테이너 정책을 사용해 NHN Cloud 오브젝트 스토리지의 컨테이너 관련 설정을 관리하는 방법을 설명합니다.

<a id="container-policy"></a>
## 컨테이너 정책 { #container-policy }

컨테이너 정책을 사용하면 컨테이너 설정을 JSON 형식의 정책 문서로 통합하여 관리할 수 있습니다.

정책 문서는 다음과 같이 기능별 최상위 키로 구성되며, 각 최상위 키 하위에 세부 설정을 정의합니다.

```json
{
  "lifecycle": { ... },
  "acl": { ... },
  "ip_acl": { ... },
  "cors": { ... },
  "lock": { ... }
}
```

각 최상위 키가 담당하는 기능은 다음과 같습니다.

| 최상위 키 | 기능 | 설명 |
|---|---|---|
| `lifecycle` | 수명 주기 | 오브젝트의 수명 주기 규칙을 설정합니다. |
| `acl` | 접근 제어(ACL) | 컨테이너의 읽기, 쓰기, 조회 접근 권한을 설정합니다. |
| `ip_acl` | IP 접근 제어(IP ACL) | IP 기반 접근 제어를 설정합니다. |
| `cors` | 교차 출처 리소스 공유(CORS) | 허용 출처 등 CORS 설정을 관리합니다. |
| `lock` | 오브젝트 잠금 | 오브젝트 잠금(WORM)의 잠금 주기를 설정합니다. |

<a id="container-policy-api"></a>
## 컨테이너 정책 API { #container-policy-api }

<a id="get-container-policy"></a>
### 정책 조회 { #get-container-policy }

컨테이너에 설정된 정책 문서를 조회합니다.

```
GET /v1/{Account}/{Container}?policy
X-Auth-Token: {token-id}
```

<a id="get-container-policy-request"></a>
#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | 토큰 ID |
| Account | URL | String | Y | 스토리지 계정 |
| Container | URL | String | Y | 컨테이너 이름 |
| policy | Query | String | Y | 정책 조회를 위한 쿼리 파라미터<br>값을 지정하지 않으면 전체 정책 문서를, 기능별 최상위 키를 지정하면 해당 기능의 정책만 조회합니다. |

<a id="get-container-policy-response"></a>
#### 응답

성공 시 HTTP 상태 코드 `200`과 함께 JSON 형식의 정책 문서를 반환합니다. 설정된 정책이 없으면 빈 JSON 오브젝트를 반환합니다.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

`policy` 쿼리 파라미터에 값을 지정하지 않으면 설정된 모든 기능의 정책을 하나의 문서로 반환합니다.

<details>
  <summary>전체 정책 문서 조회 요청 및 응답 예시</summary>

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

`policy` 쿼리 파라미터에 값을 지정하면 해당 기능의 정책만 반환합니다. 설정되지 않은 기능은 응답에 포함되지 않습니다.

<details>
  <summary>특정 기능 조회 요청 및 응답 예시</summary>

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
### 정책 설정 { #set-container-policy }

요청 본문에 JSON 정책 문서를 포함해 컨테이너 정책을 설정합니다.

정책을 설정할 때 다음 규칙이 적용됩니다.

* 최상위 키는 `lifecycle`, `acl`, `ip_acl`, `cors`, `lock`만 사용할 수 있으며, 스키마에 정의되지 않은 키나 필드를 포함하면 요청이 거부됩니다.
* 정책 문서에 포함한 최상위 키는 보낸 내용으로 전체를 덮어쓰고, 포함하지 않은 최상위 키의 설정은 그대로 유지됩니다. 최상위 키를 보내면서 일부 하위 항목만 담으면 담지 않은 항목은 해제되므로, 유지하려는 항목은 항상 함께 보내야 합니다.

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
Content-Type: application/json
```

<a id="set-container-policy-request"></a>
#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | 토큰 ID |
| Content-Type | Header | String | Y | `application/json` |
| Account | URL | String | Y | 스토리지 계정 |
| Container | URL | String | Y | 컨테이너 이름 |
| - | Body | JSON | Y | 설정할 정책 문서 |

<a id="set-container-policy-response"></a>
#### 응답

성공 시 HTTP 상태 코드 `204`를 반환합니다. 응답 본문은 없습니다.

!!! tip "알아두기"
    같은 요청에서 헤더와 정책 문서를 함께 사용하면 정책 문서가 우선 적용됩니다.
    정책 문서가 스키마에 맞지 않으면 HTTP 상태 코드 `400`과 함께 오류 위치와 사유를 담은 메시지를 반환합니다.

<br>

<a id="lifecycle"></a>
## 수명 주기 { #lifecycle }

컨테이너 정책 문서의 `lifecycle` 키를 사용해 수명 주기 규칙을 설정합니다.
수명 주기에는 두 가지 종류의 규칙이 있습니다.

| 규칙 종류 | 이름 | 설명 |
|---|---|---|
| `default_rule` | 기본 규칙 | 모든 조건 규칙에 부합하지 않는 오브젝트에 적용되는 규칙입니다. |
| `rules` | 조건 규칙 | 설정한 조건에 부합하는 오브젝트에 적용되는 규칙입니다. 기본 규칙보다 우선 적용됩니다. |

<br>

<a id="lifecycle-schema"></a>
### JSON 정책 문서 스키마 { #lifecycle-schema }

수명 주기 정책 문서의 구조는 다음과 같습니다.

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
#### 필드 설명

| 필드 | 형식 | 필수 | 설명 | 비고 |
|---|---|---|---|---|
| `default_rule` | Object | N | 기본 규칙 | |
| `default_rule.days` | Integer | Y | 오브젝트 수명 주기 | 일 단위, 최대 36,500일 |
| `default_rule.action.type` | Enum | Y | 만료 동작 유형 | `"transfer"` (이동) 또는 `"delete"` (삭제) |
| `default_rule.action.destination` | String | Conditional | 만료 시 오브젝트를 이동할 대상 컨테이너 이름 | `type`이 `"transfer"`일 때 필수 |
| `rules` | Array | N | 조건 규칙 목록 | 최대 30개 |
| `rules[*].name` | String | Y | 규칙 이름 | 컨테이너 내에서 중복될 수 없습니다. |
| `rules[*].condition.prefix` | String | Y | 오브젝트 이름의 접두사 조건 | 빈 문자열은 허용하지 않습니다. |
| `rules[*].days` | Integer | Y | 오브젝트 수명 주기 | 일 단위, 최대 36,500일 |
| `rules[*].action.type` | Enum | Y | 만료 동작 유형 | `"transfer"` (이동) 또는 `"delete"` (삭제) |
| `rules[*].action.destination` | String | Conditional | 만료 시 오브젝트를 이동할 대상 컨테이너 이름 | `type`이 `"transfer"`일 때 필수 |

<br>

<a id="lifecycle-apply"></a>
### 수명 주기 규칙 적용 { #lifecycle-apply }

<a id="lifecycle-apply-rule-priority"></a>
#### 규칙 우선순위

오브젝트 하나에는 단 하나의 수명 주기 규칙만 적용됩니다. 규칙은 아래 우선순위에 따라 적용됩니다.

| 우선순위 | 규칙 | 설명 |
|---|---|---|
| 1 | 조건 규칙 (`rules`) | 조건 규칙의 순서대로 확인하여 처음으로 부합하는 규칙을 적용합니다. |
| 2 | 기본 규칙 (`default_rule`) | 부합하는 조건 규칙이 없으면 기본 규칙을 적용합니다. |
| 3 | 삭제 | 오브젝트 만료 시 적용할 수 있는 규칙이 없으면 삭제됩니다. |

<a id="lifecycle-apply-lifecycle"></a>
#### 수명 주기

수명 주기는 **오브젝트 업로드 시점**에 컨테이너의 정책을 조회하여 적용합니다.

* 오브젝트를 업로드하는 시점의 수명 주기 규칙을 평가하여, 부합하는 규칙의 수명 주기(`days`) 값을 기반으로 오브젝트 만료 일시를 설정합니다.
* 이후 컨테이너의 수명 주기 규칙이 변경되어도 이미 업로드된 오브젝트의 만료 일시는 자동으로 갱신되지 않습니다.

<a id="lifecycle-apply-expiration-action"></a>
#### 만료 동작

만료 동작은 **오브젝트 만료 시점**에 컨테이너의 정책을 조회하여 적용합니다.

* 오브젝트 만료 시점에 수명 주기 규칙을 다시 평가하여, 부합하는 규칙의 만료 동작(`action`)에 따라 이동(`transfer`) 또는 삭제(`delete`)를 수행합니다.
* 만료 시점에 어떤 규칙과도 부합하지 않고 기본 규칙도 없으면 오브젝트는 삭제됩니다.

<a id="lifecycle-apply-application-example"></a>
#### 적용 예시

아래와 같은 수명 주기 규칙이 설정되어 있다고 가정합니다.

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

* **오브젝트 `image/test.jpg` 업로드**
    * `logs/` 접두사 조건에 부합하지 않으므로 기본 규칙 적용
    * 수명 주기 10일 설정
* **업로드 이후, `rule1`의 조건 변경**
    * 조건을 `"condition": { "prefix": "image/" }`로 변경
* **오브젝트 `image/test.jpg` 수명 주기 만료**
    * 만료 시점에 규칙을 다시 평가: `rule1`의 `image/` 접두사 조건에 부합
    * 만료 동작: `archive-container`로 이동

<br>

<a id="acl"></a>
## 접근 제어(ACL) { #acl }

컨테이너 정책 문서의 `acl` 키를 사용해 컨테이너 접근 권한을 설정합니다. 접근 제어는 읽기, 쓰기, 조회 세 가지 권한으로 구성됩니다.

| 권한 | 키 | 설명 |
|---|---|---|
| 읽기 | `read` | 컨테이너 정보와 오브젝트 정보 조회 및 다운로드를 허용합니다. |
| 쓰기 | `write` | 오브젝트 업로드, 삭제 등 변경 요청을 허용합니다. |
| 조회 | `view` | 컨테이너의 오브젝트 목록 조회를 허용합니다. |

정책 문서의 `read`, `write`, `view`는 각각 컨테이너의 `X-Container-Read`, `X-Container-Write`, `X-Container-View` 속성에 대응합니다. 각 권한의 자세한 내용은 [접근 정책 설정 가이드](acl-guide-ngsc/#role-based-access-api)를 참고합니다.

<br>

<a id="acl-schema"></a>
### JSON 정책 문서 스키마 { #acl-schema }

접근 제어 정책 문서의 구조는 다음과 같습니다.

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
#### 필드 설명

| 필드 | 형식 | 필수 | 설명 | 비고 |
|---|---|---|---|---|
| `read` | Object | N | 읽기 권한 | `grantees`, `referrers`(`allow`, `deny`), `public`, `listing`을 합산하여 최대 100개 |
| `read.public` | Boolean | N | 모든 사용자에게 읽기 허용 | `.r:*` 정책 요소에 대응하며, 인증 토큰 없이 접근을 허용합니다. |
| `read.listing` | Boolean | N | 오브젝트 목록 조회 허용 | `.rlistings` 정책 요소에 대응하며, 읽기 권한이 있는 사용자에게 목록 조회를 허용합니다. 단독으로는 설정할 수 없습니다. |
| `read.referrers.allow` | Array | N | 접근을 허용할 리퍼러(HTTP Referer) 도메인 목록 | 각 항목은 `.r:<referrer>` 정책 요소에 대응합니다. 빈 문자열이나 와일드카드 `*`는 허용하지 않으며, `,`를 포함하거나 `-`로 시작할 수 없습니다. |
| `read.referrers.deny` | Array | N | 접근을 차단할 리퍼러(HTTP Referer) 도메인 목록 | 각 항목은 `.r:-<referrer>` 정책 요소에 대응합니다. 빈 문자열이나 와일드카드 `*`는 허용하지 않으며, `,`를 포함하거나 `-`로 시작할 수 없습니다. |
| `read.grantees` | Array | N | 읽기 권한을 부여할 사용자 목록 | `<tenant>:<user>` 형식의 역할 기반 접근 정책 요소에 대응합니다. |
| `read.grantees[*].tenant` | String | Y | 테넌트(프로젝트) ID | 와일드카드 `*`를 사용할 수 있습니다. 빈 문자열은 허용하지 않으며, 값에 `,`나 `:`를 포함할 수 없습니다. |
| `read.grantees[*].user` | String | Y | API 사용자 ID | 와일드카드 `*`를 사용할 수 있습니다. 빈 문자열은 허용하지 않으며, 값에 `,`를 포함할 수 없습니다. |
| `write` | Object | N | 쓰기 권한 | `grantees` 최대 100개 |
| `write.grantees` | Array | N | 쓰기 권한을 부여할 사용자 목록 | `grantees` 형식은 `read.grantees`와 동일합니다. |
| `view` | Object | N | 조회 권한 | `grantees` 최대 100개 |
| `view.grantees` | Array | N | 조회 권한을 부여할 사용자 목록 | `grantees` 형식은 `read.grantees`와 동일합니다. |

!!! tip "알아두기"
    `public`, `listing`, `referrers`에 대응하는 요소는 접근 정책 설정 가이드의 [기타 접근 정책 요소](acl-guide-ngsc/#common-access-elements)를, `grantees`에 대응하는 요소는 [역할 기반 접근 정책 요소](acl-guide-ngsc/#role-based-access-elements)를 참고합니다.

<a id="acl-schema-application-example"></a>
#### 적용 예시

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

`view`를 포함하지 않았으므로 조회 권한은 해제됩니다.

<br>

<a id="ip-acl"></a>
## IP 접근 제어(IP ACL) { #ip-acl }

컨테이너 정책 문서의 `ip_acl` 키를 사용해 IP 기반 접근 제어를 설정합니다. 화이트리스트와 블랙리스트는 동시에 사용할 수 없으며, 둘 다 설정하면 화이트리스트만 적용됩니다. 그 외 동작 방식은 접근 정책 설정 가이드의 [IP 기반 접근 정책](acl-guide-ngsc/#ip-based-access-policies)을 참고합니다.

<br>

<a id="ip-acl-schema"></a>
### JSON 정책 문서 스키마 { #ip-acl-schema }

IP 접근 제어 정책 문서의 구조는 다음과 같습니다.

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
#### 필드 설명

| 필드 | 형식 | 필수 | 설명 | 비고 |
|---|---|---|---|---|
| `whitelist` | Array | N | 접근을 허용할 IP 목록 | 최대 100개 |
| `whitelist[*].permission` | Enum | Y | 규칙을 적용할 작업 유형 | `"read"`, `"write"`, `"full_control"` |
| `whitelist[*].cidr` | String | Y | IPv4 주소 또는 CIDR | 빈 문자열은 허용하지 않습니다. |
| `blacklist` | Array | N | 접근을 차단할 IP 목록 | 최대 100개 |
| `blacklist[*].permission` | Enum | Y | 규칙을 적용할 작업 유형 | `"read"`, `"write"`, `"full_control"` |
| `blacklist[*].cidr` | String | Y | IPv4 주소 또는 CIDR | 빈 문자열은 허용하지 않습니다. |
| `services` | Array | N | 서비스별 접근 제어 | |
| `services[*].name` | Enum | Y | 서비스 이름 | 현재 `service_gateway`만 지원합니다. |
| `services[*].permission` | Enum | Y | 허용하거나 차단할 작업 유형 | `"read"`, `"write"`, `"full_control"`, `"deny"` |

!!! tip "알아두기"
    `cidr`에는 IPv4 주소 또는 CIDR을 입력합니다. IP 기반 접근 정책은 IPv4만 지원합니다.

<a id="ip-acl-schema-application-example"></a>
#### 적용 예시

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
## 교차 출처 리소스 공유(CORS) { #cors }

컨테이너 정책 문서의 `cors` 키를 사용해 교차 출처 리소스 공유(CORS)를 설정합니다. CORS 설정과 허용 출처 형식의 자세한 내용은 API 가이드의 [교차 출처 리소스 공유(CORS)](api-guide-ngsc/#set-container-cors-policy)를 참고합니다.

<br>

<a id="cors-schema"></a>
### JSON 정책 문서 스키마 { #cors-schema }

CORS 정책 문서의 구조는 다음과 같습니다.

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
#### 필드 설명

| 필드 | 형식 | 필수 | 설명 | 비고 |
|---|---|---|---|---|
| `allow_origins` | Array | N | 허용할 출처(Origin) 목록 | 최대 100개<br>각 항목에 공백을 포함할 수 없습니다. |
| `max_age` | Integer | N | 프리플라이트 응답 캐시 시간 | 초 단위, 0 이상의 정수 |
| `expose_headers` | Array | N | 브라우저에 노출할 응답 헤더 목록 | 최대 100개<br>각 항목에 공백을 포함할 수 없습니다. |

<a id="cors-schema-application-example"></a>
#### 적용 예시

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
## 오브젝트 잠금 { #lock }

컨테이너 정책 문서의 `lock` 키를 사용해 오브젝트 잠금(WORM, Write-Once-Read-Many)의 잠금 주기를 설정합니다. 오브젝트 잠금의 개념과 제약은 API 가이드의 [오브젝트 잠금 기간 변경](api-guide-ngsc/#set-container-object-lock-cycle)을 참고합니다.

<br>

<a id="lock-schema"></a>
### JSON 정책 문서 스키마 { #lock-schema }

오브젝트 잠금 정책 문서의 구조는 다음과 같습니다.

```json
{
  "lock": {
    "days": integer
  }
}
```

<a id="lock-schema-field-descriptions"></a>
#### 필드 설명

| 필드 | 형식 | 필수 | 설명 | 비고 |
|---|---|---|---|---|
| `days` | Integer | Y | 오브젝트 잠금 주기 | 일 단위, 0~36,500일(최대 100년) |

<a id="lock-schema-application-rules"></a>
#### 적용 규칙

오브젝트 잠금은 설정할 때 다음 사항에 유의해야 합니다.

* 오브젝트 잠금은 컨테이너를 생성할 때만 활성화할 수 있습니다. 오브젝트 잠금이 설정되지 않은 기존 컨테이너에 `lock`을 설정하는 요청은 거부됩니다.
* 오브젝트 잠금은 해제할 수 없습니다. `days`를 `0`으로 설정하면 기본 잠금 주기가 0일이 될 뿐, 오브젝트 잠금은 활성 상태로 유지됩니다.

<a id="lock-schema-application-example"></a>
#### 적용 예시

```json
{
  "lock": {
    "days": 30
  }
}
```
