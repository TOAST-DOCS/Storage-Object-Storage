<!-- pre-align:aligned sig=288abf1f2a5c -->

<a id="storage-object-storage-api-guide"></a>
## Storage > Object Storage > APIガイド { #storage-object-storage-api-guide }

このドキュメントでは、NHN Cloud オブジェクトストレージが提供する API を使用してストレージアカウント、コンテナ、オブジェクトを管理する方法について説明します。

<a id="common"></a>
## オブジェクトストレージ API 共通情報 { #common }

<a id="endpoint"></a>
### API エンドポイント { #endpoint }

API を使用するには、API エンドポイントとトークンが必要です。[IaaS トークン](/nhncloud/ja/public-api/iaas-token/)を参照して、API の使用に必要な情報を準備します。
オブジェクトストレージ API は `object-store` タイプのエンドポイントを使用します。正確なエンドポイントはトークン発行レスポンスの `serviceCatalog` を参照してください。

| リージョン | エンドポイント |
| --- | --- |
| 韓国(板橋)リージョン<br>韓国(坪村)リージョン<br>韓国(光州)リージョン<br>日本(東京)リージョン | https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\*<br>https://kr2-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\*<br>https://kr3-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\*<br>https://jp1-api-object-storage.nhncloudservice.com/v1/AUTH_\*\*\*\*\* |

<a id="auth"></a>
### 認証および権限 { #auth }

オブジェクトストレージは、API 呼び出し時の認証/認可に IaaS トークンを使用します。IaaS トークンは、NHN Cloud の OpenStack ベースのインフラサービス (IaaS) で使用する認証トークンです。
IaaS トークンの発行および使用の詳細については、[IaaS トークン](/nhncloud/ja/public-api/iaas-token/)を参照してください。

!!! danger "注意"
    オブジェクトストレージは、基本インフラサービスとは異なるテナント ID を持っています。
    オブジェクトストレージのテナント ID は、オブジェクトストレージサービスページの **[API エンドポイント設定]** ボタンをクリックして確認できます。

<!-- 改行のためのコメント -->

!!! tip "ヒント"
    API パスワードは、オブジェクトストレージサービスページの **[API エンドポイント設定]** ボタンをクリックして設定することもできます。

<a id="auth-token-issuance-code-example"></a>
#### トークン発行コードの例

<details>
<summary>cURL</summary>

```
$ curl -X POST -H 'Content-Type:application/json' \
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

    // リクエストボディ用内部クラス
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

        // リクエストボディを生成
        this.tokenRequest = new TokenRequest();
        this.tokenRequest.getAuth().setTenantId(tenantId);
        this.tokenRequest.getAuth().getPasswordCredentials().setUsername(username);
        this.tokenRequest.getAuth().getPasswordCredentials().setPassword(password);

        this.restTemplate = new RestTemplate();
    }

    public String requestToken() {
        String identityUrl = this.authUrl + "/tokens";

        // ヘッダーを生成
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/json");

        HttpEntity<TokenRequest> httpEntity
            = new HttpEntity<TokenRequest>(this.tokenRequest, headers);

        // トークンをリクエスト
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
  );  // リクエスト本文を生成
  $req_header = array(
    'Content-Type: application/json'
  );  // リクエストヘッダーを生成

  $curl  = curl_init($url); // curl を初期化
  curl_setopt_array($curl, array(
    CURLOPT_POST => TRUE,
    CURLOPT_RETURNTRANSFER => TRUE,
    CURLOPT_HTTPHEADER => $req_header,
    CURLOPT_POSTFIELDS => json_encode($req_body)
  )); // パラメータを設定
  $response = curl_exec($curl); // API を呼び出し
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
## ストレージアカウント { #storage-account }
ストレージアカウント (account) は `AUTH_*****` 形式の文字列です。Object-Store API エンドポイントに含まれています。

<a id="query-the-storage-account"></a>
### ストレージアカウントの照会 { #query-the-storage-account }
ストレージアカウントの使用状況を照会します。

```
HEAD  /v1/{Account}
X-Auth-Token: {token-id}
```

<a id="query-the-storage-account-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークン ID |
| Account | URL | String | Y | ストレージアカウント。API エンドポイント設定ダイアログボックスで確認 |

<a id="query-the-storage-account-response"></a>
#### レスポンス
レスポンス本文は返しません。使用状況はヘッダーに含まれています。リクエストが正しい場合、ステータスコード 200 を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| X-Account-Container-Count | Header | String | コンテナ数 |
| X-Account-Object-Count | Header | String | 保存されているオブジェクト数 |
| X-Account-Bytes-Used | Header | String | 保存されているデータ容量 (バイト) |

<a id="query-the-storage-account-code-example"></a>
#### コード例

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

        // ヘッダーの作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
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

<br/>

<a id="list-containers"></a>
### コンテナリストの照会 { #list-containers }
ストレージアカウントのコンテナリストを照会します。

```
GET  /v1/{Account}
X-Auth-Token: {token-id}
```

<a id="list-containers-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークン ID |
| Account | URL | String | Y | ストレージアカウント。API エンドポイント設定ダイアログボックスで確認 |

<a id="list-containers-response"></a>
#### レスポンス
```
[ストレージアカウントに属するコンテナのリスト]
```

<a id="list-containers-code-example"></a>
#### コード例

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
    // AccountService クラス ...
    public List<String> getContainerList() {
        // ヘッダーの作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
        ResponseEntity<String> response
            = this.restTemplate.exchange(this.getStorageUrl(), HttpMethod.GET, requestHttpEntity, String.class);

        List<String> containerList = null;
        if (response.getStatusCode() == HttpStatus.OK) {
            // String で受け取ったリストを配列に変換
            containerList = Arrays.asList(response.getBody().split("\\r?\\n"));
        }

        // 配列を List に変換して返す
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

<br/>

<a id="container"></a>
## コンテナ { #container }

<a id="create-a-container"></a>
### コンテナの作成 { #create-a-container }
コンテナを作成します。Object Storage にファイルをアップロードするには、必ずコンテナを作成する必要があります。

!!! tip "ヒント"
    コンテナ名に特殊文字 ``' " ` < > ;`` および空白、相対パス文字 (`. ..`) は使用できません。
    IP アドレス形式の名前は使用できません。
    コンテナまたはオブジェクト名に特殊文字 `! * ' ( ) ; : @ & = + $ , / ? # [ ]` が含まれている場合、API を使用する際は必ず URL エンコード（パーセントエンコーディング）を行う必要があります。これらの文字は URL において重要な役割を持つ予約文字です。これらの文字が含まれるパスを URL エンコードせずに API リクエストを送信すると、意図したレスポンスを受け取れない場合があります。

コンテナを作成する際、`X-Storage-Policy` ヘッダーを使用してコンテナのストレージクラスを指定できます。頻繁にアクセスするデータ向けの Standard クラスと、アクセス頻度の低いデータを低コストで長期保管できる Economy クラスを選択できます。ストレージクラスを指定しない場合は、Standard クラスが適用されます。

!!! tip "ヒント"
    作成済みコンテナのストレージクラスは変更できません。
    Economy クラスコンテナにアップロードされたオブジェクトには、最低保管期間 30 日が適用されます。30 日以前に削除されたオブジェクトに対しても、残余保管期間分の料金が課金されます。
    Economy クラスコンテナは API リクエスト 1,000 件ごとに料金が適用されます（HEAD/DELETE リクエストを除く）。

コンテナを作成する際、`X-Container-Worm-Retention-Day` ヘッダーを使用してオブジェクトロック周期を設定すると、オブジェクトロックコンテナを作成できます。オブジェクトロックコンテナにアップロードしたオブジェクトは **WORM (Write-Once-Read-Many)** モデルを使用して保存されます。オブジェクトロックコンテナにアップロードしたオブジェクトにはロック有効期限日が設定されます。各オブジェクトに設定されたロック有効期限日より前は、オブジェクトを上書きまたは削除することはできません。

<br/>

```
PUT  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

<a id="create-a-container-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークン ID |
| Account | URL | String | Y | ストレージアカウント。API エンドポイント設定ダイアログボックスで確認 |
| Container | URL | String | Y | 作成するコンテナ名 |
| X-Storage-Policy | Header | String | N | コンテナのストレージクラス<br/><b>Standard</b>: 頻繁にアクセスするデータ向けのデフォルトクラス<br/><b>Economy</b>: アクセス頻度の低いデータを長期保管するのに適したクラス |
| X-Container-Worm-Retention-Day | Header | Integer | N | コンテナのデフォルトオブジェクトロック周期を日単位で設定 |


<a id="create-a-container-response"></a>
#### レスポンス
レスポンス本文は返しません。コンテナが作成された場合、ステータスコード 201 を返します。

<a id="create-a-container-code-example"></a>
#### コード例
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

        // ヘッダーの作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
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

<br/>

<a id="get-a-container"></a>
### コンテナの照会 { #get-a-container }
指定したコンテナの情報と、その中に保存されているオブジェクトのリストを照会します。コンテナの情報は応答ヘッダーで確認できます。

```
GET   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

<a id="get-a-container-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Account | URL | String | Y | ストレージアカウント。API エンドポイント設定ダイアログボックスで確認 |
| Container | URL | String | Y | 照会するコンテナ名 |
| marker | Query | String | N | 基準オブジェクト名 |
| prefix | Query | String | N | 検索する接頭辞 |
| limit | Query | Integer | N | リストに表示するオブジェクト数 |
| format | Query | String | N | 応答形式。json または xml |

!!! tip "注記"
    コンテナ照会 API はいくつかのクエリを提供します。すべてのクエリは `&` で連結して組み合わせることができます。

<a id="list-objects-over-10k"></a>
#### 1万件以上のオブジェクトリストの照会
コンテナ照会 API で照会できるリストのオブジェクト数は 1 万件に制限されています。1 万件以上のオブジェクトリストを照会するには、`marker` クエリを使用する必要があります。marker クエリは、指定したオブジェクトの次のオブジェクトから最大 1 万件のリストを返します。

<br/>

<a id="list-objects-with-a-prefix"></a>
#### 接頭辞で始まるオブジェクトリストの照会
`prefix` クエリを使用すると、指定した接頭辞で始まるオブジェクトのリストを返します。prefix クエリを使用して、サブフォルダーのオブジェクトリストを照会できます。

<br/>

<a id="list-objects-with-limit"></a>
#### リストの最大オブジェクト数の指定
`limit` クエリを使用すると、返すオブジェクトリストの最大オブジェクト数を指定できます。

<br/>

<a id="list-objects-with-format"></a>
#### 応答形式の指定
`format` クエリを使用して、`json` または `xml` の応答形式を指定できます。応答形式を指定すると、応答本文に各オブジェクトのメタデータ（サイズ、コンテンツタイプ、最終更新日時、ETag）が含まれます。

<br/>

<a id="get-a-container-response"></a>
#### レスポンス

```
[コンテナのオブジェクトリスト]
```

<a id="get-a-container-code-example"></a>
#### コード例
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
        // ヘッダーを生成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
        ResponseEntity<String> response
            = this.restTemplate.exchange(url, HttpMethod.GET, requestHttpEntity, String.class);

        if (response.getStatusCode() == HttpStatus.OK) {
            // String で受け取ったリストを配列に変換
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

<br/>

<a id="change-container-settings"></a>
### コンテナ設定の変更 { #change-container-settings }

コンテナの設定を変更します。コンテナの設定は、コンテナ照会時の応答ヘッダーで確認できます。

```
POST  /v1/{Account}/{Container}
X-Auth-Token: {token-id}
X-Container-Read: {コンテナ読み取りに対するロールベースのアクセスルール}
X-Container-Write: {コンテナ書き込みに対するロールベースのアクセスルール}
X-Container-View: {コンテナ照会に対するロールベースのアクセスルール}
X-Container-Ip-Acl-Allowed-List: {コンテナアクセスに対するIPベースのアクセスルール}
X-Container-Ip-Acl-Denied-List: {コンテナアクセスに対するIPベースのアクセスルール}
X-Container-Object-Lifecycle: {コンテナのオブジェクトライフサイクル}
X-Container-Object-Transfer-To: {オブジェクトのライフサイクルが期限切れになったときに移動先となるコンテナ}
X-History-Location: {オブジェクトの旧バージョンを保存するコンテナ}
X-Versions-Retention: {オブジェクトの旧バージョンのライフサイクル}
X-Container-Meta-Web-Index: {静的ウェブサイトのインデックスドキュメントオブジェクト}
X-Container-Meta-Web-Error: {静的ウェブサイトのエラードキュメントオブジェクトのサフィックス}
X-Container-Meta-Access-Control-Allow-Origin: {クロスオリジンリソース共有の許可リスト}
X-Container-Rfc-Compliant-Etags: {RFC準拠のETag形式を使用するかどうか}
X-Container-Worm-Retention-Day: {コンテナのオブジェクトロック周期}
X-Container-Object-Deny-Extension-Policy: {オブジェクトアップロードポリシーの拡張子ブラックリスト}
X-Container-Object-Deny-Keyword-Policy: {オブジェクトアップロードポリシーのファイル名ブラックリスト}
X-Container-Object-Allow-Extension-Policy: {オブジェクトアップロードポリシーの拡張子ホワイトリスト}
X-Container-Object-Allow-Keyword-Policy: {オブジェクトアップロードポリシーのファイル名ホワイトリスト}
```

<a id="change-container-settings-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| X-Container-Read | Header | String | N | コンテナ読み取りに対するロールベースのアクセスルール設定 |
| X-Container-Write | Header | String | N | コンテナ書き込みに対するロールベースのアクセスルール設定 |
| X-Container-View | Header | String | N | コンテナ照会に対するロールベースのアクセスルール設定 |
| X-Container-Ip-Acl-Allowed-List | Header | String | N | コンテナアクセスに対するIPベースのアクセスルール設定 |
| X-Container-Ip-Acl-Denied-List | Header | String | N | コンテナアクセスに対するIPベースのアクセスルール設定 |
| X-Container-Object-Lifecycle | Header | Integer | N | コンテナのデフォルトオブジェクトライフサイクルを日単位で設定 |
| X-Container-Object-Transfer-To | Header | String | N | オブジェクトのライフサイクルが期限切れになったときに移動先となるコンテナ |
| X-History-Location | Header | String | N | オブジェクトの旧バージョンを保管するコンテナを設定 |
| X-Versions-Retention | Header | Integer | N | オブジェクトの旧バージョンのライフサイクルを日単位で設定 |
| X-Container-Meta-Web-Index | Header | String | N | 静的ウェブサイトのインデックスドキュメントオブジェクトを設定<br/>英字、数字、一部の特殊文字（`-`, `_`, `.`, `/`）のみ使用可 |
| X-Container-Meta-Web-Error | Header | String | N | 静的ウェブサイトのエラードキュメントオブジェクトのサフィックスを設定<br/>英字、数字、一部の特殊文字（`-`, `_`, `.`, `/`）のみ使用可 |
| X-Container-Meta-Access-Control-Allow-Origin | Header | String | N | CORS許可ホストリスト。`*` ですべてのホストを許可するか、スペース区切りのホストリストを入力できます。 |
| X-Container-Rfc-Compliant-Etags | Header | String | N | RFCに準拠したETag形式を使用するかどうかを設定。true または false |
| X-Container-Worm-Retention-Day | Header | Integer | N | コンテナのデフォルトオブジェクトロック期間を日単位で設定<br/>オブジェクトロックコンテナでのみ変更可能 |
| X-Container-Object-Deny-Extension-Policy | Header | String | N | オブジェクトアップロードポリシーの拡張子ブラックリスト |
| X-Container-Object-Deny-Keyword-Policy | Header | String | N | オブジェクトアップロードポリシーのファイル名ブラックリスト |
| X-Container-Object-Allow-Extension-Policy | Header | String | N | オブジェクトアップロードポリシーの拡張子ホワイトリスト |
| X-Container-Object-Allow-Keyword-Policy | Header | String | N | オブジェクトアップロードポリシーのファイル名ホワイトリスト |
| Account | URL | String | Y | ストレージアカウント。APIエンドポイント設定ダイアログで確認できます |
| Container | URL | String | Y | 変更するコンテナ名 |
<br/>

<a id="set-container-rbac-policy"></a>
##### アクセスポリシー設定
`X-Container-Read`、`X-Container-Write`、`X-Container-View`、`X-Container-Ip-Acl-Allowed-List`、`X-Container-Ip-Acl-Denied-List`、`X-Container-Ip-Acl-Service-Gateway-Control` ヘッダーを使用して、コンテナのアクセスポリシーを設定できます。詳細については、[アクセスポリシー設定ガイド](acl-guide/)を参照してください。

<br/>

<a id="set-container-object-lifecycle"></a>
##### オブジェクトライフサイクル設定
`X-Container-Object-Lifecycle` ヘッダーを使用すると、コンテナに保存されるオブジェクトのライフサイクルを日単位で設定できます。設定後にアップロードしたオブジェクトにのみ適用されます。
`X-Container-Object-Transfer-To` ヘッダーを使用すると、ライフサイクルが期限切れになったオブジェクトを指定したコンテナに移動して保管できます。コンテナが指定されていない場合、期限切れのオブジェクトは削除されます。

!!! tip "ヒント"
    コンテナポリシーを使用して、より詳細なライフサイクルルールを設定できます。
    詳細については、[コンテナポリシー設定ガイド](container-policy-guide/#lifecycle)を参照してください。

<!-- 改行用コメント -->

!!! tip "ヒント"
    Standard クラスのコンテナに保存されたオブジェクトをライフサイクルに従って Economy クラスのコンテナに移動することで、長期保管にかかるコストを削減できます。

<br/>

<a id="set-container-object-version-policy"></a>
##### バージョン管理ポリシー設定
[オブジェクト内容の変更](api-guide/#update-an-object)に記載のとおり、オブジェクトをアップロードする際に同じ名前のオブジェクトがすでに存在する場合、オブジェクトは更新されます。既存オブジェクトの内容を保管したい場合は、`X-History-Location` ヘッダーを使用して旧バージョンを保管する**アーカイブコンテナ**を指定できます。

旧バージョンのオブジェクトはアーカイブコンテナに次の形式で保管されます。
```
`{3桁の16進数で表されたオブジェクト名の長さ}{オブジェクト名}/{以前のバージョンのオブジェクトが保管されたUnix時間}`
```
例えば、`picture.jpg` というオブジェクトを更新すると、アーカイブコンテナに `00bpicture.jpg/1610606551.82539` というオブジェクトが作成されます。

バージョン管理ポリシーが設定されたコンテナでオブジェクトを削除すると、アーカイブコンテナに削除されたオブジェクトが保管され、削除マーカーオブジェクトが作成されます。アーカイブコンテナに保管された旧バージョンのオブジェクトにはいつでもアクセスできます。

`X-Versions-Retention` ヘッダーを併用すると、旧バージョンのオブジェクトのライフサイクルを日単位で設定できます。1 を設定した場合、保管されたオブジェクトは 1 日後に自動的に削除されます。設定しない場合、旧バージョンのオブジェクトはユーザーが削除するまで保管されます。設定後に保管された旧バージョンのオブジェクトにのみ適用されます。

!!! danger "注意"
    アーカイブコンテナを元のコンテナより先に削除すると、元のコンテナでオブジェクトを更新または削除する際にエラーが発生します。すでに削除した場合は、アーカイブコンテナを新たに作成するか、元のコンテナのバージョン管理ポリシーを解除することで解決できます。

アーカイブコンテナとして使用するコンテナ名には、できる限りUnicode文字を使用しないことをお勧めします。アーカイブコンテナとして指定するコンテナ名にUnicode文字が含まれている場合は、必ずURLエンコード後にリクエストヘッダーに入力してください。

暗号化コンテナをアーカイブコンテナに指定した後、暗号化コンテナの対称キーをSecure Key Managerサービスで削除すると、元のコンテナへのオブジェクトのアップロードと削除が失敗します。

<br/>

<a id="set-container-static-website"></a>
##### 静的ウェブサイト設定
コンテナの読み取りアクセス権限をすべてのユーザーに許可した後、`X-Container-Meta-Web-Index`、`X-Container-Meta-Web-Error` ヘッダーを使用して静的ウェブサイトのインデックスドキュメントとエラードキュメントを設定すると、コンテナURLを使用して静的ウェブサイトをホスティングできます。

静的ウェブサイトのインデックスドキュメントおよびエラードキュメントとして使用するオブジェクトは、1文字以上の英字、数字、または一部の特殊文字（`-`, `_`, `.`, `/`）で構成された名前である必要があり、ファイル拡張子が `.html` のハイパーテキスト形式でなければなりません。条件を満たさない場合、設定できないか、静的ウェブサイトが正常に動作しない場合があります。
静的ウェブサイトのエラードキュメント名は `{レスポンスコード}{サフィックス}` の形式です。例えば、エラードキュメントを `error.html` と設定した場合、404エラーが発生したときに表示されるエラードキュメントの名前は `404error.html` になります。各エラー状況に合わせてエラードキュメントをアップロードして使用できます。エラードキュメントを定義しない場合、またはレスポンスコードに対応するエラードキュメントオブジェクトが存在しない場合は、ウェブブラウザのデフォルトエラードキュメントが表示されます。
<br/>

<a id="set-container-cors-policy"></a>
##### Cross-Origin Resource Sharing (CORS)

ブラウザから Object Storage API を直接呼び出すには、Cross-Origin Resource Sharing (CORS) の設定が必要です。`X-Container-Meta-Access-Control-Allow-Origin` ヘッダーを使用して、許可するオリジンのリストを設定します。スペース（` `）区切りで1つ以上のオリジンを入力するか、`*` を入力してすべてのオリジンを許可できます。

!!! tip "ヒント"
    `X-Container-Meta-Access-Control-Allow-Origin` に設定できる許可オリジンは最大100件です。この制限は[コンテナポリシー](container-policy-guide/#cors)で設定する場合も同様に適用されます。

<details>
<summary>CORS設定確認の例</summary>

コンテナにCORS設定を追加します。

```
$ curl -X POST \
-H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Container-Meta-Access-Control-Allow-Origin: https://example.com' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container
```
<br>
ブラウザでCORSを許可したサイトに移動した後、以下のスクリプトを実行します。スクリプトはブラウザが提供する開発者ツールのコンソールで実行できます。

<br/>
例) https://example.com/

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
CORS設定に問題がなければ、コンソールで以下のような成功レスポンスを確認できます。

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
CORS設定をしていない場合や、許可されていないサイトからAPIを呼び出すと、以下のようなエラーレスポンスが返されます。

```
Access to XMLHttpRequest at 'https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/container/object' from origin 'https://example.com' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.

Status: 0
```

</details>

<br/>

<a id="set-container-rfc-compliant-etag"></a>
##### RFCに準拠したETag形式の使用設定
一部のアプリケーションでは、[RFC7232](https://www.rfc-editor.org/rfc/rfc7232#section-2.3) の仕様に従い、ダブルクォートで囲まれたETag値を要求します。`X-Container-Rfc-Compliant-Etags` ヘッダーを使用すると、コンテナに保存されたオブジェクトを照会する際にダブルクォートで囲まれたETag値を返すように設定できます。

<br/>

<a id="set-container-object-lock-cycle"></a>
##### オブジェクトロック期間の変更
`X-Container-Worm-Retention-Day` ヘッダーを使用して、オブジェクトロックコンテナのオブジェクトロック期間を変更します。ロック期間は日単位で入力でき、解除することはできません。変更されたロック期間は、変更後にアップロードするオブジェクトに適用されます。オブジェクトロック期間はオブジェクトロックコンテナでのみ変更できます。

!!! tip "ヒント"
    通常のコンテナをオブジェクトロックコンテナに変更したり、オブジェクトロックコンテナを通常のコンテナに変更したりすることはできません。
    オブジェクトロックコンテナは、アーカイブコンテナまたはレプリケーション対象コンテナとして指定できません。

<br/>

<a id="set-container-upload-policy"></a>
##### アップロードポリシー設定の変更
`X-Container-Object-Deny-Extension-Policy`、`X-Container-Object-Deny-Keyword-Policy`、`X-Container-Object-Allow-Extension-Policy`、`X-Container-Object-Allow-Keyword-Policy` ヘッダーを使用して、コンテナにオブジェクト名ベースのアップロードポリシーを設定できます。アップロードポリシー設定を活用すると、名前に特定の拡張子やキーワードが含まれるオブジェクトのみをアップロードする、またはアップロードできないように制限できます。

アップロードポリシーは、ポリシーが設定された以降にアップロードされるオブジェクトに適用されます。パスが含まれるオブジェクトは、パスを除いたオブジェクト名にポリシーが適用されます。
すべてのアップロードポリシーヘッダーは `,` 区切り文字を使用して複数のルールを入力でき、区切り文字 `,` を除く各ルールはURLエンコード（パーセントエンコーディング）する必要があります。
拡張子ルールはファイルの拡張子を、ファイル名ルールはオブジェクト名への含有を検査します。拡張子ルールは `.` を除いて入力する必要があります。例えば、txt拡張子を入力する場合は `.txt` ではなく `txt` のみ入力します。

アップロードポリシーはホワイトリストとブラックリストを同時に使用できません。両方の属性を設定するようリクエストした場合、失敗レスポンスが返されます。

<details>
<summary>ホワイトリスト設定例を確認</summary>

コンテナにホワイトリストのアップロードポリシー設定を追加します。

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
<summary>ブラックリスト設定例を確認</summary>

コンテナにブラックリストのアップロードポリシー設定を追加します。

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
##### コンテナ設定の解除
値のないヘッダーを使用すると、設定が解除されます。たとえば、オブジェクトのライフサイクルが 3 日に設定されている場合に `'X-Container-Object-Lifecycle: '` を使用してコンテナの変更をリクエストすると、オブジェクトのライフサイクル設定が解除され、以降コンテナに保存されるオブジェクトにはライフサイクルが自動的に設定されなくなります。
<br/>

<a id="change-container-settings-response"></a>
#### レスポンス
レスポンス本文は返しません。リクエストが正しい場合は、ステータスコード 204 を返します。
<br/>

<a id="change-container-settings-code-example"></a>
#### コード例
すべてのユーザーにコンテナの読み取りおよび書き込みアクセスを許可する設定変更リクエストの例です。同じ方法で他の設定も必要なヘッダーを選択してリクエストできます。

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

    // ContainerService クラス ...

    public void setContainerReadACL(String containerName, boolean isPublic) {
        final String PUBLIC_ACL = ".r:*";
        final String PRIVATE_ACL = "";

        String permission = isPublic ? PUBLIC_ACL : PRIVATE_ACL;

        String url = this.getUrl(containerName);

        // ヘッダーの作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Container-Read", permission);    // ヘッダーに権限を追加

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
        this.restTemplate.exchange(url, HttpMethod.POST, requestHttpEntity, String.class);
    }

    public static void main(String[] args) {
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
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
    $req_header[] = 'X-Container-Read: ' . $permission;  // ヘッダーにアクセス許可を追加

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

<br/>

<a id="delete-a-container"></a>
### コンテナの削除 { #delete-a-container }

指定したコンテナを削除します。削除するコンテナは必ず空である必要があります。

```
DELETE   /v1/{Account}/{Container}
X-Auth-Token: {token-id}
```

<a id="delete-a-container-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Account | URL | String | Y | ストレージアカウント、APIエンドポイント設定ダイアログボックスで確認 |
| Container | URL| String | Y | 削除するコンテナ名 |

<a id="delete-a-container-response"></a>
#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード 204 を返します。

<br/>

<a id="delete-a-container-code-example"></a>
#### コード例
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

        // ヘッダーの作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
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

<br/>

<a id="object"></a>
## オブジェクト { #object }

<a id="upload-an-object"></a>
### オブジェクトのアップロード { #upload-an-object }
指定したコンテナに新しいオブジェクトをアップロードします。

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

<a id="upload-an-object-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Content-type | Header | String | Y | オブジェクトのコンテンツタイプ |
| X-Delete-At | Header | Timestamp | N | オブジェクトの有効期限、Unix時間（秒） |
| X-Delete-After | Header | Timestamp | N | オブジェクトの有効時間、Unix時間（秒） |
| Account | URL | String | Y | ストレージアカウント、APIエンドポイント設定ダイアログボックスで確認 |
| Container | URL | String | Y | コンテナ名 |
| Object | URL | String | Y | 作成するオブジェクト名 |
| - | Body | Binary | Y | 作成するオブジェクトの内容 |

<a id="set-object-lifecycle"></a>
##### オブジェクトのライフサイクル設定
`X-Delete-At` または `X-Delete-After` ヘッダーを使用すると、オブジェクトのライフサイクルを秒単位で設定できます。
<br/>

!!! danger "注意"
    オブジェクト名が `./` または `../` で始まる場合、ブラウザがそれをパス文字として認識し、Webコンソールからアクセスできません。
    APIを使用してこのような名前のオブジェクトをアップロードした場合は、APIからアクセスする必要があります。

<a id="upload-an-object-response"></a>
#### レスポンス
レスポンス本文は返しません。リクエストが正しければ、ステータスコード 201 を返します。

<a id="upload-an-object-code-example"></a>
#### コード例
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

        // InputStreamをリクエストボディに追加できるようにRequestCallbackをオーバーライド
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
        final String storageUrl = "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067";
        final String tokenId = "b587ae461278419da6ecd21a2344c8aa";
        final String containerName = "test";
        final String objectPath = "/home/example/";
        final String objectName = "46432aa503ab715f288c4922911d2035.jpg";

        ObjectService objectService = new ObjectService(storageUrl, tokenId);

        try {
            // ファイルからInputStreamを生成
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

    $fd = fopen($filename, 'r');  // ファイルを開く。

    $curl  = curl_init($req_url);
    curl_setopt_array($curl, array(
      CURLOPT_PUT => TRUE,
      CURLOPT_RETURNTRANSFER => TRUE,
      CURLOPT_INFILE => $fd,  // ファイルストリームをパラメータに渡す。
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

<br/>

<a id="multipart-upload"></a>
### マルチパートアップロード { #multipart-upload }
5GB を超えるサイズのオブジェクトは、5GB 以下のセグメントに分割してアップロードする必要があります。セグメントオブジェクトをアップロードした後、マニフェストオブジェクトを作成すると、1つのオブジェクトとして使用できます。

<br/>

<a id="upload-segment-object"></a>
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
| X-Auth-Token | Header | String | Y | トークンID |
| Content-type | Header | String | Y | オブジェクトのコンテンツタイプ |
| Account | URL | String | Y | ストレージアカウント、APIエンドポイント設定ダイアログボックスで確認 |
| Container | URL | String | Y | コンテナ名 |
| Object | URL | String | Y | 作成するオブジェクト名 |
| Count | URL | Integer | Y | 分割したオブジェクトの順番、例) 001, 002 |
| - | Body | Binary | Y | 分割したオブジェクトの内容 |

<br/>

##### レスポンス
レスポンス本文は返しません。リクエストが正しければ、ステータスコード 201 を返します。

<br/>

<a id="upload-manifest-object"></a>
#### マニフェストオブジェクトの作成
マニフェストオブジェクトは **DLO**(Dynamic Large Object) と **SLO**(Static Large Object) の 2 種類の方法で作成できます。

!!! tip "ヒント"
    マニフェストオブジェクトはセグメントオブジェクトのパス情報を保持しているため、セグメントオブジェクトとマニフェストオブジェクトを必ず同じコンテナにアップロードする必要はありません。セグメントオブジェクトとマニフェストオブジェクトが同一のコンテナに存在して管理が難しい場合は、セグメントオブジェクトを別のコンテナにアップロードし、元々アップロードしようとしていたコンテナにはマニフェストオブジェクトのみ作成することをお勧めします。

**DLO**
DLO マニフェストオブジェクトは、`X-Object-Manifest` ヘッダーに入力したセグメントオブジェクトのパスを使用して、自動的にセグメントオブジェクトを検索して連結します。

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Manifest: {Segment-Container}/{Segment-Object}/
```

<br/>

##### リクエスト
| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header| String | Y | トークン ID |
| X-Object-Manifest | Header| String | Y | 分割したセグメントオブジェクトをアップロードしたパス、`{Segment-Container}/{Segment-Object}/` |
| Account | URL | String | Y | ストレージアカウント、API エンドポイント設定ダイアログボックスで確認 |
| Container |	URL | String | Y | コンテナ名 |
| Object |	URL | String | Y | 作成するマニフェストオブジェクト名 |
| - | Body| Binary | Y | 空のデータ |

<br/>

**SLO**
SLO マニフェストオブジェクトは、リクエスト本文にセグメントオブジェクトのリストを順番に記述して入力する必要があります。最大 1 万個のセグメントオブジェクトを入力できます。
SLO マニフェストオブジェクトの作成リクエストを行うと、各セグメントオブジェクトが入力されたパスに存在するか、etag 値とオブジェクトのサイズが一致するかを確認します。情報が一致しない場合、マニフェストオブジェクトは作成されません。

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
| X-Auth-Token | Header| String | Y | トークン ID |
| Account | URL | String | Y | ストレージアカウント、API エンドポイント設定ダイアログボックスで確認 |
| Container |	URL | String | Y | コンテナ名 |
| Object |	URL | String | Y | 作成するマニフェストオブジェクト名 |
| multipart-manifest | Query| String | Y | マニフェスト作成時に put を設定 |
| path | Body | String | Y | セグメントオブジェクトのパス |
| etag | Body | String | Y | セグメントオブジェクトの etag |
| size_bytes | Body | Integer | Y | セグメントオブジェクトのサイズ（バイト単位） |

!!! tip "ヒント"
    SLO マニフェストファイルが保持するセグメント情報を照会するには、`multipart-manifest=get` クエリを使用する必要があります。

<br/>

##### レスポンス
レスポンス本文は返しません。リクエストが正しい場合、ステータスコード 201 を返します。

<br/>

<a id="multipart-upload-code-example"></a>
#### コード例
DLO 方式を使用したマルチパートアップロードの例

<details>
<summary>cURL</summary>

```
// 200MB 単位でファイルを分割
$ split -d -b 209715200 large_obj.img large_obj.img.

// 分割されたオブジェクトをアップロード
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/large_obj.img/001 \
-T large_obj.img.00

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/large_obj.img/002 \
-T large_obj.img.01

$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/large_obj.img/003 \
-T large_obj.img.02

// マニフェストオブジェクトをアップロード
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

    // マニフェストオブジェクトのアップロード
    public void uploadManifestObject(String containerName, String objectName) {
        String url = this.getUrl(containerName, objectName);
        String manifestName = containerName + "/" + objectName + "/"; // マニフェスト名の生成

        // ヘッダーの生成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Object-Manifest", manifestName);  // ヘッダーにマニフェストを指定

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
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

        final int defaultChunkSize = 100 * 1024; // 100 KB 単位で分割
        int chunkSize = defaultChunkSize;
        int chunkNo = 0;  // 分割オブジェクトの名前を生成するためのチャンク番号
        int totalBytesRead = 0;

        try {
            // ファイルから InputStream を生成
            InputStream inputStream = new BufferedInputStream(new FileInputStream(objFile));
            while(totalBytesRead < fileSize) {

                // 残りデータサイズを計算
                int remainedBytes = fileSize - totalBytesRead;
                if(remainedBytes < chunkSize) {
                    chunkSize = remainedBytes;
                }

                // バイトバッファにチャンクサイズ分のデータを読み込む
                byte[] chunkBuffer = new byte[chunkSize];
                int bytesRead = inputStream.read(chunkBuffer, 0, chunkSize);

                if(bytesRead > 0) {
                    // バッファのデータを InputStream に変換してアップロード。オブジェクトアップロード例の uploadObject() メソッドを使用
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
    $fd = fopen($filename, 'r');  // ファイルを開く。
    $obj_size = filesize($filename);

    while($total_bytes_read < $obj_size) {
      // 分割するサイズを計算する
      $remained_bytes = $obj_size - $total_bytes_read;
      if ($remained_bytes < $chunk_size) {
        $chunk_size = $remained_bytes;
      }
      $chunk = fread($fd, $chunk_size);
      // パート名を生成する
      $temp_file = sprintf("./multipart-%03d", $chunk_index);
      $req_url = sprintf("%s/%03d", $url, $chunk_index);

      // パートの一時ファイルを生成する
      $part_fd = fopen($temp_file, 'w+');
      fwrite($part_fd, $chunk);
      fseek($part_fd, 0);

      $curl  = curl_init($req_url);
      curl_setopt_array($curl, array(
        CURLOPT_PUT => TRUE,
        CURLOPT_HEADER => TRUE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_INFILE => $part_fd,  // パートファイルのストリームをパラメータとして入力する
        CURLOPT_HTTPHEADER => $req_header
      ));
      $response = curl_exec($curl);
      curl_close($curl);
      printf("$response");

      // 一時ファイルを削除する
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

<br/>

<a id="update-an-object"></a>
### オブジェクトの内容の変更 { #update-an-object }
オブジェクトアップロード API と同じですが、オブジェクトがすでにコンテナに存在する場合、そのオブジェクトの内容が変更されます。

```
PUT   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
Content-Type: {content-type}
```

<a id="update-an-object-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークン ID |
| Content-type | Header | String | Y | オブジェクトのコンテンツタイプ |
| X-Delete-At | Header | Timestamp | N | オブジェクトの有効期限日、Unix 時間 (秒) |
| X-Delete-After | Header | Timestamp | N | オブジェクトの有効時間、Unix 時間 (秒) |
| Account | URL | String | Y | ストレージアカウント。API エンドポイント設定ダイアログボックスで確認できます |
| Container | URL | String | Y | コンテナ名 |
| Object | URL | String | Y | 内容を変更するオブジェクト名 |
| - | Body | Binary | Y | 変更するオブジェクトの内容 |

<a id="update-an-object-response"></a>
#### レスポンス
レスポンス本文は返しません。リクエストが正しい場合、ステータスコード 201 を返します。

<br/>

<a id="query-object-information"></a>
### オブジェクト情報の照会 { #query-object-information }
指定したオブジェクトの情報を照会します。オブジェクト情報は応答ヘッダーで確認できます。

```
HEAD   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

<a id="query-object-information-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Account | URL | String | Y | ストレージアカウント、APIエンドポイント設定ダイアログボックスで確認 |
| Container | URL | String | Y | コンテナ名 |
| Object | URL | String | Y | ダウンロードするオブジェクト名 |

<a id="query-object-information-response"></a>
#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード 200 を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| Content-Type | Header | String | オブジェクトのコンテンツタイプ |
| Content-Length | Header | Integer | オブジェクトのサイズ |
| Etag | Header | String | オブジェクトの ETag 値<br/>オブジェクトの MD5 Hash 値です。<br/>オブジェクトの整合性確認に使用できます。 |
| Last-Modified | Header | String | オブジェクトの最終更新日時 |
| X-Timestamp | Header | Timestamp | オブジェクトの最終更新日時、Unix 時間（秒） |
| X-Delete-At | Header | Timestamp | オブジェクトの有効期限、Unix 時間（秒） |
| X-Object-Worm-Retain-Until | Header | Timestamp | オブジェクトロックの有効期限、Unix 時間（秒） |
| X-Object-Manifest | Header | String | DLO 方式マルチパートオブジェクトのセグメントオブジェクトパス |
| X-Static-Large-Object | Header | Boolean | SLO 方式マルチパートオブジェクトかどうか |
| X-Manifest-Etag | Header | String | SLO 方式マルチパートオブジェクトのマニフェスト ETag 値（MD5） |


<a id="query-object-information-code-example"></a>
#### コード例
<details>
<summary>cURL</summary>

```
$ curl -O -X HEAD -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
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

<br/>

<a id="download-an-object"></a>
### オブジェクトのダウンロード { #download-an-object }
オブジェクトをダウンロードします。

```
GET   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

<a id="download-an-object-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Account | URL | String | Y | ストレージアカウント、APIエンドポイント設定ダイアログで確認 |
| Container | URL | String | Y | コンテナ名 |
| Object | URL | String | Y | ダウンロードするオブジェクト名 |

<a id="download-an-object-response"></a>
#### レスポンス
オブジェクトの内容がストリームとして返されます。リクエストが正しい場合、ステータスコード 200 を返します。

<a id="download-an-object-code-example"></a>
#### コード例
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
        
        // リクエストヘッダーにトークンを追加する RequestCallback
        RequestCallback callback = (request) -> {
            HttpHeaders headers = request.getHeaders();
            headers.add("X-Auth-Token", tokenId);
            headers.setAccept(Collections.singletonList(MediaType.APPLICATION_OCTET_STREAM));
        };
        
        // レスポンスを受け取って保存する Extractor
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
            // オブジェクトのダウンロード
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

<br/>

<a id="copy-an-object"></a>
### オブジェクトのコピー { #copy-an-object }
オブジェクトを別のコンテナにコピーします。元のオブジェクトのすべての属性が一緒にコピーされます。

```
COPY   /v1/{Account}/{SourceContainer}/{SourceObject}
X-Auth-Token: {token-id}
Destination: {TargetContainer}/{TargetObject}
```

```
PUT   /v1/{Account}/{TargetContainer}/{TargetObject}
X-Auth-Token: {token-id}
X-Copy-From: {SourceContainer}/{SourceObject}
```

<a id="copy-an-object-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Destination | Header | String | N | 対象オブジェクトのパス、`{対象コンテナ}/{対象オブジェクト}`<br/>COPY メソッドを使用するときに必要 |
| X-Copy-From | Header | String | N | 元のオブジェクトのパス、`{元のコンテナ}/{元のオブジェクト}`<br/>PUT メソッドを使用するときに必要 |
| X-Fresh-Metadata | Header | Boolean | N | オブジェクトの属性を初期化するかどうか<br/>値が true の場合、元のオブジェクトの属性はコピーされません。<br/>デフォルト値は false です。 |
| X-Object-Meta-{Key} | Header | String | N | 対象オブジェクトのメタデータ |
| X-Delete-At | Header | Timestamp | N | 対象オブジェクトの有効期限、Unix 時間（秒） |
| X-Delete-After | Header | Timestamp | N | 対象オブジェクトの有効時間、Unix 時間（秒） |
| Account | URL | String | Y | ストレージアカウント、API エンドポイント設定ダイアログボックスで確認 |
| Container | URL | String | Y | コンテナ名<br/>COPY メソッド：元のコンテナ<br/>PUT メソッド：対象コンテナ |
| Object | URL | String | Y | オブジェクト名<br/>COPY メソッド：元のオブジェクト<br/>PUT メソッド：対象オブジェクト |
| multipart-manifest | Query | String | N | 値が get の場合、マニフェストオブジェクトのみコピー<br/>省略した場合、セグメントをマージして単一オブジェクトとしてコピーします。<br/>COPY メソッド：クエリパラメータとして追加<br/>PUT メソッド：`X-Copy-From` ヘッダー値に追加 |

<a id="preserve-object-properties"></a>
##### オブジェクト属性の保存
オブジェクトをコピーすると、元のオブジェクトの属性が一緒にコピーされます。保存される属性は次のとおりです。

| 名前 | 説明 |
|---|---|
| X-Delete-At | オブジェクトの有効期限 |
| X-Object-Worm-Retain-Until | オブジェクトロックの有効期限 |
| X-Object-Meta-{key} | ユーザー定義メタデータ |

!!! tip "ヒント"
    オブジェクトをコピーするときに `X-Delete-At` または `X-Object-Meta-{key}` ヘッダーを追加すると、コピーされたオブジェクトの属性を新しい値に設定できます。
    ただし、ロックの有効期限は変更できず、元のオブジェクトの値がそのまま維持されます。

<a id="copy-a-multipart-object"></a>
##### マルチパートオブジェクトのコピー
マルチパートオブジェクトをコピーすると、マニフェストが参照するセグメントが 1 つのオブジェクトにマージされてコピーされます。そのため、5 GB を超えるマルチパートオブジェクトは通常の方法でコピーすることはできません。5 GB を超えるマルチパートオブジェクトをコピーするには、マニフェストオブジェクトのみをコピーする必要があります。リクエスト時に `multipart-manifest=get` パラメータを追加してマニフェストを元として指定できます。

```
COPY   /v1/{Account}/{SourceContainer}/{SourceObject}?multipart-manifest=get
X-Auth-Token: {token-id}
Destination: {TargetContainer}/{TargetObject}
```

```
PUT   /v1/{Account}/{TargetContainer}/{TargetObject}
X-Auth-Token: {token-id}
X-Copy-From: {SourceContainer}/{SourceObject}; multipart-manifest=get
```

!!! tip "ヒント"
    PUT メソッドでマニフェストをコピーするときは、`X-Copy-From` ヘッダー値に `multipart-manifest=get` パラメータをセミコロンで区切って追加する必要があります。

<!-- 改行のためのコメント -->

!!! danger "警告"
    コピーされたマニフェストは元のセグメントパスを参照するため、元のセグメントオブジェクトを削除するとデータにアクセスできなくなります。
    元のセグメントオブジェクトを別のコンテナにコピーした場合は、マニフェストオブジェクトを新たに作成する必要があります。

マニフェストをコピーするときは、マニフェストの属性が一緒にコピーされます。

| 種類 | コピーされる属性 |
|---|---|
| SLO マニフェスト | X-Static-Large-Object, X-Manifest-Etag |
| DLO マニフェスト | X-Object-Manifest |

<a id="copy-an-object-response"></a>
#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければ、ステータスコード 201 を返します。

<a id="copy-an-object-code-example"></a>
#### コード例
<details>
<summary>cURL</summary>

**単一オブジェクトのコピー**
```
// COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/3a45e9.jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/3a45e9.jpg

// PUT method
$ curl -X PUT -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'X-Copy-From: curl_example/3a45e9.jpg' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/copy_con/3a45e9.jpg
```

**マルチパートマニフェストオブジェクトのコピー**
```
// COPY method
$ curl -X COPY -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H 'Destination: copy_con/419da6e.mp4' \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/419da6e.mp4?multipart-manifest=get

// PUT method
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

    // ObjectService クラス ...

    public void copyObject(String srcContainerName, String objectName, String destContainerName) {
        String url = this.getUrl(destContainerName, objectName);
        String srcObject = "/" + srcContainerName + "/" + objectName;

        // ヘッダーを作成します
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add("X-Copy-From", srcObject);    // コピー元オブジェクトを指定します
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // HttpMethod は COPY メソッドをサポートしていないため、PUT メソッドを使用する代替 API を呼び出します。
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

<br/>

<a id="modify-object-metadata"></a>
### オブジェクトメタデータの修正 { #modify-object-metadata }
指定したオブジェクトのメタデータを修正します。

```
POST   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
X-Object-Meta-{Key}: {Value}
```

<a id="modify-object-metadata-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| X-Object-Meta-{Key} | Header | String | N | 変更するメタデータ |
| X-Delete-At | Header | Timestamp | N | オブジェクトの有効期限、Unixタイム（秒） |
| X-Delete-After | Header | Timestamp | N | オブジェクトの有効時間、Unixタイム（秒） |
| X-Object-Worm-Retain-Until | Header | Timestamp | N | オブジェクトロックの有効期限、Unixタイム（秒）<br/>設定された時刻以降にのみ変更可能で、オブジェクトロックコンテナでのみ変更できます |
| Account | URL | String | Y | ストレージアカウント、APIエンドポイント設定ダイアログボックスで確認 |
| Container | URL| String | Y  | コンテナ名 |
| Object | URL| String | Y  | メタデータを修正するオブジェクト名 |

!!! tip "ヒント"
    オブジェクトロックコンテナにアップロードされたオブジェクトには、自動的にロック有効期限が設定されます。
    ロック有効期限が経過していないオブジェクトは、上書きまたは削除することはできません。
    オブジェクトのメタデータは、ロック有効期限前であっても変更できます。

<a id="modify-object-metadata-response"></a>
#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しければ、ステータスコード 202 を返します。

<a id="modify-object-metadata-code-example"></a>
#### コード例
<details>
<summary>cURL</summary>

```
// オブジェクトにメタデータを追加
$ curl -X POST -H 'X-Auth-Token: b587ae461278419da6ecd21a2344c8aa' \
-H "X-Object-Meta-Type: photo" \
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/curl_example/ba6610.jpg

// オブジェクトのヘッダーから追加したメタデータを確認
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

        // メタデータキーの生成
        String metaKey = "X-Object-Meta-" + key;

        // ヘッダーの生成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        headers.add(metaKey, value);    // ヘッダーにメタデータを設定

        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
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
    $req_header[] = 'X-Object-Meta-'.$key.': '.$value;  // ヘッダーにメタデータを追加

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

<br/>

<a id="delete-an-object"></a>
### オブジェクトの削除 { #delete-an-object }
指定したオブジェクトを削除します。

!!! tip "ヒント"
    マルチパートアップロードしたオブジェクトを削除する際は、セグメントデータをすべて削除する必要があります。マニフェストのみを削除すると、セグメントオブジェクトがそのまま残り、課金される可能性があります。

```
DELETE   /v1/{Account}/{Container}/{Object}
X-Auth-Token: {token-id}
```

<a id="delete-an-object-request"></a>
#### リクエスト
リクエスト本文は必要ありません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | トークンID |
| Account | URL | String | Y | ストレージアカウント、APIエンドポイント設定ダイアログボックスで確認 |
| Container | URL| String | Y  | コンテナ名 |
| Object | URL| String | Y  | 削除するオブジェクト名 |

<a id="delete-an-object-response"></a>
#### レスポンス
このリクエストはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード 204 を返します。

<br/>

<a id="delete-an-object-code-example"></a>
#### コード例
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

        // ヘッダーの作成
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Auth-Token", tokenId);
        HttpEntity<String> requestHttpEntity = new HttpEntity<String>(null, headers);

        // API 呼び出し
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

<br/>

<a id="limiting-policy"></a>
## 制限ポリシー { #limiting-policy }

<a id="request-rate-limit"></a>
### リクエストレート制限 { #request-rate-limit }
Object Storage は、システムの安定性を確保するため、ストレージアカウント (account) 単位で書き込みリクエストのレート制限 (rate limit) を適用します。

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>区分</th>
    <th>項目</th>
    <th>説明</th>
  </tr>
  <tr>
    <td>制限条件</td>
    <td>リクエストレート制限</td>
    <td>500 リクエスト/秒</td>
  </tr>
  <tr>
    <td rowspan="5">適用対象</td>
    <td>適用単位</td>
    <td>ストレージアカウント (account) 単位</td>
  </tr>
  <tr>
    <td rowspan="4">適用メソッド</td>
    <td>POST: コンテナ設定の変更、オブジェクトの属性/メタデータの変更</td>
  </tr>
  <tr>
    <td>PUT: コンテナの作成、オブジェクトのアップロード</td>
  </tr>
  <tr>
    <td>DELETE: コンテナ/オブジェクトの削除</td>
  </tr>
  <tr>
    <td>COPY: オブジェクトのコピー</td>
  </tr>
  <tr>
    <td>動作方式</td>
    <td>制限超過時の処理方式</td>
    <td>遅延後に処理、遅延時間が 60 秒を超えた場合は 429 レスポンスを返す</td>
  </tr>
</table>

リクエストレート制限を超えた書き込みリクエストに適用されるポリシーは次のとおりです。

* 超過した書き込みリクエストは即座に拒否されず、遅延処理されます。
* 遅延時間は超過リクエスト量に応じて段階的に増加し、最大 60 秒まで延びる場合があります。
* 遅延時間が 60 秒を超えた場合、リクエストは失敗し、429 Too Many Requests レスポンスを返します。

レスポンスの遅延や失敗を防ぐには、書き込みリクエストがレート制限を超えないように調整してください。


<br/>

<a id="references"></a>
## References { #references }

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)