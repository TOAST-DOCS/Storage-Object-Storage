## Storage > Object Storage > API 가이드

## 사전 준비

오브젝트 스토리지 API를 사용하려면 먼저 인증 토큰(Token)을 발급받아야 합니다. 인증 토큰은 오브젝트 스토리지의 REST API를 사용할 때 필요한 인증 키입니다. 외부 공개로 설정하지 않은 컨테이너나 개체들에 접근하려면 반드시 토큰이 필요합니다. 토큰은 계정별로 관리됩니다.

### 테넌트 아이디(Tenant ID) 및 API 엔드포인트(Endpoint) 확인

토큰 발급을 위한 테넌트 아이디와 API의 엔드포인트는 Object Storage 서비스 페이지의 **API Endpoint 설정** 버튼을 클릭해 확인할 수 있습니다.

| 항목 | API Endpoint | 용도 |
|---|---|---|
| Identity | https://api-compute.cloud.toast.com/identity/v2.0 | 인증 토큰 발급 |
| Object-Store | https://api-storage.cloud.toast.com/v1/{Account} | 오브젝트 스토리지 제어 |
| Tenant ID | 숫자 + 영문자로 구성된 32자 길이의 문자열 | 인증 토큰 발급 |

> [참고]  
> API에 사용되는 사용자의 계정(Account)은 `AUTH_***` 형태의 문자열입니다. Object-Store API 엔드포인트에 포함되어 있습니다.

### API 비밀번호 설정

API 비밀번호는 Object Storage 서비스 페이지의 **API Endpoint 설정** 버튼을 클릭한 다음 설정할 수 있습니다.

1. **API Endpoint 설정** 버튼을 클릭합니다.
2. **API Endpoint 설정** 아래 **API 비밀번호 설정** 입력 상자에 토큰 발급 시 사용할 비밀번호를 입력합니다.
3. **저장** 버튼을 클릭합니다.

## 인증 토큰 발급

**[Method, URL]**
```
POST    https://api-compute.cloud.toast.com/identity/v2.0/tokens
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|tenantId|	Body or Plain|	String|	Tenant ID. API Endpoint 설정 대화창에서 확인 가능.|
|username|	Plain|	String|	TOAST 계정 ID(이메일) 입력|
|password|	Plain|	String|	**API Endpoint 설정**에서 저장한 비밀번호|

**[Request Body]**
```
{
  "auth": {
    "tenantId": "{Tenant ID}",
    "passwordCredentials": {
      "username": "{TOAST ID}",
      "password": "{API Password}"
    }
  }
}
```

**[Response Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|access.token.id|	Body or Plain|	String|	발급된 토큰 ID|
|access.token.tenant.id|	Plain|	String|	토큰을 요청한 프로젝트에 대응하는 Tenant ID|
|access.token.expires|	Plain|	String|	발급된 토큰이 만료되는 시간, <br/> 토큰 발급 시간으로부터 1시간|

**[Response Body]**
```
{
  "access": {
    "token": {
      "expires": "{Expires Time}",
      "id": "{Token ID}",
      "tenant": {
        "description": "",
        "enabled": true,
        "id": "{Tenant ID}",
        "name": "{TOAST ID}",
        "groupId": "{TOAST Project ID}",
        "project_domain": "NORMAL",
        "swift": true
      },
      "issued_at": "{Token Issued Time}"
    },
    "serviceCatalog" : []
  }
}
```


**[Example]**
```
$ curl -X POST -H 'Content-Type:application/json' \
https://api-compute.cloud.toast.com/identity/v2.0/tokens \
-d '{"auth": {"tenantId": "*****", "passwordCredentials": {"username": "*****", "password": "*****"}}}'

{
  "access": {
    "token": {
      "expires": "2018-01-15T08:05:05Z",
      "id": "b587ae461278419da6ecd21a2344c8aa",
      "tenant": {
        "description": "",
        "enabled": true,
        "id": "*****",
        "name": "*****",
        "groupId": "*****",
        "project_domain": "NORMAL",
        "swift": true
      },
      "issued_at": "2018-01-15T07:05:05.719672"
    },
    "serviceCatalog" : [
      {
        "endpoints" : [
          {
            "region" : "KR1",
            "publicURL" : "https://api-storage.cloud.toast.com/v1/AUTH_*****"
          }
        ],
        "type" : "object-store",
        "name" : "swift",
        "endpoints_links" : []
      }
    ]
  }
}
```

> [주의]  
> 토큰은 유효 시간이 있습니다. 토큰 발급 요청의 응답에 포함된 "expires" 항목은 발급받은 토큰이 만료되는 시간입니다. 토큰이 만료되면 새로운 토큰을 발급받아야 합니다.


## 컨테이너

### 컨테이너 생성
오브젝트 스토리지에 파일을 올리기 위해서는 반드시 컨테이너를 생성해야 합니다.

**[Method, URL]**
```
PUT https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|URL|String|생성할 컨테이너 이름|

> [참고]
> 이 API는 응답 본문을 반환하지 않습니다. 컨테이너가 생성되었다면 상태 코드 201을 반환합니다.

**[Example]**
```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
```

### 컨테이너 조회
지정한 컨테이너의 정보와 내부에 저장된 개체들의 목록을 조회합니다.

**[Method, URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Container|URL|String|조회할 컨테이너 이름|

**[Response Body]**
```
[지정한 컨테이너에 속한 개체 목록]
```

**[Example]**
```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
ba6610.jpg
20d33f.jpg
31466f.jpg
```

#### 질의
컨테이너 조회 API는 다음과 같이 몇 가지 질의(query)를 제공합니다. 모든 질의는 `&`로 연결해 혼용할 수 있습니다.

##### 1만 개 이상의 개체 목록 조회
컨테이너 조회 API로 조회할 수 있는 목록의 개체 수는 1만 개로 제한되어 있습니다. 1만 개 이상의 개체 목록을 조회하려면 `marker` 질의를 이용해야 합니다. marker 질의는 지정한 개체의 다음 개체부터 최대 1만 개의 목록을 반환합니다.

**[Method, URL]**
```
GET    https://api-storage.cloud.toast.com/v1/{Account}/{Container}?marker={Object}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Container|URL|String|조회할 컨테이너 이름|
|Object|URL|String|기준 개체 이름|

**[Response Body]**
```
[지정한 컨테이너에 속한 지정한 개체 다음 개체 목록]
```

**[Example]**
```
// `20d33f.jpg` 이후의 개체 목록 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?maker=20d33f.jpg
{지정한 개체(20d33f.jpg) 이후의 목록}
```

##### 폴더 단위의 개체 목록 조회
컨테이너에 여러 개의 폴더를 생성하고, 폴더에 개체를 업로드했다면 `path` 질의를 이용해 폴더 단위로 개체 목록을 조회할 수 있습니다. path 질의는 하위 폴더의 개체 목록은 조회할 수 없습니다.

**[Method, URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}?path={Path}
```

**[Request Parameters]**

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Container|URL|String|조회할 컨테이너 이름|
|Path|URL|String|조회할 폴더 이름|

**[Response Body]**
```
[지정한 컨테이너에 속한 지정한 폴더의 개체 목록]
```

**[Example]**
```
// ex 폴더의 개체 목록 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?path=ex
ex/20d33f.jpg
ex/31466f.jpg
```

##### 접두어로 시작하는 개체 목록 조회
`prefix` 질의를 사용하면 지정한 접두어로 시작하는 개체들의 목록을 반환합니다. path 질의로는 조회할 수 없는 하위 폴더를 포함한 폴더의 개체 목록을 조회하는데 사용할 수 있습니다.

**[Method, URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}?prefix={Prefix}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Container|URL|String|조회할 컨테이너 이름|
|Prefix|URL|String|검색할 접두어|

**[Response Body]**
```
[지정한 컨테이너에 속하고 지정한 접두어로 시작하는 개체 목록]
```

**[Example]**
```
// 314로 시작하는 개체 목록 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?prefix=314
3146f0.jpg
3147a6.jpg
31486f.jpg
```

##### 목록의 최대 개체 수 지정
`limit` 질의를 사용하면 반환할 개체 목록의 최대 개체 수를 지정할 수 있습니다.

**[Method, URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}?limit={Number}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|Container|URL|String|조회할 컨테이너 이름|
|Number|URL|String|목록에 표시할 개체 수|

**[Response Body]**
```
[지정한 컨테이너에 속한 지정된 수 만큼의 개체 목록]
```

**[Example]**
```
// 10개의 개체만 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?limit=10
...{9개의 개체}...
31466f0.jpg
```

### 컨테이너 수정

컨테이너의 메타데이터를 변경하여 접근 규칙을 지정할 수 있습니다.

**[Method, URL]**
```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {토큰 ID}
X-Container-Read: {컨테이너 읽기 정책}
X-Container-Write: {컨테이너 쓰기 정책}
```

**[Request Parameters]**

|이름|종류|속성|설명|
|---|---|---|---|
|X-Auth-Token|Header|String|발급받은 토큰 ID|
|X-Container-Read|Header|String|컨테이너 읽기에 대한 접근 규칙 지정<br/>.r:* - 모든 사용자에 대해 접근 허용<br/>.r:example.com,test.com – 특정 주소에 대해 접근 허용, ‘,’로 구분<br/>.rlistings. – 컨테이너 목록 조회 허용<br/>AUTH_.... – 특정 계정에 대해 접근 허용|
|X-Container-Write|Header|String|컨테이너 쓰기에 대한 접근 규칙 지정|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|URL|String|수정할 컨테이너 이름|

**[Example]**
```
// 모든 사용자에게 읽기/쓰기 허용
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Read: .r:*' \
-H 'X-Container-Write: .r:*' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
```

> [참고]
> 이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

읽기 권한을 공개로 설정한 후에는 `curl`, `wget` 등의 도구를 사용하거나 브라우저를 통해 토큰 없이 조회되는지 확인할 수 있습니다.

**[Verification Example]**
```
$ curl https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{개체의 내용}
```

### 컨테이너 삭제

지정한 컨테이너를 삭제합니다.

**[Method, URL]**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	삭제할 컨테이너 이름|

**[Example]**
```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
```

> [참고]
> 이 요청은 응답 본문을 반환하지 않습니다. 삭제할 컨테이너는 반드시 비어 있어야 합니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## 개체

### 개체 업로드

지정한 컨테이너에 새로운 개체를 업로드합니다.

**[Method, URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	생성할 개체 이름|
|-|	Body|	Plain| Text	생성할 개체의 내용|

**[Example]**
```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg \
-T ./ba6610.jpg
```

> [참고]
> 요청 헤더에 개체 속성에 맞는 Content-type 항목을 설정해야 합니다. 요청이 올바르면 상태 코드 201을 반환합니다.


### 멀티파트 업로드

5GB를 초과하는 용량을 가진 개체는 5GB 이하의 세그먼트로 나누어 업로드해야 합니다.

**[Method, URL]**
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

**[Method, URL]**
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


**[Example]**
```
// 200MB 단위로 파일 분할
$ split -d -b 209715200 large_obj.img large_obj.img.

// 분할된 개체 업로드
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/003 \
-T large_obj.img.02

// 매니패스트 개체 업로드
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Object-Manifest: curl_example/large_obj.img/' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img \
-d ''
```

### 개체 내용 수정

개체 업로드 API와 같지만, 개체가 이미 컨테이너에 있다면 해당 개체의 내용이 수정됩니다.

**[Method, URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	내용을 수정할 개체 이름|

> [참고]
> 요청 헤더에 개체 속성에 맞는 Content-type 항목을 설정해야 합니다. 요청이 올바르면 상태 코드 201을 반환합니다.

### 개체 다운로드

**[Method, URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	다운로드할 개체 이름|

**[Example]**
```
$ curl -O -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 17166  100 17166    0     0   566k      0 --:--:-- --:--:-- --:--:--  578k
```

> [참고]
> 개체의 내용이 스트림으로 반환됩니다. 요청이 올바르면 상태 코드 200을 반환합니다.

### 개체 복사

**[Method, URL]**
```
COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Destination|	Header|	String|	개체를 복사할 대상, `{컨테이너 이름} / {복사된 개체의 이름}`|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	복사할 개체 이름|

**[Example]**
```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/3a45e9.jpg' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/3a45e9.jpg
```

> [참고]
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

### 개체 메타데이터 수정

**[Method, URL]**
```
POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|X-Object-Meta-{Key}|	Header|	String| 변경할 메타데이터 |
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	속성을 수정할 개체 이름|

**[Example]**
```
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg

$ curl -I -H "X-Auth-Token: b587ae461278419da6ecd21a2344c8aa" \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg
HTTP/1.1 200 OK
...
X-Object-Meta-Type: photo
...
```

> [참고]
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 202를 반환합니다.

### 개체 삭제

**[Method, URL]**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [토큰 ID]
```

**[Request Parameters]**

|이름|	종류|	속성|	설명|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	발급받은 토큰 ID|
|Account|URL|String|사용자 계정명, API Endpoint에 포함되어 있음|
|Container|	URL|	String|	컨테이너 이름|
|Object|	URL|	String|	삭제할 개체 이름|

**[Example]**
```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg
```

> [참고]
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## References

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
