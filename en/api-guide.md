## Storage > Object Storage > API Guide

## Prerequisites

### Check Tenant Name

To use API, enter tenant name in parameters. Tenant name refers to **Project ID** which is available on the Project Setting page.  

1. Click **Project Setting** under the organiz## Storage > Object Storage > API Guide

## Prerequities

To enable object storage API, an authentication token must be issued first. Authentication token is required to use REST API of object storage: it is a must to access container or object which is not open to public. Tokens are managed by each account. 

### Check Tenant ID and API Endpoint 

Click **API Endpoint Setting** on the object storage service page to check tenant ID and API endpoint to issue a token. 

| Item | API Endpoint | Usage |
|---|---|---|
| Identity | https://api-compute.cloud.toast.com/identity/v2.0 | Issue certificate token |
| Object-Store | https://api-storage.cloud.toast.com/v1/{Account} | Control object storage: depends on each region  |
| Tenant ID | Character strings composed of 32 characters in combination of numbers and alphabets | Issue certificate token  |

> [Note]
> User account for API refers to character strings in the format of `AUTH_***`, which is included in Object-Store API endpoint.

### Set API Password 

To set API password, go to the object storage service page and click **API Endpoint Setting**. 

1. Click **API Endpoint Setting**.
2. Go to **API Password Setting** under **API Endpoint Setting** and enter password to issue a token. 
3. Click **Save**. 

## Certificate Token Issuance

```
POST    https://api-compute.cloud.toast.com/identity/v2.0/tokens
Content-Type: application/json
```

### Request

| Name  | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | Body | String | O | Tenant ID, to be found on the setup box for API Endpoint |
| username | Body | String | O | Enter ID (email) of TOAST Account |
| password | Body | String | O | Password saved on the setup box for API Endpoint  |

<details>
<summary>Example</summary>

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

### Response

| Name | Type | Format | Description |
|---|---|---|---|
| access.token.id | Body | String |	ID of issued token |
| access.token.tenant.id | Body | String | Tenant ID for a project requesting for token |
| access.token.expires | Body | String | Expiration time of issued token <br/> in the ssZ:MM:HHTdd-mm-yyyy format. e.g.) 50Z:17:03T16-05-2017 |

> [Caution]
> A token includes expiration. The 'expires' item included in the reponse to request for token issuance refers to token expiration time. When a token is expired, a new token must be issued.  

<details>
<summary>Example</summary>

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

### Code Example
<details>
<summary>cURL</summary>

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

        // Create a request body 요청 본문 생성
        this.tokenRequest = new TokenRequest();
        this.tokenRequest.getAuth().setTenantId(tenantId);
        this.tokenRequest.getAuth().getPasswordCredentials().setUsername(username);
        this.tokenRequest.getAuth().getPasswordCredentials().setPassword(password);

        this.restTemplate = new RestTemplate();
    }

    public String requestToken() {
        String identityUrl = this.authUrl + "/tokens";

        // Create a header 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/json");

        HttpEntity<TokenRequest> httpEntity 
            = new HttpEntity<TokenRequest>(this.tokenRequest, headers);

        // Request for a token 토큰 요청
        ResponseEntity<String> response
            = this.restTemplate.exchange(identityUrl, HttpMethod.POST, httpEntity, String.class);

        return response.getBody();
    }

    public static void main(String[] args) {
        final String authUrl = "https://api-compute.cloud.toast.com/identity/v2.0";
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
    AUTH_URL = 'https://api-compute.cloud.toast.com/identity/v2.0'
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
  );  // 요청 본문 생성 Create a request body 
  $req_header = array(
    'Content-Type: application/json'
  );  // 요청 헤더 생성 Create a request header

  $curl  = curl_init($url); // Initialize curl 초기화
  curl_setopt_array($curl, array(
    CURLOPT_POST => TRUE,
    CURLOPT_RETURNTRANSFER => TRUE,
    CURLOPT_HTTPHEADER => $req_header,
    CURLOPT_POSTFIELDS => json_encode($req_body)
  )); // Set parameters 파라미터 설정
  $response = curl_exec($curl); // All API 호출
  curl_close($curl);

  return $response;
}

$AUTH_URL = 'https://api-compute.cloud.toast.com/identity/v2.0';
$TENANT_ID = '{Tenant ID}';
$USERNAME = '{TOAST Account}';
$PASSWORD = '{API Password}';

$token = get_token($AUTH_URL, $TENANT_ID, $USERNAME, $PASSWORD);
printf("%s\n", $token);
?>
```

</details>

## Container

### 컨테이너 생성 Create Container
Create a container. To upload files to object storage, a container must be created. 

> [Note]
> If a container or object name includes special characters such as `! * ' ( ) ; : @ & = + $ , / ? # [ ]`, it must be URL encoded (percent-encoding). These are reserved characters that are considered important for URL. Unless the paths including the characters are encoded before sending an API request, you may not receive response as needed. 

```
PUT  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body. 

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, to be found on the setup box for API Endpoint |
| Container | URL | String | O | Name of container to be created  |

#### Response
This API does not return response body. When a container is created, return status code 201. 

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
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

        // Create a header 헤더 생성
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API 호출
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->create($CONTAINER_NAME);
?>
```

</details>


### Get Container
Get container information as specified and list objects that are saved within. 

```
GET   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body. 

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, to be found on the setup box for API Endpoint |
| Container | URL | String | O | Container name to get |

#### Response

```
[List of objects included to a specified container]
```

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
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
        // 헤더 생성 Create a header 
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API 호출
        ResponseEntity<String>response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, String.class);

        List<String> objectList = null;
        if (response.getStatusCode() == HttpStatus.OK) {
            // Convert list on the string into sequence 으로 받은 목록을 배열로 변환
            objectList = Arrays.asList(response.getBody().split("\\r?\\n"));
        }
        // 배열을 List로 변환하여 반환 Convert the sequence into list and return 
        return new ArrayList<String>(objectList);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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

### 컨테이너 조회 질의 Query for Get Container 
Get Container API provides some queries as follows. All queries can be connected with '&' for common use. 컨테이너 조회 API는 다음과 같이 몇 가지 질의(query)를 제공합니다. 모든 질의는 `&`로 연결해 혼용할 수 있습니다. 

#### 1만 개 이상의 오브젝트 목록 조회 List More than 10,000 Objects
컨테이너 조회 API로 조회할 수 있는 목록의 오브젝트 수는 1만 개로 제한되어 있습니다. 1만 개 이상의 오브젝트 목록을 조회하려면 `marker` 질의를 이용해야 합니다. marker 질의는 지정한 오브젝트의 다음 오브젝트부터 최대 1만 개의 목록을 반환합니다. No more than 10,000 objects can be listed with Get Container API. To list more than 10,000 objects, use the 'marker' query. The marker query returns up to 10,000 object lists from the next object after a specified object. 

```
GET    /v1/{Account}/{Container}?marker={Object}
X-Auth-Token: {token-id}
```

##### Request
This API does not require a request body. 

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, to be found on the set up box for API Endpoint |
| Container | URL | String | O | Container name to query |
| Object | Query | String | O | 기준 오브젝트 이름 |

##### Response
```
[Object list next to a specified object which is included to a specified container  지정한 컨테이너에 속한 지정한 오브젝트 다음 오브젝트 목록]
```

##### Code Example
<details>
<summary>cURL</summary>

```
// `20d33f.jpg` 이후의 오브젝트 목록 조회 List objects after '20d33f.jpg'
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?maker=20d33f.jpg
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

        // 오브젝트 목록 조회 List objects 
        List<String> objectList = this.getList(url);
        while ((objectList.size() % LIMIT_COUNT) == 0) {
            // If the length of object list is the multiples of 10,000, specify the last object on the list to list  오브젝트 목록의 길이가 1만개의 배수라면 목록의 마지막 오브젝트를 지정하여 이후의 목록 조회
            String lastObject = objectList.get(objectList.size() - 1);
            List<String> nextObjList = this.getObjectList(conatinerName, lastObject);
            objectList.addAll(nextObjList);			
        }		

        return objectList;
    }

    // The usage example of getObjectList() is same as get container  사용 예제는 컨테이너 조회와 동일
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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

#### 폴더 단위의 오브젝트 목록 조회 List Objects by Folder 
컨테이너에 여러 개의 폴더를 만들어 사용하고 있다면 `path` 질의를 이용해 폴더 단위로 오브젝트 목록을 조회할 수 있습니다. path 질의는 하위 폴더의 오브젝트 목록은 조회할 수 없습니다. If a container has many folders, use the 'path' query to list objects by folder. The path query cannot list objects of the lower-level folder. 

```
GET   /v1/{Account}/{Container}?path={Path}
X-Auth-Token: {token-id}
```

##### Request
This API does not require a request body. 

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name for Windows, to be found on the setup box for API Endpoint  |
| Container | URL | String | O | Container name to query 조회할 컨테이너 이름 |
| Path | Query | String | O | Folder name to query 조회할 폴더 이름 |

##### Response
```
[Object list of a specified folder which is included to a specified container 지정한 컨테이너에 속한 지정한 폴더의 오브젝트 목록]
```

##### Code Example
<details>
<summary>cURL</summary>

```
// ex 폴더의 오브젝트 목록 조회 List objects of the ex folder 
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?path=ex
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
        // Create query URL by using specified folder name 지정한 폴더 이름을 이용하여 질의 URL 생성
        String url = this.getUrl(conatinerName) + "?path=" + folderName;
        // Call the 컨테이너 조회 예제의 getList() method from the get container example 메서드 호출
        return this.getList(url);
    }

    // The getObjectListOfFolder() usage example is same as get container 사용 예제는 컨테이너 조회와 동일
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

#### 접두어로 시작하는 오브젝트 목록 조회 List Objects starting with Prefix 
`prefix` 질의를 사용하면 지정한 접두어로 시작하는 오브젝트들의 목록을 반환합니다. path 질의로는 조회할 수 없는 하위 폴더의 오브젝트 목록을 조회하는 데 사용할 수 있습니다.

```
GET   /v1/{Account}/{Container}?prefix={Prefix}
X-Auth-Token: {token-id}
```

##### Request
This API does not require a request body. 는 요청 본문을 요구하지 않습니다.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Toekn ID |
| Account | URL | String | O | User account name, to be found on the setup box for API Endpoint |
| Container | URL | String | O | Container name to query 조회할 컨테이너 이름 |
| Prefix | Query | String | O | Prefix to search 검색할 접두어 |

##### Response
```
[지정한 컨테이너에 속하고 지정한 접두어로 시작하는 오브젝트 목록]
```

##### Code Example
<details>
<summary>cURL</summary>

```
// 314로 시작하는 오브젝트 목록 조회 List objects starting with 314 
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?prefix=314
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
        // 지정한 접두어를 이용하여 질의 URL 생성 Create query URL by using specified prefix 
        String url = this.getUrl(conatinerName) + "?prefix=" + prefix;
        // Call 컨테이너 조회 예제의 getList() method of the get container example 메서드 호출 
        return this.getList(url);
    }

    // Usage example of getObjectListWithPrefix() is same as get container example 사용 예제는 컨테이너 조회 예제와 동일
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

#### 목록의 최대 오브젝트 수 지정 Specify the Maximum Object Count on List 
`limit` 질의를 사용하면 반환할 오브젝트 목록의 최대 오브젝트 수를 지정할 수 있습니다. Use the 'limit' query to specify the maximum object count of the list to return 

```
GET   /v1/{Account}/{Container}?limit={limit}
X-Auth-Token: {token-id}
```

##### Request
This API does not require a request body. 

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, to be found on the setup box for API Endpoint |
| Container | URL | String | O | Container name to query 조회할 컨테이너 이름 |
| limit | Query | Integer | O | Object count to show on the list 목록에 표시할 오브젝트 수 |

##### Response
```
[지정한 컨테이너에 속한 지정된 수 만큼의 오브젝트 목록]
```

##### Code Example

<details>
<summary>cURL</summary>

```curl
// 10개의 오브젝트만 조회
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example?limit=10
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


### 컨테이너 수정 Modify Container

컨테이너의 메타데이터를 변경하여 접근 규칙을 지정할 수 있습니다.

```
POST  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {컨테이너 읽기 정책}
X-Container-Write: {컨테이너 쓰기 정책}
```

#### Request
This API does not require a request body.  API는 요청 본문을 요구하지 않습니다.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| X-Container-Read | Header | String | - | 컨테이너 읽기에 대한 접근 규칙 지정<br/>.r:* - 모든 사용자에게 접근 허용<br/>.r:example.com,test.com – 특정 주소에만 접근 허용, ‘,’로 구분<br/>.rlistings. – 컨테이너 목록 조회 허용<br/>AUTH_.... – 특정 계정에만 접근 허용 |
| X-Container-Write | Header | String | - | 컨테이너 쓰기에 대한 접근 규칙 지정<br/>\*:\* - 모든 사용자에게 쓰기 허용<br/>AUTH_.... – 특정 계정에만 쓰기 허용 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O | 수정할 컨테이너 이름 |

#### Response
This API does not return response body. When the request is appropriate, return status code 204. 이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

#### Code Example
<details>
<summary>cURL</summary>

```
// 모든 사용자에게 읽기/쓰기 허용 Allow Read/Wrote for all users 
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Read: .r:*' \
-H 'X-Container-Write: *:*' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
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

        // 헤더 생성 Create a header 
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Container-Read", permission);    // 헤더에 권한 추가 Add authority to header 

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API 호출
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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
<summary>Example</summary>

```
$ curl https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{오브젝트의 내용}
```

</details>

### 컨테이너 삭제 Delete Container

지정한 컨테이너를 삭제합니다. 삭제할 컨테이너는 반드시 비어 있어야 합니다.

```
DELETE   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body. 

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL| String |	O | 삭제할 컨테이너 이름 |

#### Response
This request does not return response body. When request is appropriate, return status code 204. 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example
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
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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

#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Content-type | Header | String | O | Content type of object 오브젝트의 콘텐츠 타입 |
| X-Delete-At | Header | Timestamp | - | Unix time (seconds) to delete object (오브젝트를 삭제할 유닉스 시간(초) |
| X-Delete-After | Header | Timestamp | - | 오브젝트 유효 시간, 유닉스 시간(초) |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object | URL | String |	O | 생성할 오브젝트 이름 |
| - |	Body | Binary | O | 생성할 오브젝트의 내용 |

> [Caution]
> 오브젝트의 이름이 `./` 또는 `../`으로 시작한다면 브라우저가 이를 경로 문자로 인식해 웹 콘솔에서 접근할 수 없습니다.
> API를 이용하여 이러한 이름의 오브젝트를 업로드했다면 API를 통해 접근해야 합니다.

#### Response
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg \
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
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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

##### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Content-type | Header | String | O | Content type of object 오브젝트의 콘텐츠 타입 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | Container name |
| Object |	URL | String | O | 생성할 오브젝트 이름 |
| Count | URL | Integer | O | 분할한 오브젝트의 순번, 예) 001, 002 |
| - |	Body | Binary | O | 분할한 오브젝트의 내용 |

##### Response
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

#### 매니페스트 생성
모든 오브젝트의 세그먼트를 업로드한 다음 매니페스트 오브젝트를 생성하면 하나의 오브젝트처럼 사용할 수 있습니다. 매니페스트 오브젝트는 세그먼트들이 저장된 경로를 가리킵니다.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Container}/{Object}/
```

##### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | Token ID |
| X-Object-Manifest | Header| String | O | 분할한 오브젝트들을 업로드한 경로, `{Container}/{Object}/` |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object |	URL | String | O | 생성할 매니페스트 오브젝트 이름 |
| - | Body| Binary | O | 빈 데이터 |

##### Response
This API does not return response body. When the request is appropriate, return status code 201. 이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.


#### Code Example
<details>
<summary>cURL</summary>

```
// 200MB 단위로 파일 분할
$ split -d -b 209715200 large_obj.img large_obj.img.

// 분할된 오브젝트 업로드
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img/003 \
-T large_obj.img.02

// 매니페스트 오브젝트 업로드
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Object-Manifest: curl_example/large_obj.img/' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/large_obj.img \
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
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Content-type | Header | String | O | 오브젝트의 콘텐츠 타입 |
| X-Delete-At | Header | Timestamp | - | 오브젝트를 삭제할 유닉스 시간(초) |
| X-Delete-After | Header | Timestamp | - | 오브젝트 유효 시간, 유닉스 시간(초) |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object | URL | String | O | 내용을 수정할 오브젝트 이름 |

#### Response
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

### 오브젝트 다운로드
오브젝트를 다운로드합니다.

```
GET   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

#### Request
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container |	URL | String | O | 컨테이너 이름 |
| Object | URL | String | O | 내용을 수정할 오브젝트 이름 |

#### Response
오브젝트의 내용이 스트림으로 반환됩니다. 요청이 올바르면 상태 코드 200을 반환합니다.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -O -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg

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
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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

#### Request
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 토큰 ID |
| Destination | Header | String |	- | 오브젝트를 복사할 대상, `{컨테이너} / {오브젝트}`<br/>COPY 메서드를 사용할 때 필요 |
| X-Copy-From | Header | String |	- | 원본 오브젝트, `{컨테이너} / {오브젝트}`<br/>PUT 메서드를 사용할 때 필요 |
| Account | URL | String | O | 사용자 계정명, API Endpoint 설정 대화 상자에서 확인 |
| Container | URL | String | O |	컨테이너 이름<br/>COPY 메서드: 원본 컨테이너<br/>PUT 메서드: 복사할 컨테이너 |
| Object | URL | String |	복사할 오브젝트 이름 |

#### Response
이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 201을 반환합니다.

#### Code Example
<details>
<summary>cURL</summary>

```
// COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/3a45e9.jpg' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/3a45e9.jpg

// PUT method
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Copy-From: curl_example/3a45e9.jpg' \
https://api-storage.cloud.toast.com/v1/AUTH_*****/copy_con/3a45e9.jpg
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
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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

#### Request
This API does not require a request body.  요청 본문을 요구하지 않습니다.

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
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg

// 오브젝트 헤더에서 추가한 메타데이터 확인
$ curl -I -H "X-Auth-Token: b587ae461278419da6ecd21a2344c8aa" \
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg
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
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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
https://api-storage.cloud.toast.com/v1/AUTH_*****/curl_example/ba6610.jpg
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
        final String storageUrl = "https://api-storage.cloud.toast.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://api-storage.cloud.toast.com/v1/AUTH_*****';
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
ation name on the left of the web console.
2. Check **Project ID**.

### Check API Endpoint

To check API endpoint, click **API Endpoint Setting** on the Object Storage service page.

| Item | API Endpoint | Usage |
|---|---|---|
| Object-Store | https://api-storage.cloud.toast.com/v1/{Account} | Control object storage |
| Identity | https://api-compute.cloud.toast.com/identity/v2.0 | Issue authentication token |

> [Note]  
> User account for API is composed of character strings in the `AUTH_***` format: included in the Object-Store API endpoint.

### Set API Password

To set API password, click **API Endpoint Setting** on the Object Storage service page.

1. Click **API Endpoint Setting**.
2. Enter password to issue a token in **API Password Setting** under **API Endpoint Setting**.
3. Click **Save**.

## Certificate Token Issuance

A certificate token is required to use REST API of object storage: it is a must to access container or object which is not open to public. Tokens are managed by each account.

**[Method, URL]**
```
POST    https://api-compute.cloud.toast.com/identity/v2.0/tokens
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|tenantName|	Body or Plain|	String| TOAST Project ID |
|username|	Plain|	String| Enter TOAST account ID (email) |
|password|	Plain|	String| Password saved in **API Endpoint Setting** |

**[Request Body]**
```
{
  "auth": {
    "tenantName": "{Project ID}",
    "passwordCredentials": {
      "username": "{TOAST ID}",
      "password": "{API Password}"
    }
  }
}
```

**[Response Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|access.token.id|	Body or Plain|	String| ID of issued token |
|access.token.tenant.id|	Plain|	String| Tenant ID in response to the project requesting for a token |
|access.token.expires|	Plain|	String| Expiration time of an issued token in the format of yyyy-mm-ddTHH:MM:ssZ. e.g) 2017-05-16T03:17:50Z |

**[Response Body Example]**
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

> [Note]  
> Token expires after valid period. "expires" included in the response to request for token issuance refers to token expiration time. If a token is expired, a new token needs to get issued.


## Containers

### Create Container
To upload files to object storage, containers must be created.

> [Note]
> If a container or object name includes special characters, such as `! * ' ( ) ; : @ & = + $ , / ? # [ ]`, URL encoding (percent encoding) is a must to be applied for APIs. They are important reservation characters for URL. Unless the path of these characters are requested to an API without URL encoded, you may not be returned with a response you need


**[Method, URL]**
```
PUT https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name|Type|Attribute|Description|
|---|---|---|---|
|X-Auth-Token|Header|String|ID of issued token|
|Account|URL|String|User account name: included in API Endpoint|
|Container|URL|String|Container name to create|

> [Note]
> The API does not return response body: return status code 201, if a container has been created.

### Retrieve Containers
Retrieve information of a specified container along with the list of objects saved in it.

**[Method, URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name|Type|Attribute|Description|
|---|---|---|---|
|X-Auth-Token|Header|String|ID of issued token|
|Container|URL|String|Container name to retrieve|

**[Response Body]**
```
[List of objects included in a specified container]
```

### Modify Containers

Specify access rules by changing meta data of a container.

**[Method, URL]**
```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
X-Container-Read: {Read Container Policy}
X-Container-Write: {Write Container Policy}
```

**[Request Parameters]**

|Name|Type|Attribute|Description|
|---|---|---|---|
|X-Auth-Token|Header|String|ID of issued token|
|X-Container-Read|Header|String|Specify access rules to read container <br/>.r:* - Allow access for all users <br/>.r:example.com,test.com – Allow access for particular address, delimited by ',' <br/>.rlistings. – Allow retrieval of container list <br/>AUTH_.... – Allow access for particular accounts|
|X-Container-Write|Header|String|Specify access rules for write containers|
|Account|URL|String|User account name: included in API Endpoint|
|Container|URL|String|Container name to modify|

**[Request Example]**
```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
X-Container-Read: .r:*
```

> [Note]
> This API does not return response body: return status code 204 if properly requested.

After read authority is set public, check if retrieval without token is available by using tools, like `curl`and `wget`, or browser.

**[Verification Example]**
```
$ curl https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{content of object}
```

### Delete Container

Delete containers as specified.

**[Method, URL]**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name to delete |

> [Note]
> This request does not return response body. Container to delete must be empty. Return status code 204 if properly requested.

## Object

### Upload Object

Upload new objects to containers as specified.

**[Method, URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to create |
|-|	Body|	Plain| Content of object to which text is to be created |

> [Note]
> Set appropriate content-type items for parameter attributes, at the request header. Return status code 201, if properly requested.   


### Upload Multiple Parts

Objects exceeding 5GB must be uploaded in segments with less than 5GB .

**[Method, URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: [Token ID]
```

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint.|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to create |
|Count| URL| String| Order of segmented objects |
|-|	Body|	Plain| Content of objects of which text is segmented |


Upload segments of all objects and create a manifest object, so as to apply as one object. Manifest object refers to the route where segments are saved.

**[Method, URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
X-Object-Manifest: {Container}/{Object}/
```

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|X-Object-Manifest| Header| String | Uploaded route for segmented objects, `{Container}/{Object}/` |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to create |
|-|	Body|	Plain| Content of object of which text is segmented Text |


**[Example]**
```
// Upload Segmented Objects
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/001 --data-binary '.....'
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/002 --data-binary '.....'

// Upload Manifest Objects
$ curl -X PUT -H 'X-Auth-Token: *****' -H 'X-Object-Manifest: con/sample.jpg/' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg --data-binary ''

// Download Objects
$ curl http://10.162.50.125/v1/AUTH_*****/con/sample.jpg > sample.jpg
```

### Modify Objects

If an object is already in a container, although its upload API is the same, the content shall change.

**[Method, URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Name of object of which content is to be modified |

> [Note]
> Content-type item appropriate for an object attribute must be set at the request header: return status code 201 if properly requested.  

### Download Objects

**[Method, URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to download |

> [Note]
> Object content is returned to stream: return status code 200 if properly requested.

### Copy Objects

**[Method, URL]**
```
COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Destination|	Header|	String| Target of copying for object, `{Container name} / {Name of copied object}` |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to copy |


> [Note]
> This request does not return response body: return status code 201 if properly requested.

### Modify Object Meta Data

**[Method, URL]**
```
POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Content-Type|	Header|	String| Object format to change |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name of which attribute is to be modified |

> [Note]
> This request does not return response body: return status code 202 if properly requested.

### Delete Objects

**[Method, URL]**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**[Request Parameters]**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to delete |

> [Note]
> This request does not return response body: return status code 204 if properly requested.

## Reference

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
