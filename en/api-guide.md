## Storage > Object Storage > API Guide

## Prerequisites

To use the Object Storage API, you must obtain an authentication token first. An authentication token is the authentication key required when using the REST API of Object Storage. A token is required to access a container or object that is not publicly available. Tokens are managed per NHN Cloud account.

<br/>

### Check the Tenant ID and API Endpoint

Click **API Endpoint Setting** on the Object Storage service page to check the tenant ID and API endpoint to issue a token.

| Item | API Endpoint | Usage |
|---|---|---|
| Identity | https://api-identity-infrastructure.nhncloudservice.com/v2.0 | Issue authentication token |
| Object-Store | https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_***** | Control object storage: depends on each region  |
| Tenant ID | A string in 32 characters consisting of numbers and alphabets | Issue authentication token  |

<br/>

### Set the API Password

To set the API password, go to the Object Storage service page and click **API Endpoint Setting**.

1. Click **API Endpoint Setting**.
2. In the **Set API Password** input box under **API Endpoint Setting**, enter the password to use when issuing a token.
3. Click **Save**.

> [Note]
> An API password is set by account. You can use the same password set in one project for all of your other projects. 

<br/>

## Authentication Token Issuance

```
POST    https://api-identity-infrastructure.nhncloudservice.com/v2.0/tokens
Content-Type: application/json
```

<p style='padding-top: 10px; font-size: 15px;'><b>Request</b></p>

| Name  | Type | Format | Required | Description |
|---|---|---|---|---|
| tenantId | Body | String | O | Tenant ID, which can be found in the API Endpoint setting dialog box |
| username | Body | String | O | NHN Cloud user ID (email format), IAM member ID |
| password | Body | String | O | Password saved on the API Endpoint setting dialog box |

<details>
<summary>Example</summary>

```json
{
  "auth": {
    "tenantId": "{Tenant ID}",
    "passwordCredentials": {
      "username": "{NHN Cloud ID}",
      "password": "{API Password}"
    }
  }
}
```
</details>

<p style='padding-top: 10px; font-size: 15px;'><b>Response</b></p>

| Name | Type | Format | Description |
|---|---|---|---|
| access.token.id | Body | String |	ID of issued token |
| access.token.tenant.id | Body | String | Tenant ID corresponding to the project requesting the token |
| access.token.expires | Body | String | Expiration time of issued token <br/> in the YYYY-MM-DDThh:mm:ssZ format. e.g.) 2017-05-16T03:17:50Z |
| access.user.id | Body | String | API User ID composed of 32 digits of hexadecimal numbers <br/>Used to obtain EC2 credentials to use S3 compatible API or to set access policies |

> [Caution]
> A token has expiration time. The 'expires' item included in the response to the request for token issuance refers to expiration time of the issued token. When a token expires, a new token must be issued.  

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
        "name": "{NHN Cloud ID}",
        "groupId": "{NHN Cloud Project ID}",
        "project_domain": "NORMAL",
        "swift": true
      },
      "issued_at": "{Token Issued Time}"
    },
    "serviceCatalog": [],
    "user": {
      "id": "{API User ID}",
      "name": "{User Name}"
    }
  }
}
```
</details>

<p style='padding-top: 10px; font-size: 15px;'><b>Code Example</b></p>

<details>
<summary>cURL</summary>

```
$ curl -X POST -H 'Content-Type:application/json' \
https://api-identity-infrastructure.nhncloudservice.com/v2.0/tokens \
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
            "publicURL": "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****"
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
package com.nhn.cloud.auth;

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

        // Request token
        ResponseEntity<String> response
            = this.restTemplate.exchange(identityUrl, HttpMethod.POST, httpEntity, String.class);

        return response.getBody();
    }

    public static void main(String[] args) {
        final String authUrl = "https://api-identity-infrastructure.nhncloudservice.com/v2.0";
        final String tenantId = "{Tenant ID}";
        final String username = "{NHN Cloud Account}";
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
    AUTH_URL = 'https://api-identity-infrastructure.nhncloudservice.com/v2.0'
    TENANT_ID = '{Tenant ID}'
    USERNAME = '{NHN Cloud Account}'
    PASSWORD = '{API Password}'

    token = get_token(AUTH_URL, TENANT_ID, USERNAME, PASSWORD)
    print(json.dumps(token, indent=4))
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

$AUTH_URL = 'https://api-identity-infrastructure.nhncloudservice.com/v2.0';
$TENANT_ID = '{Tenant ID}';
$USERNAME = '{NHN Cloud Account}';
$PASSWORD = '{API Password}';

$token = get_token($AUTH_URL, $TENANT_ID, $USERNAME, $PASSWORD);
printf("%s\n", $token);
?>
```
</details>

<br/>

## Storage Account
A storage account is a character string in the `AUTH_*****` format, included in the Object-Store API endpoint.

### Query the Storage Account
Retrieves usage status of a storage account.

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

| Name | Type | Format | Description |
|---|---|---|---|
| X-Account-Container-Count | Header | String | Number of containers |
| X-Account-Object-Count | Header | String | Number of stored objects |
| X-Account-Bytes-Used | Header | String | Stored data capacity (bytes) |

#### Code Example

<details>
<summary>cURL</summary>

```
$ curl -I -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****
```
</details>

<details>
<summary>Java</summary>

```java
// AccountService.java
package com.nhn.cloud.obs;
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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';

$account = new Account($STORAGE_URL, $TOKEN_ID);
$status = $account->get_status();

printf("Container-Count: %d\n", $status["X-Account-Container-Count"]);
printf("Object-Count: %d\n", $status["X-Account-Object-Count"]);
printf("Bytes-Used: %d\n", $status["X-Account-Bytes-Used"]);
?>
```
</details>

<br/>

### List Containers
Lists containers of a storage account.

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
[List of containers in a storage account]
```

#### Code Example

<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****
```
</details>

<details>
<summary>Java</summary>

```java
// AccountService.java
package com.nhn.cloud.obs;
// .. import list

@Data
public class AccountService {
    // AccountService Class ...
    public List<String> getContainerList() {
        // Create a header
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        ResponseEntity<String> response
            = this.restTemplate.exchange(this.getStorageUrl(), HttpMethod.GET, requestHttpEntity, String.class);

        List<String> containerList = null;
        if (response.getStatusCode() == HttpStatus.OK) {
            // Convert the list received as String to array
            containerList = Arrays.asList(response.getBody().split("\\r?\\n"));
        }

        // Convert the array to list and return it
        return new ArrayList<String>(containerList);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';

$account = new Account($STORAGE_URL, $TOKEN_ID);
$container_list = $account->get_container_list();
foreach($container_list as $container) {
  printf("%s\n", $container);
}
?>
```
</details>

<br/>

## Containers

### Create a Container
Creates a container. To upload files to object storage, a container must be created.
You can create an object lock container by setting the object lock cycle using the `X-Container-Worm-Retention-Day` header when creating a container. Objects uploaded to the object lock container are stored using the **WORM (Write-Once-Read-Many)** model. For the objects uploaded to the object lock container, lock expiration date is configured. You cannot overwrite or delete the object before the set lock expiration date.

> [Caution]
> A container name cannot include the special characters ``' " ` < > ;``, spaces, and relative path characters (`. ..`).

<!-- This is a comment for line break, so it must be included. -->

> [Note]
> If a container or object name includes special characters such as `! * ' ( ) ; : @ & = + $ , / ? # [ ]`, it must be URL-encoded (percent-encoding). These are reserved characters that are considered important for URL. If you send an API request without URL-encoding a path including these characters, you won't get the desired response.

```
PUT  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container | URL | String | O | Name of container to be created  |
| X-Container-Worm-Retention-Day | Header | Integer | - | Set the default container object lock cycle in days |

#### Response
This API does not return a response body. When a container is created, return status code 201.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example
```
</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.nhn.cloud.obs;

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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->create($CONTAINER_NAME);
?>
```
</details>


<br/>

### Get a Container
Retrieves the information of the specified container and the list of the objects stored in the container. The container's information can be viewed in the response header.

```
GET   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container | URL | String | O | Container name to get |
| marker | Query | String | - | Name of base object |
| prefix | Query | String | - | Prefix to search |
| limit | Query | Integer | - | The number of objects to display in the list |
| format | Query | String | - | Response format, json or xml |

> [Note]
> Get Container API provides a number of queries. Each query can be concatenated using `&`.

#### Listing More Than 10,000 Objects
The number of objects that can be retrieved using Get Container API is limited to 10,000. If you want to retrieve more than 10,000 objects, you need to use the `marker` query. The marker query returns up to 10,000 additional objects, starting from the next object of the specified object.

<br/>

#### Listing Objects Starting with a Prefix
Using the `prefix` query returns the list of objects that start with the specified prefix. 

<br/>

#### Specifying the Maximum Number of Objects in The List
Using the `limit` query allows you to specify the maximum number of objects in the list of objects to be returned.

<br/>

#### Specifying the Response Format
Using the `format` query allows you to specify a `json` or `xml` response format. If so, the response body includes the metadata for each object (size, content type, last modified time, Etag).

<br/>

#### Response

```
[The list of object in the container]
```

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example
ba6610.jpg
20d33f.jpg
31466f.jpg
```
</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java
package com.nhn.cloud.obs;

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
            // Convert list received as String to array
            return Arrays.asList(response.getBody().split("\\r?\\n"));
        }

        return Collections.emptyList();
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
# container.py
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
    TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09'
    CONTAINER_NAME = 'test'

    con_service = ContainerService(STORAGE_URL, TOKEN_ID)

    object_list = con_service.get_object_list(CONTAINER_NAME)
    for object in object_list:
        print(object)
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
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

<br/>

### Change Container Settings

Changes the container settings. The container settings can be found in the response header when retrieving the container.

```
POST  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {role-based access rules for container read}
X-Container-Write: {role-based access rules for container write}
X-Container-Ip-Acl-Allowed-List: {IP based access rules for container write}
X-Container-Ip-Acl-Denied-List: {IP based access rules for container write}
X-Container-Object-Lifecycle: {Life cycle of objects in container}
X-History-Location: {Container to store the previous object version}
X-Versions-Retention: {Life cycle of the previous object version}
X-Container-Meta-Web-Index: {Static website index document object}
X-Container-Meta-Web-Error: {Static website error document object suffix}
X-Container-Meta-Access-Control-Allow-Origin: {List that allows Cross-Origin Resource Sharing}
X-Container-Rfc-Compliant-Etags: {Whether to use the RFC compliant ETag format}
X-Container-Worm-Retention-Day: {Container's object lock cycle}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| X-Container-Read | Header | String | - | Sets the role-based access rules for container read |
| X-Container-Write | Header | String | - | Sets the role-based access rules for container write |
| X-Container-Ip-Acl-Allowed-List | Header | String | - | Sets the IP-based access rules for container write |
| X-Container-Ip-Acl-Denied-List | Header | String | - | Sets the IP-based access rules for container write |
| X-Container-Object-Lifecycle | Header | Integer | - | Sets the life cycle of the container's base object in days |
| X-History-Location | Header | String | - | Sets the container to store the previous version of the object |
| X-Versions-Retention | Header | Integer | - | Sets the life cycle of the object's previous version in days |
| X-Container-Meta-Web-Index | Header | String | - | Sets the static website index document object<br/>Only alphanumeric characters and some special characters (`-`, `_`, `.`, `/`) are allowed |
| X-Container-Meta-Web-Error | Header | String | - | Sets the static website error document object suffix<br/>Only alphanumeric characters and some special characters (`-`, `_`, `.`, `/`) are allowed |
| X-Container-Meta-Access-Control-Allow-Origin | Header | String | - | List of hosts that allows CORS. You can either allow all hosts with '*' or enter a list of hosts separated by spaces. | 
| X-Container-Rfc-Compliant-Etags | Header | String | - | Sets whether to use the RFC compliant ETag format, true or false |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container | URL | String | O | The name of the container to edit |
| X-Container-Worm-Retention-Day | Header | Integer | - | Set the default container object lock cycle in days <br/> Change is only possible in object lock containers |
<br/>

##### Set the Access Policy
You can set the container access policy using the `X-Container-Read`, `X-Container-Write`,`X-Container-Ip-Acl-Allowed-List`, and `X-Container-Ip-Acl-Denied-List` header. For more details, refer to [ACL Configuration Guide](acl-guide/).

<br/>

##### Set the Object Life Cycle
With the `X-Container-Object-Lifecycle` header, you can set the life cycle of the objects to be stored in a container in the unit of days. This applies only to objects uploaded after the setting has been applied.
<br/>

##### Set the Version Control Policy
As described in the [Update an Object](api-guide/#update-an-object), if there are duplicate object names while uploading objects, the objects are updated. If you want to store the content of existing objects, use the `X-History-Location` header to specify the **Archive Container** to store the previous version.

The previous version of objects are stored in the archive container in the following manner:
```
{length of an object name in a 3-digit hexadecimal number}{object name}/{unix time when the previous version was stored}
```
For example, if an object named `picture.jpg` is updated, the `00bpicture.jpg/1610606551.82539` object is created in the archive container.

If you delete an object from a container where version control policy is already set, the deleted object will be stored in the archive container and a delete marker object will be created. The previous object versions stored in the archive container can be accessed at any time.

With the `X-Versions-Retention` header, you can set the life cycle of a previous object version in days. If set to 1, the stored object will be automatically deleted after a day. If not set, the previous object version will be stored until users delete it. This applies only to previous object versions uploaded after the setting has been applied.

> [Cautions]
> If the archive container is deleted before the original container, an error occurs when updating or deleting objects in the original container. If the archive container has already been deleted, you can solve the issue by creating a new archive container or disabling the original container's version control policy.
>
> It is recommended that you avoid using Unicode characters in container names for archive containers. If the name of the container to set as an archive container contains Unicode characters, it must be URL-encoded before being entered in the request header.
>
> If you specify an encryption container as the archive container and then delete the symetric key from Secure Key Manager, the object of the original container fails to be uploaded and deleted.

<br/>

##### Set a Static Website
If you allow the container read access to all users and set the static website's index document and error document using the `X-Container-Meta-Web-Index` and `X-Container-Meta-Web-Error` headers, you can host a static website using the container URL.

The object to be used as an index document or error document for a static website must have a name consisting of one or more alphanumeric characters or some special characters (`-`, `_`, `.`, `/`), and it must be in hypertext format with an `html` file extension. If the conditions are not met, you may not be able to configure the setting or the static website may not work.
The format of a static website's error document name is `{response code}{suffix}`. For example, if an error document is set as `error.html`, the name of the error document to be displayed when the 404 error occurs becomes `404error.html`. You can upload and use error documents according to each error condition. If an error document is not defined or an error object that matches the response code does not exist, the default error document of a web browser will be displayed.
<br/>

##### Cross-Origin Resource Sharing (CORS)

If you directly call the Object Storage API from the browser, you need to set Cross-Origin Resource Sharing (CORS). Set an allowed-origin list using the `X-Container-Meta-Access-Control-Allow-Origin` header. You can enter one or more origins separated by spaces(` `) or allow all origins by entering `*`.


<details>
<summary>Example of checking CORS settings</summary>

Add CORS settings to a container.

```
$ curl -X POST \
-H 'X-Auth-Token: ****' \
-H 'X-Container-Meta-Access-Control-Allow-Origin: https://example.com' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```
<br>
The script below is run after moving to a site that allows CORS from the browser. You can run the script from the console of the developer tools provided by the browser.

<br/>
ex) https://example.com/

```
var token = "****";
var url = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_****/container/object";
var request = new XMLHttpRequest();
request.onreadystatechange = function (oEvent) {
  if (request.readyState == 4) {
      result = 'Status: ' + request.status;
      result = result + '\n' + request.getAllResponseHeaders();
      console.log(result)
  }
}
request.open('GET', url);
request.setRequestHeader('X-Auth-Token', token);
request.send(null);
```

<br>
If there is no problem in the CORS settings, you can see the success response as follows.

```
Status: 200
content-length: 1
content-type: application/octet-stream
etag: bad093d7f49dc495751cb3f7f8b2530c
last-modified: Mon, 30 May 2022 15:16:43 GMT
x-openstack-request-id: tx0b1637089d1841d6833d2-0062a60940
x-timestamp: 1653923802.28970
x-trans-id: tx0b1637089d1841d6833d2-0062a60940
```

<br>
If you do not set CORS or call an API from a site that is not allowed, you will receive an error response like the one below.

```
Access to XMLHttpRequest at 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_****/container/object' from origin 'https://example.com' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.

Status: 0
```

</details>


<br/>

##### Set the RFC compliant ETag format
Some applications require the ETag value enclosed in double quotes according to the specification [RFC7232](https://www.rfc-editor.org/rfc/rfc7232#section-2.3). If you use the `X-Container-Rfc-Compliant-Etags` header, the ETag value in double quotes can be returned when querying objects stored in a container.

<br/>

##### Change Object Lock Cycle
You can change the object lock cycle of the object lock container using the `X-Container-Worm-Retention-Day` header. The lock cycle can be entered in days and cannot be turned off. The changed lock cycle is applied to objects uploaded after the change. The object lock cycle can only be changed on object lock containers.

> [Note]
> You cannot change a general container to an object lock container and vice versa.
> You cannot specify an object lock container as an archive container or replication target containger. 

<br/>

##### Unset a Container
If you use a header without a value, the setting will be removed. For example, if the life cycle of an object is set to 3 days and you request to edit the container using `'X-Container-Object-Lifecycle: '`, the object life cycle will be removed and the objects that is stored in the container afterwards will not have their life cycle automatically set.
<br/>

#### Response
This API does not return a response body. For a valid request, return status code 204.
<br/>

#### Code Example
This is an example in which the user requests changing the setting so that all users may read from and write to containers. You can select the headers you need to change the settings and request in the same way.

<details>
<summary>cURL</summary>

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Read: .r:*' \
-H 'X-Container-Write: *:*' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example
```
</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java

package com.nhn.cloud.obs;

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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->set_acl($CONTAINER_NAME, TRUE);
?>
```
</details>


<br/>

### Delete a Container

Deletes the specified container. The container to be deleted must be empty.

```
DELETE   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container | URL| String |	O | Name of the container to delete |

#### Response
This request does not return a response body. For a valid request, status code 204 is returned.

<br/>

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example
```
</details>

<details>
<summary>Java</summary>

```java
// ContainerService.java

package com.nhn.cloud.obs;

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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->delete($CONTAINER_NAME);
?>
```
</details>

<br/>

## Objects

### Upload an Object
Uploads a new object to the specified container.

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
| X-Delete-After | Header | Timestamp | - | Object's expiration time, unix time (seconds) |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container |	URL | String | O | Container name  |
| Object | URL | String |	O | Name of object to create |
| - |	Body | Binary | O | Data content of the object to create |


##### Setting Object Life Cycle
If you use either the `X-Delete-At` or `X-Delete-After` header, you can set the life cycle of an object in seconds.
<br/>

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
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg \
-T ./ba6610.jpg
```
</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.nhn.cloud.obs;

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

        // Set to enable the overridden RequestCallback
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setBufferRequestBody(false);
        RestTemplate restTemplate = new RestTemplate(requestFactory);

        HttpMessageConverterExtractor<String> responseExtractor
            = new HttpMessageConverterExtractor<String>(String.class, restTemplate.getMessageConverters());

        // Call API
        restTemplate.execute(url, HttpMethod.PUT, requestCallback, responseExtractor);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
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

<br/>

### Multipart Upload
An object whose size exceeds 5GB needs to be divided into segments of 5GB or smaller before uploading. If you upload segment objects and create a manifest object, you can use them as if they are a single object.

<br/>

#### Upload Segment Objects
Uploads each of the segment objects generated from the object.

```
PUT   /v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

<br/>

##### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Content-type | Header | String | O | Content type of object |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container |	URL | String | O | Container name |
| Object |	URL | String | O | Name of object to create |
| Count | URL | Integer | O | Sequence of segment objects, e.g.) 001, 002 |
| - |	Body | Binary | O | Data content of the segment object |

<br/>

##### Response
This API does not return a request body. For a valid request, return status code 201.

<br/>

#### Create a Manifest Object
A manifest object can be created in two ways: either using **DLO** (Dynamic Large Object) or **SLO** (Static Large Object).

> [Note]
> Because a manifest object has path information for segment objects, there is no need to upload segment objects and the manifest object in the same container. If segment objects and manifest object are in a single container and it is difficult to manage them, it is recommended to upload segment objects to a separate container and create only the manifest object in the container where you originally intended to upload the objects.

**DLO**
The DLO manifest object uses the path to the segment objects entered in the `X-Object-Manifest` header to automatically find and connect segment objects.

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Segment-Container}/{Segment-Object}
```

<br/>

##### Request
| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | Token ID |
| X-Object-Manifest | Header| String | O | The path where segment objects are uploaded: `{Segment-Container}/{Segment-Object}/` |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container |	URL | String | O | Container name |
| Object |	URL | String | O | Name of the manifest object to create |
| - | Body| Binary | O | Empty data |

<br/>

**SLO**
To create an SLO manifest object, you must enter the list of segment objects in order in the request body.
If you make a request to create an SLO manifest object, the system checks whether each segment object is in the entered path and the etag value matches the size of the object. If the information does not match, the manifest object is not created.

```
PUT   /v1/{Account}/{Container}/{Object}?multipart-manifest=put
X-Auth-Token: {token-id}
```

```json
[
    {
        "path": "{Segment-Container}/{Segment-Object}",
        "etag": "{Etag-of-Segment-Object}",
        "size_bytes": 1048576
    },
    {
        "path": "{Segment-Container}/{Segment-Object1}",
        "etag": "{Etag-of-Segment-Object}",
        "size_bytes": 1048576
    },
    ...
]
```
<br/>

##### Request
| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | Token ID |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container |	URL | String | O | Container name |
| Object |	URL | String | O | Name of the manifest object to create |
| multipart-manifest | Query| String | O | put |
| path | Body | String | O | Path to segment objects |
| etag | Body | String | O | etag of segment objects |
| size_bytes | Body | Integer | O | Size of segment objects (in bytes) |

> [Note]
> To retrieve the segment information held by SLO manifest file, you must use the `multipart-manifest=get` query.

<br/>

##### Response
This API does not return a response body. For a valid request, return status code 201.

<br/>

#### Code Example
Example of multipart upload using the DLO method

<details>
<summary>cURL</summary>

```
// Segment file by 200MB
$ split -d -b 209715200 large_obj.img large_obj.img.

// Upload segment object
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/003 \
-T large_obj.img.02

// Upload manifest object
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Object-Manifest: curl_example/large_obj.img/' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img \
-d ''
```
</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.nhn.cloud.obs;

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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        File objFile = new File(objectPath + "/" + objectName);
        int fileSize = (int)objFile.length();

        final int defaultChunkSize = 100 * 1024; // Segment by 100 KB
        int chunkSize = defaultChunkSize;
        int chunkNo = 0;  // Chunk number to create names for segment objects
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
                total_bytes_read += chunk_size
                f.seek(total_bytes_read)
                chunk_index += 1

        return self._create_manifest(container, object)


if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
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

<br/>

### Update an Object
Same as the Upload an Object API, but if the object is already located in the container, the content of the object is updated.

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
| X-Delete-At | Header | Timestamp | - | Object expiration date, Unix time (seconds) |
| X-Delete-After | Header | Timestamp | - | Object's expiration time, unix time (seconds) |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container |	URL | String | O | Container name |
| Object | URL | String | O | Name of the object to be updated |

#### Response
This API does not return a response body. For a valid request, return status code 201.

<br/>

### Download an Object
Downloads an object.

```
GET   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container |	URL | String | O | Container name |
| Object | URL | String | O | Name of the object to download |

#### Response
Data content of the object is returned to stream. For a valid request, return status code 200.

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -O -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 17166  100 17166    0     0   566k      0 --:--:-- --:--:-- --:--:--  578k
```
</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.nhn.cloud.obs;

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

        // Call API, data is received as byte array
        ResponseEntity<byte[]> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, byte[].class);

        // Create InputStream from byte array data and return it
        return new ByteArrayInputStream(response.getBody());
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
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

<br/>

### Copy an Object
Copies an object to another container.

> [Note]
> Multipart objects larger than 5 GB cannot be copied. 
> If you create a manifest object of the multipart object in a container to copy, the object can be accessed without having to copy segment objects. However, you cannot access the data if you delete the source segment objects.

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
| Destination | Header | String |	- | The target object to be copied (required when using the `{container}/{object}`<br/>COPY method) |
| X-Copy-From | Header | String |	- | The source object (required when using the `{container}/{object}`<br/>PUT method) |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container | URL | String | O |	Container name<br/>COPY Method: Original container<br/>PUT Method: Container to copy |
| Object | URL | String |	Name of the object to copy |

#### Response
This request does not return a response body. For a valid request, return status code 201.

#### Code Example
<details>
<summary>cURL</summary>

```
// COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/3a45e9.jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/3a45e9.jpg

// PUT method
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Copy-From: curl_example/3a45e9.jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/copy_con/3a45e9.jpg
```
</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.nhn.cloud.obs;

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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$DEST_CONTAINER = 'dest';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$object->copy($CONTAINER_NAME, $OBJECT_NAME, $DEST_CONTAINER);
?>
```
</details>

<br/>

### Modify Object Metadata
Modifies metadata of the specified object.

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
| X-Delete-At | Header | Timestamp | - | Object expiration date, Unix time (seconds) |
| X-Delete-After | Header | Timestamp | - | Object's expiration time, unix time (seconds) |
| X-Object-Worm-Retain-Until | Header | Timestamp | - | Object lock expiration date, Unix time (seconds)<br/>You can change the date after a set time, and only possible in object lock containers |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container | URL| String |	 O | Container name |
| Object | URL| String |  O | Name of the object for which metadata is to be modified |

> [Note]
> For objects uploaded to object lock containers, lock expiration dates are configured automatically. 
> You cannot overwrite or delete objects that have not passed the lock expiration date. 
> You can change the object's metadata even before the lock expiration date.

#### Response
This request does not return a response body. For a valid request, return status code 202.

#### Code Example
<details>
<summary>cURL</summary>

```
// Add metadata to object
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo" \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg

// Check metadata added to object header
$ curl -I -H "X-Auth-Token: b587ae461278419da6ecd21a2344c8aa" \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg
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
package com.nhn.cloud.obs;

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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
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

<br/>

### Delete an Object
Deletes a specified object.

> [Note]
> When deleting a multipart-uploaded object, you need to delete all segment data. If you delete only the manifest object, the segment objects might be kept intact and you might be charged for them.

```
DELETE   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | Token ID |
| Account | URL | String | O | Storage account, which can be found in the API Endpoint setting dialog box |
| Container | URL| String |	 O | Container name |
| Object | URL| String |  O | Name of the object to delete |

#### Response
This request does not return a response body. For a valid request, return status code 204.

<br/>

#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg
```
</details>

<details>
<summary>Java</summary>

```java
// ObjectService.java
package com.nhn.cloud.obs;

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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****';
$TOKEN_ID = 'd052a0a054b745dbac74250b7fecbc09';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$object->delete($CONTAINER_NAME, $OBJECT_NAME);
?>
```
</details>

<br/>

## References

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
