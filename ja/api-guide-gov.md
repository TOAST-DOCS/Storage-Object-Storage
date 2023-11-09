## Storage > Object Storage > APIガイド

## 事前準備

オブジェクトストレージAPIを使用するには、先に認証トークン(token)を発行する必要があります。認証トークンはオブジェクトストレージのREST APIを使用する時に必要な認証キーです。外部公開に設定していないコンテナやオブジェクトへアクセスするにはトークンが必要です。トークンはアカウントごとに管理されます。

### テナントID(Tenant ID)およびAPIエンドポイント(Endpoint)確認

トークンを発行するためのテナントIDとAPIのエンドポイントは、オブジェクトストレージサービスページの**API Endpoint設定**ボタンをクリックして確認できます。

| 項目 | APIエンドポイント | 用途 |
|---|---|---|
| Identity | https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0 | 認証トークン発行 |
| Object-Store | https://kr1-api-object-storage.gov-nhncloudservice.com/v1/{Account} | オブジェクトストレージ制御、リージョンによって異なる |
| Tenant ID | 数字 + 英字で構成された32文字の文字列 | 認証トークン発行 |

### APIパスワード設定

APIパスワードはオブジェクトストレージサービスページの**API Endpoint設定**ボタンをクリックして設定できます。

1. **API Endpoint設定**ボタンをクリックします。
2. **API Endpoint設定**下の**APIパスワード設定**入力ボックスに、トークン発行時に使用するパスワードを入力します。
3. **保存**ボタンをクリックします。

## 認証トークン発行

```
POST    https://api-identity-infrastructure.gov-nhncloudservice.com/v2.0/tokens
Content-Type: application/json
```

### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | Body | String | O | Tenant ID。API Endpoint設定ダイアログボックスで確認可能 |
| username | Body | String | O | NHN CloudアカウントID(メール)入力 |
| password | Body | String | O | API Endpoint設定ダイアログボックスで保存したパスワード |

<details>
<summary>例</summary>

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

### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| access.token.id | Body | String |	発行されたトークンID |
| access.token.tenant.id | Body | String | トークンをリクエストしたプロジェクトに対応するTenant ID |
| access.token.expires | Body | String | 発行したトークンの満了時間 <br/>YYYY-MM-DDThh:mm:ssZの形式。例) 2017-05-16T03:17:50Z |

> [注意]
> トークンには有効期限があります。トークン発行リクエストのレスポンスに含まれた「expires」項目は、発行されたトークンの有効期限が終了する時間です。トークンが無効になった場合、新しいトークンを発行する必要があります。

<details>
<summary>例</summary>

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

### サンプルコード
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

        // リクエスト本文の作成
        this.tokenRequest = new TokenRequest();
        this.tokenRequest.getAuth().setTenantId(tenantId);
        this.tokenRequest.getAuth().getPasswordCredentials().setUsername(username);
        this.tokenRequest.getAuth().getPasswordCredentials().setPassword(password);

        this.restTemplate = new RestTemplate();
    }

    public String requestToken() {
        String identityUrl = this.authUrl + "/tokens";

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/json");

        HttpEntity<TokenRequest> httpEntity
            = new HttpEntity<TokenRequest>(this.tokenRequest, headers);

        // トークンリクエスト
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
  );  // リクエスト本文の作成
  $req_header = array(
    'Content-Type: application/json'
  );  // リクエストヘッダの作成

  $curl  = curl_init($url); // curl初期化
  curl_setopt_array($curl, array(
    CURLOPT_POST => TRUE,
    CURLOPT_RETURNTRANSFER => TRUE,
    CURLOPT_HTTPHEADER => $req_header,
    CURLOPT_POSTFIELDS => json_encode($req_body)
  )); // パラメータ設定
  $response = curl_exec($curl); // API呼び出し
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

## ストレージアカウント
ストレージアカウント(account)は`AUTH_*****`形式の文字列です。Object-Store APIエンドポイントに含まれています。

### ストレージアカウント照会
ストレージアカウントの使用状況を照会します。

```
HEAD  /v1/{Account}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ストレージアカウント。**API Endpoint設定**ダイアログボックスで確認 |

#### レスポンス
レスポンス本文を返しません。使用状況はヘッダに含まれています。リクエストが正しければステータスコード200を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| X-Account-Container-Count | Header | String | コンテナ数 |
| X-Account-Object-Count | Header | String | 保存されたオブジェクト数 |
| X-Account-Bytes-Used | Header | String | 保存されたデータ容量(バイト) |

#### コード例

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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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

### コンテナリスト照会
ストレージアカウントのコンテナリストを照会します。

```
GET  /v1/{Account}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ストレージアカウント。**API Endpoint設定**ダイアログボックスで確認 |

#### レスポンス
```
[ストレージアカウントに属すコンテナリスト]
```

#### コード例

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
        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
        ResponseEntity<String> response
            = this.restTemplate.exchange(this.getStorageUrl(), HttpMethod.GET, requestHttpEntity, String.class);

        List<String> containerList = null;
        if (response.getStatusCode() == HttpStatus.OK) {
            // Stringで受け取ったリストを配列に変換
            containerList = Arrays.asList(response.getBody().split("\\r?\\n"));
        }

        // 配列をListに変換して返す
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

## コンテナ

### コンテナの作成
コンテナを作成します。オブジェクトストレージにファイルをアップロードするには、コンテナを作成する必要があります。

> [参考]
> コンテナまたはオブジェクト名に特殊文字`! * ' ( ) ; : @ & = + $ , / ? # [ ]`が含まれている場合、APIを使用する時にURLエンコード(パーセントエンコード)を行う必要があります。これらの文字はURLで使用される予約文字です。この文字が含まれたパスをURLエンコードしないでAPIリクエストを送る場合、期待するレスポンスを得られません。

```
PUT  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 作成するコンテナ名 |

#### レスポンス
レスポンス本文を返しません。コンテナが作成されるとステータスコード201を返します。

#### サンプルコード
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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


### コンテナ照会
指定したコンテナの情報と、内部に保存されたオブジェクトのリストを照会します。

```
GET   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 照会するコンテナ名 |

#### レスポンス

```
[指定したコンテナに属しているオブジェクトリスト]
```

#### サンプルコード
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
        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
        ResponseEntity<String> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, String.class);

        if (response.getStatusCode() == HttpStatus.OK) {
            // Stringで受け取ったリストを配列に変換
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

### コンテナ照会クエリー
コンテナ照会APIは、次のように複数のクエリー(query)を提供します。すべてのクエリーは`&`で接続して混用できます。

#### 1万個以上のオブジェクトリスト照会
コンテナ照会APIで照会できるリストのオブジェクト数は1万個に制限されています。1万個以上のオブジェクトリストを照会するには`marker`クエリーを利用する必要があります。 Markerクエリーは指定したオブジェクトの次のオブジェクトから最大1万個のリストを返します。

```
GET    /v1/{Account}/{Container}?marker={Object}
X-Auth-Token: {token-id}
```

##### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 照会するコンテナ名 |
| Object | Query | String | O | 基準オブジェクト名 |

##### レスポンス
```
[指定したコンテナに属している指定したオブジェクトの次のオブジェクトリスト]
```

##### サンプルコード
<details>
<summary>cURL</summary>

```
// `20d33f.jpg`以降のオブジェクトリスト照会
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example?maker=20d33f.jpg
[指定したオブジェクト(20d33f.jpg)以降のリスト]
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
        // 指定したオブジェクト名を利用してクエリーURLを作成
        String url = this.getUrl(conatinerName) + "?marker=" + prevLastObject;
        // コンテナ照会例のgetList()メソッド呼び出し
        return this.getList(url);
    }

    public List<String> getObjectList(String conatinerName) {
        final int LIMIT_COUNT = 10000;

        String url = this.getUrl(conatinerName);

        // オブジェクトリスト照会
        List<String> objectList = this.getList(url);
        while ((objectList.size() % LIMIT_COUNT) == 0) {
            // オブジェクトリストの長さが1万の倍数なら、リストの最後のオブジェクトを指定して以降のリストを照会
            String lastObject = objectList.get(objectList.size() - 1);
            List<String> nextObjList = this.getObjectList(conatinerName, lastObject);
            objectList.addAll(nextObjList);
        }

        return objectList;
    }

    // getObjectList()使用例はコンテナ照会と同じ
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

##### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 照会するコンテナ名 |

##### レスポンス
```
[指定したコンテナに属している指定したフォルダのオブジェクトリスト]
```

##### サンプルコード
<details>
<summary>cURL</summary>

```
// オブジェクトリスト照会
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
        // URLを作成
        String url = this.getUrl(conatinerName);
        // コンテナ照会例のgetList()メソッド呼び出し
        return this.getList(url);
    }

    // getObjectList()使用例はコンテナ照会と同じ
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
  function get_object_list($container) {
    $req_url = $this->get_url($container);
    return $this->get_list($req_url);
  }
}
?>
```

</details>

#### プレフィックスで始まるオブジェクトリスト照会
`prefix`クエリーを使用すると、指定したプレフィックスで始まるオブジェクトのリストを返します。

```
GET   /v1/{Account}/{Container}?prefix={Prefix}
X-Auth-Token: {token-id}
```

##### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 照会するコンテナ名 |
| Prefix | Query | String | O | 検索するプレフィックス |

##### レスポンス
```
[指定したコンテナに属し、指定したプレフィックスで始まるオブジェクトリスト]
```

##### サンプルコード
<details>
<summary>cURL</summary>

```
// 314で始まるオブジェクトリスト照会
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
        // 指定したプレフィックスを利用してクエリーURLを作成
        String url = this.getUrl(conatinerName) + "?prefix=" + prefix;
        // コンテナ照会例のgetList()メソッド呼び出し
        return this.getList(url);
    }

    // getObjectListWithPrefix()使用例はコンテナ照会例と同じ
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

#### リストの最大オブジェクト数を指定
`limit`クエリーを使用すると、返すオブジェクトリストの最大オブジェクト数を指定できます。

```
GET   /v1/{Account}/{Container}?limit={limit}
X-Auth-Token: {token-id}
```

##### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 照会するコンテナ名 |
| limit | Query | Integer | O | リストに表示するオブジェクト数 |

##### レスポンス
```
[指定したコンテナに属している指定された数のオブジェクトリスト]
```

##### サンプルコード

<details>
<summary>cURL</summary>

```curl
// 10個のオブジェクトのみ照会
$ curl -X GET -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example?limit=10
...{9個のオブジェクト}...
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
        // 指定した最大オブジェクト数を利用してクエリーURLを作成
        String url = this.getUrl(conatinerName) + "?limit=" + limit;
        // コンテナ照会例のgetList()メソッド呼び出し
        return this.getList(url);
    }

    // getObjectListWithPrefix()使用例はコンテナ照会例と同じ
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
  }}
}
?>
```

</details>


### コンテナ修正

コンテナのメタデータを変更してアクセスルールを指定できます。

```
POST  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {コンテナ読み取りポリシー}
X-Container-Write: {コンテナ書き込みポリシー}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| X-Container-Read | Header | String | - | コンテナの読み取りに対するアクセスルール指定<br/>.r:* - すべてのユーザーにアクセス許可<br/>.r:example.com,test.com –特定アドレスにのみアクセス許可、 ‘,’で区切る<br/>.rlistings. –コンテナリスト照会許可<br/>AUTH_.... –特定アカウントにのみアクセス許可 |
| X-Container-Write | Header | String | - | コンテナの書き込みに対するアクセスルール指定<br/>\*:\* - すべてのユーザーに書き込み許可<br/>AUTH_.... –特定アカウントにのみ書き込み許可 |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 修正するコンテナ名 |

#### レスポンス
レスポンス本文を返しません。リクエストが正しければステータスコード204を返します。

#### サンプルコード
<details>
<summary>cURL</summary>

```
// すべてのユーザーに読み取り/書き込み許可
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Container-Read", permission);    // ヘッダに権限を追加

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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
    $req_header[] = 'X-Container-Read: ' . $permission;  // ヘッダに権限を追加

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

#### ACL確認
読み取り権限を公開に設定した後は、`curl`、`wget`などのツールを使用したり、ブラウザからトークンなしで照会されるかを確認できます。

<details>
<summary>例</summary>

```
$ curl https://kr1-api-object-storage.gov-nhncloudservice.com/v1/{Account}/{Container}/{Object}

{オブジェクトの内容}
```

</details>

### コンテナ削除

指定したコンテナを削除します。削除するコンテナは空になっている必要があります。

```
DELETE   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL| String |	O | 削除するコンテナ名 |

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード204を返します。

#### サンプルコード
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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

## オブジェクト

### オブジェクトアップロード
指定したコンテナに新しいオブジェクトをアップロードします。

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Content-type | Header | String | O | オブジェクトのコンテンツタイプ |
| X-Delete-At | Header | Timestamp | - | オブジェクトを削除するUNIX時間(秒) |
| X-Delete-After | Header | Timestamp | - | オブジェクト有効時間、 UNIX時間(秒) |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object | URL | String |	O | 作成するオブジェクト名 |
| - |	Body | Binary | O | 作成するオブジェクトの内容 |

> [注意]
> オブジェクト名が`./`または`../`で始まる場合、ブラウザがこれをパス文字と認識し、Webコンソールからアクセスできません。
> APIを利用してこのような名前のオブジェクトをアップロードした場合、APIでアクセスする必要があります。

#### レスポンス
レスポンス本文を返しません。リクエストが正しくない場合はステータスコード201を返します。

#### サンプルコード
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

        // InputStreamをリクエスト本文に追加できるようにRequestCallbackオーバーライド
        final RequestCallback requestCallback = new RequestCallback() {
            public void doWithRequest(final ClientHttpRequest request) throws IOException {
                request.getHeaders().add("X-Auth-Token", tokenId);
                IOUtils.copy(inputStream, request.getBody());
            }
        };

        // オーバーライドしたRequestCallbackを使用できるように設定
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setBufferRequestBody(false);
        RestTemplate restTemplate = new RestTemplate(requestFactory);

        HttpMessageConverterExtractor<String> responseExtractor
            = new HttpMessageConverterExtractor<String>(String.class, restTemplate.getMessageConverters());

        // API呼び出し
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
            // ファイルからInputStreamを作成
            File objFile = new File(objectPath + "/" + objectName);
            InputStream inputStream = new FileInputStream(objFile);

            // アップロード
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

    $fd = fopen($filename, 'r');  // ファイルを開く。

    $curl  = curl_init($req_url);
    curl_setopt_array($curl, array(
      CURLOPT_PUT => TRUE,
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_INFILE => $fd,  // ファイルストリームを引数に入れる。
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

### マルチパートアップロード
容量が5GBを超えるオブジェクトは、5GB以下のセグメントに分割してアップロードする必要があります。

#### セグメントアップロード

```
PUT   /v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

##### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Content-type | Header | String | O | オブジェクトのコンテンツタイプ |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object |	URL | String | O | 作成するオブジェクト名 |
| Count | URL | Integer | O | 分割したオブジェクトの順番、例) 001, 002 |
| - |	Body | Binary | O | 分割したオブジェクトの内容 |

##### レスポンス
レスポンス本文を返しません。リクエストが正しくない場合はステータスコード201を返します。

#### マニフェスト作成
すべてのオブジェクトのセグメントをアップロードしてマニフェストオブジェクトを作成すると、1つのオブジェクトのように使用できます。マニフェストオブジェクトはセグメントが保存されたパスを指し示します。

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Container}/{Object}/
```

##### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | トークンID |
| X-Object-Manifest | Header| String | O | 分割したオブジェクトをアップロードしたパス、 `{Container}/{Object}/` |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object |	URL | String | O | 作成するマニフェストオブジェクト名 |
| - | Body| Binary | O | 空のデータ |

##### レスポンス
レスポンス本文を返しません。リクエストが正しくない場合はステータスコード201を返します。


#### サンプルコード
<details>
<summary>cURL</summary>

```
// 200MB単位でファイル分割
$ split -d -b 209715200 large_obj.img large_obj.img.

// 分割されたオブジェクトのアップロード
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/003 \
-T large_obj.img.02

// マニフェストオブジェクトのアップロード
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

    // マニフェストオブジェクトのアップロード
    public void uploadManifestObject(String containerName, String objectName) {
        String url = this.getUrl(containerName, objectName);
        String manifestName = containerName + "/" + objectName + "/"; // マニフェスト名の作成

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Object-Manifest", manifestName);  // ヘッダにマニフェストを表記

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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

        final int defaultChunkSize = 100 * 1024; // 100 KB単位で分割
        int chunkSize = defaultChunkSize;
        int chunkNo = 0;  // 分割オブジェクトの名前を作成するためのチャンク番号
        int totalBytesRead = 0;

        try {
            // ファイルからInputStreamを作成
            InputStream inputStream = new BufferedInputStream(new FileInputStream(objFile));
            while(totalBytesRead < fileSize) {

                // 残りデータサイズを計算
                int remainedBytes = fileSize - totalBytesRead;
                if(remainedBytes < chunkSize) {
                    chunkSize = remainedBytes;
                }

                // ByteBufferにチャンクサイズ分のデータを読み込む
                byte[] chunkBuffer = new byte[chunkSize];
                int bytesRead = inputStream.read(chunkBuffer, 0, chunkSize);

                if(bytesRead > 0) {
                    // バッファのデータをInputStreamで作ってアップロード。オブジェクトアップロード例のuploadObject()メソッドを使用
                    String objPartName = String.format("%s/%03d", objectName, ++chunkNo);
                    InputStream chunkInputStream = new ByteArrayInputStream(chunkBuffer);
                    objectService.uploadObject(containerName, objPartName, chunkInputStream);

                    totalBytesRead += bytesRead;
                }
            }

            // マニフェストファイルをアップロード
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
    $fd = fopen($filename, 'r');  // ファイルを開く
    $obj_size = filesize($filename);

    while($total_bytes_read < $obj_size) {
      // 分割する容量を計算
      $remained_bytes = $obj_size - $total_bytes_read;
      if ($remained_bytes < $chunk_size) {
        $chunk_size = $remained_bytes;
      }
      $chunk = fread($fd, $chunk_size);
      // パート名を作成
      $temp_file = sprintf("./multipart-%03d", $chunk_index);
      $req_url = sprintf("%s/%03d", $url, $chunk_index);

      // パート一時ファイルを作成
      $part_fd = fopen($temp_file, 'w+');
      fwrite($part_fd, $chunk);
      fseek($part_fd, 0);

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
        CURLOPT_PUT => TRUE,
        CURLOPT_HEADER => TRUE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_INFILE => $part_fd,  // パートファイルストリームをパラメータに入力
        CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
      printf("$response");

      // 一時ファイルを削除
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

### オブジェクト内容の修正
オブジェクトアップロードAPIと同じですが、オブジェクトがすでにコンテナにある場合、該当オブジェクトの内容が修正されます。

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Content-type | Header | String | O | オブジェクトのコンテンツタイプ |
| X-Delete-At | Header | Timestamp | - | オブジェクトを削除するUNIX時間(秒) |
| X-Delete-After | Header | Timestamp | - | オブジェクト有効時間、 UNIX時間(秒) |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object | URL | String | O | 内容を修正するオブジェクト名 |

#### レスポンス
レスポンス本文を返しません。リクエストが正しくない場合はステータスコード201を返します。

### オブジェクトのダウンロード
オブジェクトをダウンロードします。

```
GET   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object | URL | String | O | ダウンロードするオブジェクト名 |

#### レスポンス
オブジェクトの内容がストリームで返されます。リクエストが正しければステータスコード200を返します。

#### サンプルコード
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.setAccept(Arrays.asList(MediaType.APPLICATION_OCTET_STREAM));

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し、データをバイト配列で受け取る
        ResponseEntity<byte[]> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, byte[].class);

        // バイト配列データをInputStreamで作って返す
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
            // オブジェクトのダウンロード
            InputStream inputStream = objectService.downloadObject(containerName, objectName);

            // ダウンロードしたデータをByteBufferに読み込む
            byte[] buffer = new byte[inputStream.available()];
            inputStream.read(buffer);

            // バッファのデータをファイルに記録
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

### オブジェクトのコピー
オブジェクトを他のコンテナへコピーします。

```
COPY   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Destination：{オブジェクトをコピーする対象}
```

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Copy-From：{原本オブジェクト}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Destination | Header | String |	- | オブジェクトをコピーする対象、 `{コンテナ} / {オブジェクト}`<br/>COPYメソッドを使用する時に必要 |
| X-Copy-From | Header | String |	- | 原本オブジェクト、 `{コンテナ} / {オブジェクト}`<br/>PUTメソッドを使用する時に必要 |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O |	コンテナ名<br/>COPYメソッド：原本コンテナ<br/>PUTメソッド：コピーするコンテナ |
| Object | URL | String |	コピーするオブジェクト名 |

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード201を返します。

#### サンプルコード
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Copy-From", srcObject);    // 原本オブジェクトを指定
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // HttpMethodはCOPYメソッドをサポートしていないため、PUTメソッドを使用する代替APIを呼び出します。
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

### オブジェクトのメタデータを修正
指定したオブジェクトのメタデータを修正します。

```
POST   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Meta-{Key}: {Value}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| X-Object-Meta-{Key} | Header | String | - | 変更するメタデータ |
| X-Delete-At | Header | Timestamp | - | オブジェクトを削除するUNIX時間(秒) |
| X-Delete-After | Header | Timestamp | - | オブジェクト有効時間、 UNIX時間(秒) |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL| String |	 O | コンテナ名 |
| Object | URL| String |  O | メタデータを修正するオブジェクト名 |

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード202を返します。

#### サンプルコード
<details>
<summary>cURL</summary>

```
// オブジェクトにメタデータを追加
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo" \
https://kr1-api-object-storage.gov-nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg

// オブジェクトヘッダで追加したメタデータの確認
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

        // メタデータキーの作成
        String metaKey = "X-Object-Meta-" + key;

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add(metaKey, value);    // ヘッダにメタデータを設定

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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
    $req_header[] = 'X-Object-Meta-'.$key.': '.$value;  // ヘッダにメタデータを追加

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

### オブジェクトの削除
指定したオブジェクトを削除します。

```
DELETE   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| X-Object-Meta-{Key} | Header | String | - | 変更するメタデータ |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL| String |	 O | コンテナ名 |
| Object | URL| String |  O | メタデータを修正するオブジェクト名 |

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ユーザーアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL| String |	 O | コンテナ名 |
| Object | URL| String |  O | 削除するオブジェクト名 |

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード204を返します。

#### サンプルコード
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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
