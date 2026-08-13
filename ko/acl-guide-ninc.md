<!-- pre-align:aligned sig=57f0cee284f8 -->

<a id="storage-object-storage-acl-configuration-guide"></a>
## Storage > Object Storage > 접근 정책 설정 가이드 { #storage-object-storage-acl-configuration-guide }

이 문서는 NHN Cloud 오브젝트 스토리지의 컨테이너에 역할 기반 접근 정책과 IP 기반 접근 정책을 설정하는 방법을 설명합니다.

<a id="role-based-access-policies"></a>
## 역할 기반 접근 정책 { #role-based-access-policies }

콘솔 또는 API를 사용해 다른 사용자에게 컨테이너의 읽기/쓰기 접근 권한을 부여할 수 있습니다.

<a id="role-based-access-console"></a>
### 콘솔 { #role-based-access-console }

콘솔에서는 [컨테이너 생성](console-guide-ninc/#create-container) 대화 상자 또는 [컨테이너 관리](console-guide-ninc/#manage-container) 창의 컨테이너 접근 정책 설정 대화 상자에서 컨테이너 접근 정책을 선택할 수 있습니다. 선택할 수 있는 정책은 `PRIVATE`과 `PUBLIC` 두 가지로 제한됩니다.

<a id="role-based-access-private"></a>
#### PRIVATE

`PRIVATE`은 컨테이너가 속한 프로젝트의 사용자에게만 접근 권한을 부여하는 기본 접근 정책입니다. 콘솔을 사용하거나 인증 토큰을 발급받아 API로 컨테이너에 접근할 수 있습니다. API 섹션의 `컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용` 항목과 같은 정책입니다.
<br>

<a id="role-based-access-public"></a>
#### PUBLIC

`PUBLIC`은 누구에게나 읽기와 오브젝트 목록 조회를 허용하는 정책입니다. 컨테이너를 `PUBLIC`으로 설정하면 콘솔에서 URL을 얻을 수 있습니다. 이 URL로 누구나 컨테이너에 접근할 수 있습니다. API 섹션의 `모든 사용자에게 읽기/목록 조회 허용` 항목과 같은 정책입니다.
<br>

<a id="role-based-access-api"></a>
### API { #role-based-access-api }

API를 사용해 컨테이너의 `X-Container-Read`, `X-Container-Write`, `X-Container-View` 속성에 역할 기반 접근 정책 요소를 입력하면 여러 가지 상황에 맞게 접근 정책을 설정할 수 있습니다. 각 속성은 다음과 같습니다.

| 속성 | 설명                                                                                       |
| --- |------------------------------------------------------------------------------------------|
| X-Container-Read | 컨테이너 정보 조회와 컨테이너 내 오브젝트 정보 조회 및 다운로드를 허용합니다. 컨테이너 및 오브젝트의 GET, HEAD 요청이 해당됩니다.           |
| X-Container-Write | 컨테이너 내 오브젝트 변경 요청을 허용합니다. 오브젝트에 대한 PUT, POST, DELETE, COPY 요청이 해당됩니다.                    |
| X-Container-View | 컨테이너 내 오브젝트 목록 조회 및 오브젝트의 정보 조회를 허용합니다. 컨테이너에 대한 GET, HEAD 요청 및 오브젝트에 대한 HEAD 요청이 해당됩니다. |

!!! tip "알아두기"
    `X-Container-Read`, `X-Container-Write`, `X-Container-View`에 설정할 수 있는 접근 정책 요소는 각 속성별로 최대 100개입니다. 이 제한은 [컨테이너 정책](container-policy-guide-ninc/#acl)으로 설정할 때도 동일하게 적용됩니다.

<br>

<a id="role-based-access-elements"></a>
#### 역할 기반 접근 정책 요소

설정할 수 있는 역할 기반 접근 정책 요소는 다음과 같습니다. 모든 정책 요소는 쉼표(`,`)로 구분해 조합할 수 있습니다.

| 정책 요소 | 설명 |
| --- | --- |
| `{tenant-id}:{api-user-id}` | 특정 프로젝트에 속한 특정 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `{tenant-id}:*` | 특정 프로젝트에 속한 모든 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `*:{api-user-id}` | 프로젝트와 관계없이 특정 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `*:*` | 프로젝트와 관계없이 인증 토큰을 발급받을 수 있는 사용자라면 누구나 오브젝트에 접근할 수 있습니다.<br>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |

!!! tip "알아두기"
    `{api-user-id}`는 콘솔의 API 엔드포인트 설정 대화 상자의 **API 사용자 ID** 항목이나 인증 토큰 발급 API 응답 본문의 **access.user.id** 필드에서 확인할 수 있습니다.
    인증 토큰 발급 API를 사용하려면 API 가이드의 [인증 및 권한](api-guide-ninc/#auth) 항목을 참고합니다.

!!! tip "알아두기"
    `{tenant-id}:`나 `:{api-user-id}`처럼 콜론의 한쪽이 비어 있는 값, `.`으로 시작하는 값은 사용할 수 없습니다.

<a id="common-access-elements"></a>
#### 기타 접근 정책 요소

`X-Container-Read` 속성에는 역할 기반 접근 정책 요소 외에 다음 정책 요소도 입력할 수 있습니다.

| 정책 요소 | 설명 |
| --- | --- |
| `.r:*` | 누구나 인증 토큰 없이 오브젝트에 접근할 수 있습니다. |
| `.r:{referrer}` | 요청 헤더를 참조하여 설정된 HTTP 리퍼러의 접근을 허용합니다.<br>인증 토큰은 필요하지 않습니다. |
| `.r:-{referrer}` | 요청 헤더를 참조하여 설정된 HTTP 리퍼러의 접근을 제한합니다.<br>리퍼러 앞에 마이너스 기호(-)를 붙여 설정합니다. |
| `.rlistings` | 인증 토큰 없이 읽기가 허용된 사용자에게 컨테이너 조회(GET 또는 HEAD 요청)를 허용합니다.<br>이 정책 요소가 없으면 오브젝트 목록을 조회할 수 없습니다.<br>이 정책 요소는 단독으로 설정할 수 없습니다. |

!!! tip "알아두기"
    리퍼러에서 `*`는 전체 공개를 뜻하는 `.r:*`로만 사용할 수 있습니다. `*`를 다른 문자와 함께 넣은 값, 전체를 차단하는 `.r:-*`, 빈 값은 사용할 수 없습니다.

<br>

<a id="role-based-access-allow-rw-to-project-users"></a>
#### 컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용

역할 기반 접근 정책 요소를 설정하지 않았을 때 적용하는 기본 접근 정책입니다. API를 사용해 컨테이너에 접근하려면 반드시 유효한 인증 토큰이 필요합니다.
컨테이너의 `X-Container-Read`, `X-Container-Write` 속성값을 모두 삭제하면 컨테이너가 속한 프로젝트의 사용자에게만 접근을 허용하는 `PRIVATE` 컨테이너가 됩니다.

<br>

<details>
<summary>모든 역할 기반 접근 정책 요소 제거 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

!!! tip "알아두기"
    curl로 값이 없는 헤더를 보낼 때는 헤더 이름에 세미콜론(;)을 붙여야 합니다.

유효한 인증 토큰 없이 요청하면 오류 메시지를 응답합니다.

```
$ curl -X GET \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

반드시 요청 헤더에 유효한 인증 토큰이 있어야 원하는 응답을 받을 수 있습니다.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]
```
</details>
<br>

<a id="role-based-access-allow-read-and-list-for-all-users"></a>
#### 모든 사용자에게 읽기/목록 조회 허용

컨테이너의 `X-Container-Read` 속성을 `.r:*, .rlistings`로 설정하면 모든 사용자에게 오브젝트 읽기와 목록 조회를 허용합니다. 인증 토큰은 필요하지 않습니다. 콘솔 섹션의 `PUBLIC` 항목과 같은 정책입니다.
<br>

<details>
<summary>모든 사용자에게 오브젝트 읽기 및 목록 조회 허용 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]
```

`.r:*`만 설정하면 컨테이너의 오브젝트에는 접근할 수 있지만, 오브젝트 목록은 조회할 수 없습니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br>

<a id="role-based-access-allow-or-deny-by-referer"></a>
#### 특정 HTTP 리퍼러 요청에 읽기 허용/거부

HTTP 리퍼러(HTTP Referer)는 하이퍼링크로 요청한 웹 페이지의 주소 정보이며, 요청 헤더에 포함됩니다.
컨테이너의 `X-Container-Read` 속성에 `.r:{referrer}` 또는 `.r:-{referrer}` 형태의 역할 기반 접근 정책 요소를 설정하면 특정 리퍼러의 접근 요청을 허용하거나 차단할 수 있습니다. 역할 기반 접근 정책 요소로 HTTP 리퍼러를 설정할 때는 프로토콜과 하위 경로를 제외한 도메인 이름을 입력해야 합니다.

HTTP 리퍼러 접근 허용/차단 정책은 입력 순서와 관계없이 차단 정책을 우선 적용합니다. 따라서 차단 대상으로 지정된 HTTP 리퍼러의 접근 요청은 모든 접근을 허용하는 `.r:*` 정책 요소를 함께 입력하더라도 거부됩니다.

!!! danger "주의"
    HTTP 리퍼러 헤더는 위변조할 수 있으므로 접근 제어 수단으로 권장하지 않습니다.

<details>
<summary>특정 HTTP 리퍼러 읽기 요청 허용 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

API 요청 헤더에 허용된 HTTP 리퍼러 주소를 명시해 요청하면 오브젝트에 접근할 수 있습니다.

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com/some/path' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```

API 요청 헤더에 허용된 리퍼러 주소가 없거나 리퍼러 주소에 프로토콜이 포함되어 있지 않으면 접근이 차단됩니다.

```
$ curl -X GET \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: https://example.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

다음과 같이 HTTP 리퍼러 설정에 `.`으로 시작하는 도메인 이름을 입력하면 설정된 도메인의 모든 서브도메인 주소를 포함하는 리퍼러에 읽기를 허용합니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://qux.baz.foo.com/some/path' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```

서브도메인이 포함되어 있지 않은 요청은 차단됩니다.

```
$ curl -X GET \
  -H 'Referer: https://foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

특정 도메인 이름을 가진 모든 리퍼러의 접근 요청을 허용하려면 다음과 같이 콤마 리스트를 사용하여 설정합니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:foo.com, .r:.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://baz.foo.com/some/path' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```
</details>

<details>
<summary>특정 HTTP 리퍼러의 읽기 요청 차단 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

HTTP 리퍼러 도메인 이름 앞에 마이너스 기호를 붙여 설정하면 해당 HTTP 리퍼러의 요청을 차단합니다.

```
$ curl -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>

<details>
<summary>특정 HTTP 리퍼러를 제외한 모든 접근 요청 허용 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br>

<a id="role-based-access-allow-rw-project-or-user"></a>
#### 특정 프로젝트 또는 특정 사용자에게 읽기/쓰기 허용

컨테이너의 `X-Container-Read`와 `X-Container-Write` 속성에 `{tenant-id}:{api-user-id}` 형태의 역할 기반 접근 정책 요소를 설정하면 특정 프로젝트 또는 특정 사용자에게 읽기/쓰기 권한을 각각 부여할 수 있습니다. 테넌트 ID 또는 API 사용자 ID 대신 와일드카드 문자 `*`를 입력하면 모든 프로젝트 또는 모든 사용자에게 접근 권한을 부여합니다. 접근을 요청할 때는 반드시 유효한 인증 토큰이 필요합니다.

!!! tip "알아두기"
    인증 토큰이 필요한 접근 정책으로 부여한 읽기 권한에는 오브젝트 목록 조회 권한이 포함됩니다.

<details>
<summary>특정 프로젝트의 특정 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{api-user-id}' \
  -H 'X-Container-Write: {tenant-id}:{api-user-id}' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

오브젝트에 접근을 요청할 때는 반드시 허가된 테넌트 ID와 API 사용자 ID로 발급한 유효한 인증 토큰이 필요합니다.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]


$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```
</details>

<details>
<summary>특정 프로젝트의 모든 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:*' \
  -H 'X-Container-Write: {tenant-id}:*' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

오브젝트에 접근을 요청할 때는 반드시 허가된 테넌트 ID와 API 사용자 ID로 발급한 유효한 인증 토큰이 필요합니다.
<br><br>
</details>

<details>
<summary>프로젝트와 관계없이 특정 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{api-user-id}' \
  -H 'X-Container-Write: *:{api-user-id}' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

오브젝트에 접근을 요청할 때는 프로젝트와 관계없이 허가된 API 사용자 ID로 발급한 유효한 인증 토큰이 필요합니다.
<br><br>
</details>

<details>
<summary>모든 NHN Cloud 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

오브젝트에 접근을 요청할 때는 반드시 유효한 인증 토큰이 필요합니다.
</details>
<br>

<a id="role-based-access-delete-access-policies"></a>
#### 접근 정책 삭제

빈 헤더를 입력하면 설정된 역할 기반 접근 정책 요소를 모두 삭제할 수 있습니다. 역할 기반 접근 정책 요소가 없는 컨테이너는 허가된 사용자만 접근할 수 있는 **PRIVATE** 컨테이너가 됩니다. `컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용` 항목을 참고합니다.

<a id="role-based-access-references"></a>
### References { #role-based-access-references }

Swift Access Control Lists(ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)

<a id="ip-based-access-policies"></a>
## IP 기반 접근 정책 { #ip-based-access-policies }

콘솔 또는 API를 사용해 화이트리스트와 블랙리스트를 지정하여 특정 IP에서 컨테이너의 읽기/쓰기 접근 권한을 제한할 수 있습니다. 화이트리스트와 블랙리스트를 함께 설정할 수 있지만, 이 경우 화이트리스트만 적용되고 블랙리스트는 무시됩니다. IP 기반 접근 정책은 IPv4만 지원합니다. 서비스 게이트웨이 요청에는 별도의 예외를 지정할 수 있습니다.

!!! danger "주의"
    IP 기반 접근 정책은 퍼블릭 IP를 통한 접근을 제어하는 용도입니다. 화이트리스트에 프라이빗 IP만 등록하면 접근할 수 없는 컨테이너가 될 수 있습니다.
    잘못된 설정으로 접근 권한이 없는 컨테이너가 되었다면 더 이상 정책을 변경할 수 없습니다. 이러한 문제가 발생할 경우 고객 센터로 문의하세요.

<a id="ip-based-access-console"></a>
### 콘솔 { #ip-based-access-console }

컨테이너 관리 창의 컨테이너 접근 정책 설정 대화 상자에서 IP 기반 컨테이너 접근 정책을 선택합니다.

!!! danger "주의"
    읽기 권한이 없으면 콘솔에서 컨테이너를 조작할 수 없습니다.

<a id="ip-based-access-whitelist"></a>
#### 화이트리스트

허용된 IP 또는 네트워크 대역을 제외한 모든 요청을 거부합니다. 요청을 허용할 읽기, 쓰기 권한을 지정할 수 있습니다.

<a id="ip-based-access-blacklist"></a>
#### 블랙리스트

지정된 IP 또는 네트워크 대역의 요청을 거부합니다. 그 외의 모든 요청은 허용합니다. 화이트리스트와 함께 설정하면 블랙리스트는 무시됩니다. 요청을 거부할 읽기, 쓰기 권한을 지정할 수 있습니다.

<a id="ip-based-access-service-gateway-ip"></a>
#### 서비스 게이트웨이 IP

서비스 게이트웨이를 통한 요청을 제어합니다. 설정하지 않으면 화이트리스트와 블랙리스트 설정에 따라 요청이 거부될 수 있습니다.

<a id="ip-based-access-api"></a>
### API { #ip-based-access-api }

API를 사용해 컨테이너의 `X-Container-Ip-Acl-Allowed-List`, `X-Container-Ip-Acl-Denied-List` 속성에 IP 기반 접근 정책 요소를 입력하면 IP 기반 접근 정책을 활성화할 수 있습니다. `X-Container-Ip-Acl-Allowed-List`는 화이트리스트, `X-Container-Ip-Acl-Denied-List`는 블랙리스트를 의미합니다.

IP 기반 접근 정책이 설정된 컨테이너의 속성을 변경하려면 허가된 테넌트 ID와 API 사용자 ID로 발급한 유효한 인증 토큰이 필요하며, 허용된 IP에서 요청해야 합니다.

!!! tip "알아두기"
    `X-Container-Ip-Acl-Allowed-List`(화이트리스트)와 `X-Container-Ip-Acl-Denied-List`(블랙리스트)에 설정할 수 있는 정책 요소는 각각 최대 100개입니다. 이 제한은 [컨테이너 정책](container-policy-guide-ninc/#ip-acl)으로 설정할 때도 동일하게 적용됩니다.

<br>

IP 기반 접근 정책 요소는 접근 권한과 IP 또는 네트워크 대역으로 이루어지며 쉼표(`,`)로 구분해 여러 개의 값을 입력할 수 있습니다. 접근 권한은 다음과 같습니다.

| 접근 권한 | 설명 |
| --- | --- |
| `r` | 읽기 권한입니다. GET, HEAD 요청이 해당됩니다. |
| `w` | 쓰기 권한입니다. PUT, POST, DELETE, COPY 요청이 해당됩니다. |
| `a` | 읽기와 쓰기 권한을 모두 의미합니다. GET, HEAD, PUT, POST, DELETE, COPY 요청이 해당됩니다. |

서비스 게이트웨이 요청을 제어하려면 컨테이너의 X-Container-Ip-Acl-Service-Gateway-Control 속성에 권한을 설정합니다. 설정할 수 있는 권한은 다음과 같습니다.

| 권한 | 설명 |
| --- | --- |
| `read` | 읽기 요청을 허용합니다. GET, HEAD 요청이 해당됩니다. |
| `write` | 쓰기 요청을 허용합니다. PUT, POST, DELETE, COPY 요청이 해당됩니다. |
| `rw` | 읽기와 쓰기 모든 요청을 허용합니다. GET, HEAD, PUT, POST, DELETE, COPY 요청이 해당됩니다. |
| `deny` | 읽기와 쓰기 모든 요청을 허용하지 않습니다. |

<details>
<summary>화이트리스트 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

192.168.0.1은 읽기 요청만, 192.168.0.2는 쓰기 요청만 할 수 있으며 172.16.0.0/24 대역의 모든 IP는 읽기와 쓰기 요청을 모두 할 수 있습니다. 그 외의 모든 IP는 요청이 거부됩니다.

<br><br>
</details>

<details>
<summary>블랙리스트 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Denied-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

192.168.0.1은 읽기 요청이, 192.168.0.2는 쓰기 요청이 거부되며 172.16.0.0/24 대역의 모든 IP는 읽기와 쓰기 요청을 모두 할 수 없습니다. 그 외의 모든 IP는 요청이 허용됩니다.

<br><br>
</details>

<details>
<summary>서비스 게이트웨이 요청 제어 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Service-Gateway-Control: rw' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

설정된 IP 기반 접근 정책과 관계없이 서비스 게이트웨이 요청은 모두 허용합니다.

<br><br>
</details>

<a id="ip-based-access-delete-access-policies"></a>
#### 접근 정책 삭제

빈 헤더를 입력하면 설정된 IP 기반 접근 정책 요소를 모두 삭제할 수 있습니다. 정책 요소가 없는 컨테이너는 IP에 따른 접근 제한을 받지 않습니다.

<details>
<summary>IP 기반 접근 정책 요소 제거 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List;' \
  -H 'X-Container-Ip-Acl-Denied-List;' \
  -H 'X-Container-Ip-Acl-Service-Gateway-Control;' \
  https://kr4-api-object-storage.ninc.go.kr/v1/AUTH_*****/container
```

<br><br>
</details>
