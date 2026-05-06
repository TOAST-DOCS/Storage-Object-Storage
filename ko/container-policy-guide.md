## Storage > Object Storage > 컨테이너 정책 설정 가이드

<a id="container-policy"></a>
### 컨테이너 정책

컨테이너 정책을 사용하면 컨테이너 설정을 JSON 형식 정책 문서로 통합하여 관리할 수 있습니다.

정책 문서는 다음과 같이 기능별 최상위 키로 구성되며, 각 최상위 키 하위에 세부 설정을 정의합니다.

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

> [참고]
> 2026년 5월 기준 **수명 주기** 설정만 지원하며, 추후 더 많은 기능으로 확대 적용될 예정입니다.

<a id="container-policy-api"></a>
## 컨테이너 정책 API

<a id="get-container-policy"></a>
### 정책 조회

컨테이너에 설정된 정책 문서를 조회합니다.

```
GET /v1/{Account}/{Container}?policy
X-Auth-Token: {token-id}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 스토리지 계정 |
| Container | URL | String | O | 컨테이너 이름 |
| policy | Query | - | O | 정책 조회를 위한 쿼리 파라미터 (값 없이 사용) |

#### 응답

성공 시 HTTP 상태 코드 `200`과 함께 JSON 형식의 정책 문서를 반환합니다. 설정된 정책이 없으면 빈 JSON 오브젝트를 반환합니다.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

<details>
  <summary>응답 예시</summary>

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
### 정책 설정

요청 본문에 JSON 정책 문서를 포함해 컨테이너 정책을 설정합니다.

정책은 요청에 포함된 최상위 키(예: lifecycle) 단위로 덮어쓰며, 포함되지 않은 키의 설정은 유지됩니다.

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
Content-Type: application/json
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Content-Type | Header | String | O | `application/json`|
| Account | URL | String | O | 스토리지 계정 |
| Container | URL | String | O | 컨테이너 이름 |
| - | Body | JSON | O | 설정할 정책 문서 |

#### 응답

성공 시 HTTP 상태 코드 `204`를 반환합니다. 응답 본문은 없습니다.

> [참고]
> 같은 요청에 헤더와 정책 문서를 함께 사용하면 정책 문서가 우선 적용됩니다.

<br/>

<a id="lifecycle"></a>
## 수명 주기

컨테이너 정책 문서의 `lifecycle` 키를 사용해 수명 주기 규칙을 설정합니다.
수명 주기에는 두 가지 종류의 규칙이 있습니다.

| 규칙 종류 | 이름 | 설명 |
|---|---|---|
| `default_rule` | 기본 규칙 | 모든 조건 규칙에 부합하지 않는 오브젝트에 적용되는 규칙입니다. |
| `rules` | 조건 규칙 | 설정한 조건에 부합하는 오브젝트에 적용되는 규칙입니다. 기본 규칙보다 우선 적용됩니다. |

<br/>

<a id="lifecycle-schema"></a>
### JSON 정책 문서 스키마

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

#### 필드 설명

| 필드 | 형식 | 필수 | 설명 | 비고 |
|---|---|---|---|---|
| `default_rule` | Object | - | 기본 규칙 | 생략하거나 빈 오브젝트(`{}`)로 설정하면 기본 규칙이 해제됩니다. |
| `default_rule.days` | Integer | O | 오브젝트 수명 주기 | 일 단위, 최대 36,500일 |
| `default_rule.action.type` | Enum | O | 만료 동작 유형 | `"transfer"` (이동) 또는 `"delete"` (삭제) |
| `default_rule.action.destination` | String | - | 만료 시 오브젝트를 이동할 대상 컨테이너 이름 | `type`이 `"transfer"`일 때 필수 |
| `rules` | Array | - | 조건 규칙 목록 | 생략하거나 빈 배열(`[]`)로 설정하면 조건 규칙이 모두 삭제됩니다. |
| `rules[*].name` | String | O | 규칙 이름 | 컨테이너 내에서 중복될 수 없습니다. |
| `rules[*].condition.prefix` | String | O | 오브젝트 이름의 접두사 조건 | 빈 문자열은 허용하지 않습니다. |
| `rules[*].days` | Integer | O | 오브젝트 수명 주기 | 일 단위, 최대 36,500일 |
| `rules[*].action.type` | Enum | O | 만료 동작 유형 | `"transfer"` (이동) 또는 `"delete"` (삭제) |
| `rules[*].action.destination` | String | - | 만료 시 오브젝트를 이동할 대상 컨테이너 이름 | `type`이 `"transfer"`일 때 필수 |

<br/>

<a id="lifecycle-apply"></a>
### 수명 주기 규칙 적용

#### 규칙 우선순위

오브젝트 하나에는 단 하나의 수명 주기 규칙만 적용됩니다. 규칙은 아래 우선순위에 따라 적용됩니다.

| 우선순위 | 규칙 | 설명 |
|---|---|---|
| 1 | 조건 규칙 (`rules`) | 조건 규칙의 순서대로 확인하여 처음으로 부합하는 규칙을 적용합니다. |
| 2 | 기본 규칙 (`default_rule`) | 부합하는 조건 규칙이 없으면 기본 규칙을 적용합니다. |
| 3 | 삭제 | 오브젝트 만료 시 적용할 수 있는 규칙이 없으면 삭제됩니다. |

#### 수명 주기

수명 주기는 **오브젝트 업로드 시점**에 컨테이너의 정책을 조회하여 적용합니다.

* 오브젝트를 업로드하는 시점의 수명 주기 규칙을 평가하여, 부합하는 규칙의 수명 주기(`days`) 값을 기반으로 오브젝트 만료 일시를 설정합니다.
* 이후 컨테이너의 수명 주기 규칙이 변경되어도 이미 업로드된 오브젝트의 만료 일시는 자동으로 갱신되지 않습니다.

#### 만료 동작

만료 동작은 **오브젝트 만료 시점**에 컨테이너의 정책을 조회하여 적용합니다.

* 오브젝트 만료 시점에 수명 주기 규칙을 다시 평가하여, 부합하는 규칙의 만료 동작(`action`)에 따라 이동(`transfer`) 또는 삭제(`delete`)를 수행합니다.
* 만료 시점에 어떤 규칙과도 부합하지 않고 기본 규칙도 없으면 오브젝트는 삭제됩니다.

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
