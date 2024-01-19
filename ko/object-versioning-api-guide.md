## Storage > Object Storage > 오브젝트 버전 관리 API 가이드

## 오브젝트 버전 관리

기존 버전 관리 정책의 단점을 보완하여 보다 용이하게 버전 오브젝트를 조회 및 관리할 수 있습니다. 오브젝트 버전 관리 정책을 사용하는 컨테이너에 오브젝트를 업로드하면 자동으로 버전 오브젝트가 생성됩니다. 언제든 버전 오브젝트로부터 복구, 다운로드, 복사 및 서명된 URL을 생성할 수 있습니다.

### 컨테이너

#### 오브젝트 버전 관리 설정

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Versions-Enabled: {오브젝트 버전 관리 정책 사용 여부}
X-Versions-Lifecycle: {오브젝트의 이전 버전 수명 주기}
X-Versions-Delete-Marker-Expiration: {삭제 마커 만료 설정}
```

##### 요청

요청 본문은 필요하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
| --- | --- | --- | --- | --- |
| X-Auth-Token | Header | String | O | 토큰 ID |
| X-Versions-Enabled | Header | String | - | 오브젝트 버전 관리 정책 사용 여부를 설정, true 또는 false<br/>`X-History-Location` 헤더와 동시에 사용할 수 없습니다. |
| X-Versions-Lifecycle | Header | Integer | - | 오브젝트의 이전 버전의 수명 주기를 일 단위로 설정 |
| X-Versions-Delete-Marker-Expiration | Header | String | - | 삭제 마커 만료 설정, true 또는 false |

###### 오브젝트 버전 관리 정책 사용 여부

`X-Versions-Enabled` 헤더를 사용하여 컨테이너의 오브젝트 버전 관리 정책 사용 여부를 설정할 수 있습니다.

###### 버전 오브젝트 수명 주기

`X-Versions-Lifecycle` 헤더를 사용하여 버전 오브젝트 수명 주기를 일 단위로 설정할 수 있습니다. `1 ~ 36500` 범위 내에서 지정할 수 있고, `'X-Versions-Lifecycle: '`를 사용해 컨테이너 수정을 요청하면 버전 오브젝트 수명 주기 설정이 해제되어 이후 컨테이너에 저장되는 버전 오브젝트는 자동으로 수명 주기가 설정되지 않습니다.

###### 삭제 마커 만료 설정

`X-Versions-Delete-Marker-Expiration` 헤더를 사용하여 삭제 마커에 수명 주기를 사용할 수 있습니다.

##### 응답

응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

#### 버전 오브젝트 목록 조회

```
GET /v1/{Account}/{Container}?versions
X-Auth-Token: {token-id}
```

##### 요청

요청 본문은 필요하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
| --- | --- | --- | --- | --- |
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 스토리지 계정, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 컨테이너 이름 |
| versions | Query | Flag | O | 버전 오브젝트 목록 조회 |
| object | Query | String | - | 조회할 오브젝트 이름 |
| marker | Query | String | - | 지정한 오브젝트의 다음 오브젝트부터 조회 |
| version_marker | Query | String | - | 지정한 버전의 다음 버전부터 조회 |
| prefix | Query | String | - | 검색할 접두어 |
| limit | Query | Integer | - | 목록에 표시할 오브젝트 수 |
| delimiter | Query | String | - | |
| reverse | Query | String | - | |

> [참고]
> 버전 오브젝트 목록 조회 API는 몇 가지 질의(query)를 제공합니다. 모든 질의는 `&`로 연결해 혼용할 수 있습니다.

##### 응답

```
[컨테이너의 버전 오브젝트 목록]
```

| 이름 | 종류 | 형식 | 설명 |
| --- | --- | --- | --- |
| X-Versions-Enabled | Header | String | 오브젝트 버전 관리 정책 사용 여부, 사용 시 True |
| X-Versions-Lifecycle | Header | Integer | 버전 오브젝트 수명 주기 (일) |
| X-Versions-Delete-Marker-Expiration | Header | Boolean | 삭제 마커 만료 설정 |
| X-Container-Bytes-Used | Header | Integer | 컨테이너의 전체 오브젝트 용량 |
| X-Container-Bytes-Used-By-Versions | Header | Integer | 컨테이너의 버전 오브젝트 용량 |
| hash | Body | String | 오브젝트 MD5 체크섬 값 |
| last_modified | Body | String | 오브젝트 마지막 수정 일시 |
| bytes | Body | Integer | 오브젝트 크기 |
| name | Body | String | 오브젝트 이름 |
| content_type | Body | String | 컨텐츠 타입 |
| version_id | Body | String | 버전 오브젝트 ID |
| is_latest | Body | String | 최신 버전 여부 |

###### 최신 오브젝트

`is_latest`가 `true`이면 최신 오브젝트입니다.

###### 삭제 마커

`content_type`이 `"application/x-deleted;swift_versions_deleted=1"`이면 삭제 마커입니다.

###### 버전 ID

버전 오브젝트와 삭제 마커는 `version_id`로 고유한 해시값을 갖습니다. 버전 관리되지 않는 오브젝트의 `version_id`는 `"null"`입니다.

### 오브젝트

#### 버전 오브젝트 조회

```
GET /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

##### 요청

요청 본문은 필요하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
| --- | --- | --- | --- | --- |
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 스토리지 계정, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 컨테이너 이름 |
| Object | URL | String | O | 오브젝트 이름 |
| version-id | Query | String | - | 버전 식별자, 버전 오브젝트 목록 조회 응답 본문에서 확인 |

###### 버전 오브젝트 조회

대상 오브젝트가 버전 관리되는 경우 자동으로 최신 버전 오브젝트를 조회합니다.

###### 버전 ID 지정 조회

`version-id` 쿼리로 버전 ID를 지정하여 버전 오브젝트를 조회할 수 있습니다.

> [주의]
> 삭제 마커 오프젝트는 `version-id` 쿼리를 사용하여 조회할 수 없습니다.

#### 버전 오브젝트 삭제

```
DELETE /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

##### 요청

요청 본문은 필요하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
| --- | --- | --- | --- | --- |
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 스토리지 계정, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 컨테이너 이름 |
| Object | URL | String | O | 오브젝트 이름 |
| version-id | Query | String | - | 버전 식별자, 버전 오브젝트 목록 조회 응답 본문에서 확인 |

###### 버전 오브젝트 삭제

대상 오브젝트가 버전 관리되는 경우 버전 오브젝트를 삭제하면 삭제 마커가 생성됩니다.

###### 버전 ID 지정 삭제

`version-id` 쿼리로 버전 ID를 지정하여 버전 오브젝트 또는 삭제 마커를 삭제할 수 있습니다.

###### 버전 오브젝트 자동 복구

최신 버전 오브젝트를 지정하여 객체 삭제할 때 삭제 마커를 제외한 이전 버전 오브젝트가 있으면 이전 버전이 복원됩니다. 

> [참고]
> `version-id` 쿼리를 사용하여 오브젝트를 삭제하면 삭제 마커가 생성되지 않습니다.

##### 응답

이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

#### 버전 오브젝트 생성

```
PUT /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

##### 요청

요청 본문은 필요하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
| --- | --- | --- | --- | --- |
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 스토리지 계정, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 컨테이너 이름 |
| Object | URL | String | O | 오브젝트 이름 |

###### 버전 오브젝트 생성

오브젝트 버전 관리 정책을 사용하는 컨테이너에 오브젝트를 업로드하면 자동으로 버전 오브젝트가 생성됩니다.

> [주의]
> `version-id` 쿼리를 사용하여 기존 버전 오브젝트를 업데이트할 수 없습니다.

## 레거시 버전 관리 (Deprecated)

오브젝트를 업로드할 때 같은 이름의 오브젝트가 이미 있으면 오브젝트를 업데이트합니다. 기존 오브젝트의 내용을 보관하고 싶다면 `X-History-Location` 헤더를 사용해 이전 버전을 보관할 **아카이브 컨테이너**를 지정할 수 있습니다.

이전 버전 오브젝트는 아카이브 컨테이너에 다음과 같은 형태로 보관됩니다.
```
{3자리 16진수로 표현된 오브젝트 이름의 길이}{오브젝트 이름}/{이전 버전 오브젝트가 보관된 유닉스 시간}
```
예를 들어 `picture.jpg`라는 오브젝트를 업데이트하면 아카이브 컨테이너에는 `00bpicture.jpg/1610606551.82539`라는 오브젝트가 생성됩니다.

버전 관리 정책이 설정된 컨테이너에서 오브젝트를 삭제하면, 아카이브 컨테이너에 삭제된 오브젝트가 보관되고 삭제 마커가 생성됩니다. 아카이브 컨테이너에 보관된 이전 버전 오브젝트에는 언제든 접근할 수 있습니다.

`X-Versions-Retention` 헤더를 함께 사용하면 이전 버전 오브젝트의 수명 주기를 일 단위로 설정할 수 있습니다. 1을 설정했다면 보관된 오브젝트는 1일 이후에 자동으로 삭제됩니다. 설정하지 않으면 이전 버전 오브젝트는 사용자가 삭제하기 전까지 보관됩니다. 설정 이후 보관된 이전 버전 오브젝트에만 적용됩니다.

> [주의]
> 아카이브 컨테이너를 원본 컨테이너보다 먼저 삭제하면, 원본 컨테이너의 오브젝트 업데이트 또는 삭제 시에 오류가 발생합니다. 이미 삭제하였다면 아카이브 컨테이너를 새로 생성하거나 원본 컨테이너의 버전 관리 정책을 해제해 해결할 수 있습니다.
>
> 아카이브 컨테이너로 사용할 컨테이너 이름에는 가급적 유니코드 문자를 사용하지 않는 것을 권장합니다. 아카이브 컨테이너로 지정할 컨테이너 이름에 유니코드 문자가 포함되어 있다면 반드시 URL 인코딩 후 요청 헤더에 입력해야 합니다.
>
> 암호화 컨테이너를 아카이브 컨테이너로 지정한 뒤 암호화 컨테이너의 대칭 키를 Secure Key Manager 서비스에서 삭제하면 원본 컨테이너의 오브젝트 업로드와 삭제에 실패합니다.

### 아카이브 컨테이너 설정

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-History-Location: {오브젝트의 이전 버전을 저장할 컨테이너}
X-Versions-Retention: {오브젝트의 이전 버전 수명 주기}
```

##### 요청

요청 본문은 필요하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
| ---- | ---- | ---- | ---- | ---- |
| X-Auth-Token | Header | String | O | 토큰 ID |
| X-History-Location | Header | String | - | 오브젝트의 이전 버전을 보관할 컨테이너를 설정 |
| X-Versions-Retention | Header | Integer | - | 오브젝트의 이전 버전의 수명 주기를 일 단위로 설정 |

##### 응답

응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.