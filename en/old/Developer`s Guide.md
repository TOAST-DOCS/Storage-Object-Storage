## Infrastructure > Object Storage > Developer's Guide

## 사전 준비

### API Password 설정

1. [Infrastructure] > [Object Storage] 메뉴 클릭
2. 상단에 보이는 [API Endpoint 설정] 버튼 클릭
3. &lt;API Endpoint> 대화창에서 원하는 API 비빌번호를 지정

### Tenant Name 확인
API 이용 시 요청 파라미터로 사용하는 Tenant Name을 얻어오기 위해서는 [그림 1]과 같이 Project List에서 [프로젝트 ID] 항목의 값을 참조합니다.

1. Web Console에 로그인
2. Project List 확인
3. Project List의 프로젝트 ID 항목의 값이 API에서 사용하는 Tenant Name

![[그림 1] Tenant Name 확인](http://static.toastoven.net/prod_infrastructure/object_storage/img_111.png)
<center>[그림 1] Tenant Name 확인</center>

### Account 확인
API 이용 시 요청 파라미터로 사용하는 Account를 얻어오기 위해서는 [그림 2]와 같이 &lt;API Endpoint> 대화창에서 Account 이름을 참조합니다.

![[그림 2] Account 확인](http://static.toastoven.net/prod_infrastructure/object_storage/img_112_sc.png)
<center>[그림 2] Account 확인</center>

![[그림 3] Account 확인 대화창](http://static.toastoven.net/prod_infrastructure/object_storage/img_113.png)
<center>[그림 3] Account 확인 대화창</center>

## 인증 Token 발급

Token은 Object Storage 의 RESTful API사용을 위해 발급받아야 하는 인증키입니다. 외부로 공개 설정하지 않은 Container나 Object들은 이 Token을 가지고 요청을 하며 그렇지 않은 경우 접근할 수 없습니다. Token은 Account 별로 관리됩니다.

[Method, URL]
```
POST   https://api-compute.cloud.toast.com/identity/v2.0/tokens
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|TenantName|	Body or Plain|	String|	TOAST Cloud의 Project ID|
|Username|	Plain|	String|	TOAST Cloud의 사용자 ID, 현재 Project에 member로 등록된 사용자|
|Password|	Plain|	String|	사용자 비밀번호, &lt;API Endpoint> 대화창에서 지정한 비밀번호|

[Request Body Example]

```
{
  "auth": {
    "tenantName": "X1k372c9",
    "passwordCredentials": {
      "username": "test@nhnent.com",
      "password": "..."
    }
  }
}
```

[Response Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|access.token.id|	Body or Plain|	String|	인증 Token|
|access.token.tenant.id|	Plain|	String|	Token을 요청한 Project에 대응하는 Tenant ID|
|access.token.expires|	Plain|	String|	발급한 Token의 만료 시간. yyyy-mm-ddTHH:MM:ssZ의 형태. 예) 2017-05-16T03:17:50Z |

[Response Body Example]

```
{
    "access": {
        "token": {
            "expires": "2015-11-23T03:14:23Z",
            "id": "bbefb86f8e7d41cfb40e582ca5ca39a8",
            "tenant": {
                "description": "",
                "enabled": true,
                "id": "70c11b619c3f4a53af8d2466ee1b0234",
                "name": "X1k372c9",
                "project_domain": "NORMAL",
                "swift": true
            },
            "issued_at": "2015-11-23T02:14:23.482540"
        },
        "serviceCatalog": [
            …
        ],
        "user": {
            "username": "…",
            "id": "…",
            "name": "…",
            "roles": [
                {
                    "name": "project_admin"
                }
            ],
            "roles_links": []
        },
        "metadata": {
            "roles": [
                "…"
            ],
            "is_admin": 0
        }
    }
}
```

> [주의]  
> Token 발급 응답은 “expires” field를 포함하고 있습니다. 이field는 발급받은 Token이 언제까지 유효한지를 알려줍니다. 매번 새로운 Token을 발급받을 필요 없이, 이 정보를 사용해 Token의 유효 기간을 확인하고 갱신할 수 있도록 합니다.

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
```
## Containers

### Container 생성
Object Storage에 파일을 올리기 위해서는 반드시 Container를 생성합니다.

[Method, URL]
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	생성할 Container 이름|

> [참고]  
> 이 API는 응답 본문을 반환하지 않습니다. Container가 성공적으로 생성되었으면 상태코드로 201을 반환합니다.

### Container 조회

지정한 Container에 대한 정보와 내부에 저장된 Object들의 목록을 조회합니다.

[Method, URL]
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	조회할 Container 이름|

[Response Body Example]

```
{지정한 Container에 속한 Object 목록}
```

### Container 수정

Container의 metadata를 변경하여 접근 규칙을 지정할 수 있습니다.

[Method, URL]

```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
X-Container-Read: {Container 읽기 정책}
X-Container-Write: {Container 쓰기 정책}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|X-Container-Read|	Header|	String|	Container 읽기에 대한 접근 규칙 지정<br/>.r:* - 모든 사용자에 대해 접근 허용<br/>.r:example.com,test.com – 특정 주소에 대해 접근 허용, ‘,’로 구분<br/>.rlistings. – Container의 목록 조회 허용<br/>AUTH_.... – 특정 Account에 대해 접근 허용|
|X-Container-Write|	Header|	String|	Container 쓰기에 대한 접근 규칙 지정|
|Account|	URL|	String|	사용자 계정명, 대화창에서 확인|
|Container|	URL|	String|	수정할 Container 이름|

[Request Example]

```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
X-Container-Read: .r:*
```

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 정상적으로 요청된 경우 204를 반환합니다.

읽기 권한을 공개로 설정한 후에는 wget 등의 명령을 사용하여 Token 없이 조회가 되는지 확인 할 수 있습니다.

[Verification Example]
```
GET https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{Object의 내용}
```

> [참고]  
> 이 API는 응답 본문을 반환하지 않습니다. 정상적으로 요청된 경우 204를 반환합니다.

### Container 삭제

지정한 Container를 삭제합니다.

[Method, URL]

```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	삭제할 Container 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 삭제할 Container는 반드시 빈 Container이어야 합니다. 그렇지 않은 경우 Error를 발생합니다. 정상적으로 생성된 경우 상태 코드로 204를 반환합니다.

## Object

### Object 올리기

지정한 Container에 새로운 Object를 생성합니다.

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	Container 이름|
|Object|	URL|	String|	생성할 Object 이름|
|-|	Body|	Plain| Text	생성할 Object의 내용|

> [참고]  
> 요청 시 Header의 Content-type 항목의 값을 Object의 속성에 맞게끔 설정 해야 합니다. 정상적으로 요청된 경우 상태코드로 201를 반환합니다.

### Object 내용 수정하기

생성하려는 Object가 이미 존재하는 경우 해당 Object의 내용이 수정되어 반영됩니다.

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	Container 이름|
|Object|	URL|	String|	내용을 수정할 Object 이름|

> [참고]  
> 요청 시 Header의 Content-type 항목의 값을 Object의 속성에 맞게끔 설정 해야 합니다. 정상적인 요청 시 상태코드로 201를 반환합니다.

### Object 내려 받기

[Method, URL]

```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	Container 이름|
|Object|	URL|	String|	내려 받을 Object 이름|

> [참고]  
> Object의 내용이 stream으로 변환되어 반환됩니다. 정상적인 요청 시 상태코드로 200을 반환합니다.

### Object 복사 하기

[Method, URL]

```
COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Destination|	Header|	String|	Object를 복사할 대상, Container 이름 + 복사된 Object의 이름|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	Container 이름|
|Object|	URL|	String|	복사할 Object 이름|

> [참고]  
> 정상적인 요청 시 상태코드로 201을 반환합니다.

### Object의 속성 수정 하기

[Method, URL]

```
POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Content-Type|	Header|	String|	변경할 Object의 Type|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	Container 이름|
|Object|	URL|	String|	속성을 수정할 Object 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 정상적인 요청 시 상태코드로 202를 반환합니다.

### Object 지우기

[Method, URL]
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: {Token ID}
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 Token의 ID|
|Account|	URL|	String|	사용자 계정명, &lt;API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	Container 이름|
|Object|	URL|	String|	삭제할 Object 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 정상적인 요청 시 상태코드로 204를 반환합니다.

### References

Swift API v1 - http://developer.openstack.org/api-ref-objectstorage-v1.html <br />
Identity API v2 - http://developer.openstack.org/api-ref-identity-v2.html
