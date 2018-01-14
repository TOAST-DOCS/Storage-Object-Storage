## Storage > Object Storage > API 가이드

## 사전 준비
### User Access Key ID 발급
1. 웹 콘솔 상단 오른쪽의 마이페이지 버튼 클릭
2. 드롭다운 메뉴에서 [API 보안 설정] 메뉴 클릭
3. API 보안 설정 메뉴에서 User Access Key ID 발급

### Tenant Name 확인
API를 이용할 때 요청 파라미터로 사용하는 Tenant Name을 얻어오기 위해서는 [그림 1]과 같이 프로젝트 설정의 프로젝트 기본 정보에서 [프로젝트 ID] 값을 참조합니다.

1. 웹 콘솔의 프로젝트 설정 버튼 클릭
3. 프로젝트 설정의 프로젝트 ID 값이 API에서 사용하는 Tenant Name

![[그림 1] Tenant Name 확인](http://static.toastoven.net/prod_infrastructure/object_storage/img_111.png)
<center>[그림 1] Tenant Name 확인</center>

### Account 확인
API를 이용할 때 요청 파라미터로 사용하는 Account를 얻어오기 위해서는 [그림 2]와 같이 &lt;API Endpoint> 대화창에서 Account 항목을 참조합니다.

![[그림 2] Account 확인](http://static.toastoven.net/prod_infrastructure/object_storage/img_112_sc.png)
<center>[그림 2] Account 확인</center>

![[그림 3] Account 확인 대화창](http://static.toastoven.net/prod_infrastructure/object_storage/img_113.png)
<center>[그림 3] Account 확인 대화창</center>

## 인증 토큰 발급

토큰은 오브젝트 스토리지의 RESTful API를 사용할 때 필요한 인증키입니다. 외부로 공개 설정하지 않은 컨테이너나 개체들에 접근하기 위해서는 토큰이 필요합니다. 토큰은 계정 별로 관리됩니다.

[Method, URL]
```
POST   https://api-compute.cloud.toast.com/identity/v2.0/tokens
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|TenantName|	Body or Plain|	String|	TOAST 프로젝트 ID|
|Username|	Plain|	String|	TOAST 사용자 ID, 현재 프로젝트에 멤버로 등록된 사용자|
|Password|	Plain|	String|	사용자 비밀번호, 사전에 발급받은 User Access Key ID

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
|access.token.id|	Body or Plain|	String|	발급된 토큰 ID|
|access.token.tenant.id|	Plain|	String|	토큰을 요청한 프로젝트에 대응하는 Tenant ID|
|access.token.expires|	Plain|	String|	발급된 토큰이 만료되는 시간, <br/> 토큰 발급 시간으로 부터 1시간으로 설정|

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
> 토큰 발급 응답은 “expires” 항목을 포함하고 있습니다. 이 항목은 발급받은 토큰이 언제까지 유효한지를 알려줍니다. 매번 새로운 토큰을 발급받을 필요 없이, 이 정보를 사용해 토큰의 유효 기간을 확인하고 갱신할 수 있도록 합니다.

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```
## 컨테이너

### 컨테이너 생성
오브젝트 스토리지에 파일을 올리기 위해서는 반드시 컨테이너를 생성해야 합니다.

[Method, URL]
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Account|	URL|	String|	사용자 계정명, API Endpoint> 대화창에서 확인|
|Container|	URL|	String|	생성할 컨테이너 이름|

> [참고]  
> 이 API는 응답 본문을 반환하지 않습니다. 컨테이너가 생성에 성공했다면 상태코드 201을 반환합니다.

### 컨테이너 조회

지정한 컨테이너에 대한 정보와 내부에 저장된 개체들의 목록을 조회합니다.

[Method, URL]
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	조회할 컨테이너 이름|

[Response Body Example]

```
{지정한 컨테이너에 속한 개체 목록}
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

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|X-Container-Read|	Header|	String|	컨테이너 읽기에 대한 접근 규칙 지정<br/>.r:* - 모든 사용자에 대해 접근 허용<br/>.r:example.com,test.com – 특정 주소에 대해 접근 허용, ‘,’로 구분<br/>.rlistings. – 컨테이너 목록 조회 허용<br/>AUTH_.... – 특정 계정에 대해 접근 허용|
|X-Container-Write|	Header|	String|	컨테이너 쓰기에 대한 접근 규칙 지정|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	수정할 컨테이너 이름|

[Request Example]

```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
X-Container-Read: .r:*
```

> [참고]  
> 이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태코드 204를 반환합니다.

읽기 권한을 공개로 설정한 후에는 wget 등의 명령을 사용하여 토큰 없이 조회가 되는지 확인 할 수 있습니다.

[Verification Example]
```
GET https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

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
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	삭제할 컨테이너 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 삭제할 컨테이너는 반드시 비어 있어야 합니다. 그렇지 않으면 에러코드를 반환합니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## 개체

### 개체 업로드

지정한 컨테이너에 새로운 개체를 업로드합니다..

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	생성할 개체 이름|
|-|	Body|	Plain| Text	생성할 개체의 내용|

> [참고]  
> 요청 Header에서 Content-type 항목의 값을 개체 속성에 맞게 설정 해야 합니다. 요청이 올바르면 상태코드 201를 반환합니다.

### 개체 내용 수정

생성하려는 개체가 이미 존재하는 경우 해당 개체의 내용이 수정되어 반영됩니다.

[Method, URL]

```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	내용을 수정할 개체 이름|

> [참고]  
> 요청 Header에서 Content-type 항목의 값을 개체 속성에 맞게 설정 해야 합니다. 요청이 올바르면 상태코드 201를 반환합니다.

### 개체 다운로드

[Method, URL]

```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	내려 받을 개체 이름|

> [참고]  
> 개체의 내용이 스트림으로 반환됩니다. 요청이 올바르면 상태코드 200을 반환합니다.

### 개체 복사

[Method, URL]

```
COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Destination|	Header|	String|	개체를 복사할 대상, 컨테이너 이름 + 복사된 개체의 이름|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	복사할 개체 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태코드 201을 반환합니다.

### 개체 속성 수정

[Method, URL]

```
POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Content-Type|	Header|	String|	변경할 개체 형식|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	속성을 수정할 개체 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태코드 202를 반환합니다.

### 개체 삭제

[Method, URL]
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

[Request Parameters]

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급 받은 토큰 ID|
|Account|	URL|	String|	사용자 계정명|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	삭제할 개체 이름|

> [참고]  
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태코드 204를 반환합니다.

### References

Swift API v1 - http://developer.openstack.org/api-ref-objectstorage-v1.html <br />
Identity API v2 - http://developer.openstack.org/api-ref-identity-v2.html
