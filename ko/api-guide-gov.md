## Storage > Object Storage > API 가이드

## 사전 준비

오브젝트 스토리지 API를 사용하려면 먼저 인증 토큰(token)을 발급받아야 합니다. 인증 토큰은 오브젝트 스토리지의 REST API를 사용할 때 필요한 인증 키입니다. 외부 공개로 설정하지 않은 컨테이너나 오브젝트에 접근하려면 반드시 토큰이 필요합니다. 토큰은 계정별로 관리됩니다.

### 테넌트 아이디(Tenant ID) 및 API 엔드포인트(Endpoint) 확인

토큰 발급을 위한 테넌트 아이디와 API의 엔드포인트는 오브젝트 스토리지 서비스 페이지의 **API Endpoint 설정** 버튼을 클릭해 확인할 수 있습니다.

| 항목 | API 엔드포인트 | 용도 |
|---|---|---|
| Identity | https://gov-api-compute.cloud.toast.com/identity/v2.0 | 인증 토큰 발급 |
| Object-Store | https://gov-api-storage.cloud.toast.com/v1/{Account} | 오브젝트 스토리지 제어, 리전에 따라 다름 |
| Tenant ID | 숫자 + 영문자로 구성된 32자 길이의 문자열 | 인증 토큰 발급 |

### API 비밀번호 설정

API 비밀번호는 오브젝트 스토리지 서비스 페이지의 **API Endpoint 설정** 버튼을 클릭해 설정할 수 있습니다.

1. **API Endpoint 설정** 버튼을 클릭합니다.
2. **API Endpoint 설정** 아래 **API 비밀번호 설정** 입력 상자에 토큰 발급 시 사용할 비밀번호를 입력합니다.
3. **저장** 버튼을 클릭합니다.

## 인증 토큰 발급

```
POST    https://gov-api-compute.cloud.toast.com/identity/v2.0/tokens
Content-Type: application/json
```

### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| tenantId | Body | String | O | Tenant ID. API Endpoint 설정 대화 상자에서 확인 가능 |
| username | Body | String | O | TOAST 계정 ID(이메일) 입력 |
| password | Body | String | O | API Endpoint 설정 대화 상자에서 저장한 비밀번호 |

<details>
<summary>예시</summary>

```json
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

</details>

### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| access.token.id | Body | String |	발급된 토큰 ID |
| access.token.tenant.id | Body | String | 토큰을 요청한 프로젝트에 대응하는 Tenant ID |
| access.token.expires | Body | String | 발급한 토큰의 만료 시간 <br/>yyyy-mm-ddTHH:MM:ssZ의 형태. 예) 2017-05-16T03:17:50Z |

> [주의]
> 토큰에는 유효 시간이 있습니다. 토큰 발급 요청의 응답에 포함된 'expires' 항목은 발급받은 토큰이 만료되는 시간입니다. 토큰이 만료되면 새로운 토큰을 발급받아야 합니다.

<details>
<summary>예시</summary>

```json
{
  "access": {
    "token": {
      "expires": "{Expires Time}",
      "id": "{token-id}",
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
    "serviceCatalog" : [],
    "user" : {
      "id": "{User UUID}",
      "name": "{User Name}"
    }
  }
}
```

</details>

### 코드 예시
<details>
<summary>cURL</summary>

```
$ curl -X POST -H 'Content-Type:application/json' \
https://gov-api-compute.cloud.toast.com/identity/v2.0/tokens \
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
            "publicURL" : "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****"
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

</details>

<details>
<summary>Java</summary>

```java
// AuthService.java
package com.toast.swift.auth;

// .. import list

@Data
public class AuthService {

    // Inner class for the request body
    @Data
    public class TokenRequest {

        private Auth auth = new Auth();

        @Data
        public class Auth {
            private String tenantId;
            private PasswordCredentials passwordCredentials = new PasswordCredentials();
        }

        @Data
        public class PasswordCredentials {
            private String username;
            private String password;
        }
    }

    private String authUrl;
    private TokenRequest tokenRequest;
    private RestTemplate restTemplate;

    public AuthService(String authUrl, String tenantId, String username, String password) {		
        this.authUrl = authUrl;		

        // 요청 본문 생성
        this.tokenRequest = new TokenRequest();
        this.tokenRequest.getAuth().setTenantId(tenantId);
        this.tokenRequest.getAuth().getPasswordCredentials().setUsername(username);
        this.tokenRequest.getAuth().getPasswordCredentials().setPassword(password);

        this.restTemplate = new RestTemplate();
    }

    public String requestToken() {
        String identityUrl = this.authUrl + "/tokens";

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/json");

        HttpEntity<TokenRequest> httpEntity
            = new HttpEntity<TokenRequest>(this.tokenRequest, headers);

        // 토큰 요청
        ResponseEntity<String> response
            = this.restTemplate.exchange(identityUrl, HttpMethod.POST, httpEntity, String.class);

        return response.getBody();
    }

    public static void main(String[] args) {
        final String authUrl = "https://gov-api-compute.cloud.toast.com/identity/v2.0";
        final String tenantId = "{Tenant ID}";
        final String username = "{TOAST Account}";
        final String password = "{API Password}";

        AuthService authService = new AuthService(authUrl, tenantId, username, password);
        String token = authService.requestToken();

        System.out.println(token);
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# auth.py
import json
import requests


def get_token(auth_url, tenant_id, username, password):
    token_url = auth_url + '/tokens'
    req_header = {'Content-Type': 'application/json'}
    req_body = {
        'auth': {
            'tenantId': tenant_id,
            'passwordCredentials': {
                'username': username,
                'password': password
            }
        }
    }

    response = requests.post(token_url, headers=req_header, json=req_body)
    return response.json()


if __name__ == '__main__':
    AUTH_URL = 'https://gov-api-compute.cloud.toast.com/identity/v2.0'
    TENANT_ID = '{Tenant ID}'
    USERNAME = '{TOAST Account}'
    PASSWORD = '{API Password}'

    token = get_token(AUTH_URL, TENANT_ID, USERNAME, PASSWORD)
    print json.dumps(token, indent=4, separators=(',', ': '))
```

</details>

<details>
<summary>PHP</summary>

```php
// auth.php
<?php
function get_token($auth_url, $tenant_id, $username, $password) {
  $url = "$auth_url/tokens";
  $req_body = array(
      'auth' => array(
          'tenantId' => $tenant_id,
          'passwordCredentials' => array(
              'username' => $username,
              'password' => $password
          )
      )
  );  // 요청 본문 생성
  $req_header = array(
    'Content-Type: application/json'
  );  // 요청 헤더 생성

  $curl  = curl_init($url); // curl 초기화
  curl_setopt_array($curl, array(
    CURLOPT_POST => TRUE,
    CURLOPT_RETURNTRANSFER => TRUE,
    CURLOPT_HTTPHEADER => $req_header,
    CURLOPT_POSTFIELDS => json_encode($req_body)
  )); // 파라미터 설정
  $response = curl_exec($curl); // API 호출
  curl_close($curl);

  return $response;
}

$AUTH_URL = 'https://gov-api-compute.cloud.toast.com/identity/v2.0';
$TENANT_ID = '{Tenant ID}';
$USERNAME = '{TOAST Account}';
$PASSWORD = '{API Password}';

$token = get_token($AUTH_URL, $TENANT_ID, $USERNAME, $PASSWORD);
printf("%s\n", $token);
?>
```

</details>

## 어카운트
OBS 어카운트(Account)는 `AUTH_*****` 형태의 문자열입니다. Object-Store API 엔드포인트에 포함되어 있습니다.

### 어카운트 조회
어카운트의 사용 현황을 조회합니다.

```
HEAD  /v1/{Account}
X-Auth-Token: {token-id}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 사용 현황은 헤더에 포함되어 있습니다. 요청이 올바르면 상태 코드 200을 반환합니다.

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| X-Account-Container-Count | Header | String | 컨테이너 개수 |
| X-Account-Object-Count | Header | String | 저장된 오브젝트 개수 |
| X-Account-Bytes-Used | Header | String | 저장된 데이터 용량 (Bytes) |

#### 코드 예시

<details>
<summary>cURL</summary>

```
$ curl -I -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****
```

</details>

<details>
<summary>Java</summary>

```java
// AccountService.java
package com.toast.swift.service;

// .. import list

@Data
public class AccountService {

    private String tokenId;
    private String storageUrl;
    private RestTemplate restTemplate;

    public AccountService(String storageUrl, String tokenId) {
        this.setStorageUrl(storageUrl);
        this.setTokenId(tokenId);

        this.restTemplate = new RestTemplate();
    }

    public HashMap<String, String> getStatus() {
        String url = this.getStorageUrl();

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출
        HashMap<String, String> status = new HashMap<String, String>();
        ResponseEntity<String> response
            = this.restTemplate.exchange(this.getStorageUrl(), HttpMethod.GET, requestHttpEntity, String.class);
        if (response.getStatusCode() == HttpStatus.OK) {
            HttpHeaders responseHeaders = response.getHeaders();
            status.put("ContainerCount", responseHeaders.getFirst("X-Account-Container-Count"));
            status.put("ObjectCount", responseHeaders.getFirst("X-Account-Object-Count"));
            status.put("BytesUsed", responseHeaders.getFirst("X-Account-Bytes-Used"));
        }
        return status;
    }

    public static void main(String[] args) {
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";

        AccountService accountService = new AccountService(storageUrl, tokenId);

        try {
            HashMap<String, String> status = accountService.getStatus();
            System.out.println(status.toString());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# account.py
import json
import requests


class AccountService:
    def __init__(self, storage_url, token_id):
        self.storage_url = storage_url
        self.token_id = token_id

    def _get_url(self, container):
        return self.storage_url

    def _get_request_header(self):
        return {'X-Auth-Token': self.token_id}

    def get_stat(self):
        req_header = self._get_request_header()
        resp = requests.head(self.storage_url, headers=req_header)
        return resp.headers


if __name__ == '__main__':
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'

    acc_service = AccountService(STORAGE_URL, TOKEN_ID)

    # Get the account status
    stat = acc_service.get_stat()
    print(json.dumps(dict(stat), indent=4))
```

</details>

<details>
<summary>PHP</summary>

```php
// account.php
<?php
class Account {
  private $storage_url;
  private $token_id;

  function __construct($storage_url,  $token_id) {
     $this->storage_url = $storage_url;
     $this->token_id = $token_id;
  }

  function get_request_header(){
    return array(
      'X-Auth-Token: ' . $this->token_id
    );
  }

  function get_status() {
    $req_header = $this->get_request_header();

    $curl = curl_init($this->storage_url); // initialize curl
    curl_setopt_array($curl, array(
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_HTTPHEADER => $req_header,
      CURLOPT_HEADER => TRUE,
    )); // set parameters of curl
    $response = curl_exec($curl); // call api
    curl_close($curl);  // close curl
    $data = explode("\n", $response);

    // parse response headers
    $headers = [];
    foreach($data as $part){
        $middle = explode(":", $part, 2);
        $headers[trim($middle[0])] = trim($middle[1]);
    }

    return $headers;
  }
}

// main
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';

$account = new Account($STORAGE_URL, $TOKEN_ID);

$status = $account->get_status();
printf("Container-Count: %d\n", $status["X-Account-Container-Count"]);
printf("Object-Count: %d\n", $status["X-Account-Object-Count"]);
printf("Bytes-Used: %d\n", $status["X-Account-Bytes-Used"]);
?>

```

</details>

### 컨테이너 목록 조회
어카운트의 컨테이너 목록을 조회합니다.

```
GET  /v1/{Account}
X-Auth-Token: {token-id}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |

#### 응답
```
[어카운트에 속한 컨테이너 목록]
```

#### 코드 예시

<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****
```

</details>

<details>
<summary>Java</summary>

```java
// AccountService.java
package com.toast.swift.service;

// .. import list

@Data
public class AccountService {

  // AccountService Class ...

  public List<String> getContainerList() {
      // 헤더 생성
      HttpHeaders headers = new HttpHeaders();
      headers.add("X-Auth-Token", tokenId);

      HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

      // API 호출
      ResponseEntity<String>response
              = this.restTemplate.exchange(this.getStorageUrl(), HttpMethod.GET, requestHttpEntity, String.class);

      List<String> containerList = null;
      if (response.getStatusCode() == HttpStatus.OK) {
          // String으로 받은 목록을 배열로 변환
          containerList = Arrays.asList(response.getBody().split("\\r?\\n"));
      }
      // 배열을 List로 변환하여 반환
      return new ArrayList<String>(containerList);
  }

  public static void main(String[] args) {
    final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
    final String tokenId = "d052a0a054b745dbac74250b7fecbc09";

      AccountService accountService = new AccountService(storageUrl, tokenId);

      try {
          List<String> containerList = accountService.getContainerList();
          if (containerList != null) {
              for (String object : containerList) {
                  System.out.println(object);
              }
          }
      } catch (Exception e) {
          e.printStackTrace();
      }
  }
}
```

</details>

<details>
<summary>Python</summary>

```python
# account.py
class AccountService:
    # ...

    def get_container_list(self):
      req_header = self._get_request_header()
      resp = requests.get(self.storage_url, headers=req_header)
      return resp.content.split('\n')


if __name__ == '__main__':
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'

    acc_service = AccountService(STORAGE_URL, TOKEN_ID)

    # Get the container list
    container_list = acc_service.get_container_list()
    for container in container_list:
        print(container)
```

</details>

<details>
<summary>PHP</summary>

```php
// account.php
<?php
class Account {
  // ...
  function get_container_list() {
    $req_header = $this->get_request_header();

    $curl  = curl_init($this->storage_url); // initialize curl
    curl_setopt_array($curl, array(
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_HTTPHEADER => $req_header,
    )); // set parameters of curl
    $response = curl_exec($curl); // call api
    curl_close($curl);  // close curl
    $container_list = explode("\n", $response);
    return $container_list;
  }
}

// main
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';

$account = new Account($STORAGE_URL, $TOKEN_ID);

$container_list = $account->get_container_list();
foreach($container_list as $container){
    printf("%s\n", $container);
}
?>
```

</details>

## 컨테이너

### 컨테이너 생성
컨테이너를 생성합니다. 오브젝트 스토리지에 파일을 업로드하려면 반드시 컨테이너를 생성해야 합니다.

> [참고]
> 컨테이너 또는 오브젝트 이름에 특수 문자 `! * ' ( ) ; : @ & = + $ , / ? # [ ]`가 포함되어 있다면 API를 사용할 때 반드시 URL 인코딩(퍼센트 인코딩)을 해야 합니다. 이 문자들은 URL에서 중요하게 사용되는 예약 문자입니다. 이 문자들이 포함된 경로를 URL 인코딩하지 않고 API 요청을 보낸다면 원하는 응답을 받을 수 없습니다.

```
PUT  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 생성할 컨테이너 이름 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 컨테이너가 생성되었다면 상태 코드 201을 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.toast.swift.service;

// .. import list

@Data
public class ContainerService {

    private String tokenId;
    private String storageUrl;
    private RestTemplate restTemplate;

    public ContainerService(String storageUrl, String tokenId) {
        this.setStorageUrl(storageUrl);
        this.setTokenId(tokenId);

        this.restTemplate = new RestTemplate();
    }

    private String getUrl(@NonNull String containerName) {
        return this.getStorageUrl() + "/" + containerName;
    }

    public void createContainer(String containerName) {
        String url = this.getUrl(containerName);

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";

        ContainerService containerService = new ContainerService(storageUrl, tokenId);

        try {
            containerService.createContainer(containerName);
            System.out.println("Container " + containerName + " created");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
import requests


class ContainerService:
    def __init__(self, storage_url, token_id):
        self.storage_url = storage_url
        self.token_id = token_id

    def _get_url(self, container):
        return self.storage_url + '/' + container

    def _get_request_header(self):
        return {'X-Auth-Token': self.token_id}

    def create(self, container):
        req_url = self._get_url(container)
        req_header = self._get_request_header()
        return requests.put(req_url, headers=req_header)


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'

    con_service = ContainerService(STORAGE_URL, TOKEN_ID)

    # Create the container
    new_container = 'test'
    con_service.create(new_container)
```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  private $storage_url;
  private $token_id;

  function __construct($storage_url,  $token_id) {
     $this->storage_url = $storage_url;
     $this->token_id = $token_id;
  }

  function get_url($container = null){
    $url = $this->storage_url;
    if ($container != null) {
      $url .= '/' . $container;
    }
    return $url;
  }

  function get_request_header(){
    return array(
      'X-Auth-Token: ' . $this->token_id
    );
  }

  function create($container){
    $req_url = $this->get_url($container);
    $req_header = $this->get_request_header();

    $curl  = curl_init($req_url);
    curl_setopt_array($curl, array(
      CURLOPT_PUT => TRUE,
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_HTTPHEADER => $req_header
    ));
    $response = curl_exec($curl);
    curl_close($curl);
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->create($CONTAINER_NAME);
?>
```

</details>


### 컨테이너 조회
지정한 컨테이너의 정보와 내부에 저장된 오브젝트들의 목록을 조회합니다.

```
GET   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 조회할 컨테이너 이름 |

#### 응답

```
[지정한 컨테이너에 속한 오브젝트 목록]
```

#### 코드 예시
<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
ba6610.jpg
20d33f.jpg
31466f.jpg
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.toast.swift.service;

// .. import list

public class ContainerService {

    // ContainerService Class ...

    public List<String> getObjectList(String conatinerName) {
        return this.getList(this.getUrl(conatinerName));
    }

    public List<String> getList(String url) {
        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출
        ResponseEntity<String>response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, String.class);

        List<String> objectList = null;
        if (response.getStatusCode() == HttpStatus.OK) {
            // String으로 받은 목록을 배열로 변환
            objectList = Arrays.asList(response.getBody().split("\\r?\\n"));
        }
        // 배열을 List로 변환하여 반환
        return new ArrayList<String>(objectList);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";

        ContainerService containerService = new ContainerService(storageUrl, tokenId);

        List<String> objectList = containerService.getObjectList(containerName);

        if (objectList != null) {
            for (String object : objectList) {
                System.out.println(object);
            }
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
class ContainerService:
    # ...

    def _get_list(self, req_url):
        req_header = self._get_request_header()
        response = requests.get(req_url, headers=req_header)
        return response.content.split('\n')

    def get_object_list(self, container):
        req_url = self._get_url(container)
        return self._get_list(req_url)


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'

    con_service = ContainerService(STORAGE_URL, TOKEN_ID)

    object_list = con_service.get_object_list(CONTAINER_NAME)
    for object in object_list:
        print object
```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  // ...
  function get_list($req_url) {
    $req_header = $this->get_request_header();

    $curl  = curl_init($req_url); // initialize curl
    curl_setopt_array($curl, array(
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_HTTPHEADER => $req_header,
    )); // set parameters of curl
    $response = curl_exec($curl); // call api
    curl_close($curl);  // close curl
    $object_list = explode("\n", $response);
    return $object_list;
  }

  function get_object_list($container, $last_object = null){
    $req_url = $this->get_url($container);
    return $this->get_list($req_url);
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$object_list = $container->get_object_list($CONTAINER_NAME);
foreach ($object_list as $obj){
  printf("%s\n", $obj);
}
?>
```

</details>

### 컨테이너 조회 질의
컨테이너 조회 API는 다음과 같이 몇 가지 질의(query)를 제공합니다. 모든 질의는 `&`로 연결해 혼용할 수 있습니다.

#### 1만 개 이상의 오브젝트 목록 조회
컨테이너 조회 API로 조회할 수 있는 목록의 오브젝트 수는 1만 개로 제한되어 있습니다. 1만 개 이상의 오브젝트 목록을 조회하려면 `marker` 질의를 이용해야 합니다. marker 질의는 지정한 오브젝트의 다음 오브젝트부터 최대 1만 개의 목록을 반환합니다.

```
GET    /v1/{Account}/{Container}?marker={Object}
X-Auth-Token: {token-id}
```

##### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 조회할 컨테이너 이름 |
| Object | Query | String | O | 기준 오브젝트 이름 |

##### 응답
```
[지정한 컨테이너에 속한 지정한 오브젝트 다음 오브젝트 목록]
```

##### 코드 예시
<details>
<summary>cURL</summary>

```
// `20d33f.jpg` 이후의 오브젝트 목록 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?maker=20d33f.jpg
[지정한 오브젝트(20d33f.jpg) 이후의 목록]
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.toast.swift.service;

// ... import list

public class ContainerService {

    // ContainerService Class ...

    public List<String> getObjectList(String conatinerName, String prevLastObject) {
        // 지정한 오브젝트 이름을 이용하여 질의 URL 생성
        String url = this.getUrl(conatinerName) + "?marker=" + prevLastObject;
        // 컨테이너 조회 예제의 getList() 메서드 호출
        return this.getList(url);
    }

    public List<String> getObjectList(String conatinerName) {
        final int LIMIT_COUNT = 10000;

        String url = this.getUrl(conatinerName);

        // 오브젝트 목록 조회
        List<String> objectList = this.getList(url);
        while ((objectList.size() % LIMIT_COUNT) == 0) {
            // 오브젝트 목록의 길이가 1만개의 배수라면 목록의 마지막 오브젝트를 지정하여 이후의 목록 조회
            String lastObject = objectList.get(objectList.size() - 1);
            List<String> nextObjList = this.getObjectList(conatinerName, lastObject);
            objectList.addAll(nextObjList);			
        }		

        return objectList;
    }

    // getObjectList() 사용 예제는 컨테이너 조회와 동일
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
class ContainerService:
    # ...
    _MAX_LIST_COUNT = 10000

    def get_object_list(self, container, last_object=None):
        req_url = self._get_url(container)
        if last_object:
            req_url += '?marker=' + last_object
        return self._get_list(req_url)

    def get_all_object_list(self, container):
        object_list = self.get_object_list(container)
        while (len(object_list) % self._MAX_LIST_COUNT) == 0:
            next_object_list = self.get_object_list(container, object_list[-1])
            object_list.append(next_object_list)
        return object_list

```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  const MAX_LIST_COUNT = 10;

  // ...

  function get_object_list($container, $last_object = null){
    $req_url = $this->get_url($container);
    if ($last_object) {
      $req_url .= '?marker='.last_object;
    }
    return $this->get_list($req_url);
  }

  function get_all_object_list($container){
    $object_list = $this->get_object_list($container);
    while ((count($object_list) % self::MAX_LIST_COUNT) == 0) {
      $next_object_list = $this->get_object_list($container, end($object_list));
      array_merge($object_list, $next_object_list);
    }

    return $object_list;
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$object_list = $container->get_all_object_list($CONTAINER_NAME);
foreach ($object_list as $obj){
  printf("%s\n", $obj);
}
?>
```

</details>

#### 폴더 단위의 오브젝트 목록 조회
컨테이너에 여러 개의 폴더를 만들어 사용하고 있다면 `path` 질의를 이용해 폴더 단위로 오브젝트 목록을 조회할 수 있습니다. path 질의는 하위 폴더의 오브젝트 목록은 조회할 수 없습니다.

```
GET   /v1/{Account}/{Container}?path={Path}
X-Auth-Token: {token-id}
```

##### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 조회할 컨테이너 이름 |
| Path | Query | String | O | 조회할 폴더 이름 |

##### 응답
```
[지정한 컨테이너에 속한 지정한 폴더의 오브젝트 목록]
```

##### 코드 예시
<details>
<summary>cURL</summary>

```
// ex 폴더의 오브젝트 목록 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?path=ex
ex/20d33f.jpg
ex/31466f.jpg
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.toast.swift.service;

// ... import list

public class ContainerService {

    // ContainerService Class ...

    public List<String> getObjectListOfFolder(String conatinerName, String folderName) {
        // 지정한 폴더 이름을 이용하여 질의 URL 생성
        String url = this.getUrl(conatinerName) + "?path=" + folderName;
        // 컨테이너 조회 예제의 getList() 메서드 호출
        return this.getList(url);
    }

    // getObjectListOfFolder() 사용 예제는 컨테이너 조회와 동일
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
class ContainerService:
    # ...
    def get_object_list_of_folder(self, container, folder):
        req_url = self._get_url(container) + "?path=" + folder
        return self._get_list(req_url)

```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  // ...
  function get_object_list_of_folder($container, $folder){
    $req_url = $this->get_url($container)."?path=".$folder;
    return $this->get_list($req_url);
  }
}
?>
```

</details>

#### 접두어로 시작하는 오브젝트 목록 조회
`prefix` 질의를 사용하면 지정한 접두어로 시작하는 오브젝트들의 목록을 반환합니다. path 질의로는 조회할 수 없는 하위 폴더의 오브젝트 목록을 조회하는 데 사용할 수 있습니다.

```
GET   /v1/{Account}/{Container}?prefix={Prefix}
X-Auth-Token: {token-id}
```

##### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 조회할 컨테이너 이름 |
| Prefix | Query | String | O | 검색할 접두어 |

##### 응답
```
[지정한 컨테이너에 속하고 지정한 접두어로 시작하는 오브젝트 목록]
```

##### 코드 예시
<details>
<summary>cURL</summary>

```
// 314로 시작하는 오브젝트 목록 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?prefix=314
3146f0.jpg
3147a6.jpg
31486f.jpg
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.toast.swift.service;

// ... import list

public class ContainerService {

    // ContainerService Class ...

    public List<String> getObjectListWithPrefix(String conatinerName, String prefix) {
        // 지정한 접두어를 이용하여 질의 URL 생성
        String url = this.getUrl(conatinerName) + "?prefix=" + prefix;
        // 컨테이너 조회 예제의 getList() 메서드 호출
        return this.getList(url);
    }

    // getObjectListWithPrefix() 사용 예제는 컨테이너 조회 예제와 동일
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
class ContainerService:
    # ...
    def get_object_list_of_prefix(self, container, prefix):
        req_url = self._get_url(container) + "?prefix=" + prefix
        return self._get_list(req_url)

```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  // ...
  function get_object_list_of_prefix($container, $prefix){
    $req_url = $this->get_url($container)."?prefix=".$prefix;
    return $this->get_list($req_url);
  }
}
?>
```

</details>

#### 목록의 최대 오브젝트 수 지정
`limit` 질의를 사용하면 반환할 오브젝트 목록의 최대 오브젝트 수를 지정할 수 있습니다.

```
GET   /v1/{Account}/{Container}?limit={limit}
X-Auth-Token: {token-id}
```

##### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 조회할 컨테이너 이름 |
| limit | Query | Integer | O | 목록에 표시할 오브젝트 수 |

##### 응답
```
[지정한 컨테이너에 속한 지정된 수 만큼의 오브젝트 목록]
```

##### 코드 예시

<details>
<summary>cURL</summary>

```curl
// 10개의 오브젝트만 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?limit=10
...{9개의 오브젝트}...
31466f0.jpg
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.toast.swift.service;

// ... import list

public class ContainerService {

    // ContainerService Class ...

    public List<String> getObjectList(String conatinerName, int limit) {
        // 지정한 최대 오브젝트 수를 이용하여 질의 URL 생성
        String url = this.getUrl(conatinerName) + "?limit=" + limit;
        // 컨테이너 조회 예제의 getList() 메서드 호출
        return this.getList(url);
    }

    // getObjectListWithPrefix() 사용 예제는 컨테이너 조회 예제와 동일
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
class ContainerService:
    # ...
    def get_object_list_with_limit(self, container, limit=0):
        req_url = self._get_url(container) + "?limit=%d" % limit
        return self._get_list(req_url)

```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  // ...
  function get_object_list_with_limit($container, $limit){
    $req_url = $this->get_url($container)."?limit=".$limit;
    return $this->get_list($req_url);
  }}
}
?>
```

</details>


### 컨테이너 수정

컨테이너의 메타데이터를 변경하여 접근 규칙을 지정할 수 있습니다.

```
POST  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {컨테이너 읽기 정책}
X-Container-Write: {컨테이너 쓰기 정책}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| X-Container-Read | Header | String | - | 컨테이너 읽기에 대한 접근 규칙 지정<br/>.r:* - 모든 사용자에게 접근 허용<br/>.r:example.com,test.com – 특정 주소에만 접근 허용, ‘,’로 구분<br/>.rlistings. – 컨테이너 목록 조회 허용<br/>AUTH_.... – 특정 계정에만 접근 허용 |
| X-Container-Write | Header | String | - | 컨테이너 쓰기에 대한 접근 규칙 지정<br/>\*:\* - 모든 사용자에게 쓰기 허용<br/>AUTH_.... – 특정 계정에만 쓰기 허용 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 수정할 컨테이너 이름 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
// 모든 사용자에게 읽기/쓰기 허용
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Read: .r:*' \
-H 'X-Container-Write: *:*' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java

package com.toast.swift.service;

// ... import list

public class ContainerService {

    // ContainerService Class ...

    public void setContainerReadACL(String containerName, boolean isPublic) {
        final String PUBLIC_ACL = ".r:*";
        final String PRIVATE_ACL = "";

        String permission = isPublic ? PUBLIC_ACL : PRIVATE_ACL;

        String url = this.getUrl(containerName);

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Container-Read", permission);    // 헤더에 권한 추가

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";

        ContainerService containerService = new ContainerService(storageUrl, tokenId);

        try {
            containerService.setContainerReadACL(containerName, true);
            System.out.println("Container " + containerName + " became to public.");
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
class ContainerService:
    # ...
    def set_read_acl(self, container, is_public):
        req_url = self._get_url(container)
        req_header = self._get_request_header()
        req_header['X-Container-Read'] = '.r:*' if is_public else ''
        return requests.post(req_url, headers=req_header)

if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'

    con_service = ContainerService(STORAGE_URL, TOKEN_ID)

    con_service.set_read_acl(CONTAINER_NAME, True)
```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  const PUBLIC_ACL = '.r:*';
  const PRIVATE_ACL = '';  
  // ...  
  function set_acl($container, $is_public){
    $req_url = $this->get_url($container);

    $permission = $is_public ? self::PUBLIC_ACL : self::PRIVATE_ACL;
    $req_header = $this->get_request_header();
    $req_header[] = 'X-Container-Read: ' . $permission;  // 헤더에 권한 추가

    $curl  = curl_init($req_url);
    curl_setopt_array($curl, array(
      CURLOPT_POST => TRUE,
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_HTTPHEADER => $req_header
    ));
    $response = curl_exec($curl);
    curl_close($curl);

    return $response;
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->set_acl($CONTAINER_NAME, TRUE);
?>
```

</details>

#### ACL 확인
읽기 권한을 공개로 설정한 후에는 `curl`, `wget` 등의 도구를 사용하거나 브라우저를 통해 토큰 없이 조회되는지 확인할 수 있습니다.

<details>
<summary>예시</summary>

```
$ curl https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{오브젝트의 내용}
```

</details>

### 컨테이너 삭제

지정한 컨테이너를 삭제합니다. 삭제할 컨테이너는 반드시 비어 있어야 합니다.

```
DELETE   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL| String |	O | 삭제할 컨테이너 이름 |

#### 응답
이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
```

</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java

package com.toast.swift.service;

// ... import list

public class ContainerService {

    // ContainerService Class ...

    public void deleteContainer(String containerName){
        String url = this.getUrl(containerName);

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출        
        this.restTemplate.exchange(url, HttpMethod.DELETE, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";

        ContainerService containerService = new ContainerService(storageUrl, tokenId);

        try {
            containerService.deleteContainer(containerName);
            System.out.println("Container " + containerName + " deleted.");
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
class ContainerService:
    # ...
    def delete(self, container):
        req_url = self._get_url(container)
        req_header = self._get_request_header()
        return requests.delete(req_url, headers=req_header)

if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'

    con_service = ContainerService(STORAGE_URL, TOKEN_ID)

    con_service.delete(CONTAINER_NAME)
```

</details>

<details>
<summary>PHP</summary>

```php
// container.php
<?php
class Container {
  // ...  
  function delete($container){
    $req_url = $this->get_url($container);
    $req_header = $this->get_request_header();

    $curl  = curl_init($req_url); // initialize curl
    curl_setopt_array($curl, array(
      CURLOPT_CUSTOMREQUEST => "DELETE",
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_HTTPHEADER => $req_header
    )); // set parameters of curl
    $response = curl_exec($curl); // call api
    curl_close($curl);  // close curl
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->delete($CONTAINER_NAME);
?>
```

</details>

## 오브젝트

### 오브젝트 업로드
지정한 컨테이너에 새로운 오브젝트를 업로드합니다.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Content-type | Header | String | O | 오브젝트의 콘텐츠 타입 |
| X-Delete-At | Header | Timestamp | - | 오브젝트를 삭제할 유닉스 시간(초) |
| X-Delete-After | Header | Timestamp | - | 오브젝트 유효 시간, 유닉스 시간(초) |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object | URL | String |	O | 생성할 오브젝트 이름 |
| - |	Body | Binary | O | 생성할 오브젝트의 내용 |

> [주의]
> 오브젝트의 이름이 `./` 또는 `../`으로 시작한다면 브라우저가 이를 경로 문자로 인식해 웹 콘솔에서 접근할 수 없습니다.
> API를 이용하여 이러한 이름의 오브젝트를 업로드했다면 API를 통해 접근해야 합니다.

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg \
-T ./ba6610.jpg
```

</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.toast.swift.service;

// ... import list

@Data
public class ObjectService {

    private String tokenId;
    private String storageUrl;
    private RestTemplate restTemplate;

    public ObjectService(String storageUrl, String tokenId) {
        this.setStorageUrl(storageUrl);
        this.setTokenId(tokenId);

        this.restTemplate = new RestTemplate();
    }

    private String getUrl(@NonNull String containerName, @NonNull String objectName) {
        return this.getStorageUrl() + "/" + containerName + "/" + objectName;
    }

    public void uploadObject(String containerName, String objectName, final InputStream inputStream) {
        String url = this.getUrl(containerName, objectName);

        // InputStream을 요청 본문에 추가할 수 있도록 RequestCallback 오버라이드
        final RequestCallback requestCallback = new RequestCallback() {
            public void doWithRequest(final ClientHttpRequest request) throws IOException {
                request.getHeaders().add("X-Auth-Token", tokenId);
                IOUtils.copy(inputStream, request.getBody());
            }
        };

        // 오버라이드한 RequestCallback을 사용할 수 있도록 설정
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setBufferRequestBody(false);
        RestTemplate restTemplate = new RestTemplate(requestFactory);

        HttpMessageConverterExtractor<String> responseExtractor
            = new HttpMessageConverterExtractor<String>(String.class, restTemplate.getMessageConverters());

        // API 호출
        restTemplate.execute(url, HttpMethod.PUT, requestCallback, responseExtractor);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // 파일로 부터 InputStream 생성
            File objFile = new File(objectPath + "/" + objectName);            
            InputStream inputStream = FileUtils.openInputStream(objFile);

            // 업로드
            objectService.uploadObject(containerName, objectName, inputStream);
            System.out.println("\nUpload OK");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

</details>

<details>
<summary>Python</summary>

```python
# object.py
import os
import requests


class ObjectService:
    def __init__(self, storage_url, token_id):
        self.storage_url = storage_url
        self.token_id = token_id

    def _get_url(self, container, object):
        return '/'.join([self.storage_url, container, object])

    def _get_request_header(self):
        return {'X-Auth-Token': self.token_id}

    def upload(self, container, object, object_path):
        req_url = self._get_url(container, object)
        req_header = self._get_request_header()

        path = '/'.join([object_path, object])
        with open(path, 'rb') as f:
            requests.put(req_url, headers=req_header, data=f.read())


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'
    OBJECT_NAME = 'd03bda22ffb649a97958d4a5bf4b6eaf.jpg'
    OBJECT_PATH = '/home/example/'

    obj_service = ObjectService(STORAGE_URL, TOKEN_ID)

    obj_service.upload(CONTAINER_NAME, OBJECT_NAME, OBJECT_PATH)
```

</details>

<details>
<summary>PHP</summary>

```php
// object.php
<?php
class Object {
  private $storage_url;
  private $token_id;

  function __construct($storage_url,  $token_id) {
      $this->storage_url = $storage_url;
      $this->token_id = $token_id;
  }

  function get_url($container, $object){
      return $this->storage_url . '/' . $container . '/' . $object;
  }

  function get_request_header(){
      return array(
          'X-Auth-Token: ' . $this->token_id
      );
  }

  function upload($container, $object, $filename){
      $req_url = $this->get_url($container, $object);

      $req_header = $this->get_request_header();

      $fd = fopen($filename, 'r');  // 파일을 연다.

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
          CURLOPT_PUT => TRUE,
          CURLOPT_RETURNTRANSFER => TRUE,
          CURLOPT_INFILE => $fd,  // 파일 스트림을 매개변수로 넣는다.
          CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);l

      fclose($fd);
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';
$OBJ_PATH = '/home/example';

$object = new Object($STORAGE_URL, $TOKEN_ID);

// upload object
$filename = $OBJ_PATH.'/'.$OBJECT_NAME;
$object->upload($CONTAINER_NAME, $OBJECT_NAME, $filename);
?>
```

</details>

### 멀티 파트 업로드
5GB를 초과하는 용량을 가진 오브젝트는 5GB 이하의 세그먼트로 분할해 업로드해야 합니다.

#### 세그먼트 업로드

```
PUT   /v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

##### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Content-type | Header | String | O | 오브젝트의 콘텐츠 타입 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object |	URL | String | O | 생성할 오브젝트 이름 |
| Count | URL | Integer | O | 분할한 오브젝트의 순번, 예) 001, 002 |
| - |	Body | Binary | O | 분할한 오브젝트의 내용 |

##### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

#### 매니페스트 생성
모든 오브젝트의 세그먼트를 업로드한 다음 매니페스트 오브젝트를 생성하면 하나의 오브젝트처럼 사용할 수 있습니다. 매니페스트 오브젝트는 세그먼트들이 저장된 경로를 가리킵니다.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Container}/{Object}/
```

##### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | 토큰 ID |
| X-Object-Manifest | Header| String | O | 분할한 오브젝트들을 업로드한 경로, `{Container}/{Object}/` |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object |	URL | String | O | 생성할 매니페스트 오브젝트 이름 |
| - | Body| Binary | O | 빈 데이터 |

##### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.


#### 코드 예시
<details>
<summary>cURL</summary>

```
// 200MB 단위로 파일 분할
$ split -d -b 209715200 large_obj.img large_obj.img.

// 분할된 오브젝트 업로드
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/003 \
-T large_obj.img.02

// 매니페스트 오브젝트 업로드
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Object-Manifest: curl_example/large_obj.img/' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img \
-d ''
```

</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.toast.swift.service;

// ... import list

@Data
public class ObjectService {

    // ObjectService Class ...

    // 매니페스트 오브젝트 업로드
    public void uploadManifestObject(String containerName, String objectName) {
        String url = this.getUrl(containerName, objectName);        
        String manifestName = containerName + "/" + objectName + "/"; // 매니페스트 이름 생성

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Object-Manifest", manifestName);  // 헤더에 매니페스트 표기

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";        

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        File objFile = new File(objectPath + "/" + objectName);
        int fileSize = (int)objFile.length();

        final int defaultChunkSize = 100 * 1024; // 100 KB 단위로 분할
        int chunkSize = defaultChunkSize;
        int chunkNo = 0;  // 분할 오브젝트의 이름을 만들기 위한 청크 번호
        int totalBytesRead = 0;

        try {
            // 파일로부터 InputStream 생성
            InputStream inputStream = new BufferedInputStream(new FileInputStream(objFile));			
            while(totalBytesRead < fileSize) {

                // 남은 데이터 크기 계산
                int remainedBytes = fileSize - totalBytesRead;
                if(remainedBytes < chunkSize) {
                    chunkSize = remainedBytes;
                }

                // 바이트 버퍼에 청크 크기만큼 데이터를 읽음
                byte[] chunkBuffer = new byte[chunkSize];
                int bytesRead = inputStream.read(chunkBuffer, 0, chunkSize);

                if(bytesRead > 0) {
                    // 버퍼의 데이터를 InputStream으로 만들어 업로드, 오브젝트 업로드 예제의 uploadObject() 메서드 사용
                    String objPartName = String.format("%s/%03d", objectName, ++chunkNo);
                    InputStream chunkInputStream = new ByteArrayInputStream(chunkBuffer);
                    objectService.uploadObject(containerName, objPartName, chunkInputStream);

                    totalBytesRead += bytesRead;
                }
            }

            // 매니페스트 파일을 업로드
            objectService.uploadManifestObject(containerName, objectName);

            System.out.println("Upload OK");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# object.py
class ObjectService:
    CHUNK_SIZE = 100 * 1024  # 100 KB
    # ...

    def _create_manifest(self, container, object):
        req_url = self._get_url(container, object)
        req_header = self._get_request_header()
        req_header['X-Object-Manifest'] = '/'.join([container, object])
        requests.put(req_url, headers=req_header)

    def upload_large_object(self, container, object, object_path):
        url = self._get_url(container, object)
        req_header = self._get_request_header()

        path = '/'.join([object_path, object])
        with open(path, 'rb') as f:
            chunk_index = 1
            chunk_size = self.CHUNK_SIZE
            total_bytes_read = 0
            obj_size = os.path.getsize(path)

            while total_bytes_read < obj_size:
                remained_bytes = obj_size - total_bytes_read
                if remained_bytes < chunk_size:
                    chunk_size = remained_bytes

                req_url = '%s/%03d' % (url, chunk_index)
                requests.put(
                    req_url, headers=req_header, data=f.read(chunk_size))
                f.seek(chunk_size)
                total_bytes_read += chunk_size
                chunk_index += 1

        self._create_manifest(container, object)


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'
    LARGE_OBJECT = 'dfa10eec828f4a228a34fb4da1d037ff.jpg'
    OBJECT_PATH = '/home/example/'

    obj_service = ObjectService(STORAGE_URL, TOKEN_ID)

    obj_service.upload_large_object(CONTAINER_NAME, LARGE_OBJECT, OBJECT_PATH)
```

</details>

<details>
<summary>PHP</summary>

```php
// object.php
<?php
class Object {
  const CHUNK_SIZE = 100 * 1024;  // 100 KB
  // ...

  function create_manifest($container, $object){
      $req_url = $this->get_url($container, $object);
      $req_header = $this->get_request_header();
      $req_header[] = 'X-Object-Manifest: '.$container.'/'.$object.'/';

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
          CURLOPT_PUT => TRUE,
          CURLOPT_RETURNTRANSFER => TRUE,
          CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
  }

  function upload_large_object($container, $object, $filename){
      $url = $this->get_url($container, $object);
      $req_header = $this->get_request_header();

      $chunk_index = 1;
      $chunk_size = self::CHUNK_SIZE;
      $total_bytes_read = 0;
      $fd = fopen($filename, 'r');  // 파일을 연다.
      $obj_size = filesize($filename);

      while($total_bytes_read < $obj_size){
          // 분할할 용량 계산
          $remained_bytes = $obj_size - $total_bytes_read;
          if ($remained_bytes < $chunk_size){
              $chunk_size = $remained_bytes;
          }
          $chunk = fread($fd, $chunk_size);
          // 파트 이름 생성
          $temp_file = sprintf("./multipart-%03d", $chunk_index);
          $req_url = sprintf("%s/%03d", $url, $chunk_index);

          // 파트 임시 파일 생성
          $part_fd = fopen($temp_file, 'w+');
          fwrite($part_fd, $chunk);
          fseek($part_fd, 0);

          $curl  = curl_init($req_url);
          curl_setopt_array($curl, array(
              CURLOPT_PUT => TRUE,
              CURLOPT_HEADER => TRUE,
              CURLOPT_RETURNTRANSFER => TRUE,
              CURLOPT_INFILE => $part_fd,  // 파트 파일 스트림을 매개 변수로 입력
              CURLOPT_HTTPHEADER => $req_header
          ));
          $response = curl_exec($curl);
          curl_close($curl);
          printf("$response");

          // 임시 파일 삭제
          fclose($part_fd);
          unlink($temp_file);

          $total_bytes_read += $chunk_size;
          $chunk_index += 1;
      }
      fclose($fd);

      $this->create_manifest($container, $object);
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$LARGE_OBJECT = '8cb0d624f8c14c69b52f2cd89e5e59b7.jpg';
$OBJ_PATH = '/home/example';

$object = new Object($STORAGE_URL, $TOKEN_ID);

$filename = $OBJ_PATH.'/'.$LARGE_OBJECT;
$object->upload_large_object($CONTAINER_NAME, $LARGE_OBJECT, $filename);
?>
```

</details>

### 오브젝트 내용 수정
오브젝트 업로드 API와 같지만, 오브젝트가 이미 컨테이너에 있다면 해당 오브젝트의 내용이 수정됩니다.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Content-type | Header | String | O | 오브젝트의 콘텐츠 타입 |
| X-Delete-At | Header | Timestamp | - | 오브젝트를 삭제할 유닉스 시간(초) |
| X-Delete-After | Header | Timestamp | - | 오브젝트 유효 시간, 유닉스 시간(초) |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object | URL | String | O | 내용을 수정할 오브젝트 이름 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

### 오브젝트 다운로드
오브젝트를 다운로드합니다.

```
GET   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object | URL | String | O | 내용을 수정할 오브젝트 이름 |

#### 응답
오브젝트의 내용이 스트림으로 반환됩니다. 요청이 올바르면 상태 코드 200을 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
$ curl -O -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 17166  100 17166    0     0   566k      0 --:--:-- --:--:-- --:--:--  578k
```

</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.toast.swift.service;

// ... import list

@Data
public class ObjectService {

    // ObjectService Class ...

    public InputStream downloadObject(String containerName, String objectName) {
        String url = this.getUrl(containerName, objectName);

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.setAccept(Arrays.asList(MediaType.APPLICATION_OCTET_STREAM));

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출, 데이터를 바이트 배열로 받음
        ResponseEntity<byte[]> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, byte[].class);

        // 바이트 배열 데이터를 InputStream으로 만들어 반환
        return new ByteArrayInputStream(response.getBody());
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";
        final String downloadPath = "/home/example/download";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // 오브젝트 다운로드
            InputStream inputStream = objectService.downloadObject(containerName, objectName);

            // 다운로드한 데이터를 바이트 버퍼로 읽어 들임
            byte[] buffer = new byte[inputStream.available()];
            inputStream.read(buffer);

            // 버퍼의 데이터를 파일에 기록
            File target = new File(downloadPath + "/" + objectName);
            OutputStream outputStream = new FileOutputStream(target);
            outputStream.write(buffer);
            outputStream.close();

            System.out.println("\nDownload OK");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

</details>

<details>
<summary>Python</summary>

```python
# object.py
class ObjectService:
    # ...
    def download(self, container, object, download_path):
        req_url = self._get_url(container, object)
        req_header = self._get_request_header()

        response = requests.get(req_url, headers=req_header)

        dn_path = '/'.join([download_path, object])
        with open(dn_path, 'wb') as f:
            f.write(response.content)


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'
    OBJECT_NAME = 'dfa10eec828f4a228a34fb4da1d037ff.jpg'
    DOWNLOAD_PATH = '/home/example/download'

    obj_service = ObjectService(STORAGE_URL, TOKEN_ID)

    obj_service.download(CONTAINER_NAME, OBJECT_NAME, DOWNLOAD_PATH)
```

</details>

<details>
<summary>PHP</summary>

```php
// object.php
<?php
class Object {
  //...
  function download($container, $object, $filename){
      $req_url = $this->get_url($container, $object);

      $req_header = $this->get_request_header();

      $fd = fopen($filename, 'w');

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
          CURLOPT_RETURNTRANSFER => TRUE,
          CURLOPT_FILE => $fd,
          CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);

      fclose($fd);
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';
$DOWNLOAD_PATH = '/home/example/download';

$object = new Object($STORAGE_URL, $TOKEN_ID);

$filename = $DOWNLOAD_PATH.'/'.$OBJECT_NAME;
$object->download($CONTAINER_NAME, $OBJECT_NAME, $filename);
?>
```

</details>

### 오브젝트 복사
오브젝트를 다른 컨테이너로 복사합니다.

```
COPY   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Destination: {오브젝트를 복사할 대상}
```

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Copy-From: {원본 오브젝트}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Destination | Header | String |	- | 오브젝트를 복사할 대상, `{컨테이너} / {오브젝트}`<br/>COPY 메서드를 사용할 때 필요 |
| X-Copy-From | Header | String |	- | 원본 오브젝트, `{컨테이너} / {오브젝트}`<br/>PUT 메서드를 사용할 때 필요 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O |	컨테이너 이름<br/>COPY 메서드: 원본 컨테이너<br/>PUT 메서드: 복사할 컨테이너 |
| Object | URL | String |	복사할 오브젝트 이름 |

#### 응답
이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
// COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/3a45e9.jpg' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/3a45e9.jpg

// PUT method
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Copy-From: curl_example/3a45e9.jpg' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/copy_con/3a45e9.jpg
```

</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.toast.swift.service;

// ... import list

@Data
public class ObjectService {

    // ObjectService Class ...

    public void copyObject(String srcContainerName, String objectName, String destContainerName) {
        String url = this.getUrl(destContainerName, objectName);
        String srcObject = "/" + srcContainerName + "/" + objectName;

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Copy-From", srcObject);    // 원본 오브젝트 지정
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // HttpMethod는 COPY 메서드를 지원하지 않아 PUT 메서드를 사용하는 대체 API를 호출한다.
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);			
    }    

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String srcContainerName = "test";
        final String destContainerName = "test2";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            objectService.copyObject(srcContainerName, objectName, destContainerName);
            System.out.println("Copy OK");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# object.py
class ObjectService:
    # ...
    def copy(self, src_container, object, dest_container):
        req_url = self._get_url(dest_container, object)
        req_header = self._get_request_header()
        req_header['X-Copy-From'] = '/'.join([src_container, object])
        requests.put(req_url, headers=req_header)


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'
    OBJECT_NAME = 'dfa10eec828f4a228a34fb4da1d037ff.jpg'
    DEST_CONTAINER = 'dest'

    obj_service = ObjectService(STORAGE_URL, TOKEN_ID)

    obj_service.copy(CONTAINER_NAME, OBJECT_NAME, DEST_CONTAINER)
```

</details>

<details>
<summary>PHP</summary>

```php
// object.php
<?php
class Object {
  //...
  function copy($src_container, $object, $dest_container){
      $req_url = $this->get_url($dest_container, $object);

      $req_header = $this->get_request_header();
      $req_header[] = 'X-Copy-From: '.$src_container.'/'.$object;

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
          CURLOPT_PUT => TRUE,
          CURLOPT_RETURNTRANSFER => TRUE,
          CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);l
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new Object($STORAGE_URL, $TOKEN_ID);

$META_KEY = 'Type';
$META_VALUE = 'photo';
$object->set_metadata($CONTAINER_NAME, $OBJECT_NAME, $META_KEY, $META_VALUE);
?>
```

</details>

### 오브젝트 메타데이터 수정
지정한 오브젝트의 메타데이터를 수정합니다.

```
POST   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Meta-{Key}: {Value}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| X-Object-Meta-{Key} | Header | String | - | 변경할 메타데이터 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL| String |	 O | 컨테이너 이름 |
| Object | URL| String |  O | 메타데이터를 수정할 오브젝트 이름 |

#### 응답
이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 202를 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
// 오브젝트에 메타 데이터 추가
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg

// 오브젝트 헤더에서 추가한 메타데이터 확인
$ curl -I -H "X-Auth-Token: b587ae461278419da6ecd21a2344c8aa" \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg
HTTP/1.1 200 OK
...
X-Object-Meta-Type: photo
...
```

</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.toast.swift.service;

// ... import list

@Data
public class ObjectService {

    // ObjectService Class ...

    public void setObjectMetadata(String containerName, String objectName, String key, String value) {
        String url = this.getUrl(containerName, objectName);

        // 메타데이터 키 생성
        String metaKey = "X-Object-Meta-" + key;

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add(metaKey, value);    // 헤더에 메타데이터 설정

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";
        final String key = "Type";
        final String value = "photo";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            objectService.setObjectMetadata(containerName, objectName, key, value);
            System.out.println("Set metadata OK");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }    
}
```

</details>

<details>
<summary>Python</summary>

```python
# object.py
class ObjectService:
    # ...
    def set_metadata(self, container, object, key, value):
        req_url = self._get_url(container, object)
        req_header = self._get_request_header()
        req_header['X-Object-Meta-' + key] = value
        requests.post(req_url, headers=req_header)


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'
    OBJECT_NAME = 'dfa10eec828f4a228a34fb4da1d037ff.jpg'
    META_KEY = 'Type'
    META_VALUE = 'photo'

    obj_service = ObjectService(STORAGE_URL, TOKEN_ID)

    obj_service.set_metadata(CONTAINER_NAME, OBJECT_NAME, META_KEY, META_VALUE)    
```

</details>

<details>
<summary>PHP</summary>

```php
<?php
class Object {
  //...
  function set_metadata($container, $object, $key, $value){
      $req_url = $this->get_url($container, $object);
      $req_header = $this->get_request_header();
      $req_header[] = 'X-Object-Meta-'.$key.': '.$value;  // 헤더에 메타데이터 추가

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
          CURLOPT_POST => TRUE,
          CURLOPT_RETURNTRANSFER => TRUE,
          CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$DEST_CONTAINER = 'dest';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new Object($STORAGE_URL, $TOKEN_ID);

$object->copy($CONTAINER_NAME, $OBJECT_NAME, $DEST_CONTAINER);
?>
```

</details>

### 오브젝트 삭제
지정한 오브젝트를 삭제합니다.

```
DELETE   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| X-Object-Meta-{Key} | Header | String | - | 변경할 메타데이터 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL| String |	 O | 컨테이너 이름 |
| Object | URL| String |  O | 메타데이터를 수정할 오브젝트 이름 |

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL| String |	 O | 컨테이너 이름 |
| Object | URL| String |  O | 삭제할 오브젝트 이름 |

#### 응답
이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

#### 코드 예시
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://gov-api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg
```

</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.toast.swift.service;

// ... import list

@Data
public class ObjectService {

    // ObjectService Class ...

    public void deleteObject(String containerName, String objectName) {
        String url = this.getUrl(containerName, objectName);

        // 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);		
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 호출
        this.restTemplate.exchange(url, HttpMethod.DELETE, requestHttpEntity, String.class);			
    }

    public static void main(String[] args) {
        final String storageUrl = "https://gov-api-storage.cloud.toast.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            objectService.deleteObject(containerName, objectName);
            System.out.println("Delete OK");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

</details>

<details>
<summary>Python</summary>

```python
# object.py
class ObjectService:
    # ...
    def delete(self, container, object):
        req_url = self._get_url(container, object)
        req_header = self._get_request_header()
        requests.delete(req_url, headers=req_header)


if __name__ == '__main__':
    STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'
    OBJECT_NAME = 'dfa10eec828f4a228a34fb4da1d037ff.jpg'

    obj_service = ObjectService(STORAGE_URL, TOKEN_ID)

    obj_service.delete(CONTAINER_NAME, OBJECT_NAME)   
```

</details>

<details>
<summary>PHP</summary>

```php
// object.php
<?php
class Object {
  //...
  function delete($container, $object){
      $req_url = $this->get_url($container, $object);
      $req_header = $this->get_request_header();

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
          CURLOPT_CUSTOMREQUEST => "DELETE",
          CURLOPT_RETURNTRANSFER => TRUE,
          CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
  }
}

// main
$STORAGE_URL = 'https://gov-api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new Object($STORAGE_URL, $TOKEN_ID);

$object->delete($CONTAINER_NAME, $OBJECT_NAME);
?>
```

</details>

## References

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
