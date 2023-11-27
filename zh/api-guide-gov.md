## Storage > Object Storage > API Guide

## Prerequisites

To enable object storage API, an authentication token must be issued first. Authentication token is required to use REST API of object storage: it is a must to access container or object which is not open to public. Tokens are managed by each account.

### Check Tenant ID and API Endpoint

Click **API Endpoint Setting** on the object storage service page to check tenant ID and API endpoint to issue a token.

| Item | API Endpoint | Usage |
|---|---|---|
| Identity | https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0 | Issue certificate token |
| Object-Store | https://kr1-api-object-storage.gov-nhncloudservice.com/v1/{Account} | Control object storage: depends on each region  |
| Tenant ID | Character strings composed of 32 characters in combination of numbers and alphabets | Issue certificate token  |

### Set API Password

To set API password, go to the object storage service page and click **API Endpoint Setting**.

1. Click **API Endpoint Setting**.
2. Go to **API Password Setting** under **API Endpoint Setting** and enter password to issue a token.
3. Click **Save**.

## Certificate Token Issuance

```
POST    https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0/tokens
Content-Type: application/json
```

### Request

| Name  | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | Body | String | O | Tenant ID, available on the setup box for API Endpoint |
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
| access.token.tenant.id | Body | String | Tenant ID of a project requesting for token |
| access.token.expires | Body | String | Expiration time of issued token <br/> in the YYYY-MM-DDThh:mm:ssZ format. e.g.) 2017-05-16T03:17:50Z |

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
    "serviceCatalog": [],
    "user": {
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
https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0/tokens \
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
    "serviceCatalog": [
      {
        "endpoints": [
          {
            "region": "KR1",
            "publicURL": "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****"
          }
        ],
        "type": "object-store",
        "name": "swift",
        "endpoints_links": []
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

        // Create request body
        this.tokenRequest = new TokenRequest();
        this.tokenRequest.getAuth().setTenantId(tenantId);
        this.tokenRequest.getAuth().getPasswordCredentials().setUsername(username);
        this.tokenRequest.getAuth().getPasswordCredentials().setPassword(password);

        this.restTemplate = new RestTemplate();
    }

    public String requestToken() {
        String identityUrl = this.authUrl + "/tokens";

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/json");

        HttpEntity<TokenRequest> httpEntity
            = new HttpEntity<TokenRequest>(this.tokenRequest, headers);

        // Request for token
        ResponseEntity<String> response
            = this.restTemplate.exchange(identityUrl, HttpMethod.POST, httpEntity, String.class);

        return response.getBody();
    }

    public static void main(String[] args) {
        final String authUrl = "https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0";
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
    AUTH_URL = 'https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0'
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
  );  // Create request body
  $req_header = array(
    'Content-Type: application/json'
  );  // Create a request header

  $curl  = curl_init($url); // Initialize curl
  curl_setopt_array($curl, array(
    CURLOPT_POST => TRUE,
    CURLOPT_RETURNTRANSFER => TRUE,
    CURLOPT_HTTPHEADER => $req_header,
    CURLOPT_POSTFIELDS => json_encode($req_body)
  )); // Set parameters
  $response = curl_exec($curl); // Call API
  curl_close($curl);

  return $response;
}

$AUTH_URL = 'https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0';
$TENANT_ID = '{Tenant ID}';
$USERNAME = '{TOAST Account}';
$PASSWORD = '{API Password}';

$token = get_token($AUTH_URL, $TENANT_ID, $USERNAME, $PASSWORD);
printf("%s\n", $token);
?>
```

</details>

## Storage Account
A storage account is a character string in the `AUTH_*****` format, included in the Object-Store API endpoint.

### Query Storage Account
Query usage status of a storage account.

```
HEAD  /v1/{Account}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account, available on the **API Endpoint Setting** popup |

#### Response
This API does not return a response body. Usage status is included in the header. For a valid request, return status code 200.  

| Name  | Type | Format | Description |
|---|---|---|---|
| X-Account-Container-Count | Header | String | Number of containers |
| X-Account-Object-Count | Header | String | Number of saved objects |
| X-Account-Bytes-Used | Header | String | Saved data capacity (bytes) |

#### Code Example

<details>
<summary>cURL</summary>

```
$ curl -I -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****
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
        // Create a header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);
        // Call API
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
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
  function get_request_header() {
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
    foreach($data as $part) {
      $middle = explode(":", $part, 2);
      $headers[trim($middle[0])] = trim($middle[1]);
    }
    return $headers;
  }
}
// main
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$account = new Account($STORAGE_URL, $TOKEN_ID);
$status = $account->get_status();
printf("Container-Count: %d\n", $status["X-Account-Container-Count"]);
printf("Object-Count: %d\n", $status["X-Account-Object-Count"]);
printf("Bytes-Used: %d\n", $status["X-Account-Bytes-Used"]);
?>
```

</details>

### List Containers
List containers of a storage account.

```
GET  /v1/{Account}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account, available on the **API Endpoint Setting** popup |

#### Response
```
[List of containers included in a storage account]
```

#### Code Example

<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****
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
        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);
        // Call API
        ResponseEntity<String> response
            = this.restTemplate.exchange(this.getStorageUrl(), HttpMethod.GET, requestHttpEntity, String.class);
        List<String> containerList = null;
        if (response.getStatusCode() == HttpStatus.OK) {
            // Convert the list of string into sequence
            containerList = Arrays.asList(response.getBody().split("\\r?\\n"));
        }
        // Convert sequence into list and return
        return new ArrayList<String>(containerList);
    }
    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        AccountService accountService = new AccountService(storageUrl, tokenId);
        try {
            List<String> containerList = accountService.getContainerList();
            if (containerList != null) {
                for (String object: containerList) {
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
      return resp.text.split('\n')


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$account = new Account($STORAGE_URL, $TOKEN_ID);
$container_list = $account->get_container_list();
foreach($container_list as $container) {
  printf("%s\n", $container);
}
?>
```

</details>

## Containers

### Create
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
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL | String | O | Name of container to be created  |

#### Response
This API does not return a response body. When a container is created, return status code 201.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example
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

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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

  function get_url($container = null) {
    $url = $this->storage_url;
    if ($container != null) {
      $url .= '/' . $container;
    }
    return $url;
  }

  function get_request_header() {
    return array(
      'X-Auth-Token: ' . $this->token_id
    );
  }

  function create($container) {
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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->create($CONTAINER_NAME);
?>
```

</details>


### Get
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
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
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
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example
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
        // Create a header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        ResponseEntity<String> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, String.class);

        if (response.getStatusCode() == HttpStatus.OK) {
            // Convert list on the string into sequence
            return Arrays.asList(response.getBody().split("\\r?\\n"));
        }

        return Collections.emptyList();
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";

        ContainerService containerService = new ContainerService(storageUrl, tokenId);

        List<String> objectList = containerService.getObjectList(containerName);

        if (objectList != null) {
            for (String object: objectList) {
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
        return response.text.split('\n')

    def get_object_list(self, container):
        req_url = self._get_url(container)
        return self._get_list(req_url)


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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

  function get_object_list($container, $last_object = null) {
    $req_url = $this->get_url($container);
    return $this->get_list($req_url);
  }
}

// main
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$object_list = $container->get_object_list($CONTAINER_NAME);
foreach ($object_list as $obj) {
  printf("%s\n", $obj);
}
?>
```

</details>

### Query  
Get Container API provides some queries as follows. All queries can be connected with '&' for common use.

#### List of More than 10,000 Objects
No more than 10,000 objects can be listed with Get Container API. To list more than 10,000 objects, use the `marker` query. The marker query returns up to 10,000 object lists from the next object after a specified object.

```
GET    /v1/{Account}/{Container}?marker={Object}
X-Auth-Token: {token-id}
```

##### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL | String | O | Container name to query |
| Object | Query | String | O | Criteria object name |

##### Response
```
[Object list next to a specified object which is included to a specified container]
```

##### Code Example
<details>
<summary>cURL</summary>

```
// List objects after '20d33f.jpg'
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example?maker=20d33f.jpg
[List after specified object (20d33f.jpg)]
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
        // Create query URL by using specified object name
        String url = this.getUrl(conatinerName) + "?marker=" + prevLastObject;
        // Call getList() method from the get container example
        return this.getList(url);
    }

    public List<String> getObjectList(String conatinerName) {
        final int LIMIT_COUNT = 10000;

        String url = this.getUrl(conatinerName);

        // List objects
        List<String> objectList = this.getList(url);
        while ((objectList.size() % LIMIT_COUNT) == 0) {
            // If the length of object list is a multiple of 10,000, specify the last object on the list to list
            String lastObject = objectList.get(objectList.size() - 1);
            List<String> nextObjList = this.getObjectList(conatinerName, lastObject);
            objectList.addAll(nextObjList);
        }

        return objectList;
    }

    // The usage example of getObjectList() is same as get container
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

  function get_object_list($container, $last_object = null) {
    $req_url = $this->get_url($container);
    if ($last_object) {
      $req_url .= '?marker='.last_object;
    }
    return $this->get_list($req_url);
  }

  function get_all_object_list($container) {
    $object_list = $this->get_object_list($container);
    while ((count($object_list) % self::MAX_LIST_COUNT) == 0) {
      $next_object_list = $this->get_object_list($container, end($object_list));
      array_merge($object_list, $next_object_list);
    }

    return $object_list;
  }
}

// main
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$object_list = $container->get_all_object_list($CONTAINER_NAME);
foreach ($object_list as $obj) {
  printf("%s\n", $obj);
}
?>
```

</details>


##### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name for Windows, available on the setup box for API Endpoint  |
| Container | URL | String | O | Container name to query |

##### Response
```
[Object list of a specified folder which is included to a specified container]
```

##### Code Example
<details>
<summary>cURL</summary>

```
// List objects
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example
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

    public List<String> getObjectList(String conatinerName) {
        // Create query URL
        String url = this.getUrl(conatinerName);
        // Call getList() method from the get container example
        return this.getList(url);
    }

    // The usage example of getObjectListOfFolder() is same as get container
}
```

</details>

<details>
<summary>Python</summary>

```python
# container.py
class ContainerService:
    # ...
    def get_object_list(self, container):
        req_url = self._get_url(container)
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
  function get_object_list_of_folder($container) {
    $req_url = $this->get_url($container);
    return $this->get_list($req_url);
  }
}
?>
```

</details>

#### List of Objects Starting with Prefix
The `prefix` query can return list of objects starting with a specified prefix. 

```
GET   /v1/{Account}/{Container}?prefix={Prefix}
X-Auth-Token: {token-id}
```

##### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Toekn ID |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL | String | O | Container name to query |
| Prefix | Query | String | O | Prefix to search |

##### Response
```
[Object list which belongs to a specified container and starts with a specified prefix]
```

##### Code Example
<details>
<summary>cURL</summary>

```
// List objects starting with 314
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example?prefix=314
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
        // Create query URL by using a specified prefix
        String url = this.getUrl(conatinerName) + "?prefix=" + prefix;
        // Call getList() method from the get container example
        return this.getList(url);
    }

    // The usage example of getObjectListWithPrefix() is same as get container example.
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
  function get_object_list_of_prefix($container, $prefix) {
    $req_url = $this->get_url($container)."?prefix=".$prefix;
    return $this->get_list($req_url);
  }
}
?>
```

</details>

#### Specify Maximum Object Count on List
Use the `limit` query to specify the maximum object count of the list to return.

```
GET   /v1/{Account}/{Container}?limit={limit}
X-Auth-Token: {token-id}
```

##### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL | String | O | Container name to query  |
| limit | Query | Integer | O | Object count to show on the list |

##### Response
```
[Object list as much as specified for a specified container]
```

##### Code Example

<details>
<summary>cURL</summary>

```curl
// Get only 10 objects
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example?limit=10
...{9 objects}...
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
        // Create query URL by using the maximum number of specified objects
        String url = this.getUrl(conatinerName) + "?limit=" + limit;
        // Call getList() method from the get container example
        return this.getList(url);
    }

    // The usage example of getObjectListWithPrefix() is same as get container example
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
  function get_object_list_with_limit($container, $limit) {
    $req_url = $this->get_url($container)."?limit=".$limit;
    return $this->get_list($req_url);
  }
}
?>
```

</details>


### Modify

Access rules can be specified by changing container metadata.

```
POST  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {Read-container policy}
X-Container-Write: {Write-container policy}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| X-Container-Read | Header | String | - | Specify access rules for reading container <br/>.r:* - Allow access for all users <br/>.r:example.com,test.com – Access allowed to particular address only, to be delimited by ‘,’<br/>.rlistings. – Allow listing containers <br/>AUTH_.... – Allow access to particular accounts only |
| X-Container-Write | Header | String | - | Specify access rules for writing container <br/>\*:\* - Allow reading for all users <br/>AUTH_.... – Allow reading to particular accounts only |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL | String | O | Container name to modify |

#### Response
This API does not return a response body. For a valid request, return status code 204.

#### Code Example
<details>
<summary>cURL</summary>

```
// Allow Read/Write for all users
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Read: .r:*' \
-H 'X-Container-Write: *:*' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example
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

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Container-Read", permission);    // Add permission to header

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
  function set_acl($container, $is_public) {
    $req_url = $this->get_url($container);

    $permission = $is_public ? self::PUBLIC_ACL : self::PRIVATE_ACL;
    $req_header = $this->get_request_header();
    $req_header[] = 'X-Container-Read: ' . $permission;  // Add permission to header

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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->set_acl($CONTAINER_NAME, TRUE);
?>
```

</details>

#### Check ACL
After setting public for the permission to read, you may use tools such as `curl` or `wget`, or check whether to get without token via browser.   

<details>
<summary>Example</summary>

```
$ curl https://kr1-api-object-storage.gov-nhncloudservice.com/v1/{Account}/{Container}/{Object}

{Object Details}
```

</details>

### Delete

Delete specified containers. To be deleted, containers must be empty.

```
DELETE   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL| String |	O | Container name to delete |

#### Response
This request does not return a response body. For a valid request, return status code 204.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example
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

    public void deleteContainer(String containerName) {
        String url = this.getUrl(containerName);

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.DELETE, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
  function delete($container) {
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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->delete($CONTAINER_NAME);
?>
```

</details>

## Objects

### Upload
Upload new objects to a specified container.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Content-type | Header | String | O | Content type of object |
| X-Delete-At | Header | Timestamp | - | Unix time (seconds) to delete object |
| X-Delete-After | Header | Timestamp | - | Valid object time, unix time (seconds)  |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container |	URL | String | O | Container name  |
| Object | URL | String |	O | Object name to create |
| - |	Body | Binary | O | Object details to create  |

> [Caution]
> If an object name starts with `./` or `../`, the browser regards it as path character and access is unavailable on web console.
> If you have uploaded an object of such name via API, it must also be accessed via API.

#### Response
This API does not return a response body. For a valid request, return status code 201.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg \
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

        // Override RequestCallback to add InputStream to the request body
        final RequestCallback requestCallback = new RequestCallback() {
            public void doWithRequest(final ClientHttpRequest request) throws IOException {
                request.getHeaders().add("X-Auth-Token", tokenId);
                IOUtils.copy(inputStream, request.getBody());
            }
        };

        // Set to enable the override RequestCallback
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setBufferRequestBody(false);
        RestTemplate restTemplate = new RestTemplate(requestFactory);

        HttpMessageConverterExtractor<String> responseExtractor
            = new HttpMessageConverterExtractor<String>(String.class, restTemplate.getMessageConverters());

        // Call API
        restTemplate.execute(url, HttpMethod.PUT, requestCallback, responseExtractor);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // Create InputStream from file
            File objFile = new File(objectPath + "/" + objectName);
            InputStream inputStream = new FileInputStream(objFile);

            // Upload
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
            return requests.put(req_url, headers=req_header, data=f.read())


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
class ObjectService {
  private $storage_url;
  private $token_id;

  function __construct($storage_url,  $token_id) {
    $this->storage_url = $storage_url;
    $this->token_id = $token_id;
  }

  function get_url($container, $object) {
    return $this->storage_url . '/' . $container . '/' . $object;
  }

  function get_request_header() {
    return array(
      'X-Auth-Token: ' . $this->token_id
    );
  }

  function upload($container, $object, $filename) {
    $req_url = $this->get_url($container, $object);

    $req_header = $this->get_request_header();

    $fd = fopen($filename, 'r');  // Open file.

    $curl  = curl_init($req_url);
    curl_setopt_array($curl, array(
      CURLOPT_PUT => TRUE,
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_INFILE => $fd,  // Put FileStream as parameter.
      CURLOPT_HTTPHEADER => $req_header
    ));
    $response = curl_exec($curl);
    curl_close($curl);l

    fclose($fd);
  }
}

// main
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';
$OBJ_PATH = '/home/example';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

// upload object
$filename = $OBJ_PATH.'/'.$OBJECT_NAME;
$object->upload($CONTAINER_NAME, $OBJECT_NAME, $filename);
?>
```

</details>

### Multipart Upload
Objects that are over 5GB must be segmented into less than 5GB-parts before uploaded.

#### Segment Uploading  

```
PUT   /v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

##### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Content-type | Header | String | O | Content type of object |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container |	URL | String | O | Container name |
| Object |	URL | String | O | Object name to create |
| Count | URL | Integer | O | Sequence of segmented objects, e.g.) 001, 002 |
| - |	Body | Binary | O | Segmented object details |

##### Response
This API does not return a request body. For a valid request, return status code 201.

#### Manifest Creation
Upload all segments of an object, and then create manifest object; then you can use it as one object. The manifest object refers to the path in which segments are saved.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Container}/{Object}/
```

##### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | Token ID |
| X-Object-Manifest | Header| String | O | Upload path for segmented objects, `{Container}/{Object}/` |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container |	URL | String | O | Container name |
| Object |	URL | String | O | Manifest object name to create |
| - | Body| Binary | O | Empty data |

##### Response
This API does not return a response body. For a valid request, return status code 201.


#### Code Example
<details>
<summary>cURL</summary>

```
// Segment file by 200MB
$ split -d -b 209715200 large_obj.img large_obj.img.

// Upload segmented object
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/003 \
-T large_obj.img.02

// Upload manifest object
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Object-Manifest: curl_example/large_obj.img/' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img \
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

    // Upload manifest object
    public void uploadManifestObject(String containerName, String objectName) {
        String url = this.getUrl(containerName, objectName);
        String manifestName = containerName + "/" + objectName + "/"; // Create manifest name

        // Create a header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Object-Manifest", manifestName);  // Show manifest at the header

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        File objFile = new File(objectPath + "/" + objectName);
        int fileSize = (int)objFile.length();

        final int defaultChunkSize = 100 * 1024; // Segment by 100 KB
        int chunkSize = defaultChunkSize;
        int chunkNo = 0;  // Chunk number to create names for segmented objects
        int totalBytesRead = 0;

        try {
            // Create InputStream from file
            InputStream inputStream = new BufferedInputStream(new FileInputStream(objFile));
            while(totalBytesRead < fileSize) {

                // Calculate remaining data size
                int remainedBytes = fileSize - totalBytesRead;
                if(remainedBytes < chunkSize) {
                    chunkSize = remainedBytes;
                }

                // Read data as much as the chunk size for byte buffer
                byte[] chunkBuffer = new byte[chunkSize];
                int bytesRead = inputStream.read(chunkBuffer, 0, chunkSize);

                if(bytesRead > 0) {
                    // Create buffered data as InputStream to be uploaded, by using the uploadObject() method from upload object example
                    String objPartName = String.format("%s/%03d", objectName, ++chunkNo);
                    InputStream chunkInputStream = new ByteArrayInputStream(chunkBuffer);
                    objectService.uploadObject(containerName, objPartName, chunkInputStream);

                    totalBytesRead += bytesRead;
                }
            }

            // Upload manifest file
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
        return requests.put(req_url, headers=req_header)

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

        return self._create_manifest(container, object)


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
class ObjectService {
  const CHUNK_SIZE = 100 * 1024;  // 100 KB
  // ...

  function create_manifest($container, $object) {
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

  function upload_large_object($container, $object, $filename) {
    $url = $this->get_url($container, $object);
    $req_header = $this->get_request_header();

    $chunk_index = 1;
    $chunk_size = self::CHUNK_SIZE;
    $total_bytes_read = 0;
    $fd = fopen($filename, 'r');  // Open file.
    $obj_size = filesize($filename);

    while($total_bytes_read < $obj_size) {
      // Calculate volume to segment
      $remained_bytes = $obj_size - $total_bytes_read;
      if ($remained_bytes < $chunk_size) {
        $chunk_size = $remained_bytes;
      }
      $chunk = fread($fd, $chunk_size);
      // Create part names
      $temp_file = sprintf("./multipart-%03d", $chunk_index);
      $req_url = sprintf("%s/%03d", $url, $chunk_index);

      // Create temporary part files
      $part_fd = fopen($temp_file, 'w+');
      fwrite($part_fd, $chunk);
      fseek($part_fd, 0);

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
        CURLOPT_PUT => TRUE,
        CURLOPT_HEADER => TRUE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_INFILE => $part_fd,  // Enter part filestream as parameter
        CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
      printf("$response");

      // Delete temporary files
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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$LARGE_OBJECT = '8cb0d624f8c14c69b52f2cd89e5e59b7.jpg';
$OBJ_PATH = '/home/example';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$filename = $OBJ_PATH.'/'.$LARGE_OBJECT;
$object->upload_large_object($CONTAINER_NAME, $LARGE_OBJECT, $filename);
?>
```

</details>

### Modify Object Details
Same as Upload Object API, but if object is already located in the container, the object details are to be modified.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Content-type | Header | String | O | Content type of object |
| X-Delete-At | Header | Timestamp | - | Unix time to delete object (seconds) |
| X-Delete-After | Header | Timestamp | - | Valid object time, unitx time (seconds) |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint  |
| Container |	URL | String | O | Container name |
| Object | URL | String | O | Object name of which details are to be modified |

#### Response
This API does not return a response body. For a valid request, return status code 201.

### Download
Download objects.

```
GET   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account ID, available on the setup box for API Endpoint |
| Container |	URL | String | O | Container name |
| Object | URL | String | O | Object name to download  |

#### Response
Object details are returned to stream. For a valid request, return status code 200.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -O -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg

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

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.setAccept(Arrays.asList(MediaType.APPLICATION_OCTET_STREAM));

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API, with data received as byte sequence
        ResponseEntity<byte[]> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, byte[].class);

        // Create byte sequence data as InputStream to be returned
        return new ByteArrayInputStream(response.getBody());
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";
        final String downloadPath = "/home/example/download";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // Download object
            InputStream inputStream = objectService.downloadObject(containerName, objectName);

            // Read downloaded data as byte buffer
            byte[] buffer = new byte[inputStream.available()];
            inputStream.read(buffer);

            // Record buffered data onto file
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
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
class ObjectService {
  //...
  function download($container, $object, $filename) {
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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';
$DOWNLOAD_PATH = '/home/example/download';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$filename = $DOWNLOAD_PATH.'/'.$OBJECT_NAME;
$object->download($CONTAINER_NAME, $OBJECT_NAME, $filename);
?>
```

</details>

### Copy
Copy object to another container.

```
COPY   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Destination: {Target to copy objects}
```

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Copy-From: {Original object}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Destination | Header | String |	- | Target to copy object, `{Container} / {Object}`<br/>Required to use the COPY method |
| X-Copy-From | Header | String |	- | Original object, `{Container} / {Object}`<br/>Required to use the PUT method |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL | String | O |	Container name<br/>COPY Method: Original container<br/>PUT Method: Container to copy |
| Object | URL | String |	Object name to copy |

#### Response
This request does not return a response body. For a valid request, return status code 201.

#### Code Example
<details>
<summary>cURL</summary>

```
// COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/3a45e9.jpg' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/3a45e9.jpg

// PUT method
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Copy-From: curl_example/3a45e9.jpg' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/copy_con/3a45e9.jpg
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

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Copy-From", srcObject);    // Specify original object
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call alternative API since the COPY method is not supported by HttpMethod.
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
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
        return requests.put(req_url, headers=req_header)


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
class ObjectService {
  //...
  function copy($src_container, $object, $dest_container) {
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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$DEST_CONTAINER = 'dest';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$object->copy($CONTAINER_NAME, $OBJECT_NAME, $DEST_CONTAINER);
?>
```

</details>

### Modify Object Metadata
Modify metadata of a specified object.

```
POST   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Meta-{Key}: {Value}
```

#### Request
This API does not require a request body.  

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| X-Object-Meta-{Key} | Header | String | - | Metadata to change |
| X-Delete-At | Header | Timestamp | - | Unix time to delete object (seconds) |
| X-Delete-After | Header | Timestamp | - | Valid object time, unitx time (seconds) |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL| String |	 O | Container name |
| Object | URL| String |  O | Object name of which metadata is to be modified |

#### Response
This request does not return a response body. For a valid request, return status code 202.

#### Code Example
<details>
<summary>cURL</summary>

```
// Add metada to object
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo" \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg

// Check metadata added to object header
$ curl -I -H "X-Auth-Token: b587ae461278419da6ecd21a2344c8aa" \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg
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

        // Create metadata key
        String metaKey = "X-Object-Meta-" + key;

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add(metaKey, value);    // Set metadata at the header

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
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
        return requests.post(req_url, headers=req_header)


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
class ObjectService {
  //...
  function set_metadata($container, $object, $key, $value) {
    $req_url = $this->get_url($container, $object);
    $req_header = $this->get_request_header();
    $req_header[] = 'X-Object-Meta-'.$key.': '.$value;  // Add metadata to the header

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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$META_KEY = 'Type';
$META_VALUE = 'photo';
$object->set_metadata($CONTAINER_NAME, $OBJECT_NAME, $META_KEY, $META_VALUE);
?>
```

</details>

### Delete
Delete specified object.

```
DELETE   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| X-Object-Meta-{Key} | Header | String | - | Metadata to change |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL| String |	 O | Container name |
| Object | URL| String |  O | Object name of which metadata is to be modified |

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | User account name, available on the setup box for API Endpoint |
| Container | URL| String |	 O | Container name |
| Object | URL| String |  O | Object name to delete |

#### Response
This request does not return a response body. For a valid request, return status code 204.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg
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

        // Create header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.DELETE, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****";
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
        return requests.delete(req_url, headers=req_header)


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****'
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
class ObjectService {
  //...
  function delete($container, $object) {
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
$STORAGE_URL = 'https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$object->delete($CONTAINER_NAME, $OBJECT_NAME);
?>
```

</details>

## References

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
