## Storage > Object Storage > API 가이드

## 사전 준비
### User Access Key ID 발급

API를 이용하기 위해 인증 토큰을 발급받으려면 Secret Access Key가 필요합니다. 이 키는 User Access Key ID와 함께 발급받을 수 있습니다.

1. 웹 콘솔 상단 오른쪽의 계정명 클릭
2. 드롭다운 메뉴에서 [API 보안 설정] 클릭
3. API 보안 설정 메뉴에서 User Access Key ID 발급 버튼 클릭
4. Secret Access Key 발급


### Tenant Name 확인

API를 이용할 때 Tenant Name을 파라미터로 입력해야 합니다. Tenant Name은 프로젝트 설정 페이지에서 확인할 수 있는 [프로젝트 ID]입니다.

1. 웹 콘솔의 프로젝트 설정 버튼 클릭
2. 프로젝트 ID 값 확인


### API Endpoint 확인

API의 엔드포인트는 `[API Endpoint]` 버튼을 클릭해 확인할 수 있습니다.

| 항목 | API Endpoint | 용도 |
|---|---|---|
| Identity | https://api-compute.cloud.toast.com/identity/v2.0 | 인증 토큰 발급 |
| Object-Store | https://api-storage.cloud.toast.com/v1/{Account} | 오브젝트 스토리지 제어 |

> [참조]
> API에 사용되는 사용자의 Account는 `AUTH_***` 형태의 문자열입니다. Object-Store API 엔드포인트에 포함되어 있습니다.


## 인증 토큰 발급

인증 토큰은 오브젝트 스토리지의 RESTful API를 사용할 때 필요한 인증키입니다. 외부 공개로 설정하지 않은 컨테이너나 개체들에 접근하려면 반드시 토큰이 필요합니다. 토큰은 계정별로 관리됩니다.

[Method, URL]

```
POST    https://api-compute.cloud.toast.com/identity/v2.0/tokens
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|TenantName|	Body or Plain|	String|	TOAST 프로젝트 ID|
|Username|	Plain|	String|	User Access Key ID |
|Password|	Plain|	String|	Secret Access Key |

[Request Body Example]

```
{
  "auth": {
    "tenantName": "{Project ID}",
    "passwordCredentials": {
      "username": "{User Access Key ID}",
      "password": "{Secret Access Key}"
    }
  }
}
```

[Response Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|access.token.id|	Body or Plain|	String|	발급된 토큰 ID|
|access.token.tenant.id|	Plain|	String|	토큰을 요청한 프로젝트에 대응하는 Tenant ID|
|access.token.expires|	Plain|	String|	발급된 토큰이 만료되는 시간, <br/> 토큰 발급 시간으로부터 1시간|

[Response Body Example]

```
{
    "access": {
        "token": {
            "expires": "2018-01-15T08:05:05Z",
            "id": "b587ae461278419da6ecd21a2344c8aa",
            "tenant": {
                "description": "",
                "enabled": true,
                "id": "c6439197d5e74e4593ca37a62bbc09a6",
                "name": "9AD5KRlH",
                "groupId": "XEj2zkHrbA7modGU",
                "project_domain": "NORMAL",
                "swift": true
            },
            "issued_at": "2018-01-15T07:05:05.719672"
        },
        …
    }
}
```

> [주의]  
> 토큰은 유효 시간이 있습니다. 토큰 발급 요청의 응답에 포함된 "expires" 항목은 발급받은 토큰이 만료되는 시간입니다. 토큰이 만료되면 새로운 토큰을 발급 받아야 합니다.


## 컨테이너

### 컨테이너 생성
오브젝트 스토리지에 파일을 올리기 위해서는 반드시 컨테이너를 생성해야 합니다.

[Method, URL]

```
PUT https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|URL|String|생성할 컨테이너 이름|

> [참고]  
> 이 API는 응답 본문을 반환하지 않습니다. 컨테이너가 생성되었다면 상태 코드 201을 반환합니다.

### 컨테이너 조회
지정한 컨테이너의 정보와 내부에 저장된 개체들의 목록을 조회합니다.

[Method, URL]
```
GET   /{Container}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Container|URL|String|조회할 컨테이너 이름|

[Response Body Example]

```
[지정한 컨테이너에 속한 개체 목록]
```

### 컨테이너 수정

컨테이너의 메타데이터를 변경하여 접근 규칙을 지정할 수 있습니다.

[Method, URL]

```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {토큰 ID}
X-Container-Read: {컨테이너 읽기 정책}
X-Container-Write: {컨테이너 쓰기 정책}
```

[Request Parameters]

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|X-Container-Read|Header|String|컨테이너 읽기에 대한 접근 규칙 지정<br/>.r:* - 모든 사용자에 대해 접근 허용<br/>.r:example.com,test.com – 특정 주소에 대해 접근 허용, ‘,’로 구분<br/>.rlistings. – 컨테이너 목록 조회 허용<br/>AUTH_.... – 특정 계정에 대해 접근 허용|
|X-Container-Write|Header|String|컨테이너 쓰기에 대한 접근 규칙 지정|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|URL|String|수정할 컨테이너 이름|

[Request Example]

```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
X-Container-Read: .r:*
```

> [참고]  
> 이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

읽기 권한을 공개로 설정한 후에는 `curl`, `wget` 등의 도구를 사용하거나 브라우저를 통해 토큰 없이 조회되는지 확인할 수 있습니다.

[Verification Example]
```
$ curl https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{개체의 내용}
```

### 컨테이너 삭제

지정한 컨테이너를 삭제합니다.

[Method, URL]

```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	삭제할 컨테이너 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 삭제할 컨테이너는 반드시 비어 있어야 합니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## 개체

### 개체 업로드

지정한 컨테이너에 새로운 개체를 업로드합니다.

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	생성할 개체 이름|
|-|	Body|	Plain| Text	생성할 개체의 내용|

> [참고]  
> 요청 헤더에 개체 속성에 맞는 Content-type 항목을 설정해야 합니다. 요청이 올바르면 상태 코드 201을 반환합니다.


### 멀티파트 업로드

5GB를 초과하는 용량을 가진 개체는 5GB 이하의 세그먼트로 나누어 업로드해야 합니다.

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: [토큰 ID]
```

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	생성할 개체 이름|
|Count| URL| String| 분할한 개체의 순번|
|-|	Body|	Plain| Text 분할한 개체의 내용|


모든 개체의 세그먼트를 업로드한 다음 매니패스트 개체를 생성하면 하나의 개체처럼 사용할 수 있습니다. 매니패스트 개체는 세그먼트들이 저장된 경로를 가리킵니다.

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
X-Object-Manifest: {Container}/{Object}/
```

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|X-Object-Manifest| Header| String | 분한 개체들을 업로드한 경로, `{Container}/{Object}/` |
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	생성할 개체 이름|
|-|	Body|	Plain| Text 분할한 개체의 내용|


[Example]

```
// 분할된 개체 업로드
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/001 --data-binary '.....'
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/002 --data-binary '.....'

// 매니패스트 개체 업로드
$ curl -X PUT -H 'X-Auth-Token: *****' -H 'X-Object-Manifest: con/sample.jpg/' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg --data-binary ''

// 개체 다운로드
$ curl http://10.162.50.125/v1/AUTH_*****/con/sample.jpg > sample.jpg
```

### 개체 내용 수정

개체 업로드 API와 같지만, 개체가 이미 컨테이너에 있다면 해당 개체의 내용이 수정됩니다.

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	내용을 수정할 개체 이름|

> [참고]  
> 요청 헤더에 개체 속성에 맞는 Content-type 항목을 설정해야 합니다. 요청이 올바르면 상태 코드 201을 반환합니다.

### 개체 다운로드

[Method, URL]

```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	다운로드할 개체 이름|

> [참고]  
> 개체의 내용이 스트림으로 반환됩니다. 요청이 올바르면 상태 코드 200을 반환합니다.

### 개체 복사

[Method, URL]

```
COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Destination|	Header|	String|	개체를 복사할 대상, `{컨테이너 이름} / {복사된 개체의 이름}`|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	복사할 개체 이름|


> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

### 개체 메타데이터 수정

[Method, URL]

```
POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Content-Type|	Header|	String|	변경할 개체 형식|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	속성을 수정할 개체 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 202를 반환합니다.

### 개체 삭제

[Method, URL]

```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	삭제할 개체 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

### References

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
Identity API v2 - [http://developer.openstack.org/api-ref-identity-v2.html](http://developer.openstack.org/api-ref-identity-v2.html)
