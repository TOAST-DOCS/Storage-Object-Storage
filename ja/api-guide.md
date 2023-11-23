## Storage > Object Storage > APIガイド

## 事前準備

オブジェクトストレージAPIを使用するには、先に認証トークン(token)を発行する必要があります。認証トークンはオブジェクトストレージのREST APIを使用する時に必要な認証キーです。外部公開に設定していないコンテナやオブジェクトへアクセスするにはトークンが必要です。トークンはNHN Cloudアカウントごとに管理されます。

<br/>

### テナントID(Tenant ID)およびAPIエンドポイント(Endpoint)確認

トークンを発行するためのテナントIDとAPIのエンドポイントは、オブジェクトストレージサービスページの**API Endpoint設定**ボタンをクリックして確認できます。

| 項目 | APIエンドポイント | 用途 |
|---|---|---|
| Identity | https://api-identity-infrastructure.nhncloudservice.com/v2.0 | 認証トークン発行 |
| Object-Store | https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_***** | オブジェクトストレージ制御、リージョンによって異なる |
| Tenant ID | 数字 + 英字で構成された32文字の文字列 | 認証トークン発行 |

<br/>

### APIパスワード設定

APIパスワードはオブジェクトストレージサービスページの**API Endpoint設定**ボタンをクリックして設定できます。

1. **API Endpoint設定**ボタンをクリックします。
2. **API Endpoint設定**下の**APIパスワード設定**入力ボックスに、トークン発行時に使用するパスワードを入力します。
3. **保存**ボタンをクリックします。

> [参考]
> APIパスワードはアカウントごとに設定されます。あるプロジェクトで設定されたパスワードはユーザーが属すすべてのプロジェクトで使用できます。

<br/>

## 認証トークン発行

```
POST    https://api-identity-infrastructure.nhncloudservice.com/v2.0/tokens
Content-Type: application/json
```

<p style='padding-top: 10px; font-size: 15px;'><b>リクエスト</b></p>

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| tenantId | Body | String | O | テナントID。API Endpoint設定ダイアログボックスで確認可能 |
| username | Body | String | O | NHN Cloud会員ID(メール形式)、IAMメンバーID |
| password | Body | String | O | API Endpoint設定ダイアログボックスで保存したパスワード |

<details>
<summary>例</summary>

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

<p style='padding-top: 10px; font-size: 15px;'><b>レスポンス</b></p>

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| access.token.id | Body | String |	発行されたトークンID |
| access.token.tenant.id | Body | String | トークンをリクエストしたプロジェクトに対応するテナントID |
| access.token.expires | Body | String | 発行したトークンの満了時間 <br/>YYYY-MM-DDThh:mm:ssZの形式。例) 2017-05-16T03:17:50Z |
| access.user.id | Body | String | 32個の16進数で構成されたAPIユーザーID<br/>S3互換APIを使用するためのEC2資格証明を発行したり、アクセスポリシーを設定するのに使用 |

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

<p style='padding-top: 10px; font-size: 15px;'><b>サンプルコード</b></p>

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

## コンテナ

### コンテナの作成
コンテナを作成します。オブジェクトストレージにファイルをアップロードするには、コンテナを作成する必要があります。
コンテナを作成する時、`X-Container-Worm-Retention-Day`ヘッダを利用してオブジェクトロック周期を設定すると、オブジェクトロックコンテナを作成できます。オブジェクトロックコンテナにアップロードしたオブジェクトは、**WORM (Write-Once-Read-Many)**モデルを使用して保存されます。オブジェクトロックコンテナにアップロードしたオブジェクトにはロック有効期限が設定されます。各オブジェクトに設定されたロック有効期限以前にはオブジェクトの上書きや削除ができません。

> [注意]
> コンテナ名に特殊文字``' " ` < > ;``と　、相対パス文字(`. ..`)は使用できません。

<!-- 改行用。 削除しないでください。 -->

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
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 作成するコンテナ名 |
| X-Container-Worm-Retention-Day | Header | Integer | - | コンテナの基本オブジェクトロック周期を日単位で設定 |

#### レスポンス
レスポンス本文を返しません。リクエストが正しくない場合はステータスコード201を返します。

#### サンプルコード
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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

### コンテナ照会
指定したコンテナの情報と、内部に保存されたオブジェクトのリストを照会します。コンテナの情報はレスポンスヘッダで確認できます。

```
GET   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 照会するコンテナ名 |
| marker | Query | String | - | 基準オブジェクト名 |
| prefix | Query | String | - | 検索するプレフィックス |
| limit | Query | Integer | - | リストに表示するオブジェクト数 |
| format | Query | String | - | レスポンス形式、 jsonまたはxml |

> [参考]
> コンテナ照会APIは複数のクエリ(query)を提供します。すべてのクエリは`&`で接続して使用できます。

#### 1万個以上のオブジェクトリスト照会
コンテナ照会APIで照会できるリストのオブジェクト数は1万個に制限されています。1万個以上のオブジェクトリストを照会するには`marker`クエリを利用する必要があります。Markerクエリは指定したオブジェクトの次のオブジェクトから最大1万個のリストを返します。

<br/>

#### プレフィックスで始まるオブジェクトリスト照会
`prefix`クエリを使用すると、指定したプレフィックスで始まるオブジェクトのリストを返します。Prefixクエリでサブフォルダのオブジェクトリストを照会できます。

<br/>

#### リストの最大オブジェクト数指定
`limit`クエリを使用する際に返すオブジェクトリストの最大オブジェクト数を指定できます。

<br/>

#### レスポンス形式指定
`format`クエリを使用して`json`または`xml`のレスポンス形式を指定できます。レスポンス形式を指定するとレスポンス本文に各オブジェクトのメタデータ(サイズ、コンテンツタイプ、最終修正時刻、ETag)が含まれます。

<br/>

#### レスポンス

```
[コンテナのオブジェクトリスト]
```

#### サンプルコード
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

### コンテナ設定変更

コンテナ設定を変更します。コンテナ設定はコンテナ照会時にレスポンスヘッダで確認できます。

```
POST  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {コンテナ読み取りに対するロールベースのアクセスルール}
X-Container-Write: {コンテナ書き込みに対するロールベースのアクセスルール}
X-Container-Ip-Acl-Allowed-List: {コンテナ書き込みに対するIPベースのアクセスルール}
X-Container-Ip-Acl-Denied-List: {コンテナ書き込みに対するIPベースのアクセスルール}
X-Container-Object-Lifecycle: {コンテナのオブジェクトのライフサイクル}
X-History-Location: {オブジェクトの以前バージョンを保存するコンテナ}
X-Versions-Retention: {オブジェクトの以前のバージョンのライフサイクル}
X-Container-Meta-Web-Index: {静的Webサイトインデックス文書オブジェクト}
X-Container-Meta-Web-Error: {静的Webサイトエラー文書オブジェクトのサフィックス}
X-Container-Meta-Access-Control-Allow-Origin: {オリジン間リソース共有許可リスト}
X-Container-Rfc-Compliant-Etags: {RFCを遵守するETag形式を使用するかどうか}
X-Container-Worm-Retention-Day: {コンテナのオブジェクトロック周期}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| X-Container-Read | Header | String | - | コンテナ読み取りに対するロールベースのアクセスルール設定 |
| X-Container-Write | Header | String | - | コンテナ書き込みに対するロールベースのアクセスルール設定 |
| X-Container-Ip-Acl-Allowed-List | Header | String | - | コンテナ書き込みに対するIPベースのアクセスルール設定 |
| X-Container-Ip-Acl-Denied-List | Header | String | - | コンテナ書き込みに対するIPベースのアクセスルール設定 |
| X-Container-Object-Lifecycle | Header | Integer | - | コンテナの基本オブジェクトライフサイクルを日単位で設定 |
| X-History-Location | Header | String | - | オブジェクトの以前バージョンを保管するコンテナを設定 |
| X-Versions-Retention | Header | Integer | - | オブジェクトの以前のバージョンのライフサイクルを日単位で設定 |
| X-Container-Meta-Web-Index | Header | String | - | 静的Webサイトインデックス文書オブジェクト設定<br/>英数字、一部の特殊文字(`-`, `_`, `.`, `/`)のみ許可 |
| X-Container-Meta-Web-Error | Header | String | - | 静的Webサイトエラー文書オブジェクトサフィックス設定<br/>英数字、一部の特殊文字(`-`, `_`, `.`, `/`)のみ許可 |
| X-Container-Meta-Access-Control-Allow-Origin | Header | String | - | CORS許可ホストリスト。 '*'ですべてのホストを許可するか、スペースで区切られたホストリストを入力できます。 | 
| X-Container-Rfc-Compliant-Etags | Header | String | - | RFCを遵守するETag形式を使用するかどうかの設定。trueまたはfalse |
| X-Container-Worm-Retention-Day | Header | Integer | - | コンテナの基本オブジェクトロック周期を日単位で設定<br/>オブジェクトロックコンテナでのみ変更可能 |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O | 修正するコンテナ名 |
<br/>

##### アクセスポリシー設定
`X-Container-Read`, `X-Container-Write`, `X-Container-Ip-Acl-Allowed-List`, `X-Container-Ip-Acl-Denied-List`ヘッダを使用してコンテナアクセスポリシーを設定できます。詳細な内容は[アクセスポリシー設定ガイド](acl-guide/)を参照してください。

<br/>

##### オブジェクトライフサイクル設定
`X-Container-Object-Lifecycle`ヘッダを使用するとコンテナに保存されるオブジェクトのライフサイクルを日単位で設定できます。設定後にアップロードしたオブジェクトにのみ適用されます。
<br/>

##### バージョン管理ポリシー設定
[オブジェクト内容の修正](api-guide/#_51)項目に記述したとおりにオブジェクトをアップロードする時、同じ名前のオブジェクトがすでにある場合、オブジェクトをアップデートします。既存オブジェクトの内容を保管したい場合は`X-History-Location`ヘッダを使用して以前のバージョンを保管する**アーカイブコンテナ**を指定できます。

以前のバージョンのオブジェクトは、アーカイブコンテナに次のような形式で保管されます。
```
{3桁16進数で表現されたオブジェクト名の長さ}{オブジェクト名}/{以前のバージョンのオブジェクトが保管されたUNIX時間}
```
例えば`picture.jpg`というオブジェクトをアップデートすると、アーカイブコンテナには`00bpicture.jpg/1610606551.82539`というオブジェクトが作成されます。

バージョン管理ポリシーが設定されたコンテナでオブジェクトを削除すると、アーカイブコンテナに削除されたオブジェクトが保管され、削除マーカーオブジェクトが作成されます。アーカイブコンテナに保管された以前のバージョンのオブジェクトにはいつでもアクセスできます。

`X-Versions-Retention`ヘッダを一緒に使用すると、以前のバージョンのオブジェクトのライフサイクルを日単位で設定できます。1に設定すると保管されたオブジェクトは1日後に自動的に削除されます。設定しない場合、以前のバージョンのオブジェクトはユーザーが削除するまで保管されます。設定後に保管された以前のバージョンのオブジェクトにのみ適用されます。

> [注意]
バージョン管理ポリシーを設定した場合、アーカイブコンテナを原本コンテナより先に削除してはいけません。原本コンテナのオブジェクトをアップデートまたは削除する時、アーカイブコンテナに以前のバージョンを保存することができず、エラーが発生します。アーカイブコンテナを先に削除してエラーが発生した場合、アーカイブコンテナを新たに作成するか、原本コンテナのバージョン管理ポリシーを解除して問題を解決できます。
>
> アーカイブコンテナに使用するコンテナ名には、なるべくUnicode文字を使用しないことを推奨します。アーカイブコンテナに指定するコンテナ名にUnicode文字が含まれている場合、必ずURLエンコード後にリクエストヘッダに入力する必要があります。
>
> 暗号化コンテナをアーカイブコンテナとして指定した後、暗号化コンテナの対称鍵をSecure Key Managerサービスから削除すると、原本コンテナのオブジェクトアップロードと削除に失敗します。

<br/>

##### 静的Webサイト設定
コンテナ読み取り権限をすべてのユーザーに許可した後、`X-Container-Meta-Web-Index`, `X-Container-Meta-Web-Error`ヘッダを利用して静的Webサイトインデックス文書とエラー文書を設定すると、コンテナURLを利用して静的Webサイトをホスティングできます。

静的Webサイトのインデックス文書、エラー文書に使用するオブジェクトは、1つ以上の英字、数字または一部特殊文字(`-`, `_`, `.`, `/`)で構成された名前でなければならず、拡張子が`html`のハイパーテキスト形式でなければいけません。この条件を満たさない場合は、設定できなかったり、静的Webサイトが動作しない可能性があります。
静的Webサイトのエラー文書名は`{エラーコード}{サフィックス}`形式で、ヘッダには`サフィックス`を入力する必要があります。例えば`X-Container-Meta-Web-Error: error.html`をリクエストした場合、404エラーが発生した時に表示されるエラー文書の名前は`404error.html`になります。各エラーの状況に合わせてエラー文書をアップロードして使用できます。エラー文書を定義していなかったり、エラーコードに合ったエラー文書オブジェクトがない場合はWebブラウザの基本エラー文書が表示されます。
<br/>

##### オリジン間リソース共有(CORS)

ブラウザでObject Storage APIを直接呼び出すには、クロスソースリソース共有(CORS)設定が必要です。 `X-Container-Meta-Access-Control-Allow-Origin`ヘッダを利用して許可するソースリストを設定します。スペース(` `)で区切られた1つ以上のソースを入力するか、`*`を入力してすべてのソースを許可できます。


<details>
<summary>CORS設定確認例</summary>

コンテナにCORS設定を追加します。

```
$ curl -X POST \
-H 'X-Auth-Token: ****' \
-H 'X-Container-Meta-Access-Control-Allow-Origin: https://example.com' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```
<br>
ブラウザでCORSを許可したサイトに移動し、以下のスクリプトを実行します。スクリプトはブラウザが提供する開発者ツールのコンソールで実行できます。

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
CORS設定に問題がなければ、コンソールで次のような成功レスポンスを確認できます。

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
CORS設定を行っていない場合や、許可されていないサイトでAPIを呼び出すと、次のようなエラーレスポンスを受け取ります。

```
Access to XMLHttpRequest at 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_****/container/object' from origin 'https://example.com' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
Status: 0
```

</details>


<br/>

##### RFCを遵守するETag形式の使用設定
一部アプリケーションでは[RFC7232](https://www.rfc-editor.org/rfc/rfc7232#section-2.3)仕様に基づいて二重引用符で囲まれたETag値を要求します。 `X-Container-Rfc-Compliant-Etags`ヘッダを使用するとコンテナに保存されたオブジェクトを照会するとき、二重引用符で囲まれたETag値を返すように設定できます。

<br/>

##### オブジェクトロック期間の変更
`X-Container-Worm-Retention-Day`ヘッダを使用してオブジェクトロックコンテナのオブジェクトロック周期を変更します。ロック周期は日単位で入力でき、解除できません。変更されたロック周期は変更後にアップロードするオブジェクトに適用されます。オブジェクトロック周期はオブジェクトロックコンテナでのみ変更できます。

> [参考]
> 一般コンテナをオブジェクトロックコンテナに変更したり、オブジェクトロックコンテナを一般コンテナに変更できません。
> オブジェクトロックコンテナはアーカイブコンテナまたは複製対象コンテナに指定できません。
<br/>

##### コンテナ設定解除
値がないヘッダを使用すると設定が解除されます。例えばオブジェクトライフサイクルが3日に設定されている時、`'X-Container-Object-Lifecycle: '`を使用してコンテナ修正リクエストを行うと、オブジェクトライフサイクル設定が解除され、その後にコンテナに保存されるオブジェクトは自動的にライフサイクルが設定されません。
<br/>

#### レスポンス
レスポンス本文を返しません。リクエストが正しければステータスコード204を返します。
<br/>

#### コード例
すべてのユーザーにコンテナの読み取り、書き込みを許可する設定変更リクエストを行う例です。同じ方法で他の設定も必要なヘッダを選択してリクエストできます。

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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Container-Read", permission);    // ヘッダに権限追加

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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
    $req_header[] = 'X-Container-Read: ' . $permission;  // ヘッダに権限追加

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
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL| String |	O | 削除するコンテナ名 |

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード204を返します。

<br/>

#### コード例
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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
| X-Delete-At | Header | Timestamp | - | オブジェクトの有効期限、UNIX時間(秒) |
| X-Delete-After | Header | Timestamp | - | オブジェクト有効時間、 UNIX時間(秒) |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object | URL | String |	O | 作成するオブジェクト名 |
| - |	Body | Binary | O | 作成するオブジェクトの内容 |


##### オブジェクトライフサイクル設定
`X-Delete-At`または`X-Delete-After`ヘッダを使用するとオブジェクトのライフサイクルを秒単位で設定できます。
<br/>

> [注意]
> オブジェクトの名前が`./`または`../`で始まる場合、ブラウザがこれをパス文字として認識してWebコンソールからアクセスできません。
> APIを利用してこのような名前のオブジェクトをアップロードした場合、APIを通してアクセスする必要があります。

#### レスポンス
レスポンス本文を返しません。リクエストが正しければステータスコード201を返します。

#### コード例
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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // ファイルからInputStream作成
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

### マルチパートアップロード
5GBを超える容量を持つオブジェクトは5GB以下のセグメントに分割してアップロードする必要があります。セグメントオブジェクトをアップロードした後、マニフェストオブジェクトを作成すると1つのオブジェクトのように使用できます。

<br/>

#### セグメントオブジェクトのアップロード
オブジェクトを分割したセグメントオブジェクトをそれぞれアップロードします。

```
PUT   /v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

<br/>

##### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Content-type | Header | String | O | オブジェクトのコンテンツタイプ |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object |	URL | String | O | 作成するオブジェクト名 |
| Count | URL | Integer | O | 分割したオブジェクトの順番、例) 001, 002 |
| - |	Body | Binary | O | 分割したオブジェクトの内容 |

<br/>

##### レスポンス
レスポンス本文を返しません。リクエストが正しければステータスコード201を返します。

<br/>

#### マニフェストオブジェクトの作成
マニフェストオブジェクトは**DLO**(Dynamic Large Object)と **SLO**(Static Large Object)の2つの方式で作成できます。

> [参考]
> マニフェストオブジェクトはセグメントオブジェクトのパス情報を持っているため、セグメントオブジェクトとマニフェストオブジェクトを必ず同じコンテナにアップロードする必要はありません。セグメントオブジェクトとマニフェストオブジェクトが1つのコンテナにあって管理が難しい場合はセグメントオブジェクトを別のコンテナにアップロードして、アップロードしようとしていたコンテナにはマニフェストオブジェクトを作成することを推奨します。

**DLO**
DLOマニフェストオブジェクトは`X-Object-Manifest`ヘッダに入力したセグメントオブジェクトのパスを利用して自動的にセグメントオブジェクトを探して接続します。

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Segment-Container}/{Segment-Object}/
```

<br/>

##### リクエスト
| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | トークンID |
| X-Object-Manifest | Header| String | O | 分割したセグメントオブジェクトをアップロードしたパス、 `{Segment-Container}/{Segment-Object}/` |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object |	URL | String | O | 作成するマニフェストオブジェクト名 |
| - | Body| Binary | O | 空のデータ |

<br/>

**SLO**
SLOマニフェストオブジェクトは、リクエスト本文にセグメントオブジェクトリストを順序どおりに作成して入力する必要があります。最大1000個のセグメントオブジェクトを入力できます。
SLOマニフェストオブジェクト作成リクエストを行うと、各セグメントオブジェクトが入力されたパスにあるかどうか、etag値とオブジェクトのサイズが一致するかどうかを確認します。情報が一致していない場合はマニフェストオブジェクトが作成されません。

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

##### リクエスト
| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header| String |	O | トークンID |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object |	URL | String | O | 作成するマニフェストオブジェクト名 |
| multipart-manifest | Query| String | O | put |
| path | Body | String | O | セグメントオブジェクトのパス |
| etag | Body | String | O | セグメントオブジェクトのetag |
| size_bytes | Body | Integer | O | セグメントオブジェクトのサイズ(バイト単位) |

> [参考]
> SLOマニフェストファイルが持っているセグメント情報を照会するには`multipart-manifest=get`クエリを利用する必要があります。

<br/>

##### レスポンス
レスポンス本文を返しません。リクエストが正しければステータスコード201を返します。

<br/>

#### コード例
DLO方式を利用したマルチパートアップロード例

<details>
<summary>cURL</summary>

```
// 200MB単位でファイル分割
$ split -d -b 209715200 large_obj.img large_obj.img.

// 分割されたオブジェクトのアップロード
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/large_obj.img/003 \
-T large_obj.img.02

// マニフェストオブジェクトのアップロード
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

    // マニフェストオブジェクトのアップロード
    public void uploadManifestObject(String containerName, String objectName) {
        String url = this.getUrl(containerName, objectName);
        String manifestName = containerName + "/" + objectName + "/"; // マニフェスト名作成

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Object-Manifest", manifestName);  // ヘッダにマニフェスト表記

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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

        final int defaultChunkSize = 100 * 1024; // 100 KB単位で分割
        int chunkSize = defaultChunkSize;
        int chunkNo = 0;  // 分割オブジェクトの名前を作成するためのチャンク番号
        int totalBytesRead = 0;

        try {
            // ファイルからInputStream作成
            InputStream inputStream = new BufferedInputStream(new FileInputStream(objFile));
            while(totalBytesRead < fileSize) {

                // 残りデータサイズ計算
                int remainedBytes = fileSize - totalBytesRead;
                if(remainedBytes < chunkSize) {
                    chunkSize = remainedBytes;
                }

                // バイトバッファにチャンクサイズ分のデータを読み取り
                byte[] chunkBuffer = new byte[chunkSize];
                int bytesRead = inputStream.read(chunkBuffer, 0, chunkSize);

                if(bytesRead > 0) {
                    // バッファのデータをInputStreamで作成し、アップロード、オブジェクトアップロード例のuploadObject()メソッド使用
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
    $fd = fopen($filename, 'r');  // ファイルを開く。
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

      // パート臨時ファイル作成
      $part_fd = fopen($temp_file, 'w+');
      fwrite($part_fd, $chunk);
      fseek($part_fd, 0);

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
        CURLOPT_PUT => TRUE,
        CURLOPT_HEADER => TRUE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_INFILE => $part_fd,  // パートファイルストリームを引数に入力
        CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
      printf("$response");

      // 臨時ファイル削除
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
| X-Delete-At | Header | Timestamp | - | オブジェクトの有効期限、UNIX時間(秒) |
| X-Delete-After | Header | Timestamp | - | オブジェクト有効時間、UNIX時間(秒) |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object | URL | String | O | 内容を修正するオブジェクト名 |

#### レスポンス
レスポンス本文を返しません。リクエストが正しければステータスコード201を返します。

<br/>

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
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container |	URL | String | O | コンテナ名 |
| Object | URL | String | O | ダウンロードするオブジェクト名 |

#### レスポンス
オブジェクトの内容がストリームで返されます。リクエストが正しければステータスコード200を返します。

#### コード例
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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****";
        final String tokenId = "d052a0a054b745dbac74250b7fecbc09";
        final String containerName = "test";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";
        final String downloadPath = "/home/example/download";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // オブジェクトのダウンロード
            InputStream inputStream = objectService.downloadObject(containerName, objectName);

            // ダウンロードしたデータをバイトバッファに読み込む
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

### オブジェクトのコピー
オブジェクトを他のコンテナにコピーします。

> [参考]
> 5GBを超えるマルチパートオブジェクトはコピーできません。 
> マルチパートオブジェクトのマニフェストオブジェクトをコピーするコンテナに作成すると、セグメントオブジェクトをコピーしなくてもオブジェクトにアクセスできます。ただし、原本セグメントオブジェクトを削除するとデータにアクセスできません。

```
COPY   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Destination: {オブジェクトをコピーする対象}
```

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Copy-From: {原本オブジェクト}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Destination | Header | String |	- | オブジェクトをコピーする対象、 `{コンテナ}/{オブジェクト}`<br/>COPYメソッドを使用する時に必要 |
| X-Copy-From | Header | String |	- | 原本オブジェクト、 `{コンテナ}/{オブジェクト}`<br/>PUTメソッドを使用する時に必要 |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL | String | O |	コンテナ名<br/>COPYメソッド：原本コンテナ<br/>PUTメソッド：コピーするコンテナ |
| Object | URL | String |	コピーするオブジェクト名 |

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード201を返します。

#### コード例
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Copy-From", srcObject);    // 原本オブジェクト指定
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // HttpMethodはCOPYメソッドをサポートしません。PUTメソッドを使用する別のAPIを呼び出します。
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

### オブジェクトメタデータ修正
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
| X-Delete-At | Header | Timestamp | - | オブジェクトの有効期限、UNIX時間(秒) |
| X-Delete-After | Header | Timestamp | - | オブジェクト有効時間、UNIX時間(秒) |
| X-Object-Worm-Retain-Until | Header | Timestamp | - | オブジェクトロック有効期限、UNIX時間(秒)<br/>設定された時間以降にのみ変更でき、オブジェクトロックコンテナでのみ変更可能 |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL| String |	 O | コンテナ名 |
| Object | URL| String |  O | メタデータを修正するオブジェクト名 |

> [参考]
> オブジェクトロックコンテナにアップロードされたオブジェクトには自動的にロック有効期限が設定されます。 
> ロック有効期限が過ぎていないオブジェクトは、上書きまたは削除できません。 
> オブジェクトのメタデータはロック有効期限より前でも変更できます。

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード202を返します。

#### コード例
<details>
<summary>cURL</summary>

```
// オブジェクトにメタデータ追加
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo" \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/curl_example/ba6610.jpg

// オブジェクトヘッダで追加したメタデータ確認
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

        // メタデータキーを作成
        String metaKey = "X-Object-Meta-" + key;

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add(metaKey, value);    // ヘッダにメタデータ設定

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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
    $req_header[] = 'X-Object-Meta-'.$key.': '.$value;  // ヘッダにメタデータ追加

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

### オブジェクト削除
指定したオブジェクトを削除します。

> [参考]
> マルチパートアップロードしたオブジェクトを削除する時はセグメントデータを全て削除する必要があります。マニフェストのみ削除するとセグメントオブジェクトが残って課金される場合があります。

```
DELETE   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | トークンID |
| Account | URL | String | O | ストレージアカウント名。API Endpoint設定ダイアログボックスで確認 |
| Container | URL| String |	 O | コンテナ名 |
| Object | URL| String |  O | 削除するオブジェクト名 |

#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければステータスコード204を返します。

<br/>

#### コード例
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

        // ヘッダ作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API呼び出し
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
