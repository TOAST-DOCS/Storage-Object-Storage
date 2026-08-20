<!-- machine_translated: true -->

<!-- pre-align:aligned sig=1c730003c3a0 -->

<a id="storage-object-storage-api-guide"></a>
## Storage > Object Storage > API Guide { #storage-object-storage-api-guide }

This document describes how to manage storage accounts, containers, and objects using the APIs provided by NHN Cloud Object Storage.

<a id="common"></a>
## Object Storage API Common Information { #common }

<a id="endpoint"></a>
### API Endpoint { #endpoint }

To use the APIs, you need an API endpoint and a token. See [IaaS token](/nhncloud/en/public-api/iaas-token/) to prepare the information required to use the APIs.
Object Storage APIs use the `object-store` type endpoint. For the exact endpoint, refer to `serviceCatalog` in the token issuance response.

| Region | Endpoint |
| --- | --- |
| Korea (Pangyo) region<br>Korea (Pyeongchon) region<br>Korea (Gwangju) region<br>Japan (Tokyo) region | https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\*<br>https://kr2-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\*<br>https://kr3-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\*<br>https://jp1-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\* |

<a id="auth"></a>
### Authentication and Authorization { #auth }

Object Storage uses the IaaS token for authentication/authorization when making API calls. The IaaS token is the authentication token used by NHN Cloud's OpenStack-based infrastructure service (IaaS).
For more information on IaaS token issuance and usage, see [IaaS token](/nhncloud/en/public-api/iaas-token/).

!!! danger "Caution"
    Object Storage has a different Tenant ID from the base infrastructure service.
    You can check the Object Storage Tenant ID by clicking the **Set API Endpoint** button on the Object Storage service page.

<!-- Line break comment -->

!!! tip "Note"
    You can also set the API password by clicking the **Set API Endpoint** button on the Object Storage service page.

<a id="auth-token-issuance-code-example"></a>
#### Token Issuance Code Examples

<details>
<summary>cURL</summary>

```
$ curl -X POST -H 'Content-Type: application/json' \
https://api-identity-infrastructure.nhncloudservice.com/v2.0/tokens \
-d '{"auth": {"tenantId": "6dbc368b94894416bec4cdfc65b5e067", "passwordCredentials": {"username": "*****", "password": "*****"}}}'

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
            "publicURL": "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067"
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

        // Create headers
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
  );  // Create request header

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

<a id="storage-account"></a>
## Storage Account { #storage-account }
A storage account is a character string in the `AUTH_*****` format, included in the `object-store` API endpoint.

<a id="query-the-storage-account"></a>
### Query the Storage Account { #query-the-storage-account }
Retrieves usage status of a storage account.

```
HEAD /v1/{Account}
X-Auth-Token: {token-id}
```

<a id="query-the-storage-account-request"></a>
#### Request
This request does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |

<a id="query-the-storage-account-response"></a>
#### Response
This API does not return a response body. Usage status is included in the header. For a valid request, return status code 200.

| Name | Type | Format | Description |
|---|---|---|---|
| X-Account-Container-Count | Header | String | Number of containers |
| X-Account-Object-Count | Header | String | Number of stored objects |
| X-Account-Bytes-Used | Header | String | Amount of stored data (bytes) |

<a id="query-the-storage-account-code-example"></a>
#### Code Examples

<details>
<summary>cURL</summary>

```
$ curl -I -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067
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

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        HashMap<String, String> status = new HashMap<String, String>();
        ResponseEntity<String> response
            = this.restTemplate.exchange(url, HttpMethod.HEAD, requestHttpEntity, String.class);
        if (response.getStatusCode() == HttpStatus.OK) {
            HttpHeaders responseHeaders = response.getHeaders();
            status.put("ContainerCount", responseHeaders.getFirst("X-Account-Container-Count"));
            status.put("ObjectCount", responseHeaders.getFirst("X-Account-Object-Count"));
            status.put("BytesUsed", responseHeaders.getFirst("X-Account-Bytes-Used"));
        }
        return status;
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";

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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'

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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';

$account = new Account($STORAGE_URL, $TOKEN_ID);
$status = $account->get_status();

printf("Container-Count: %d\n", $status["X-Account-Container-Count"]);
printf("Object-Count: %d\n", $status["X-Account-Object-Count"]);
printf("Bytes-Used: %d\n", $status["X-Account-Bytes-Used"]);
?>
```
</details>

<br>

<a id="list-containers"></a>
### List Containers { #list-containers }
Lists containers of a storage account.

```
GET /v1/{Account}
X-Auth-Token: {token-id}
```

<a id="list-containers-request"></a>
#### Request
This request does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |

<a id="list-containers-response"></a>
#### Response
```
[List of containers in a storage account]
```

<a id="list-containers-code-example"></a>
#### Code Examples

<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067
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
        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        ResponseEntity<String> response
            = this.restTemplate.exchange(this.getStorageUrl(), HttpMethod.GET, requestHttpEntity, String.class);

        List<String> containerList = null;
        if (response.getStatusCode() == HttpStatus.OK) {
            // Convert the list received as a String to an array
            containerList = Arrays.asList(response.getBody().split("\\r?\\n"));
        }

        // Convert the array to a List and return
        return new ArrayList<String>(containerList);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';

$account = new Account($STORAGE_URL, $TOKEN_ID);
$container_list = $account->get_container_list();
foreach($container_list as $container) {
  printf("%s\n", $container);
}
?>
```
</details>

<br>

<a id="container"></a>
## Container { #container }

<a id="create-a-container"></a>
### Create a Container { #create-a-container }
Creates a container. To upload files to object storage, a container must be created.

!!! tip "Note"
    A container name cannot include the special characters ``' " ` < > ;``, spaces, and relative path characters (`. ..`).

    Names in IP address format cannot be used.

    If a container or object name contains the special characters `! * ' ( ) ; : @ & = + $ , / ? # [ ]`, you must apply URL encoding (percent encoding) when using the API. These characters are reserved characters that are used in URLs. If you send an API request without URL-encoding a path that contains these characters, you will not receive the expected response.

When you create a container, you can use the `X-Storage-Policy` header to specify the storage class of the container. You can choose the Standard class for frequently accessed data and the Economy class for long-term storage of less frequently accessed data at a lower cost. If you don't specify a storage class, the Standard class is applied.

!!! tip "Note"
    You cannot change the storage class of an already-created container.

    Objects uploaded to Economy class containers are subject to a minimum storage period of 30 days. Objects deleted before 30 days are also charged for the remaining storage period.

    Economy class containers are charged per 1,000 API requests (excluding HEAD/DELETE requests).

You can create an object lock container by setting the object lock interval using the `X-Container-Worm-Retention-Day` header when creating the container. Objects uploaded to the object lock container are stored using the **WORM (Write-Once-Read-Many)** model. For objects uploaded to the object lock container, the lock expiration date is configured. You cannot overwrite or delete objects before the lock expiration date set on each object.

<br>

```
PUT /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

<a id="create-a-container-request"></a>
#### Request
No request body is required.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, available in the API endpoint configuration dialog box |
| Container | URL | String | Y | Name of the container to create |
| X-Storage-Policy | Header | String | N | Storage class of the container<br>**Standard**: Default class for frequently accessed data<br>**Economy**: Class ideal for long-term storage of infrequently accessed data |
| X-Container-Worm-Retention-Day | Header | Integer | N | Sets the default object lock interval for the container, in days |

<a id="create-a-container-response"></a>
#### Response
No response body is returned. If the container is created successfully, status code 201 is returned.

<a id="create-a-container-code-example"></a>
#### Code Examples
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example
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

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'

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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->create($CONTAINER_NAME);
?>
```
</details>

<br>

<a id="get-a-container"></a>
### Get a Container { #get-a-container }
Retrieves information about the specified container and the list of objects stored in it. The container's information can be viewed in the response header.

```
GET /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

<a id="get-a-container-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint setting dialog |
| Container | URL | String | Y | Name of the container to retrieve |
| marker | Query | String | N | Reference object name |
| prefix | Query | String | N | Prefix to search for |
| limit | Query | Integer | N | Number of objects to display in the list |
| format | Query | String | N | Response format, json or xml |

!!! tip "Note"
    Get Container API provides a number of queries. Each query can be concatenated using `&`.

<a id="list-objects-over-10k"></a>
##### Retrieve a List of More Than 10,000 Objects
The number of objects that can be retrieved using Get Container API is limited to 10,000. If you want to retrieve more than 10,000 objects, you need to use the `marker` query. The marker query returns up to 10,000 additional objects, starting from the next object of the specified object.

<br>

<a id="list-objects-with-a-prefix"></a>
##### Retrieve a List of Objects Starting with a Prefix
Using the `prefix` query returns the list of objects that start with the specified prefix. The prefix query can be used to retrieve the list of objects in subfolders.

<br>

<a id="list-objects-with-limit"></a>
##### Specify the Maximum Number of Objects in the List
Using the `limit` query allows you to specify the maximum number of objects in the list of objects to be returned.

<br>

<a id="list-objects-with-format"></a>
##### Specify the Response Format
Using the `format` query allows you to specify a `json` or `xml` response format. If the response format is specified, the response body includes metadata for each object (size, content type, last modified time, ETag).

<br>

<a id="get-a-container-response"></a>
#### Response

```
[The list of object in the container]
```

<a id="get-a-container-code-example"></a>
#### Code Examples
<details>
<summary>cURL</summary>

```
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example
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

    public List<String> getObjectList(String containerName) {
        return this.getList(this.getUrl(containerName));
    }

    public List<String> getList(String url) {
        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        ResponseEntity<String> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, String.class);

        if (response.getStatusCode() == HttpStatus.OK) {
            // Convert the list received as a String to an array
            return Arrays.asList(response.getBody().split("\\r?\\n"));
        }

        return Collections.emptyList();
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$object_list = $container->get_object_list($CONTAINER_NAME);
foreach ($object_list as $obj) {
  printf("%s\n", $obj);
}
?>
```
</details>

<br>

<a id="change-container-settings"></a>
### Change Container Settings { #change-container-settings }

Changes the container settings. The container settings can be found in the response header when retrieving the container.

```
POST /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {Role-based access rules for container read}
X-Container-Write: {Role-based access rules for container write}
X-Container-View: {Role-based access rules for container view}
X-Container-Ip-Acl-Allowed-List: {IP-based access rules for container access}
X-Container-Ip-Acl-Denied-List: {IP-based access rules for container access}
X-Container-Ip-Acl-Service-Gateway-Control: {Access permission for requests through the service gateway}
X-Container-Object-Lifecycle: {Object lifecycle for the container}
X-Container-Object-Transfer-To: {Containers to move when an object's lifecycle expires}
X-History-Location: {Container to store previous versions of objects}
X-Versions-Retention: {Lifecycle of previous versions of objects}
X-Container-Meta-Web-Index: {Static website index document object}
X-Container-Meta-Web-Error: {Static website error document object suffix}
X-Container-Meta-Access-Control-Allow-Origin: {Cross-origin resource sharing allowlist}
X-Container-Rfc-Compliant-Etags: {Whether to use RFC-compliant ETag format}
X-Container-Worm-Retention-Day: {Object lock period for the container}
X-Container-Object-Deny-Extension-Policy: {Blacklist for object upload policy extensions}
X-Container-Object-Deny-Keyword-Policy: {Blacklist for object upload policy filenames}
X-Container-Object-Allow-Extension-Policy: {Whitelist for object upload policy extensions}
X-Container-Object-Allow-Keyword-Policy: {Whitelist for object upload policy filenames}
```

<a id="change-container-settings-request"></a>
#### Request
A request body is not required.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| X-Container-Read | Header | String | N | Sets the role-based access rules for container read |
| X-Container-Write | Header | String | N | Sets the role-based access rules for container write |
| X-Container-View | Header | String | N | Sets the role-based access rules for container view |
| X-Container-Ip-Acl-Allowed-List | Header | String | N | Sets the IP-based access rules for container access |
| X-Container-Ip-Acl-Denied-List | Header | String | N | Sets the IP-based access rules for container access |
| X-Container-Ip-Acl-Service-Gateway-Control | Header | String | N | Sets the access permission for service gateway requests: `read`, `write`, `rw`, `deny` |
| X-Container-Object-Lifecycle | Header | Integer | N | Sets the default object lifecycle for the container, in days |
| X-Container-Object-Transfer-To | Header | String | N | Container to move objects to when their lifecycle expires |
| X-History-Location | Header | String | N | Sets the container for storing previous versions of objects |
| X-Versions-Retention | Header | Integer | N | Sets the life cycle of the object's previous version in days |
| X-Container-Meta-Web-Index | Header | String | N | Sets the static website index document object<br>Only alphanumeric characters and some special characters (`-`, `_`, `.`, `/`) are allowed |
| X-Container-Meta-Web-Error | Header | String | N | Sets the static website error document object suffix<br>Only alphanumeric characters and some special characters (`-`, `_`, `.`, `/`) are allowed |
| X-Container-Meta-Access-Control-Allow-Origin | Header | String | N | List of hosts allowed for CORS. You can allow all hosts by entering `*`, or enter a list of hosts separated by spaces. |
| X-Container-Rfc-Compliant-Etags | Header | String | N | Sets whether to use RFC-compliant ETag format; true or false |
| X-Container-Worm-Retention-Day | Header | Integer | N | Sets the default object lock cycle for the container, in days<br>Can only be changed in an object lock container |
| X-Container-Object-Deny-Extension-Policy | Header | String | N | Extension blacklist for the object upload policy |
| X-Container-Object-Deny-Keyword-Policy | Header | String | N | Filename blacklist for the object upload policy |
| X-Container-Object-Allow-Extension-Policy | Header | String | N | Extension whitelist for the object upload policy |
| X-Container-Object-Allow-Keyword-Policy | Header | String | N | Filename whitelist for the object upload policy |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Name of the container to modify |

<br>

<a id="set-container-rbac-policy"></a>
##### Access Policy Configuration
You can configure container access policies using the `X-Container-Read`, `X-Container-Write`, `X-Container-View`, `X-Container-Ip-Acl-Allowed-List`, `X-Container-Ip-Acl-Denied-List`, and `X-Container-Ip-Acl-Service-Gateway-Control` headers. For more information, see the [Access Policy Configuration Guide](acl-guide/).

<br>

<a id="set-container-object-lifecycle"></a>
##### Set Object Lifecycle
You can use the `X-Container-Object-Lifecycle` header to set the lifecycle of objects stored in the container, in days. This setting applies only to objects uploaded after the configuration is applied.
You can use the `X-Container-Object-Transfer-To` header to move objects whose lifecycle has expired to a specified container for storage. If no container is specified, expired objects are deleted.

!!! tip "Note"
    You can configure detailed lifecycle rules through container policies.
    For more information, see the [Container Policy Configuration Guide](container-policy-guide/#lifecycle).

<!-- Line break comment -->

!!! tip "Note"
    You can move objects stored in a Standard class container to an Economy class container according to their lifecycle to reduce costs associated with long-term storage.

<br>

<a id="set-container-object-version-policy"></a>
##### Set Version Control Policy
As described in the [Modify an Object](api-guide/#update-an-object) section, when you upload an object and an object with the same name already exists, the object is updated. If you want to retain the contents of the existing object, you can use the `X-History-Location` header to specify an **archive container** in which to store previous versions.

Previous version objects are stored in the archive container in the following format:
```
{length of an object name in a 3-digit hexadecimal number}{object name}/{unix time when the previous version was stored}
```
For example, if you update an object named `picture.jpg`, an object named `00bpicture.jpg/1610606551.82539` is created in the archive container.

When you delete an object from a container with a version control policy configured, the deleted object is stored in the archive container and a delete marker object is created. You can access previous version objects stored in the archive container at any time.

You can use the `X-Versions-Retention` header together to set the lifecycle of previous version objects in days. If you set it to 1, stored objects are automatically deleted after 1 day. If not configured, previous version objects are retained until you delete them. This setting applies only to previous version objects stored after the configuration is applied.

!!! danger "Caution"
    If the archive container is deleted before the original container, an error occurs when updating or deleting objects in the original container. If the archive container has already been deleted, you can solve the issue by creating a new archive container or disabling the original container's version control policy.

It is recommended that you avoid using Unicode characters in container names for archive containers. If the name of the container to set as an archive container contains Unicode characters, it must be URL-encoded before being entered in the request header.

If you specify an encryption container as the archive container and then delete the symmetric key from Secure Key Manager, the object of the original container fails to be uploaded and deleted.

<br>

<a id="set-container-static-website"></a>
##### Configure a Static Website
If you allow the container read access to all users and set the static website's index document and error document using the `X-Container-Meta-Web-Index` and `X-Container-Meta-Web-Error` headers, you can host a static website using the container URL.

The name for an object to be used as an index document or error document for a static website must consist of one or more alphanumeric characters, or some special characters (`-`, `_`, `.`, `/`), and the file extension must be `.html` in hypertext format. If the conditions are not satisfied, you cannot configure the settings or the static website may not work.
The name for an error document of a static website has the form of `{response code}{suffix}`. For example, if you configure the error document as `error.html`, the name for an error document to display when a 404 error occurs is `404error.html`. You can upload and use error documents for each error situation. If error documents are not defined or error objects that match the response code do not exist, the default error document of a web browser will be displayed.
<br>

<a id="set-container-cors-policy"></a>
##### Cross-Origin Resource Sharing (CORS)

If you directly call the Object Storage API from the browser, you need to set Cross-Origin Resource Sharing (CORS). Set an allowed-origin list using the `X-Container-Meta-Access-Control-Allow-Origin` header. You can enter one or more origins separated by spaces (` `) or allow all origins by entering `*`.

!!! tip "Note"
    A maximum of 100 allowed origins can be configured in `X-Container-Meta-Access-Control-Allow-Origin`. This limit also applies when configuring through [container policies](container-policy-guide/#cors).

<details>
<summary>View CORS configuration example</summary>

Add a CORS configuration to the container.

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Meta-Access-Control-Allow-Origin: https://example.com' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container
```
<br>
After navigating to the site for which CORS is allowed in the browser, run the script below. The script can be run in the console of the developer tools provided by the browser.

<br>
ex) `https://example.com/`

```
var token = "****";
var url = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container/object";
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
If there are no issues with the CORS configuration, you can confirm a successful response like the following in the console.

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
If CORS is not configured or if the API is called from a site that is not allowed, the following error response is returned.

```
Access to XMLHttpRequest at 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container/object' from origin 'https://example.com' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.

Status: 0
```

</details>

<br>

<a id="set-container-rfc-compliant-etag"></a>
##### Configure RFC-Compliant ETag Format
Some applications require ETag values enclosed in double quotes in accordance with the [RFC7232](https://www.rfc-editor.org/rfc/rfc7232#section-2.3) specification. You can use the `X-Container-Rfc-Compliant-Etags` header to configure the container to return ETag values enclosed in double quotes when retrieving objects stored in the container.

<br>

<a id="set-container-object-lock-cycle"></a>
##### Change Object Lock Period
Use the `X-Container-Worm-Retention-Day` header to change the object lock cycle of an object lock container. The lock cycle can be entered in days and cannot be disabled. The changed lock cycle is applied to objects uploaded after the change. The object lock cycle can only be changed in an object lock container.

!!! tip "Note"
    You cannot change a general container to an object lock container or an object lock container to a general container.

You cannot specify an object lock container as an archive container or replication target container.

<br>

<a id="set-container-upload-policy"></a>
##### Change Upload Policy Configuration
You can use the `X-Container-Object-Deny-Extension-Policy`, `X-Container-Object-Deny-Keyword-Policy`, `X-Container-Object-Allow-Extension-Policy`, and `X-Container-Object-Allow-Keyword-Policy` headers to configure object name-based upload policies for a container. By using upload policy settings, you can restrict uploads so that only objects whose names contain specific extensions or keywords can be uploaded, or prevent such objects from being uploaded.

The upload policy applies to objects uploaded after the policy is configured. For objects that include a path, the policy is applied to the object name without the path.
All upload policy headers can accept multiple rules using the `,` delimiter, and each rule except the `,` delimiter must be URL-encoded (percent-encoded).
Extension rules check the file extension, and filename rules check whether the object name contains the specified value. Extension rules must be entered without the `.`. For example, to enter the txt extension, enter only `txt`, not `.txt`.

Upload policies cannot use a whitelist and a blacklist at the same time. If you request that both be configured, a failure response is returned.

<details>
<summary>View whitelist configuration example</summary>

Add a whitelist upload policy configuration to the container.

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Object-Allow-Extension-Policy: exe, jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container

$ curl -X PUT \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container/test.jpg -i

HTTP/1.1 409 Conflict
Content-Length: 72
Content-Type: text/html; charset=UTF-8
X-Trans-Id: txddeb34d60f7f4b43a8b2a-0065b8b134
X-Openstack-Request-Id: txddeb34d60f7f4b43a8b2a-0065b8b134
Date: Tue, 30 Jan 2024 08:20:04 GMT

Only the objects with the following extensions can be uploaded: exe, jpg
```

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Object-Allow-Keyword-Policy: example' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container

$ curl -X PUT \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container/upload.txt -i

HTTP/1.1 409 Conflict
Content-Length: 60
Content-Type: text/html; charset=UTF-8
X-Trans-Id: tx24209f2af02b4de0a4921-0065b8b192
X-Openstack-Request-Id: tx24209f2af02b4de0a4921-0065b8b192
Date: Tue, 30 Jan 2024 08:21:38 GMT

The object name must contain the following keywords: example
```

</details>

<details>
<summary>View blacklist configuration example</summary>

Add a blacklist upload policy configuration to the container.

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Object-Deny-Extension-Policy: exe, jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container

$ curl -X PUT \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container/test.jpg -i

HTTP/1.1 409 Conflict
Content-Length: 70
Content-Type: text/html; charset=UTF-8
X-Trans-Id: tx4a0f746118e9453ca8688-0065b8b038
X-Openstack-Request-Id: tx4a0f746118e9453ca8688-0065b8b038
Date: Tue, 30 Jan 2024 08:15:52 GMT

The objects with the following extensions cannot be uploaded: exe, jpg
```

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Object-Deny-Keyword-Policy: example' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container

$ curl -X PUT \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container/upload_example.txt -i

HTTP/1.1 409 Conflict
Content-Length: 64
Content-Type: text/html; charset=UTF-8
X-Trans-Id: tx60aaa14186d84cca88a8e-0065b8b098
X-Openstack-Request-Id: tx60aaa14186d84cca88a8e-0065b8b098
Date: Tue, 30 Jan 2024 08:17:28 GMT

The object name must not contain the following keywords: example
```

</details>

<a id="unset-container-settings"></a>
##### Unset Container Settings
If you use a header without a value, the setting will be removed. For example, if the life cycle of an object is set to 3 days and you request to edit the container using `'X-Container-Object-Lifecycle: '`, the object life cycle will be removed, and the objects that are stored in the container afterwards will not have their life cycle automatically set.
<br>

<a id="change-container-settings-response"></a>
#### Response
This API does not return a response body. When the request is appropriate, return status code 204.
<br>

<a id="change-container-settings-code-example"></a>
#### Code Example
This is an example in which the user requests changing the setting so that all users may read from and write to containers. You can select the headers you need to change the settings and request in the same way.

<details>
<summary>cURL</summary>

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Read: .r:*' \
-H 'X-Container-Write: *:*' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example
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

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Container-Read", permission);    // Add permission to header

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
        final String containerName = "test";

        ContainerService containerService = new ContainerService(storageUrl, tokenId);

        try {
            containerService.setContainerReadACL(containerName, true);
            System.out.println("Container " + containerName + " became public.");
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
class ContainerService:
    # ...
    def set_read_acl(self, container, is_public):
        req_url = self._get_url(container)
        req_header = self._get_request_header()
        req_header['X-Container-Read'] = '.r:*' if is_public else ''
        return requests.post(req_url, headers=req_header)

if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->set_acl($CONTAINER_NAME, TRUE);
?>
```
</details>

<br>

<a id="delete-a-container"></a>
### Delete a Container { #delete-a-container }

Deletes the specified container. The container to be deleted must be empty.

```
DELETE /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

<a id="delete-a-container-request"></a>
#### Request
A request body is not required.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Name of the container to delete |

<a id="delete-a-container-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 204.

<br>

<a id="delete-a-container-code-example"></a>
#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example
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

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.DELETE, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
        final String containerName = "test";

        ContainerService containerService = new ContainerService(storageUrl, tokenId);

        try {
            containerService.deleteContainer(containerName);
            System.out.println("Container " + containerName + " deleted.");
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
class ContainerService:
    # ...
    def delete(self, container):
        req_url = self._get_url(container)
        req_header = self._get_request_header()
        return requests.delete(req_url, headers=req_header)

if __name__ == '__main__':
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';

$container = new Container($STORAGE_URL, $TOKEN_ID);

$container->delete($CONTAINER_NAME);
?>
```
</details>

<br>

<a id="object"></a>
## Object { #object }

<a id="upload-an-object"></a>
### Upload an Object { #upload-an-object }
Uploads a new object to the specified container.

```
PUT /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

<a id="upload-an-object-request"></a>
#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Content-Type | Header | String | Y | Content type of the object |
| X-Delete-At | Header | Timestamp | N | Object expiration date, Unix time (seconds) |
| X-Delete-After | Header | Timestamp | N | Object validity period, Unix time (seconds) |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the object to create |
| - | Body | Binary | Y | Content of the object to create |

<a id="set-object-lifecycle"></a>
##### Set Object Lifecycle
You can set the lifecycle of an object in seconds by using the `X-Delete-At` or `X-Delete-After` header.
<br>

!!! danger "Caution"
    If an object name starts with `./` or `../`, the browser regards it as path character and access is unavailable on web console.
    If you have uploaded an object of such name via API, it must also be accessed via API.

<a id="upload-an-object-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 201.

<a id="upload-an-object-code-example"></a>
#### Code Examples
<details>
<summary>cURL</summary>

```
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/ba6610.jpg \
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

        // Configure settings to use the overridden RequestCallback
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setBufferRequestBody(false);
        RestTemplate restTemplate = new RestTemplate(requestFactory);

        HttpMessageConverterExtractor<String> responseExtractor
            = new HttpMessageConverterExtractor<String>(String.class, restTemplate.getMessageConverters());

        // Call API
        restTemplate.execute(url, HttpMethod.PUT, requestCallback, responseExtractor);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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

    $fd = fopen($filename, 'r');  // Open the file.

    $curl  = curl_init($req_url);
    curl_setopt_array($curl, array(
      CURLOPT_PUT => TRUE,
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_INFILE => $fd,  // Pass the file stream as a parameter.
      CURLOPT_HTTPHEADER => $req_header
    ));
    $response = curl_exec($curl);
    curl_close($curl);

    fclose($fd);
  }
}

// main
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
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

<br>

<a id="multipart-upload"></a>
### Multipart Upload { #multipart-upload }
An object whose size exceeds 5GB needs to be divided into segments of 5GB or smaller before uploading. If you upload segment objects and create a manifest object, you can use them as if they are a single object.

<br>

<a id="upload-segment-object"></a>
#### Upload Segment Objects
Upload each segment object that the original object has been divided into.

```
PUT /v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

<br>

##### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Content-Type | Header | String | Y | Content type of the object |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the object to create |
| Count | URL | String | Y | Sequence number of the divided object, e.g. 001, 002 |
| - | Body | Binary | Y | Content of the divided object |

<br>

##### Response
This API does not return a response body. For a valid request, return status code 201.

<br>

<a id="upload-manifest-object"></a>
#### Create Manifest Object
A manifest object can be created in two ways: either using **DLO** (Dynamic Large Object) or **SLO** (Static Large Object).

!!! tip "Note"
    Because a manifest object has path information for segment objects, there is no need to upload segment objects and the manifest object in the same container. If segment objects and manifest objects are in a single container and it is difficult to manage them, it is recommended to upload segment objects to a separate container and create only the manifest object in the container where you originally intended to upload the objects.

**DLO**

The DLO manifest object uses the path to the segment objects specified in the `X-Object-Manifest` header to automatically locate and connect to segment objects.

```
PUT /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Segment-Container}/{Segment-Object}/
```

<br>

##### Request
| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| X-Object-Manifest | Header | String | Y | The path where segment objects are uploaded: `{Segment-Container}/{Segment-Object}/` |
| Account | URL | String | Y | Storage account, found in the API endpoint configuration dialog box |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the manifest object to create |
| - | Body | Binary | Y | Empty data |

<br>

**SLO**

To create an SLO manifest object, you must enter the list of segment objects in order in the request body. Up to 10,000 segment objects can be entered.
If you request the creation of an SLO manifest object, the system checks whether each segment object is in the specified path and whether the ETag value matches the object's size. If the information does not match, the manifest object is not created.

```
PUT /v1/{Account}/{Container}/{Object}?multipart-manifest=put
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
<br>

##### Request
| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, found in the API endpoint configuration dialog box |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the manifest object to create |
| multipart-manifest | Query | String | Y | Set to `put` when creating a manifest |
| path | Body | String | Y | Path of the segment object |
| etag | Body | String | Y | ETag of the segment object |
| size_bytes | Body | Integer | Y | Size of the segment object (in bytes) |

!!! tip "Note"
    To retrieve the segment information held by SLO manifest file, you must use the `multipart-manifest=get` query.

<br>

##### Response
This API does not return a response body. For a valid request, return status code 201.

<br>

<a id="multipart-upload-code-example"></a>
#### Code Examples
DLO method multipart upload example

<details>
<summary>cURL</summary>

```
// Split the file into 200 MB chunks
$ split -d -b 209715200 large_obj.img large_obj.img.

// Upload the split objects
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/large_obj.img/003 \
-T large_obj.img.02

// Upload the manifest object
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Object-Manifest: curl_example/large_obj.img/' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/large_obj.img \
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
        String manifestName = containerName + "/" + objectName + "/"; // Generate manifest name

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Object-Manifest", manifestName);  // Add manifest to header

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        File objFile = new File(objectPath + "/" + objectName);
        int fileSize = (int)objFile.length();

        final int defaultChunkSize = 100 * 1024; // Split into 100 KB chunks
        int chunkSize = defaultChunkSize;
        int chunkNo = 0;  // Chunk number used to generate the name of each split object
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

                // Read data of chunk size into byte buffer
                byte[] chunkBuffer = new byte[chunkSize];
                int bytesRead = inputStream.read(chunkBuffer, 0, chunkSize);

                if(bytesRead > 0) {
                    // Convert buffer data to InputStream and upload using the uploadObject() method from the object upload example
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
        req_header['X-Object-Manifest'] = '/'.join([container, object]) + '/'
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
    $fd = fopen($filename, 'r');  // Open the file.
    $obj_size = filesize($filename);

    while($total_bytes_read < $obj_size) {
      // Calculate the size to split
      $remained_bytes = $obj_size - $total_bytes_read;
      if ($remained_bytes < $chunk_size) {
        $chunk_size = $remained_bytes;
      }
      $chunk = fread($fd, $chunk_size);
      // Generate the part name
      $temp_file = sprintf("./multipart-%03d", $chunk_index);
      $req_url = sprintf("%s/%03d", $url, $chunk_index);

      // Create a temporary file for the part
      $part_fd = fopen($temp_file, 'w+');
      fwrite($part_fd, $chunk);
      fseek($part_fd, 0);

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
        CURLOPT_PUT => TRUE,
        CURLOPT_HEADER => TRUE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_INFILE => $part_fd,  // Pass the part file stream as a parameter
        CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
      printf("$response");

      // Delete the temporary file
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';
$LARGE_OBJECT = '8cb0d624f8c14c69b52f2cd89e5e59b7.jpg';
$OBJ_PATH = '/home/example';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$filename = $OBJ_PATH.'/'.$LARGE_OBJECT;
$object->upload_large_object($CONTAINER_NAME, $LARGE_OBJECT, $filename);
?>
```
</details>

<br>

<a id="update-an-object"></a>
### Update an Object { #update-an-object }
Same as the Upload an Object API, but if the object is already located in the container, the content of the object is updated.

```
PUT /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

<a id="update-an-object-request"></a>
#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Content-Type | Header | String | Y | Content type of the object |
| X-Delete-At | Header | Timestamp | N | Object expiration date, Unix time in seconds |
| X-Delete-After | Header | Timestamp | N | Object validity period, Unix time in seconds |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the object whose content is to be updated |
| - | Body | Binary | Y | Content to update the object with |

<a id="update-an-object-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 201.

<br>

<a id="query-object-information"></a>
### Query Object Information { #query-object-information }
Retrieves the information about the specified object. The object information can be found in the response header.

```
HEAD /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

<a id="query-object-information-request"></a>
#### Request
A request body is not required.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the object to download |

<a id="query-object-information-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 200.

| Name | Type | Format | Description |
|---|---|---|---|
| Content-Type | Header | String | Content type of the object |
| Content-Length | Header | Integer | Size of the object |
| Etag | Header | String | ETag value of the object<br>MD5 hash value of the object.<br>Can be used to verify the integrity of the object. |
| Last-Modified | Header | String | Last modified time of the object |
| X-Timestamp | Header | Timestamp | Last modified time of the object, in Unix time (seconds) |
| X-Delete-At | Header | Timestamp | Expiration date of the object, in Unix time (seconds) |
| X-Object-Worm-Retain-Until | Header | Timestamp | Object lock expiration date, in Unix time (seconds) |
| X-Object-Manifest | Header | String | Segment object path for DLO method multipart objects |
| X-Static-Large-Object | Header | Boolean | Whether the object is an SLO-style multipart object |
| X-Manifest-Etag | Header | String | Manifest ETag values (MD5) for SLO-style multipart objects |

<a id="query-object-information-code-example"></a>
#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -I -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/ba6610.jpg

HTTP/1.1 200 OK
content-type: image/jpeg
content-length: 148585
x-delete-at: 1729263600
etag: bad093d7f49dc495751cb3f7f8b2530c
last-modified: Mon, 30 May 2022 15:16:43 GMT
x-timestamp: 1653923802.28970
x-trans-id: tx3c30a8f0272c40f5979b4-0067104fa7
x-openstack-request-id: tx3c30a8f0272c40f5979b4-0067104fa7
date: Wed, 16 Oct 2024 23:43:36 GMT
```
</details>

<br>

<a id="download-an-object"></a>
### Download an Object { #download-an-object }
Downloads an object.

```
GET /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

<a id="download-an-object-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the object to download |

<a id="download-an-object-response"></a>
#### Response
Data content of the object is returned to stream. For a valid request, return status code 200.

<a id="download-an-object-code-example"></a>
#### Code Examples
<details>
<summary>cURL</summary>

```
$ curl -O -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/ba6610.jpg

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

    public File downloadObject(String containerName, String objectName, String downloadPath) {
        String url = this.getUrl(containerName, objectName);

        // RequestCallback that adds a token to the request header
        RequestCallback callback = (request) -> {
            HttpHeaders headers = request.getHeaders();
            headers.add("X-Auth-Token", tokenId);
            headers.setAccept(Collections.singletonList(MediaType.APPLICATION_OCTET_STREAM));
        };

        // Extractor that receives the response and saves it
        ResponseExtractor<File> extractor = (clientHttpResponse) -> {
            File ret = new File(downloadPath + "/" + objectName);
            StreamUtils.copy(clientHttpResponse.getBody(), Files.newOutputStream(ret.toPath()));
            return ret;
        };

        return this.restTemplate.execute(url, HttpMethod.GET, callback, extractor);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
        final String containerName = "test";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";
        final String downloadPath = "/home/example/download";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // Download object
            objectService.downloadObject(containerName, objectName, downloadPath);
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';
$DOWNLOAD_PATH = '/home/example/download';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$filename = $DOWNLOAD_PATH.'/'.$OBJECT_NAME;
$object->download($CONTAINER_NAME, $OBJECT_NAME, $filename);
?>
```
</details>

<br>

<a id="copy-an-object"></a>
### Copy an Object { #copy-an-object }
Copies an object to another container. All properties of the source object are copied along with it.

```
COPY /v1/{Account}/{SourceContainer}/{SourceObject}
X-Auth-Token: {token-id}
Destination: {TargetContainer}/{TargetObject}
```

```
PUT /v1/{Account}/{TargetContainer}/{TargetObject}
X-Auth-Token: {token-id}
X-Copy-From: {SourceContainer}/{SourceObject}
```

<a id="copy-an-object-request"></a>
#### Request
A request body is not required.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Destination | Header | String | Conditional | Target object path, `{target container}/{target object}`<br>Required when using the COPY method |
| X-Copy-From | Header | String | Conditional | Source object path, `{source container}/{source object}`<br>Required when using the PUT method |
| X-Fresh-Metadata | Header | Boolean | N | Whether to reset the object's properties<br>If set to true, the source object's properties are not copied.<br>The default value is false. |
| X-Object-Meta-{Key} | Header | String | N | Metadata of the target object |
| X-Delete-At | Header | Timestamp | N | Expiration date of the target object, in Unix time (seconds) |
| X-Delete-After | Header | Timestamp | N | Validity period of the target object, in Unix time (seconds) |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name<br>COPY method: source container<br>PUT method: target container |
| Object | URL | String | Y | Object name<br>COPY method: source object<br>PUT method: target object |
| multipart-manifest | Query | String | N | If the value is get, only the manifest object is copied<br>If omitted, segments are merged and copied as a single object.<br>COPY method: add as a query parameter<br>PUT method: add to the `X-Copy-From` header value |

<a id="preserve-object-properties"></a>
##### Preserve Object Properties
When you copy an object, the source object's properties are copied along with it. The properties that are preserved are as follows:

| Name | Description |
|---|---|
| X-Delete-At | Object expiration date |
| X-Object-Worm-Retain-Until | Object lock expiration date |
| X-Object-Meta-{Key} | User-defined metadata |

!!! tip "Note"
    When copying an object, you can set the copied object's properties to new values by adding the `X-Delete-At` or `X-Object-Meta-{key}` header.
    However, the lock expiration period cannot be changed and the value from the source object is retained as-is.

<a id="copy-a-multipart-object"></a>
##### Copy a Multipart Object
When you copy a multipart object, the segments referenced by the manifest are merged into a single object and copied. Therefore, multipart objects larger than 5 GB cannot be copied using the standard method. To copy a multipart object larger than 5 GB, you must copy only the manifest object. You can specify the manifest as the source by adding the `multipart-manifest=get` parameter to the request.

```
COPY /v1/{Account}/{SourceContainer}/{SourceObject}?multipart-manifest=get
X-Auth-Token: {token-id}
Destination: {TargetContainer}/{TargetObject}
```

```
PUT /v1/{Account}/{TargetContainer}/{TargetObject}
X-Auth-Token: {token-id}
X-Copy-From: {SourceContainer}/{SourceObject}; multipart-manifest=get
```

!!! tip "Note"
    When copying a manifest using the PUT method, you must add the `multipart-manifest=get` parameter to the `X-Copy-From` header value, separated by a semicolon.

<!-- Line break comment -->

!!! danger "Caution"
    Because the copied manifest references the source segment paths, you cannot access the data if you delete the source segment objects.
    If you have copied the source segment objects to another container, you must create a new manifest object.

When copying a manifest, its properties are copied along with it.

| Type | Copied Properties |
|---|---|
| SLO manifest | X-Static-Large-Object, X-Manifest-Etag |
| DLO manifest | X-Object-Manifest |

<a id="copy-an-object-response"></a>
#### Response
This request does not return a response body. For a valid request, return status code 201.

<a id="copy-an-object-code-example"></a>
#### Code Examples
<details>
<summary>cURL</summary>

**Copy a Single Object**
```
# COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/3a45e9.jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/3a45e9.jpg

# PUT method
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Copy-From: curl_example/3a45e9.jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/copy_con/3a45e9.jpg
```

**Copy a Multipart Manifest Object**
```
# COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/419da6e.mp4' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/419da6e.mp4?multipart-manifest=get

# PUT method
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Copy-From: curl_example/419da6e.mp4; multipart-manifest=get' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/copy_con/419da6e.mp4
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

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Copy-From", srcObject);    // Specify the source object
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // HttpMethod does not support the COPY method, so call the alternative API that uses the PUT method.
        this.restTemplate.exchange(url, HttpMethod.PUT, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
    curl_close($curl);
  }
}

// main
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';
$DEST_CONTAINER = 'dest';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$object->copy($CONTAINER_NAME, $OBJECT_NAME, $DEST_CONTAINER);
?>
```
</details>

<br>

<a id="modify-object-metadata"></a>
### Modify Object Metadata { #modify-object-metadata }
Modifies the metadata of the specified object.

```
POST /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Meta-{Key}: {Value}
```

<a id="modify-object-metadata-request"></a>
#### Request
A request body is not required.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| X-Object-Meta-{Key} | Header | String | N | Metadata to change |
| X-Delete-At | Header | Timestamp | N | Object expiration date, Unix time in seconds |
| X-Delete-After | Header | Timestamp | N | Object validity period, Unix time in seconds |
| X-Object-Worm-Retain-Until | Header | Timestamp | N | Object lock expiration date, Unix time in seconds<br>Can only be changed to a time after the configured time, and only modifiable in an object lock container |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the object whose metadata is to be modified |

!!! tip "Note"
    Objects uploaded to the Object Lock container are automatically assigned a lock expiration date.

    Objects whose lock expiration date has not yet passed cannot be overwritten or deleted.

    The metadata of an object can be changed even before the lock expiration date.

<a id="modify-object-metadata-response"></a>
#### Response
This request does not return a response body. For a valid request, return status code 202.

<a id="modify-object-metadata-code-example"></a>
#### Code Examples
<details>
<summary>cURL</summary>

```
# Add metadata to an object
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo" \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/ba6610.jpg

# Verify the added metadata in the object header
$ curl -I -H "X-Auth-Token: b587ae461278419da6ecd21a2344c8aa" \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/ba6610.jpg
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

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add(metaKey, value);    // Set metadata in headers

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
    $req_header[] = 'X-Object-Meta-'.$key.': '.$value;  // Add metadata to header

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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$META_KEY = 'Type';
$META_VALUE = 'photo';
$object->set_metadata($CONTAINER_NAME, $OBJECT_NAME, $META_KEY, $META_VALUE);
?>
```
</details>

<br>

<a id="delete-an-object"></a>
### Delete an Object { #delete-an-object }
Deletes the specified object.

!!! tip "Note"
    When deleting a multipart-uploaded object, you need to delete all segment data. If you delete only the manifest object, the segment objects might be kept intact and you might be charged for them.

```
DELETE /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

<a id="delete-an-object-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Token ID |
| Account | URL | String | Y | Storage account, which can be found in the API Endpoint Settings popup |
| Container | URL | String | Y | Container name |
| Object | URL | String | Y | Name of the object to delete |

<a id="delete-an-object-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 204.

<br>

<a id="delete-an-object-code-example"></a>
#### Code Example
<details>
<summary>cURL</summary>

```
$ curl -X DELETE -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/ba6610.jpg
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

        // Create headers
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // Call API
        this.restTemplate.exchange(url, HttpMethod.DELETE, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067'
    TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa'
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
$STORAGE_URL = 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067';
$TOKEN_ID = 'b587ae461278419da6ecd21a2344c8aa';
$CONTAINER_NAME = 'test';
$OBJECT_NAME = '0428b9e3e419d4fb7aedffde984ba5b3.jpg';

$object = new ObjectService($STORAGE_URL, $TOKEN_ID);

$object->delete($CONTAINER_NAME, $OBJECT_NAME);
?>
```
</details>

<br>

<a id="limiting-policy"></a>
## Limiting Policy { #limiting-policy }

<a id="request-rate-limit"></a>
### Request Rate Limit { #request-rate-limit }
Object Storage applies a write request rate limit per storage account to ensure system stability.

| Category | Item | Description |
|---|---|---|
| Limit condition | Request rate limit | 500 requests/second |
| Applies to | Unit | Per storage account |
| Applies to | Applied methods | `POST`: Change container settings, modify object properties/metadata<br>`PUT`: Create container, upload object<br>`DELETE`: Delete container/object<br>`COPY`: Copy object |
| Behavior | When limit is exceeded | Requests are delayed; if the delay exceeds 60 seconds, a 429 response is returned |

The following policy applies to write requests that exceed the rate limit:

* Write requests that exceed the limit are not immediately rejected but are delayed.
* The delay increases progressively depending on the volume of excess requests, and can be extended up to 60 seconds.
* If the delay exceeds 60 seconds, the request fails and a `429 Too Many Requests` response is returned.

To prevent response delays or failures, adjust your write requests so that they do not exceed the rate limit.

<br>

<a id="references"></a>
## References { #references }

Swift API v1 - [https://docs.openstack.org/api-ref/object-store/](https://docs.openstack.org/api-ref/object-store/)