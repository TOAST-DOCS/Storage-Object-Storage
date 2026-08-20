<!-- machine_translated: true -->

{% include-markdown '../_object-storage-vars.md' %}

<!-- pre-align:aligned sig=ba6b9ac2ecbb -->

<a id="storage-object-storage-presigned-url-guide"></a>
## Storage > Object Storage > 署名付き URL ガイド { #storage-object-storage-presigned-url-guide }

このドキュメントでは、署名付き URL を使用して NHN Cloud オブジェクトストレージのオブジェクトに一時的なアクセス権限を付与する方法について説明します。

<a id="overview"></a>
## 署名付き URL { #overview }

署名付き URL（presigned URL）は、秘密鍵であらかじめ署名した一時アクセスリンクです。**URL にはアクセス対象のオブジェクト、許可する HTTP メソッド（GET/PUT）、有効期限**が含まれており、これを秘密鍵で署名します。

<br>

<a id="swift-tempurl"></a>
### Swift TempURL { #swift-tempurl }

Swift TempURL(Temporary URL)は、オブジェクトの URL にクエリパラメータが追加された形式です。

```bash
$[ object_storage_url ]$/v1/my_account/container/object
?temp_url_sig=732fcac368abb10c78a4cbe95c3fab7f311584532bf779abd5074e13cbe8b88b
&temp_url_expires=1323479485
&filename=My+Test+File.pdf
```

| 構成要素 | 必須 | 説明 |
| --- | --- | --- |
| Object URL | Y | オブジェクトの完全なパス URL |
| temp_url_sig | Y | 許可された HTTP メソッド、有効期限、オブジェクトのフルパスを秘密鍵で署名した HMAC 値 |
| temp_url_expires | Y | 有効期限。UNIX Epoch タイムスタンプまたは ISO 8601 UTC タイムスタンプで表現します。<br>例: `1390852007` または `2014-01-27T19:46:47Z` |
| filename | N | デフォルトのファイル名を上書きします |
| temp_url_prefix | N | 接頭辞単位で署名する際に必要 |

<br>

<a id="s3-presigned-url"></a>
### S3互換APIの署名付き URL { #s3-presigned-url }

NHN Cloud オブジェクトストレージは S3互換API を提供しており、この場合に生成される署名付き URL は次のような形式です。

```bash
https://{endpoint}/my-container/cat.jpg
?X-Amz-Algorithm=AWS4-HMAC-SHA256
&X-Amz-Credential={your-access-key-id}/20260601/$[ base_region | lower ]$/s3/aws4_request
&X-Amz-Date=20260601T201207Z
&X-Amz-Expires=86400
&X-Amz-SignedHeaders=host
&X-Amz-Signature={signature-value}
```

| 構成要素 | 必須 | 説明 |
| --- | --- | --- |
| Object URL | Y | オブジェクトの完全パス URL (path-style: `https://{endpoint}/{bucket}/{object}`) |
| X-Amz-Algorithm | Y | AWS Signature のバージョンとアルゴリズムを識別します。SigV4 では `AWS4-HMAC-SHA256` に設定します |
| X-Amz-Credential | Y | Access Key ID と、署名が有効な scope（リージョン・サービス）を指定します。形式: `{access-key-id}/{date}/{region}/{service}/aws4_request`（サービスは `s3`、リージョンは `$[ base_region | lower ]$` など）。URL 内の `/` は `%2F` にエンコードします |
| X-Amz-Date | Y | リクエストの日時。ISO 8601 `yyyyMMddTHHmmssZ` 形式 (UTC) で表現<br>例: `20260601T223241Z` |
| X-Amz-Expires | Y | 署名付き URL が有効な期間（秒）。最小 `1`、最大 `604800`（7 日間） |
| X-Amz-SignedHeaders | Y | 署名の計算に使用したヘッダーのリスト。最低限 HTTP `host` ヘッダーを含み、リクエストに追加するすべての `x-amz-*` ヘッダーも含みます |
| X-Amz-Signature | Y | リクエストを認証する HMAC 署名値。サーバーが計算した値と一致する必要があり、一致しない場合はリクエストが拒否されます |

!!! tip "ヒント"
    S3 署名付き URL では、プレフィックス単位の署名はサポートされていません。常に単一のオブジェクトと単一の操作（GET/PUT など）単位で署名します。

<br>

<a id="preparation"></a>
## 事前準備 { #preparation }

署名付き URL を生成するには、まず署名対象オブジェクトの場所を決定し、方式ごとに必要なキーと認証情報を準備します。署名対象はストレージエンドポイントとオブジェクトパスによって決まります。

* ストレージエンドポイント: コンソールのオブジェクトストレージで確認
* コンテナ名/オブジェクトパス: (例) `my-container/photos/cat.jpg`

Swift API と S3 API ではオブジェクトパスの形式が異なるため、生成する方式に合ったパスを使用する必要があります。

<br>

<a id="set-tempurl-key"></a>
### Swift TempURL 秘密鍵の設定 { #set-tempurl-key }

TempURL は、ストレージアカウントまたはコンテナにあらかじめ登録した秘密鍵 (Secret Key) で署名します。

* **ストレージアカウントレベル**でキーを設定するには、ストレージアカウントへの POST リクエストで次のヘッダーのいずれか一方または両方を任意の値に設定します。
    * `X-Account-Meta-Temp-URL-Key`
    * `X-Account-Meta-Temp-URL-Key-2`
* **コンテナレベル**でキーを設定するには、コンテナへの POST または PUT リクエストで次のヘッダーのいずれか一方または両方を任意の値に設定します。
    * `X-Container-Meta-Temp-URL-Key`
    * `X-Container-Meta-Temp-URL-Key-2`

```http
POST /v1/{account}

X-Auth-Token: {token-id}
X-Account-Meta-Temp-URL-Key: {key}
```

```http
POST /v1/{account}/{container}

X-Auth-Token: {token-id}
X-Container-Meta-Temp-URL-Key: {key}
```

!!! tip "ヒント"
    オブジェクトストレージは、ストレージアカウントあたり 2 個、コンテナあたり 2 個の秘密鍵を保存できます。

リクエストを検証する際、Object Storageはすべてのキーの署名を確認します。各レベルで2つのキーを使用することで、既存のTempURLを維持したままキーをローテーションできます。

Swift CLI を使用すると、次のように秘密鍵を設定できます。

```bash
swift post -m "Temp-URL-Key:MYKEY"              # ストレージアカウント単位の設定
swift post my-container -m "Temp-URL-Key:MYKEY" # コンテナ単位の設定
```

!!! tip "ヒント"
    Swift CLI を使用するには、まず認証が必要です。詳細については、[Swift CLI 環境設定](cli-guide$[ file_suffix ]$/#configuration)を参照してください。

<br>

<a id="obtain-s3-credentials"></a>
### S3 API 認証情報の発行 { #obtain-s3-credentials }

S3互換APIを使用するには、まずAWS EC2形式のS3 API認証情報（Access Key ID + Secret Access Key）を発行する必要があります。認証情報はコンソールまたはAPIを使用して発行できます。コンソールを使用した認証情報の発行については、[S3 API認証情報](console-guide$[ file_suffix ]$/#s3-api-credentials)を参照してください。

```http
POST $[ identity_url ]$/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

`Access Key ID` は URL の `X-Amz-Credential` に公開され、`Secret Access Key` は署名の計算にのみ使用され、URL には公開されません。

<details>
<summary>例</summary>

```json
{
  "credential": {
    "access": "$[ access_key ]$",
    "tenant_id": "84c9e9a51aea402e95389c08ac562ac5",
    "secret": "$[ secret_key ]$",
    "user_id": "84db0c80-3c39-11e7-b29c-005056ac1497",
    "created_at": "2024-10-19T08:24:46.000000Z",
    "accessed_at": "2024-10-19T08:24:46.000000Z"
  }
}
```

</details>

`aws` CLI または SDK で署名を生成するには、発行した認証情報をローカルに設定する必要があります。詳細については、[Amazon S3互換APIガイド](s3-api-guide$[ file_suffix ]$/#aws-command-line-interface-configuration)の設定項目を参照してください。

<br>

<a id="create-presigned-url"></a>
## 署名付き URL の生成 { #create-presigned-url }

事前準備が完了したら、署名付き URL を生成する方法について説明します。

<br>

<a id="create-manual-signature"></a>
### 手動署名 { #create-manual-signature }

以下は、オブジェクトベースの TempURL 用 HMAC-SHA256 署名の例です。

```python
import hmac
from hashlib import sha256
from time import time

method = 'GET'
duration_in_seconds = 60*60*24
expires = int(time() + duration_in_seconds)
path = '/v1/my_account/container/object'
key = 'MYKEY'

hmac_body = '%s\n%s\n%s' % (method, expires, path)
signature = hmac.new(key.encode(), hmac_body.encode(), sha256).hexdigest()
```

TempURL はプレフィックスベースでも生成できます。プレフィックスで始まるすべてのオブジェクトに有効な署名を作成できます。以下は、プレフィックスベースの TempURL 用 HMAC-SHA512 署名の例です。

```python
import hmac
from hashlib import sha512
from time import time

method = 'GET'
duration_in_seconds = 60*60*24
expires = int(time() + duration_in_seconds)
path = 'prefix:/v1/my_account/container/my_prefix'
key = 'MYKEY'

hmac_body = '%s\n%s\n%s' % (method, expires, path)
signature = hmac.new(key.encode(), hmac_body.encode(), sha512).hexdigest()
```

HMAC 署名を生成する際は、パスを URL エンコードしません。ただし、実際に HTTP リクエストを送信する際は、パスを URL エンコードする必要があります。どちらの例でも、`MYKEY` の値は [Swift TempURL 秘密鍵の設定](#set-tempurl-key) で設定したキー値のいずれかです。

<details>
<summary>Java</summary>

```java
// オブジェクトベース (HMAC-SHA256)
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

public class TempUrlSha256 {
    public static void main(String[] args) throws Exception {
        String method = "GET";
        long   durationInSeconds = 60L * 60 * 24;
        long   expires = System.currentTimeMillis() / 1000 + durationInSeconds;
        String path = "/v1/my_account/container/object";
        String key  = "MYKEY";

        String hmacBody = method + "\n" + expires + "\n" + path;

        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(key.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
        byte[] raw = mac.doFinal(hmacBody.getBytes(StandardCharsets.UTF_8));

        StringBuilder sb = new StringBuilder();
        for (byte b : raw) sb.append(String.format("%02x", b));   // hex 変換
        String signature = sb.toString();

        System.out.println(signature);
    }
}
```

```java
// プレフィックスベース (HMAC-SHA512)
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

public class TempUrlSha512Prefix {
    public static void main(String[] args) throws Exception {
        String method = "GET";
        long   durationInSeconds = 60L * 60 * 24;
        long   expires = System.currentTimeMillis() / 1000 + durationInSeconds;
        String path = "prefix:/v1/my_account/container/my_prefix";
        String key  = "MYKEY";

        String hmacBody = method + "\n" + expires + "\n" + path;

        Mac mac = Mac.getInstance("HmacSHA512");
        mac.init(new SecretKeySpec(key.getBytes(StandardCharsets.UTF_8), "HmacSHA512"));
        byte[] raw = mac.doFinal(hmacBody.getBytes(StandardCharsets.UTF_8));

        StringBuilder sb = new StringBuilder();
        for (byte b : raw) sb.append(String.format("%02x", b));
        String signature = sb.toString();

        System.out.println(signature);
    }
}
```

</details>

<details>
<summary>PHP</summary>

```php
<?php
// オブジェクトベース (HMAC-SHA256)
$method            = 'GET';
$durationInSeconds = 60 * 60 * 24;
$expires           = time() + $durationInSeconds;
$path              = '/v1/my_account/container/object';
$key               = 'MYKEY';

$hmacBody  = "$method\n$expires\n$path";
$signature = hash_hmac('sha256', $hmacBody, $key);   // デフォルト出力は hex
echo $signature . "\n";
```

```php
<?php
// プレフィックスベース (HMAC-SHA512)
$method            = 'GET';
$durationInSeconds = 60 * 60 * 24;
$expires           = time() + $durationInSeconds;
$path              = 'prefix:/v1/my_account/container/my_prefix';
$key               = 'MYKEY';

$hmacBody  = "$method\n$expires\n$path";
$signature = hash_hmac('sha512', $hmacBody, $key);
echo $signature . "\n";
```

</details>

<br>

<a id="create-swift-tempurl-cli"></a>
### Swift CLI の使用 { #create-swift-tempurl-cli }

Swift CLI の `tempurl` コマンドは、`temp_url_sig` と `temp_url_expires` クエリパラメータを自動的に生成します。

```bash
# GET
$ swift tempurl GET 3600 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg "$TEMP_URL_KEY"
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199

# PUT
$ swift tempurl PUT 3600 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg "$TEMP_URL_KEY"
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=b1c4e9f2a8d7035641c2e9d8f4b1a7d063c5e8f9a2b1d4e70f3a8c1d9e2b5f40&temp_url_expires=1772755199
```

TempURL を作成するには、このパスの前にオブジェクトストレージのホスト名を付加します。

```bash
$[ object_storage_url ]$/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199
```

<br>

<a id="create-aws-cli"></a>
### AWS CLI の使用 { #create-aws-cli }

`aws` CLI で署名を生成するには、[S3 API 認証情報の発行](#obtain-s3-credentials)で発行した Access Key と Secret Key をローカルに設定する必要があります。

```bash
aws s3 presign s3://my-container/cat.jpg \
  --expires-in 3600 \
  --endpoint-url $[ object_storage_url ]$
```

`aws s3 presign` は GET 専用のため、PUT（アップロード）用の署名付き URL は SDK で生成する必要があります。

<br>

<a id="create-aws-sdk"></a>
### AWS SDK の使用 { #create-aws-sdk }

SDK では、ダウンロード（GET）とアップロード（PUT）用の署名付き URL をどちらも生成できます。

```python
import boto3
from botocore.client import Config

s3 = boto3.client(
    's3',
    endpoint_url='$[ object_storage_url ]$',
    region_name='$[ base_region | lower ]$',
    aws_access_key_id='$[ access_key ]$',
    aws_secret_access_key='$[ secret_key ]$',
    config=Config(signature_version='s3v4',
                  s3={'addressing_style': 'path'}),
)

# アップロード(PUT)用署名付き URL — GET の場合は 'get_object'
put_url = s3.generate_presigned_url(
    'put_object',
    Params={'Bucket': 'my-container', 'Key': 'cat.jpg'},
    ExpiresIn=3600,
)
print(put_url)
```

<details>
<summary>Java</summary>

```java
import software.amazon.awssdk.auth.credentials.*;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.model.PutObjectRequest;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;
import software.amazon.awssdk.services.s3.presigner.model.PutObjectPresignRequest;
import java.net.URI;
import java.time.Duration;

public class PresignPut {
    public static void main(String[] args) {
        S3Presigner presigner = S3Presigner.builder()
            .region(Region.of("$[ base_region | lower ]$"))
            .endpointOverride(URI.create("$[ object_storage_url ]$"))
            .credentialsProvider(StaticCredentialsProvider.create(
                AwsBasicCredentials.create(
                    "$[ access_key ]$",
                    "$[ secret_key ]$")))
            .serviceConfiguration(b -> b.pathStyleAccessEnabled(true))
            .build();

        PutObjectRequest put = PutObjectRequest.builder()
            .bucket("my-container").key("cat.jpg").build();

        PutObjectPresignRequest preq = PutObjectPresignRequest.builder()
            .signatureDuration(Duration.ofHours(1))
            .putObjectRequest(put)                       // GETの場合は getObjectRequest
            .build();

        System.out.println(presigner.presignPutObject(preq).url());  // presignGetObject ↔ presignPutObject
        presigner.close();
    }
}
```

</details>

<details>
<summary>PHP</summary>

```php
<?php
require 'vendor/autoload.php';
use Aws\S3\S3Client;

$s3 = new S3Client([
    'version'  => 'latest',
    'region'   => '$[ base_region | lower ]$',
    'endpoint' => '$[ object_storage_url ]$',
    'use_path_style_endpoint' => true,
    'credentials' => [
        'key'    => '$[ access_key ]$',
        'secret' => '$[ secret_key ]$',
    ],
]);

// アップロード(PUT)用の署名付き URL — GET の場合は 'GetObject'
$cmd = $s3->getCommand('PutObject', ['Bucket' => 'my-container', 'Key' => 'cat.jpg']);
echo (string) $s3->createPresignedRequest($cmd, '+1 hour')->getUri() . "\n";
```

</details>

<br>

<a id="usage"></a>
## 活用例 { #usage }

署名付き URL には認証情報が含まれているため、認証トークンや署名ヘッダーなしで署名付き URL にリクエストを送信できます。どちらの方式（TempURL、SigV4）も使用方法は同じです。

<br>

<a id="usage-download"></a>
### ダウンロード { #usage-download }

```bash
# Swift TempURL
curl -O "$[ object_storage_url ]$/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199"
```

```bash
# S3 SigV4
curl -O "$[ object_storage_url ]$/my-container/cat.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=$[ access_key ]$%2F20260601%2F$[ base_region | lower ]$%2Fs3%2Faws4_request&X-Amz-Date=20260601T201207Z&X-Amz-Expires=86400&X-Amz-SignedHeaders=host&X-Amz-Signature=8c1d9e2b5f4072a3b6c9d0e1f2a3b4c5d6e7f8091a2b3c4d5e6f70819a2b3c4d"
```

<br>

<a id="usage-upload"></a>
### アップロード { #usage-upload }

```bash
# Swift TempURL
curl -X PUT -T ./cat.jpg \
  "$[ object_storage_url ]$/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=b1c4e9f2a8d7035641c2e9d8f4b1a7d063c5e8f9a2b1d4e70f3a8c1d9e2b5f40&temp_url_expires=1772755199"
```

```bash
# S3 SigV4
curl -X PUT -T ./cat.jpg \
  "$[ object_storage_url ]$/my-container/cat.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=$[ access_key ]$%2F20260601%2F$[ base_region | lower ]$%2Fs3%2Faws4_request&X-Amz-Date=20260601T201207Z&X-Amz-Expires=86400&X-Amz-SignedHeaders=host&X-Amz-Signature=2b1d4e70f3a8c1d9e2b5f4076a3b8c1d9e2b5f40a1b2c3d4e5f60718293a4b50"
```