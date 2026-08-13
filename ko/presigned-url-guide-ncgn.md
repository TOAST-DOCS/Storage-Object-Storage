<!-- pre-align:aligned sig=ba6b9ac2ecbb -->

<a id="storage-object-storage-presigned-url-guide"></a>
## Storage > Object Storage > 서명된 URL 가이드 { #storage-object-storage-presigned-url-guide }

이 문서는 서명된 URL로 NHN Cloud 오브젝트 스토리지의 오브젝트에 한시적인 접근 권한을 부여하는 방법을 설명합니다.

<a id="overview"></a>
## 서명된 URL { #overview }

서명된 URL(presigned URL)은 비밀 키로 미리 서명해 둔 임시 접근 링크입니다. **URL에 접근 대상 오브젝트, 허용할 HTTP 메서드(GET/PUT), 만료 시각**을 포함하고 이를 비밀 키로 서명합니다.

<br>

<a id="swift-tempurl"></a>
### Swift TempURL { #swift-tempurl }

Swift TempURL(Temporary URL)은 오브젝트의 URL에 쿼리 파라미터가 추가된 형태입니다.

```bash
https://api-object-storage.gncloud.go.kr/v1/my_account/container/object
?temp_url_sig=732fcac368abb10c78a4cbe95c3fab7f311584532bf779abd5074e13cbe8b88b
&temp_url_expires=1323479485
&filename=My+Test+File.pdf
```

| 구성 요소 | 필수 | 설명 |
| --- | --- | --- |
| Object URL | Y | 오브젝트의 전체 경로 URL |
| temp_url_sig | Y | 허용된 HTTP 메서드, 만료 일시, 오브젝트의 전체 경로를 비밀 키로 서명한 HMAC 값 |
| temp_url_expires | Y | 만료 일시. UNIX Epoch 타임스탬프 또는 ISO 8601 UTC 타임스탬프로 표현.<br>예: `1390852007` 또는 `2014-01-27T19:46:47Z` |
| filename | N | 기본 파일명을 덮어씀 |
| temp_url_prefix | N | 접두사 단위로 서명할 때 필요 |

<br>

<a id="s3-presigned-url"></a>
### S3 호환 서명된 URL { #s3-presigned-url }

NHN Cloud 오브젝트 스토리지는 S3 호환 API를 제공하며, 이때 생성하는 서명된 URL은 다음과 같은 형태입니다.

```bash
https://{endpoint}/my-container/cat.jpg
?X-Amz-Algorithm=AWS4-HMAC-SHA256
&X-Amz-Credential={your-access-key-id}/20260601/kr1/s3/aws4_request
&X-Amz-Date=20260601T201207Z
&X-Amz-Expires=86400
&X-Amz-SignedHeaders=host
&X-Amz-Signature={signature-value}
```

| 구성 요소 | 필수 | 설명 |
| --- | --- | --- |
| Object URL | Y | 오브젝트의 전체 경로 URL(path-style: `https://{endpoint}/{bucket}/{object}`) |
| X-Amz-Algorithm | Y | AWS Signature 버전과 알고리즘 식별. SigV4에서는 `AWS4-HMAC-SHA256`으로 설정 |
| X-Amz-Credential | Y | Access Key ID와 서명이 유효한 scope(리전·서비스)를 제공. 형식: `{access-key-id}/{date}/{region}/{service}/aws4_request` (서비스는 `s3`, 리전은 `kr1` 등). URL에서 `/`는 `%2F`로 인코딩 |
| X-Amz-Date | Y | 요청 일시. ISO 8601 `yyyyMMddTHHmmssZ` 형식(UTC)으로 표현<br>예: `20260601T223241Z` |
| X-Amz-Expires | Y | 서명된 URL이 유효한 기간(초). 최소 `1`, 최대 `604800`(7일) |
| X-Amz-SignedHeaders | Y | 서명 계산에 사용한 헤더 목록. 최소한 HTTP `host` 헤더를 포함하며, 요청에 추가하는 모든 `x-amz-*` 헤더도 포함 |
| X-Amz-Signature | Y | 요청을 인증하는 HMAC 서명 값. 서버가 계산한 값과 일치해야 하며, 아니면 요청 거부 |

!!! tip "알아두기"
    S3 서명된 URL에서는 접두사 단위 서명을 지원하지 않습니다. 항상 단일 오브젝트와 단일 작업(GET/PUT 등) 단위로 서명합니다.

<br>

<a id="preparation"></a>
## 사전 준비 { #preparation }

서명된 URL을 생성하려면 먼저 서명 대상 오브젝트의 위치를 정하고, 방식별로 필요한 키와 자격 증명을 준비합니다. 서명 대상은 스토리지 엔드포인트와 오브젝트 경로로 정해집니다.

* 스토리지 엔드포인트: 콘솔의 오브젝트 스토리지에서 확인
* 컨테이너 이름/오브젝트 경로: (예시) `my-container/photos/cat.jpg`

Swift API와 S3 API는 오브젝트 경로 형식이 다르므로, 생성하려는 방식에 맞는 경로를 사용해야 합니다.

<br>

<a id="set-tempurl-key"></a>
### Swift TempURL 비밀 키 설정 { #set-tempurl-key }

TempURL은 스토리지 계정 또는 컨테이너에 미리 등록해 둔 비밀 키(Secret Key)로 서명합니다.

* **스토리지 계정 레벨**에서 키를 설정하려면, 스토리지 계정 POST 요청에서 다음 헤더 중 하나 또는 둘 다를 임의의 값으로 설정합니다.
    * `X-Account-Meta-Temp-URL-Key`
    * `X-Account-Meta-Temp-URL-Key-2`
* **컨테이너 레벨**에서 키를 설정하려면, 컨테이너 POST 또는 PUT 요청에서 다음 헤더 중 하나 또는 둘 다를 임의의 값으로 설정합니다.
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

!!! tip "알아두기"
    오브젝트 스토리지는 스토리지 계정당 2개, 컨테이너당 2개의 비밀 키 값을 저장할 수 있습니다.

    요청을 검증할 때 오브젝트 스토리지는 모든 키의 서명을 확인합니다. 각 레벨에서 키를 2개 사용하면, 기존 TempURL을 유지한 채로 키를 교체(rotation)할 수 있습니다.

Swift CLI를 사용하면 다음과 같이 비밀 키를 설정할 수 있습니다.

```bash
swift post -m "Temp-URL-Key:MYKEY"              # 스토리지 계정 단위 설정
swift post my-container -m "Temp-URL-Key:MYKEY" # 컨테이너 단위 설정
```

!!! tip "알아두기"
    Swift CLI를 사용하려면 먼저 인증이 필요합니다. 자세한 내용은 [Swift CLI 환경설정](cli-guide-ncgn/#configuration)을 참고합니다.

<br>

<a id="obtain-s3-credentials"></a>
### S3 API 자격 증명 발급 { #obtain-s3-credentials }

S3 호환 API를 사용하려면 먼저 AWS EC2 형태의 S3 API 자격 증명(Access Key ID + Secret Access Key)을 발급해야 합니다. 자격 증명은 콘솔 또는 API를 사용하여 발급할 수 있습니다. 콘솔을 사용한 자격 증명 발급은 [S3 API 자격 증명](console-guide-ncgn/#s3-api-credentials) 항목을 참고합니다.

```http
POST https://api-identity-infrastructure.gncloud.go.kr/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

`Access Key ID`는 URL의 `X-Amz-Credential`에 노출되고, `Secret Access Key`는 서명 계산에만 쓰이며 URL에 노출되지 않습니다.

<details>
<summary>예시</summary>

```json
{
  "credential": {
    "access": "253a3c7ca27f4731a9c757addfac29ca",
    "tenant_id": "84c9e9a51aea402e95389c08ac562ac5",
    "secret": "be057f235abf45ee8e2ba14edc5fb253",
    "user_id": "84db0c80-3c39-11e7-b29c-005056ac1497",
    "created_at": "2024-10-19T08:24:46.000000Z",
    "accessed_at": "2024-10-19T08:24:46.000000Z"
  }
}
```

</details>

`aws` CLI 또는 SDK로 서명을 생성하려면, 발급한 자격 증명을 로컬에 설정해야 합니다. 자세한 내용은 [Amazon S3 호환 API 가이드](s3-api-guide-ncgn/#aws-command-line-interface-configuration) 설정 항목을 참고합니다.

<br>

<a id="create-presigned-url"></a>
## 서명된 URL 생성 { #create-presigned-url }

사전 준비를 마친 뒤, 서명된 URL을 생성하는 방법을 설명합니다.

<br>

<a id="create-manual-signature"></a>
### 직접 서명 { #create-manual-signature }

다음은 오브젝트 기반 TempURL용 HMAC-SHA256 서명 예시입니다.

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

TempURL은 접두사 기반으로도 생성할 수 있습니다. 접두사로 시작하는 모든 오브젝트에 유효한 서명을 만들 수 있습니다. 다음은 접두사 기반 TempURL용 HMAC-SHA512 서명 예시입니다.

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

HMAC 서명을 생성할 때는 경로를 URL 인코딩하지 않습니다. 다만, 실제 HTTP 요청을 보낼 때는 경로를 URL 인코딩해야 합니다. 두 예시 모두 `MYKEY` 값은 [Swift TempURL 비밀 키 설정](#set-tempurl-key)에서 설정한 키 값 중 하나입니다.

<details>
<summary>Java</summary>

```java
// 오브젝트 기반 (HMAC-SHA256)
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
        for (byte b : raw) sb.append(String.format("%02x", b));   // hex 변환
        String signature = sb.toString();

        System.out.println(signature);
    }
}
```

```java
// 접두사 기반 (HMAC-SHA512)
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
// 오브젝트 기반 (HMAC-SHA256)
$method            = 'GET';
$durationInSeconds = 60 * 60 * 24;
$expires           = time() + $durationInSeconds;
$path              = '/v1/my_account/container/object';
$key               = 'MYKEY';

$hmacBody  = "$method\n$expires\n$path";
$signature = hash_hmac('sha256', $hmacBody, $key);   // 기본 출력이 hex
echo $signature . "\n";
```

```php
<?php
// 접두사 기반 (HMAC-SHA512)
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
### Swift CLI 사용 { #create-swift-tempurl-cli }

Swift CLI의 `tempurl` 명령은 `temp_url_sig`와 `temp_url_expires` 쿼리 파라미터를 자동으로 생성합니다.

```bash
# GET
$ swift tempurl GET 3600 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg "$TEMP_URL_KEY"
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199

# PUT
$ swift tempurl PUT 3600 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg "$TEMP_URL_KEY"
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=b1c4e9f2a8d7035641c2e9d8f4b1a7d063c5e8f9a2b1d4e70f3a8c1d9e2b5f40&temp_url_expires=1772755199
```

TempURL을 생성하려면 이 경로 앞에 오브젝트 스토리지 호스트 이름을 붙입니다.

```bash
https://api-object-storage.gncloud.go.kr/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199
```

<br>

<a id="create-aws-cli"></a>
### AWS CLI 사용 { #create-aws-cli }

`aws` CLI로 서명을 생성하려면, [S3 API 자격 증명 발급](#obtain-s3-credentials)에서 발급한 Access Key와 Secret Key를 로컬에 설정해야 합니다.

```bash
aws s3 presign s3://my-container/cat.jpg \
  --expires-in 3600 \
  --endpoint-url https://api-object-storage.gncloud.go.kr
```

`aws s3 presign`은 GET 전용이므로, PUT(업로드) 서명된 URL은 SDK로 생성해야 합니다.

<br>

<a id="create-aws-sdk"></a>
### AWS SDK 사용 { #create-aws-sdk }

SDK로는 다운로드(GET)와 업로드(PUT)용 서명된 URL을 모두 생성할 수 있습니다.

```python
import boto3
from botocore.client import Config

s3 = boto3.client(
    's3',
    endpoint_url='https://api-object-storage.gncloud.go.kr',
    region_name='kr1',
    aws_access_key_id='253a3c7ca27f4731a9c757addfac29ca',
    aws_secret_access_key='be057f235abf45ee8e2ba14edc5fb253',
    config=Config(signature_version='s3v4',
                  s3={'addressing_style': 'path'}),
)

# 업로드(PUT)용 서명된 URL — GET이면 'get_object'
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
            .region(Region.of("kr1"))
            .endpointOverride(URI.create("https://api-object-storage.gncloud.go.kr"))
            .credentialsProvider(StaticCredentialsProvider.create(
                AwsBasicCredentials.create(
                    "253a3c7ca27f4731a9c757addfac29ca",
                    "be057f235abf45ee8e2ba14edc5fb253")))
            .serviceConfiguration(b -> b.pathStyleAccessEnabled(true))
            .build();

        PutObjectRequest put = PutObjectRequest.builder()
            .bucket("my-container").key("cat.jpg").build();

        PutObjectPresignRequest preq = PutObjectPresignRequest.builder()
            .signatureDuration(Duration.ofHours(1))
            .putObjectRequest(put)                       // GET이면 getObjectRequest
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
    'region'   => 'kr1',
    'endpoint' => 'https://api-object-storage.gncloud.go.kr',
    'use_path_style_endpoint' => true,
    'credentials' => [
        'key'    => '253a3c7ca27f4731a9c757addfac29ca',
        'secret' => 'be057f235abf45ee8e2ba14edc5fb253',
    ],
]);

// 업로드(PUT)용 서명된 URL — GET이면 'GetObject'
$cmd = $s3->getCommand('PutObject', ['Bucket' => 'my-container', 'Key' => 'cat.jpg']);
echo (string) $s3->createPresignedRequest($cmd, '+1 hour')->getUri() . "\n";
```

</details>

<br>

<a id="usage"></a>
## 활용 예시 { #usage }

서명된 URL에 인증 정보가 포함되어 있으므로, 인증 토큰이나 서명 헤더 없이 서명된 URL로 요청을 보낼 수 있습니다. 두 방식(TempURL, SigV4) 모두 사용법은 동일합니다.

<br>

<a id="usage-download"></a>
### 다운로드 { #usage-download }

```bash
# Swift TempURL
curl -O "https://api-object-storage.gncloud.go.kr/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199"
```

```bash
# S3 SigV4
curl -O "https://api-object-storage.gncloud.go.kr/my-container/cat.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=253a3c7ca27f4731a9c757addfac29ca%2F20260601%2Fkr1%2Fs3%2Faws4_request&X-Amz-Date=20260601T201207Z&X-Amz-Expires=86400&X-Amz-SignedHeaders=host&X-Amz-Signature=8c1d9e2b5f4072a3b6c9d0e1f2a3b4c5d6e7f8091a2b3c4d5e6f70819a2b3c4d"
```

<br>

<a id="usage-upload"></a>
### 업로드 { #usage-upload }

```bash
# Swift TempURL
curl -X PUT -T ./cat.jpg \
  "https://api-object-storage.gncloud.go.kr/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=b1c4e9f2a8d7035641c2e9d8f4b1a7d063c5e8f9a2b1d4e70f3a8c1d9e2b5f40&temp_url_expires=1772755199"
```

```bash
# S3 SigV4
curl -X PUT -T ./cat.jpg \
  "https://api-object-storage.gncloud.go.kr/my-container/cat.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=253a3c7ca27f4731a9c757addfac29ca%2F20260601%2Fkr1%2Fs3%2Faws4_request&X-Amz-Date=20260601T201207Z&X-Amz-Expires=86400&X-Amz-SignedHeaders=host&X-Amz-Signature=2b1d4e70f3a8c1d9e2b5f4076a3b8c1d9e2b5f40a1b2c3d4e5f60718293a4b50"
```
