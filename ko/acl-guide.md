## Storage > Object Storage > 접근 정책 설정 가이드
## 역할 기반 접근 정책

콘솔 또는 API를 사용해 다른 사용자에게 컨테이너의 읽기/쓰기 접근 권한을 부여할 수 있습니다.

### 콘솔
콘솔에서는 [컨테이너 생성](console-guide/#_2) 대화 상자 또는 [컨테이너 관리](console-guide/#_5) 창의 컨테이너 접근 정책 설정 대화 상자에서 컨테이너 접근 정책을 선택할 수 있습니다. 선택할 수 있는 정책은 `PRIVATE`과 `PUBLIC` 두 가지로 제한됩니다.

#### PRIVATE
`PRIVATE`은 컨테이너가 속한 프로젝트의 사용자에게만 접근 권한을 부여하는 기본 접근 정책입니다. 콘솔을 이용하거나, 인증 토큰을 발급 받아 API로 컨테이너에 접근할 수 있습니다. API 섹션의 `컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용` 항목과 같은 정책입니다.
<br/>

#### PUBLIC
`PUBLIC`은 누구에게나 읽기와 오브젝트 목록 조회를 허용하는 정책입니다. 컨테이너를 PUBLIC으로 설정하면 콘솔에서 URL을 얻을 수 있습니다. 이 URL을 통해 누구나 컨테이너에 접근할 수 있습니다. API 섹션의 `모든 사용자에게 읽기 허용` 항목과 같은 정책입니다.
<br/>

### API
API를 사용해 컨테이너의 `X-Container-Read`, `X-Container-Write` 속성에 ACL 정책 요소를 입력하면 여러 가지 상황에 맞게 접근 정책을 설정할 수 있습니다.
<br/>

#### ACL 정책 요소

설정할 수 있는 ACL 정책 요소는 다음과 같습니다. 모든 정책 요소는 콤마(`,`)로 구분해 조합할 수 있습니다.

| 정책 요소 | 설명 |
| --- | --- |
| `.r:*` | 누구나 인증 토큰 없이 오브젝트에 접근할 수 있습니다. |
| `.rlistings` | 읽기 권한이 있는 사용자에게 컨테이너 조회(GET 또는 HEAD 요청)를 허용합니다.<br/>이 정책 요소가 없으면 오브젝트 목록을 조회할 수 없습니다.<br/>이 정책 요소는 단독으로 설정할 수 없습니다. |
| `.r:<referrer>` | 요청 헤더를 참조하여 설정된 HTTP 리퍼러(HTTP Referer)에게 접근을 허용합니다.<br/>인증 토큰은 필요하지 않습니다. |
| `.r:-<referrer>` | 요청 헤더를 참조하여 설정된 HTTP 리퍼러의 접근을 제한합니다.<br/>리퍼러 앞에 마이너스 기호(-)를 붙여 설정합니다. |
| `<tenant-id>:<api-user-id>` | 특정 프로젝트에 속한 특정 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `<tenant-id>:*` | 특정 프로젝트에 속한 모든 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `*:<api-user-id>` | 프로젝트와 관계없이 특정 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `*:*` | 프로젝트와 관계없이 인증 토큰을 발급 받을 수 있는 사용자라면 누구나 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |

> [참고]
> `<api-user-id>`는 콘솔의 API Endpoint 설정 대화 상자에서 **API 사용자 ID** 항목을 참조하거나 인증 토큰 발급 API 응답 본문의 **access.user.id** 필드에서 확인할 수 있습니다.
> 인증 토큰 발급 API를 이용하려면 API 가이드의 [인증 토큰 발급](api-guide/#_2) 항목을 참조하세요.

<br/>

#### 컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용
ACL 정책 요소를 설정하지 않았을 때 사용되는 기본 접근 정책입니다. API를 사용해 컨테이너에 접근하려면 반드시 유효한 인증 토큰이 필요합니다.
컨테이너의 `X-Container-Read`, `X-Container-Write` 속성값을 모두 삭제하면 컨테이너가 속한 프로젝트의 사용자에게만 접근을 허용하는 `PRIVATE` 컨테이너가 됩니다.

<br/>

<details>
<summary>모든 ACL 정책 요소 제거 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

<blockquote>
<p>[참고]
curl을 이용하여 값이 없는 헤더를 보낼 때는 헤더 이름에 세미콜론(;)을 붙여야 합니다.</p>
</blockquote>

유효한 인증 토큰 없이 요청하면 에러 메시지를 응답합니다.

```
$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

반드시 요청 헤더에 유효한 인증 토큰이 있어야 원하는 응답을 받을 수 있습니다.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]
```
</details>
<br/>

#### 모든 사용자에게 읽기/목록 조회 허용
컨테이너의 `X-Container-Read` 속성을 `.r:*, .rlistings`로 설정하면 모든 사용자에게 오브젝트 읽기와 목록 조회를 허용합니다. 인증 토큰은 필요하지 않습니다. 콘솔 섹션의 `PUBLIC` 항목과 같은 정책입니다.
<br/>

<details>
<summary>전체 사용자에 대해 오브젝트 읽기 및 목록 조회 허용 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]
```

<code>.r:*</code>만 설정하면 컨테이너의 오브젝트에는 접근할 수 있지만, 오브젝트 목록은 조회할 수 없습니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>


#### 특정 HTTP 리퍼러 요청에 읽기 허용/거부
HTTP 리퍼러(HTTP Referer)는 하이퍼링크를 통해 요청하는 웹 페이지의 주소 정보입니다. 요청 헤더에 포함되어 있습니다.
컨테이너의 `X-Container-Read` 속성에 `.r:<referrer>` 또는 `.r:-<referrer>` 형태의 ACL 정책 요소를 설정하면, 특정 리퍼러의 접근 요청을 허용하거나 차단할 수 있습니다. ACL 정책 요소로 HTTP 리퍼러를 설정할 때는 프로토콜과 하위 경로를 제외한 도메인 이름을 입력해야 합니다.

> [주의]
> HTTP 리퍼러는 헤더 변조를 통해 사용자가 언제든지 변경할 수 있습니다. HTTP 리퍼러를 이용한 접근 정책은 보안에 취약하기 때문에 권장하지 않습니다.

<details>
<summary>특정 HTTP 리퍼러 읽기 요청 허용 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

API 요청 헤더에 허용된 HTTP 리퍼러 주소를 명시해 요청하면 오브젝트에 접근할 수 있습니다.

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```

API 요청 헤더에 허가된 리퍼러 주소가 없거나, 리퍼러 주소에 프로토콜이 포함되어 있지 않으면 접근이 차단됩니다.

```
$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: https://example.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

다음과 같이 HTTP 리퍼러 설정에 <code>.</code>으로 시작하는 도메인 이름을 입력하면, 설정된 도메인의 모든 서브 도메인 주소를 포함하는 리퍼러에 읽기를 허용합니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://qux.baz.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```

서브 도메인이 포함되어 있지 않은 요청은 차단됩니다.

```
$ curl -X GET \
  -H 'Referer: https://foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

특정 도메인 이름을 가진 모든 리퍼러의 접근 요청을 허용하려면 다음과 같이 콤마 리스트를 이용하여 설정합니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:foo.com, .r:.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://baz.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```
</details>

<details>
<summary>특정 HTTP 리퍼러의 읽기 요청 차단 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

HTTP 리퍼러 도메인 이름 앞에 마이너스 기호를 붙여 설정하면, 설정된 HTTP 리퍼러 요청이 차단됩니다.

```
$ curl -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>

HTTP 리퍼러에 대한 접근 허용/차단 정책은 입력하는 순서에 따라 적용됩니다. 예를 들어, 리퍼러 차단 정책 요소 뒤에 모두에게 접근을 허용하는 `.r:*` 정책 요소를 입력했다면 리퍼러 차단 정책은 무시됩니다. 반대로 모두에게 접근을 허용하는 정책 요소를 먼저 입력하고 특정 리퍼러 차단 정책 요소를 뒤에 입력했다면, 설정된 리퍼러의 접근 요청을 제외한 모든 접근 요청이 허용됩니다.
<br/>

<details>
<summary>HTTP 리퍼러 차단이 무시되는 잘못된 정책 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com, .r:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```
</details>

<details>
<summary>특정 HTTP 리퍼러의 접근 요청을 제외한 모든 접근 요청을 허용하는 정책 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br/>

#### 특정 프로젝트 또는 특정 사용자에게 읽기/쓰기 허용
컨테이너의 `X-Container-Read`와 `X-Container-Write` 속성에 `<tenant-id>:<api-user-id>` 형태의 ACL 정책 요소를 설정하면, 특정 프로젝트 또는 특정 사용자에게 읽기/쓰기 권한을 각각 부여할 수 있습니다. 테넌트 ID 또는 API 사용자 ID 대신 와일드카드 문자 `*`를 입력하면 모든 프로젝트 또는 모든 사용자에게 접근 권한을 부여합니다. 접근 요청을 할 때는 반드시 유효한 인증 토큰이 필요합니다.

> [참고]
> 인증 토큰이 필요한 ACL 정책으로 부여된 읽기 권한에는 오브젝트 목록 조회 권한이 포함되어 있습니다.

<details>
<summary>특정 프로젝트의 특정 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{api-user-id}' \
  -H 'X-Container-Write: {tenant-id}:{api-user-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 허가된 테넌트 ID와 NHN Cloud 사용자 ID로 발급 받은 유효한 인증 토큰이 필요합니다.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]


$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

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
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 허가된 테넌트 ID와 해당하는 프로젝트에 속한 NHN Cloud 사용자 ID로 발급 받은 유효한 인증 토큰이 필요합니다.
<br/><br/>
</details>

<details>
<summary>프로젝트와 관계없이 특정 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{api-user-id}' \
  -H 'X-Container-Write: *:{api-user-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 허가된 NHN Cloud 사용자 ID로 발급 받은 유효한 인증 토큰이 필요합니다.
<br/><br/>
</details>

<details>
<summary>모든 NHN Cloud 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 유효한 인증 토큰이 필요합니다.
</details>
<br/>

#### 접근 정책 삭제
빈 헤더를 입력하면 설정된 ACL 정책 요소를 모두 삭제할 수 있습니다. ACL 정책 요소가 없는 컨테이너는 허가된 사용자만 접근할 수 있는 **PRIVATE** 컨테이너가 됩니다. `컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용` 항목을 참고하세요.


### References
Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)

## IP 기반 접근 정책

콘솔 또는 API를 사용해 화이트리스트와 블랙리스트를 지정하여 특정 IP에서 컨테이너의 읽기/쓰기 접근 권한을 제한할 수 있습니다. 화이트리스트와 블랙리스트는 동시에 사용할 수 없습니다. 화이트리스트와 블랙리스트를 모두 입력한 경우 화이트리스트만 사용됩니다. IP 기반 접근 정책은 IPv4만 지원합니다.


> [주의]
> 잘못된 설정으로 인하여 쓰기 권한이 없는 컨테이너가 발생할 수 있습니다. 이 경우 더 이상 설정 변경이 불가능합니다. 이로 인한 문제가 발생하면 고객센터로 문의합니다.


### 콘솔

컨테이너 관리 창의 컨테이너 접근 정책 설정 대화 상자에서 IP 기반의 컨테이너 접근 정책을 선택합니다. 

> [주의]
> 읽기 권한이 없는 경우 콘솔에서 해당 컨테이너의 조작은 더 이상 불가능합니다.

#### 화이트리스트
허용된 IP 또는 네트워크 대역을 제외한 모든 요청을 거부합니다. 요청을 허용할 읽기, 쓰기 권한 지정이 가능합니다.

#### 블랙리스트
지정된 IP 또는 네트워크 대역의 요청을 거부합니다. 그 외의 모든 요청은 허용됩니다. 허용 정책과 함께 사용되는 경우 거부 정책은 무시됩니다. 요청을 거부할 읽기, 쓰기 권한 지정이 가능합니다.


### API

API를 사용해 컨테이너의 `X-Container-Ip-Acl-Allowed-List`, `X-Container-Ip-Acl-Denied-List` 속성에 ACL 정책 요소를 입력하면 IP 기반의 ACL을 활성화할 수 있습니다. `X-Container-Ip-Acl-Allowed-List`는 화이트리스트, `X-Container-Ip-Acl-Denied-List`는 블랙리스트를 의미합니다.
<br>

정책 요소는 접근 권한과, IP 또는 네트워크 대역으로 이루어져 있으며 콤마(`,`)로 구분해 여러 개의 값을 입력할 수 있습니다. 접근 권한은 아래의 표와 같습니다.

| 접근 권한 | 설명 |
| --- | --- |
| `r` | 읽기 권한, GET, HEAD 요청이 해당됩니다. |
| `w` | 쓰기 권한, PUT, POST, DELETE, COPY 요청이 해당됩니다.|
| `a` | 읽기와 쓰기 권한을 모두 의미합니다. GET, HEAD, PUT, POST, DELETE, COPY 요청이 해당됩니다. |


<details>
<summary>화이트리스트 설정</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

192.168.0.1 IP는 읽기 요청만, 192.168.0.2 IP에서는 쓰기 요청만 가능하며 172.16.0.0/24 대역의 모든 IP는 모든 요청이 가능합니다. 그 외의 모든 IP는 요청이 거부됩니다.<br><br>

컨테이너를 변경하려면 허가된 테넌트 ID와 해당 프로젝트에 속한 NHN Cloud 사용자 ID로 발급 받은 유효한 인증 토큰이 필요하며, 반드시 허용된 IP에서 요청해야 합니다.

<br/><br/>
</details>

<details>
<summary>블랙리스트 설정</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Denied-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

192.168.0.1 IP는 읽기 요청이, 192.168.0.2 IP에서는 쓰기 요청이 거부되며 172.16.0.0/24 대역의 모든 IP는 모든 요청이 불가능합니다. 그 외의 모든 IP는 요청이 허용됩니다.<br><br>

컨테이너를 변경하려면 허가된 테넌트 ID와 해당 프로젝트에 속한 NHN Cloud 사용자 ID로 발급 받은 유효한 인증 토큰이 필요하며, 반드시 허용된 IP에서 요청해야 합니다.

<br/><br/>
</details>

<details>
<summary>ACL 설정 제거</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List;' \
  -H 'X-Container-Ip-Acl-Denied-List;' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

컨테이너를 변경하려면 허가된 테넌트 ID와 해당 프로젝트에 속한 NHN Cloud 사용자 ID로 발급 받은 유효한 인증 토큰이 필요하며, 반드시 허용된 IP에서 요청해야 합니다.

<br/><br/>
</details>


