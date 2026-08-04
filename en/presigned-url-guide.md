<!-- pre-align:aligned sig=ba6b9ac2ecbb -->

<a id="storage-object-storage-presigned-url-guide"></a>

## Storage > Object Storage > Presigned URL Guide { #storage-object-storage-presigned-url-guide }

This document describes how to grant temporary access to objects in NHN Cloud Object Storage using presigned URLs.

<a id="overview"></a>

## Presigned URL { #overview }

A presigned URL is a temporary access link that is pre-signed with a secret key. The URL contains **the target object, the allowed HTTP method (GET/PUT), and an expiration time**, all signed with a secret key.

<br>

<a id="swift-tempurl"></a>
### Swift TempURL { #swift-tempurl }

A Swift Temporary URL is a URL for an object with additional query parameters appended.

```bash
https://kr1-api-object-storage.nhncloudservice.com/v1/my_account/container/object
?temp_url_sig=732fcac368abb10c78a4cbe95c3fab7f311584532bf779abd5074e13cbe8b88b
&temp_url_expires=1323479485
&filename=My+Test+File.pdf
```

| Component | Required | Description |
| --- | --- | --- |
| Object URL | Y        | Full path URL of the object |
| temp_url_sig | Y        | HMAC value signed with a secret key, covering the allowed HTTP method, expiration time, and the full path of the object |
| temp_url_expires | Y        | Expiration time. Expressed as a UNIX Epoch timestamp or an ISO 8601 UTC timestamp.<br>Example: `1390852007` or `2014-01-27T19:46:47Z` |
| filename | N        | Overrides the default filename |
| temp_url_prefix | N        | Required when signing at the prefix level |

<br>

<a id="s3-presigned-url"></a>
### S3-Compatible Presigned URL { #s3-presigned-url }

NHN Cloud Object Storage provides an S3-compatible API. Presigned URLs generated through this API take the following form.

```bash
https://{endpoint}/my-container/cat.jpg
?X-Amz-Algorithm=AWS4-HMAC-SHA256
&X-Amz-Credential={your-access-key-id}/20260601/kr1/s3/aws4_request
&X-Amz-Date=20260601T201207Z
&X-Amz-Expires=86400
&X-Amz-SignedHeaders=host
&X-Amz-Signature={signature-value}
```

| Component | Required | Description                                                                                                                       |
| --- | --- |--------------------------------------------------------------------------------------------------------------------------|
| Object URL | Y        | Full path URL of the object (path-style: `https://{endpoint}/{bucket}/{object}`)                                                      |
| X-Amz-Algorithm | Y        | Identifies the AWS Signature version and algorithm. Set to `AWS4-HMAC-SHA256` for SigV4.                                                                |
| X-Amz-Credential | Y        | Provides the Access Key ID and the scope (region and service) in which the signature is valid. Format: `{access-key-id}/{date}/{region}/{service}/aws4_request` (service is `s3`; region is `kr1`, etc.). In URLs, `/` is encoded as `%2F`. |
| X-Amz-Date | Y        | Request date and time. Expressed in ISO 8601 `yyyyMMddTHHmmssZ` format (UTC).<br>Example: `20260601T223241Z`                                                 |
| X-Amz-Expires | Y        | Validity period of the presigned URL, in seconds. Minimum `1`, maximum `604800` (7 days).                                                                              |
| X-Amz-SignedHeaders | Y        | List of headers used in the signature calculation. Must include at least the HTTP `host` header, as well as all `x-amz-*` headers added to the request.                                                 |
| X-Amz-Signature | Y        | HMAC signature value that authenticates the request. Must match the value calculated by the server; otherwise, the request is rejected.                                                                        |

!!! tip "Note"
    S3 presigned URLs do not support prefix-level signing. They always sign for a single object and a single operation (GET, PUT, etc.).

<br>

<a id="preparation"></a>

## Preparation { #preparation }

To create a signed URL, first determine the location of the object to be signed, and then prepare the keys and credentials required for each method. The signing target is defined by the storage endpoint and the object path.

* Storage endpoint: Check in Object Storage in the console
* Container name/object path: (Example) `my-container/photos/cat.jpg`

The Swift API and S3 API use different object path formats, so you must use the path that matches the method you want to use.

<br>

<a id="set-tempurl-key"></a>
### Set the Swift TempURL Secret Key { #set-tempurl-key }

TempURL signs requests using a secret key registered in advance on the storage account or container.

* To set a key at the **storage account level**, set one or both of the following headers to an arbitrary value in a storage account POST request:
    * `X-Account-Meta-Temp-URL-Key`
    * `X-Account-Meta-Temp-URL-Key-2`
* To set a key at the **container level**, set one or both of the following headers to an arbitrary value in a container POST or PUT request:
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

!!! tip "Note"
    Object Storage can store up to two secret key values per storage account and two per container.

    When validating a request, Object Storage checks the signatures of all keys. Using two keys at each level allows you to rotate keys without invalidating existing temporary URLs.

You can set a secret key using the Swift CLI as follows:

```bash
swift post -m "Temp-URL-Key:MYKEY"              # Account-level settings
swift post my-container -m "Temp-URL-Key:MYKEY" # Container-level settings
```

!!! tip "Note"
    Authentication is required before using the Swift CLI. For more information, see [Swift CLI configuration](cli-guide/#configuration).

<br>

<a id="obtain-s3-credentials"></a>
### Obtain S3 API Credentials { #obtain-s3-credentials }

To use the S3-compatible API, you must first obtain S3 API credentials in the AWS EC2 format (Access Key ID + Secret Access Key). Credentials can be issued using the web console or API. To obtain credentials using the web console, refer to the [S3 API Credentials](console-guide/#s3-api-credentials) section.

```http
POST https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

The `Access Key ID` is exposed in the `X-Amz-Credential` parameter of the URL, while the `Secret Access Key` is used only for signature calculation and is not exposed in the URL.

<details>
<summary>Example</summary>

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

To generate a signature using the `aws` CLI or SDK, you must configure the credentials you obtained on your local machine. For more information, see the configuration section of the [Amazon S3-compatible API Guide](s3-api-guide/#aws-command-line-interface-configuration).

<br>

<a id="create-presigned-url"></a>

## Create a Presigned URL { #create-presigned-url }

After completing the prerequisites, this section explains how to create a presigned URL.

<br>

<a id="create-manual-signature"></a>
### Create a Signature Manually { #create-manual-signature }

The following is an example of an HMAC-SHA256 signature for an object-based Temporary URL.

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

You can also create a Temporary URL based on a prefix. This allows you to create a signature that is valid for all objects that start with the prefix. The following is an example of an HMAC-SHA512 signature for a prefix-based Temporary URL.

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

When generating an HMAC signature, do not URL-encode the path. However, when sending an actual HTTP request, you must URL-encode the path. In both examples, the `MYKEY` value is one of the key values that you set in [Set a Swift TempURL Secret Key](#set-tempurl-key).

<details>
<summary>Java</summary>

```java
// Object-based (HMAC-SHA256)
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
        for (byte b : raw) sb.append(String.format("%02x", b));   // hex conversion
        String signature = sb.toString();

        System.out.println(signature);
    }
}
```

```java
// Prefix-based (HMAC-SHA512)
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
// Object-based (HMAC-SHA256)
$method            = 'GET';
$durationInSeconds = 60 * 60 * 24;
$expires           = time() + $durationInSeconds;
$path              = '/v1/my_account/container/object';
$key               = 'MYKEY';

$hmacBody  = "$method\n$expires\n$path";
$signature = hash_hmac('sha256', $hmacBody, $key);   // Default output is hex
echo $signature . "\n";
```

```php
<?php
// Prefix-based (HMAC-SHA512)
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
### Use the Swift CLI { #create-swift-tempurl-cli }

The `tempurl` command of the Swift CLI automatically generates the `temp_url_sig` and `temp_url_expires` query parameters.

```bash
# GET
$ swift tempurl GET 3600 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg "$TEMP_URL_KEY"
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199

# PUT
$ swift tempurl PUT 3600 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg "$TEMP_URL_KEY"
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=b1c4e9f2a8d7035641c2e9d8f4b1a7d063c5e8f9a2b1d4e70f3a8c1d9e2b5f40&temp_url_expires=1772755199
```

To create a Temporary URL, prepend the Object Storage hostname to this path, as follows:

```bash
https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199
```

<br>

<a id="create-aws-cli"></a>
### Use the AWS CLI { #create-aws-cli }

To generate a signature with the `aws` CLI, you must configure the access key and secret key issued in [Obtain S3 API Credentials](#obtain-s3-credentials) on your local environment.

```bash
aws s3 presign s3://my-container/cat.jpg \
  --expires-in 3600 \
  --endpoint-url https://kr1-api-object-storage.nhncloudservice.com
```

Because `aws s3 presign` supports GET only, you must use an SDK to create a presigned URL for PUT (upload).

<br>

<a id="create-aws-sdk"></a>
### Use the AWS SDK { #create-aws-sdk }

With the SDK, you can create presigned URLs for both download (GET) and upload (PUT).

```python
import boto3
from botocore.client import Config

s3 = boto3.client(
    's3',
    endpoint_url='https://kr1-api-object-storage.nhncloudservice.com',
    region_name='kr1',
    aws_access_key_id='253a3c7ca27f4731a9c757addfac29ca',
    aws_secret_access_key='be057f235abf45ee8e2ba14edc5fb253',
    config=Config(signature_version='s3v4',
                  s3={'addressing_style': 'path'}),
)

# Signed URL for upload (PUT) — use 'get_object' for GET
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
            .endpointOverride(URI.create("https://kr1-api-object-storage.nhncloudservice.com"))
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
            .putObjectRequest(put)                       // For GET, use getObjectRequest
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
    'endpoint' => 'https://kr1-api-object-storage.nhncloudservice.com',
    'use_path_style_endpoint' => true,
    'credentials' => [
        'key'    => '253a3c7ca27f4731a9c757addfac29ca',
        'secret' => 'be057f235abf45ee8e2ba14edc5fb253',
    ],
]);

// Signed URL for upload (PUT) — use 'GetObject' for GET
$cmd = $s3->getCommand('PutObject', ['Bucket' => 'my-container', 'Key' => 'cat.jpg']);
echo (string) $s3->createPresignedRequest($cmd, '+1 hour')->getUri() . "\n";
```

</details>

<br>

<a id="usage"></a>

## Use Cases { #usage }

Because a presigned URL contains authentication information, you can send requests using the presigned URL without an authentication token or signature header. Both methods (TempURL and SigV4) are used in the same way.

<br>

<a id="usage-download"></a>
### Download { #usage-download }

```bash
# Swift TempURL
curl -O "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e4790a1b2c3d4e5f60718293a4b5&temp_url_expires=1772755199"
```

```bash
# S3 SigV4
curl -O "https://kr1-api-object-storage.nhncloudservice.com/my-container/cat.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=253a3c7ca27f4731a9c757addfac29ca%2F20260601%2Fkr1%2Fs3%2Faws4_request&X-Amz-Date=20260601T201207Z&X-Amz-Expires=86400&X-Amz-SignedHeaders=host&X-Amz-Signature=8c1d9e2b5f4072a3b6c9d0e1f2a3b4c5d6e7f8091a2b3c4d5e6f70819a2b3c4d"
```

<br>

<a id="usage-upload"></a>
### Upload { #usage-upload }

```bash
# Swift TempURL
curl -X PUT -T ./cat.jpg \
  "https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/my-container/cat.jpg?temp_url_sig=b1c4e9f2a8d7035641c2e9d8f4b1a7d063c5e8f9a2b1d4e70f3a8c1d9e2b5f40&temp_url_expires=1772755199"
```

```bash
# S3 SigV4
curl -X PUT -T ./cat.jpg \
  "https://kr1-api-object-storage.nhncloudservice.com/my-container/cat.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=253a3c7ca27f4731a9c757addfac29ca%2F20260601%2Fkr1%2Fs3%2Faws4_request&X-Amz-Date=20260601T201207Z&X-Amz-Expires=86400&X-Amz-SignedHeaders=host&X-Amz-Signature=2b1d4e70f3a8c1d9e2b5f4076a3b8c1d9e2b5f40a1b2c3d4e5f60718293a4b50"
```