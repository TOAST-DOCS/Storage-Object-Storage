<a id="storage-object-storage-amazon-s3-compatible-api-guide"></a>

## Storage > Object Storage > Amazon S3-compatible API Guide { #storage-object-storage-amazon-s3-compatible-api-guide }
NHN Cloud Object Storage provides an API compatible with the S3 API of AWS object storage. Therefore, you can use applications developed to use the Amazon S3 API as is, with only a few configuration changes.

The following Amazon S3 compatible API is provided.

| S3 API 메서드 | 용도 |
| --- | --- |
| PUT Bucket | Create bucket |
| HEAD Bucket | Query bucket information |
| DELETE Bucket | Delete bucket |
| PUT Bucket Object Lock | Create locked bucket |
| PUT Object Lock Configuration | Set locked bucket retention period |
| GET Object Lock Configuration | Get locked bucket retention period |
| PUT Bucket ACL | Set bucket ACL |
| GET Bucket ACL | Get bucket ACL |
| GET Bucket Location | Get region with bucket |
| GET Bucket List Objects | List bucket objects |
| GET Object | Download object |
| HEAD Object | Query object information |
| PUT Object | Upload object |
| PUT Object Copy | Copy object |
| DELETE Object | Delete object |
| Initiate Multipart Upload | Initialize multipart upload |
| Upload Part | Upload part |
| Upload Part Copy | Copy part |
| Complete Multipart Upload | Complete Multipart Upload |
| Abort Multipart Upload | Stop Multipart Upload |
| List Parts | List multipart objects |
| List Multipart Uploads | List multipart objects under uploading |
| DELETE Multiple Objects | Delete two or more objects |

This document describes only the basic usage of API. To use advanced features, it is recommended that you see [Amazon S3 API Guide](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/API/API_Operations_Amazon_Simple_Storage_Service.html) or use [AWS SDK](https://aws.amazon.com/ko/tools).

<a id="s3-api-credential"></a>

## S3 API Credentials { #s3-api-credential }

<a id="obtain-s3-api-credentials"></a>
### Obtain S3 API Credentials { #obtain-s3-api-credentials }
To use the Amazon S3-compatible API, you must first obtain S3 API credentials in the form of AWS EC2. Credentials can be issued using the web console or API. To obtain credentials using the web console, refer to [S3 API Credentials](console-guide/#s3-api-credentials).

To obtain credentials using the API, you need an authentication token. For information on issuing an authentication token, refer to the [Object Storage API Guide](api-guide/#auth).

```
POST    https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

<a id="obtain-s3-api-credentials-request"></a>
#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Issued token ID |
| api-user-id | URL | String | Y | API User ID; can be found in the API endpoint configuration dialog box |
| tenant_id | Body | String | Y | Tenant ID; can be found in the API endpoint configuration dialog box |

!!! tip "Note"
    `{api-user-id}` can be found in the **API User ID** field in the API endpoint configuration dialog box in the console, or in the **access.user.id** field in the response body of the authentication token issuance API.
    To use the authentication token issuance API, refer to the [Authentication and Authorization](api-guide/#auth) section of the API guide.

    S3 API credentials have no expiration date, and up to 3 credentials can be issued per project for each user.

<!-- This comment is for line breaks and must be included. -->

!!! danger "Caution"
    If the S3 API credentials key is leaked, anyone can access the object using the leaked key. If the key is leaked, it is recommended to delete the leaked credentials and obtain a new one.

    If the user account that obtained the S3 API credentials loses access to the project or is deleted by leaving NHN Cloud, the credentials expire immediately and cannot be used.

<details>
<summary>Example</summary>

```json
{
  "tenant_id": "84c9e9a51aea402e95389c08ac562ac5"
}
```

</details>

<a id="obtain-s3-api-credentials-response"></a>
#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| access | Body | String | S3 API credentials access key |
| secret | Body | String | S3 API credentials secret key |
| user_id | Body | String | API User ID |
| tenant_id | Body | String | Tenant ID |
| created_at | Body | String | S3 API credentials creation time |
| accessed_at | Body | String | S3 API credentials last access time |

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

<a id="get-s3-api-credentials"></a>
### Get S3 API Credentials { #get-s3-api-credentials }
Retrieves the issued S3 API credentials.

**[Method, URL]**

```
GET   https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2

X-Auth-Token: {token-id}
```

<a id="get-s3-api-credentials-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Issued token ID |
| user-id | URL | String | Y | User ID; included in the authentication token |

<a id="get-s3-api-credentials-response"></a>
#### Response

| Name | Type | Format | Description |
|---|---|---|---|
| access | Body | String | S3 API credentials access key |
| secret | Body | String | S3 API credentials secret key |
| user_id | Body | String | API User ID |
| tenant_id | Body | String | Tenant ID |
| created_at | Body | String | S3 API credentials creation time |
| accessed_at | Body | String | S3 API credentials last access time |

<details>
<summary>Example</summary>

```json
{
  "credentials": [
    {
      "access": "253a3c7ca27f4731a9c757addfac29ca",
      "tenant_id": "84c9e9a51aea402e95389c08ac562ac5",
      "secret": "be057f235abf45ee8e2ba14edc5fb253",
      "user_id": "84db0c80-3c39-11e7-b29c-005056ac1497",
      "created_at": "2024-10-19T08:24:46.000000Z",
      "accessed_at": "2024-10-19T08:30:42.000000Z"
    }
  ]
}
```

</details>

<a id="delete-s3-api-credentials"></a>
### Delete S3 API Credentials { #delete-s3-api-credentials }
Deletes the issued S3 API credentials.

**[Method, URL]**

```
DELETE   https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```

<a id="delete-s3-api-credentials-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | Issued token ID |
| user-id | URL | String | Y | User ID; included in the authentication token |
| access | URL | String | Y | S3 API credentials access key |

<a id="delete-s3-api-credentials-response"></a>
#### Response
This API does not return a response body. When the request is appropriate, it returns status code 204.

<a id="create-signature"></a>

## Create Signature { #create-signature }
To use S3 API, you must create a signature using credentials. For information on how to create a signature, refer to the [AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) documentation.

The information required to create a signature is as follows:

| Name | Value |
|---|---|
| Algorithm | AWS4-HMAC-SHA256 |
| Signature time | YYYYMMDDThhmmssZ format |
| Service name | s3 |
| Region name | KR1 - Korea (Pangyo) region<br>KR2 - Korea (Pyeongchon) region<br>KR3 - Korea (Gwangju) region<br>JP1 - Japan (Tokyo) region |
| Secret key | S3 API credentials secret key |

The `x-amz-content-sha256` header is required when generating an AWS signature V4 signature. This header is included in the Canonical Request and used in the signature calculation, and the payload processing method is determined by the header value. The available values are as follows:

| x-amz-content-sha256 value | Description |
|---|---|
| `<payload hash>` | Default method that uses the SHA-256 hash value of the entire request payload |
| `UNSIGNED-PAYLOAD` | Omits payload signing |
| `STREAMING-AWS4-HMAC-SHA256-PAYLOAD` | AWS Chunked Upload method (signature included in each chunk) |
| `STREAMING-UNSIGNED-PAYLOAD-TRAILER` | AWS Chunked Upload method (uses trailer header without chunk signature) |
| `STREAMING-AWS4-HMAC-SHA256-PAYLOAD-TRAILER` | AWS Chunked Upload method (signature included in each chunk + trailer header used) |

!!! tip "Note"
    For more information, refer to the [Authenticating Requests: Using the Authorization Header (AWS Signature Version 4)](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-auth-using-authorization-header.html) documentation.

If the `x-amz-content-sha256` value is `STREAMING-UNSIGNED-PAYLOAD-TRAILER` or `STREAMING-AWS4-HMAC-SHA256-PAYLOAD-TRAILER`, you must declare the checksum algorithm to be sent in the trailer using the `x-amz-trailer` request header. The supported algorithms are as follows:

| x-amz-trailer value | Algorithm |
|---|---|
| `x-amz-checksum-crc32` | CRC-32 |
| `x-amz-checksum-crc32c` | CRC-32C |
| `x-amz-checksum-crc64nvme` | CRC-64/NVME |
| `x-amz-checksum-sha1` | SHA-1 |
| `x-amz-checksum-sha256` | SHA-256 |

!!! tip "Note"
    For more information on signature calculation using trailer headers (chunked uploads), refer to the [Signature calculations for trailing headers(chunked uploads)](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-streaming-trailers.html) documentation.

<a id="bucket"></a>

## Bucket { #bucket }

<a id="create-bucket"></a>
### Create Bucket { #create-bucket }
Creates a bucket. Bucket names must follow Amazon S3's bucket naming rules:

* Bucket names must be between 3 and 63 characters long.
* Bucket names can consist only of lowercase letters, numbers, dots (.), and hyphens (-).
* Bucket names must begin and end with a letter or number.
* Bucket names must not be formatted as an IP address (for example, 192.168.5.4).
* Bucket names must not start with xn--.

For more information, see the [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) documentation.

```
PUT /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

!!! tip "Note"
    If a container name made via web console or object storage API violates any bucket naming rules, it cannot be accessed with S3 compatible API.

<a id="create-bucket-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Consists of S3 API credentials access key and signature |

<a id="create-bucket-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 200.

| Name | Type | Format | Description |
|---|---|---|---|
| Location | Header | String | Path of the created bucket |

<a id="list-buckets"></a>
### List Buckets { #list-buckets }
Retrieves a list of buckets.

```
GET /

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="list-buckets-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Consists of S3 API credentials access key and signature |

<a id="list-buckets-response"></a>
#### Response
For a valid request, returns status code 200 and a bucket list in XML format.

<details>
<summary>Example</summary>

```xml
<?xml version='1.0' encoding='UTF-8'?>
<ListAllMyBucketsResult
	xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
	<Owner>
		<ID>user:panther</ID>
		<DisplayName>user:panther</DisplayName>
	</Owner>
	<Buckets>
		<Bucket>
			<Name>log</Name>
			<CreationDate>2009-02-03T16:45:09.000Z</CreationDate>
		</Bucket>
		<Bucket>
			<Name>snapshot</Name>
			<CreationDate>2009-02-03T16:45:09.000Z</CreationDate>
		</Bucket>
	</Buckets>
</ListAllMyBucketsResult>
```

</details>

<a id="get-bucket"></a>
### Get Bucket { #get-bucket }
Retrieves information about the specified bucket and a list of objects stored in it.

```
GET /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

!!! tip "Note"
    If a bucket name made via web console or object storage API violates any bucket naming rules, it cannot be accessed with S3 compatible API.

<a id="get-bucket-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Consists of S3 API credentials access key and signature |

<a id="get-bucket-response"></a>
#### Response
For a valid request, returns status code 200 and a list of objects in XML format.

<details>
<summary>Example</summary>

```xml
<?xml version='1.0' encoding='UTF-8'?>
<ListBucketResult
	xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
	<Name>snapshot</Name>
	<Prefix/>
	<Marker/>
	<MaxKeys>1000</MaxKeys>
	<IsTruncated>false</IsTruncated>
	<Contents>
		<Key>cheetah</Key>
		<LastModified>2023-02-01T04:49:52.995Z</LastModified>
		<ETag>"7d793037a0760186574b0282f2f435e7"</ETag>
		<Size>5</Size>
		<Owner>
			<ID>user:panther</ID>
			<DisplayName>user:panther</DisplayName>
		</Owner>
		<StorageClass>STANDARD</StorageClass>
	</Contents>
	<Contents>
		<Key>leopard</Key>
		<LastModified>2023-02-01T04:49:52.685Z</LastModified>
		<ETag>"5d41402abc4b2a76b9719d911017c592"</ETag>
		<Size>5</Size>
		<Owner>
			<ID>user:panther</ID>
			<DisplayName>user:panther</DisplayName>
		</Owner>
		<StorageClass>STANDARD</StorageClass>
	</Contents>
</ListBucketResult>
```

</details>

<a id="delete-bucket"></a>
### Delete Bucket { #delete-bucket }
Deletes the specified bucket. The bucket to be deleted must be empty.

```
DELETE /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="delete-bucket-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Consists of S3 API credentials access key and signature |

<a id="delete-bucket-response"></a>
#### Response
This API does not return request body. When the request is appropriate, return status code 204.

<a id="create-lock-bucket"></a>
### Create Lock Bucket { #create-lock-bucket }
Creates a bucket with object lock enabled. Set the `x-amz-bucket-object-lock-enabled` header to `true` when creating the bucket. The default retention period is set to 0 days.

```
PUT /{bucket}

x-amz-bucket-object-lock-enabled: true
Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="create-lock-bucket-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| x-amz-bucket-object-lock-enabled | Header | Boolean | Y | Whether to enable object lock (`true`) |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Consists of S3 API credentials access key and signature |

<a id="create-lock-bucket-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 200.

| Name | Type | Format | Description |
|---|---|---|---|
| Location | Header | String | Path of the created bucket |

<a id="put-object-lock-configuration"></a>
### Set Lock Bucket Retention Period { #put-object-lock-configuration }
Sets the default retention period for a lock bucket.

```
PUT /{bucket}?object-lock

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="put-object-lock-configuration-request"></a>
#### Request

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Consists of S3 API credentials access key and signature |

The request body must contain the object lock configuration in JSON format.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| ObjectLockEnabled | Body | String | Y | Object lock activation status. Only `Enabled` is allowed |
| Rule | Body | Object | N | Default retention rule |
| Rule.DefaultRetention | Body | Object | Conditional | Default retention period settings. Required when Rule is specified |
| Rule.DefaultRetention.Mode | Body | String | Conditional | Retention mode. Only `COMPLIANCE` is allowed |
| Rule.DefaultRetention.Days | Body | Integer | Conditional | Retention period (days). Positive integer. Either Days or Years is required |
| Rule.DefaultRetention.Years | Body | Integer | Conditional | Retention period (years). Positive integer. Either Days or Years is required |

!!! tip "Note"
    If `Rule` is omitted, the default retention period is set to 0 days.
    Even if set using `Years`, the value is always converted to and returned as `Days` when retrieved.

<details>
<summary>Example</summary>

```json
{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
        "DefaultRetention": {
            "Mode": "COMPLIANCE",
            "Days": 1
        }
    }
}
```

</details>

<a id="put-object-lock-configuration-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 200.

<a id="get-object-lock-configuration"></a>
### Get Lock Bucket Retention Period { #get-object-lock-configuration }
Retrieves the object lock configuration of a lock bucket.

```
GET /{bucket}?object-lock

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="get-object-lock-configuration-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Consists of S3 API credentials access key and signature |

<a id="get-object-lock-configuration-response"></a>
#### Response
For a valid request, returns status code 200 and the object lock configuration in JSON format.

| Name | Type | Format | Description |
|---|---|---|---|
| ObjectLockEnabled | Body | String | Object lock activation status |
| Rule | Body | Object | Default retention rule |
| Rule.DefaultRetention | Body | Object | Default retention period settings |
| Rule.DefaultRetention.Mode | Body | String | Retention mode |
| Rule.DefaultRetention.Days | Body | Integer | Retention period (days) |

!!! tip "Note"
    Retention periods set using `Years` are also converted to and returned as `Days` when retrieved.

<details>
<summary>Example</summary>

```json
{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
        "DefaultRetention": {
            "Mode": "COMPLIANCE",
            "Days": 1
        }
    }
}
```

</details>

<a id="object"></a>

## Object { #object }

<a id="upload-object"></a>
### Upload Object { #upload-object }
Uploads an object to the specified bucket.

```
PUT /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="upload-object-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| obj | URL | String | Y | Object name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Comprised of S3 API credentials access key and signature |

<a id="upload-object-response"></a>
#### Response
This API does not return a response body. For a valid request, return status code 200.

| Name | Type | Format | Description |
|---|---|---|---|
| ETag | Header | String | MD5 hash value of the object |
| Last-Modified | Header | String | Last modified date and time of the object (e.g. Wed, 01 Mar 2006 12:00:00 GMT) |

<a id="download-object"></a>
### Download Object { #download-object }
Downloads an object.

```
GET /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="download-object-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| obj | URL | String | Y | Object name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Comprised of S3 API credentials access key and signature |

<a id="download-object-response"></a>
#### Response
For a valid request, return status code 200.

| Name | Type | Format | Description |
|---|---|---|---|
| Last-Modified | Header | String | Last modified date and time of the object (e.g. Wed, 01 Mar 2006 12:00:00 GMT) |
| ETag | Header | String | MD5 hash value of the object |

<a id="delete-object"></a>
### Delete Object { #delete-object }
Deletes the specified object.

```
DELETE /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="delete-object-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| obj | URL | String | Y | Object name |
| Date | Header | String | Y | Request time |
| Authorization | Header | String | Y | Comprised of S3 API credentials access key and signature |

<a id="delete-object-response"></a>
#### Response
This API does not return request body. When the request is appropriate, return status code 204.

<a id="presigned-url"></a>

## Create Signed URL { #presigned-url }
A URL that carries **AWS Signature Version 4 (SigV4)** signing in query parameters, allowing access to an object for a set period of time without an authentication token (Authorization header). Use `GET` for downloads and `PUT` for uploads.

<a id="presigned-url-format"></a>
### Signed URL Format { #presigned-url-format }

```
GET /{bucket}/{obj}
?X-Amz-Algorithm=AWS4-HMAC-SHA256
&X-Amz-Credential={access}%2F{date}%2F{region}%2Fs3%2Faws4_request
&X-Amz-Date={date}
&X-Amz-Expires={seconds}
&X-Amz-SignedHeaders=host
&X-Amz-Signature={signature}
```

<a id="presigned-url-format-request"></a>
#### Request
This API does not require a request body.

| Name | Type | Format | Required | Description |
|---|---|---|---|---|
| bucket | URL | String | Y | Bucket name |
| obj | URL | String | Y | Object name |
| X-Amz-Algorithm | Query | String | Y | Signing algorithm. Set to AWS4-HMAC-SHA256 |
| X-Amz-Credential | Query | String | Y | Access key and signing scope. Format: `{access}/{date}/{region}/s3/aws4_request` (`/` encoded as `%2F`) |
| X-Amz-Date | Query | String | Y | Signing time. ISO 8601 `yyyyMMddTHHmmssZ` (UTC) |
| X-Amz-Expires | Query | String | Y | Validity period in seconds. Minimum `1`, maximum `604800` (7 days) |
| X-Amz-SignedHeaders | Query | String | Y | List of headers included in the signature. Must include at least `host` |
| X-Amz-Signature | Query | String | Y | HMAC signature value that authenticates the request |

<a id="presigned-url-format-response"></a>
#### Response
For a valid request, return status code 200.

!!! tip "Note"
    For more information, including the Swift TempURL method and language-specific direct signing examples, see the [Signed URL Guide](presigned-url-guide/).

<a id="aws-command-line-interface"></a>

## AWS Command Line Interface (CLI) { #aws-command-line-interface }
You can use NHN Cloud Object Storage with the [AWS Command Line Interface](https://aws.amazon.com/ko/cli/) using the S3 compatible API.

<a id="aws-command-line-interface-installation"></a>

### Installation { #aws-command-line-interface-installation }
See [Installing past releases of the AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-version.html) to install the AWS Command Line Interface.

!!! tip "Note"
    NHN Cloud Object Storage supports AWS CLI up to version 2.34.38.

<a id="aws-command-line-interface-configuration"></a>

### Configuration { #aws-command-line-interface-configuration }
To use the AWS Command Line Interface, you must first configure the S3 API credentials and environment.

```shell
$ aws configure
AWS Access Key ID [None]: {access}
AWS Secret Access Key [None]: {secret}
Default region name [None]: {region name}
Default output format [None]: json
```

| Name | Description |
|---|---|
| access | S3 API Credentials access key |
| secret | S3 API Credentials secret key |
| region name | KR1 - Korea (Pangyo) region<br>KR2 - Korea (Pyeongchon) region<br>KR3 - Korea (Gwangju) region<br>JP1 - Japan (Tokyo) region |

<a id="how-to-use-the-s3-commands"></a>

### How to Use S3 Commands { #how-to-use-the-s3-commands }

```shell
aws --endpoint-url={endpoint} s3 {command} s3://{bucket}
```

| Name | Description |
|---|---|
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - Korea (Pangyo) region<br>https://kr2-api-object-storage.nhncloudservice.com - Korea (Pyeongchon) region<br>https://kr3-api-object-storage.nhncloudservice.com - Korea (Gwangju) region<br>https://jp1-api-object-storage.nhncloudservice.com - Japan (Tokyo) region |
| command | AWS Command Line Interface command |
| bucket | Bucket name |

!!! tip "Note"
    Since the AWS CLI is provided for use with AWS, it is configured to use the AWS domain. Therefore, to use NHN Cloud Object Storage, you must specify an endpoint for every command.
    For AWS CLI commands, see [Using high-level (s3) commands with the AWS CLI](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-services-s3-commands.html).

<details>
<summary>Create a bucket</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 mb s3://example-bucket
make_bucket: example-bucket
```

</details>

<details>
<summary>List buckets</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls
2020-07-13 10:07:13 example-bucket
```

</details>

<details>
<summary>View a bucket</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls s3://example-bucket
2020-07-13 10:08:49     104389 0428b9e3e419d4fb7aedffde984ba5b3.jpg
2020-07-13 10:09:09      74448 6dd6d48eef889a5dab5495267944bdc6.jpg
```

</details>

<details>
<summary>Delete a bucket</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rb s3://example-bucket
remove_bucket: example-bucket
```

</details>

<details>
<summary>Locked bucket</summary>

Locked buckets are managed using the <code>aws s3api</code> subcommand.
<br>
Using the <code>--object-lock-enabled-for-bucket</code> option with the <code>create-bucket</code> command creates a bucket with object lock enabled. The default retention period is set to 0 days.

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3api create-bucket \
  --bucket example-bucket \
  --object-lock-enabled-for-bucket
```

To set the default retention period, use the <code>put-object-lock-configuration</code> command.

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3api put-object-lock-configuration \
    --bucket example-bucket \
    --object-lock-configuration '{
        "ObjectLockEnabled": "Enabled",
        "Rule": {
            "DefaultRetention": {
                "Mode": "COMPLIANCE",
                "Days": 1
            }
        }
    }'
```

To view the lock configuration, use the <code>get-object-lock-configuration</code> command.

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3api get-object-lock-configuration --bucket example-bucket
{
    "ObjectLockConfiguration": {
        "ObjectLockEnabled": "Enabled",
        "Rule": {
            "DefaultRetention": {
                "Mode": "COMPLIANCE",
                "Days": 1
            }
        }
    }
}
```

</details>

<details>
<summary>Upload an object</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp ./3b5ab489edffdea7bf4d914e3e9b8240.jpg s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
upload: ./3b5ab489edffdea7bf4d914e3e9b8240.jpg to s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

!!! tip "Note"
    If an object is 8 MB or larger, the AWS CLI splits the object into multiple parts and uploads them. Part objects are stored in a bucket named <code style="display: inline;">{bucket}+segments</code> with names in the format <code style="display: inline;">{object-name}/{upload-id}/{part-number}</code>. After all parts are uploaded, an object that links the part objects is created in the bucket where the upload was requested.

    The <code style="display: inline;">{bucket}+segments</code> bucket where part objects are stored cannot be accessed via the S3-compatible API; it can only be accessed through the Object Storage API or the console.

    The ETag of a multipart object is the MD5 hash of the concatenation of each part object's ETag value converted to binary data in order.

!!! danger "Caution"
    If you delete some or all of the part objects of an object uploaded via multipart upload, the object becomes inaccessible.

</details>

<details>
<summary>Download an object</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg ./3b5ab489edffdea7bf4d914e3e9b8240.jpg
download: s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg to ./0428b9e3e419d4fb7aedffde984ba5b3.jpg
```

</details>

<details>
<summary>Delete an object</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rm s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
delete: s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

</details>

<details>
<summary>Create a presigned URL</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 presign s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg --expires-in 3600
https://kr1-api-object-storage.nhncloudservice.com/example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Date=...&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=...
```

</details>

<a id="aws-command-line-interface-virtual-hosted-style"></a>

### Use Domain-Style Endpoints { #aws-command-line-interface-virtual-hosted-style }
The S3-compatible API supports both Path-style and Virtual Hosted-style as bucket access methods. Virtual Hosted-style uses the bucket name as a subdomain of the endpoint.

| Method | Format |
|---|---|
| Path-style | `https://{endpoint}/{bucket}/{object}` |
| Virtual Hosted-style | `https://{bucket}.{endpoint}/{object}` |

<br>

To use a Virtual Hosted-style endpoint with the AWS CLI, set the `addressing_style` option to `virtual`. With this setting, the AWS CLI automatically combines the endpoint and bucket name to send requests using a Virtual Hosted-style URL.

```shell
$ aws configure set default.s3.addressing_style virtual
```

Alternatively, add the following settings to the profile section you are using in the `~/.aws/config` file.

```ini
[default]
s3 =
  addressing_style = virtual
```

| Name | Description |
|---|---|
| addressing_style | `virtual` - Use Virtual Hosted-style<br>`path` - Use Path-style<br>`auto` - Automatic selection (default; when using a custom endpoint such as NHN Cloud Object Storage, behaves as Path-style) |

!!! danger "Caution"
    If the bucket name contains a period (`.`), using Virtual Hosted-style may result in certificate validation failure because the bucket name falls outside the scope of the wildcard SSL certificate. In this case, use Path-style instead.

<a id="aws-sdk"></a>

## AWS SDK { #aws-sdk }
AWS provides SDKs for many types of programming languages. By using the S3 compatible API, you can use NHN Cloud Object Storage with AWS SDK.

!!! tip "Note"
    For more information, see the [AWS SDK](https://aws.amazon.com/ko/tools) documentation.

The key parameters required to use the AWS SDK are as follows:

| Name | Description |
|---|---|
| access | S3 API Credentials access key |
| secret | S3 API Credentials secret key |
| region name | KR1 - Korea (Pangyo) region<br>KR2 - Korea (Pyeongchon) region<br>KR3 - Korea (Gwangju) region<br>JP1 - Japan (Tokyo) region |
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - Korea (Pangyo) region<br>https://kr2-api-object-storage.nhncloudservice.com - Korea (Pyeongchon) region<br>https://kr3-api-object-storage.nhncloudservice.com - Korea (Gwangju) region<br>https://jp1-api-object-storage.nhncloudservice.com - Japan (Tokyo) region |

<a id="aws-sdk-boto3-python"></a>

### Boto3 - Python SDK { #aws-sdk-boto3-python }

!!! tip "Note"
    For more information, see the [AWS SDK for Python(Boto3)](https://docs.aws.amazon.com/ko_kr/pythonsdk/?icmpid=docs_homepage_sdktoolkits) documentation.

<a id="aws-sdk-boto3-python-context"></a>
#### Context

<details>
<summary>Boto3 client class</summary>

```python
# boto3example.py
from boto3 import client
from boto3.s3.transfer import TransferConfig
from botocore.exceptions import ClientError


class Boto3Example(object):
    _REGION = '{region name}'
    _ENDPOINT = '{endpoint}'
    _ACCESS = '{access}'
    _SECRET = '{secret}'

    def __init__(self):
        self.s3 = client(service_name='s3',
                         region_name=self._REGION,
                         endpoint_url=self._ENDPOINT,
                         aws_access_key_id=self._ACCESS,
                         aws_secret_access_key=self._SECRET)
```

</details>

<details>
<summary>Create a bucket</summary>

```python
def create_bucket(self, bucket_name):
    try:
        return self.s3.create_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>List buckets</summary>

```python
def list_buckets(self):
    try:
        return self.s3.list_buckets().get('Buckets')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Get a bucket (list objects)</summary>

```python
def list_objs(self, bucket_name):
    try:
        return self.s3.list_objects_v2(Bucket=bucket_name).get('Contents')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Delete a bucket</summary>

```python
def delete_bucket(self, bucket_name):
    try:
        return self.s3.delete_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Locked bucket</summary>

Setting <code>ObjectLockEnabledForBucket=True</code> in the <code>create_bucket</code> method creates a locked bucket. The default retention period is set to 0 days.

```python
def create_bucket_with_lock(self, bucket_name):
    try:
        return self.s3.create_bucket(
            Bucket=bucket_name,
            ObjectLockEnabledForBucket=True
        )
    except ClientError as e:
        raise RuntimeError(e)
```

To set the default retention period, use the <code>put_object_lock_configuration</code> method.

```python
def put_object_lock_configuration(self, bucket_name, days):
    lock_configuration = {
        'ObjectLockEnabled': 'Enabled',
        'Rule': {
            'DefaultRetention': {
                'Mode': 'COMPLIANCE',
                'Days': days
            }
        }
    }
    try:
        return self.s3.put_object_lock_configuration(
            Bucket=bucket_name,
            ObjectLockConfiguration=lock_configuration
        )
    except ClientError as e:
        raise RuntimeError(e)
```

To retrieve the lock configuration, use the <code>get_object_lock_configuration</code> method.

```python
def get_object_lock_configuration(self, bucket_name):
    try:
        return self.s3.get_object_lock_configuration(
            Bucket=bucket_name
        )
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Upload an object</summary>

!!! tip "Note"
    The number of part objects is determined by the size of the uploaded object and the part size you set. The default part size is 8 MiB, and the minimum part size you can specify is 5 MiB. The maximum number of part objects is 10,000.

```python
def upload(self, bucket_name, key, filename, part_size):
    config = TransferConfig(multipart_chunksize=part_size)
    try:
        self.s3.upload_file(Filename=filename,
                            Bucket=bucket_name,
                            Key=key,
                            Config=config)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Download an object</summary>

```python
def download(self, bucket_name, key, filename):
    try:
        response = self.s3.get_object(Bucket=bucket_name, Key=key)

        with io.FileIO(filename, 'w') as fd:
            for chunk in response['Body']:
                fd.write(chunk)

        response.pop('Body')
    except ClientError as e:
        raise RuntimeError(e)
    except OSError as e:
        raise RuntimeError(e)

    return response
```

</details>

<details>
<summary>Delete an object</summary>

```python
def delete(self, bucket_name, key):
    try:
        return self.s3.delete_object(Bucket=bucket_name, Key=key)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Generate a signed URL</summary>

```python
def generate_presigned_url(self, bucket_name, key, expires_in):
    try:
        # Use 'put_object' for uploads
        return self.s3.generate_presigned_url(
            'get_object',
            Params={'Bucket': bucket_name, 'Key': key},
            ExpiresIn=expires_in)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<a id="aws-sdk-java"></a>

### Java SDK { #aws-sdk-java }

!!! tip "Note"
    For more information, see the [AWS SDK for Java documentation](https://docs.aws.amazon.com/ko_kr/sdk-for-java/index.html).

<a id="aws-sdk-java-context"></a>
#### Context

<details>
<summary>Java SDK client class</summary>

```java
// AwsSdkExample.java
public class AwsSdkExample {
    private static final String access = "{access}";
    private static final String secret = "{secret}";
    private static final String region = "{region name}";
    private static final String endpoint = "{endpoint}";

    private AmazonS3 s3Client;

    public AwsSdkExample() {
        BasicAWSCredentials awsCredentials =
            new BasicAWSCredentials(access, secret);
        s3Client = AmazonS3ClientBuilder.standard()
            .withEndpointConfiguration(
                new AwsClientBuilder.EndpointConfiguration(endpoint, region)
            )
            .withCredentials(
                new AWSStaticCredentialsProvider(awsCredentials)
            )
            .enablePathStyleAccess()
            .disableChunkedEncoding()
            .build();
    }
}
```

</details>

<details>
<summary>Create a bucket</summary>

```java
public String createBucket(String bucketName) throws RuntimeException {
    try {
        return s3Client.createBucket(bucketName).toString();
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<details>
<summary>List buckets</summary>

```java
public List<Bucket> listBuckets() throws RuntimeException {
    try {
        return s3Client.listBuckets();
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<details>
<summary>Get a bucket (list objects)</summary>

```java
public ListObjectsV2Result listObjects(
    String bucketName
) throws RuntimeException {
    try {
        return s3Client.listObjectsV2(bucketName);
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<details>
<summary>Delete a bucket</summary>

```java
public void deleteBucket(String bucketName) throws RuntimeException {
    try {
        s3Client.deleteBucket(bucketName);
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<details>
<summary>Lock bucket</summary>

Setting <code>withObjectLockEnabledForBucket(true)</code> in <code>CreateBucketRequest</code> creates a lock bucket. The default retention period is set to 0 days.

```java
public String createBucketWithLock(String bucketName) throws RuntimeException {
    try {
        CreateBucketRequest request = new CreateBucketRequest(bucketName)
            .withObjectLockEnabledForBucket(true);
        return s3Client.createBucket(request).toString();
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

To set the default retention period, use the <code>setObjectLockConfiguration</code> method.

```java
public void putObjectLockConfiguration(
    String bucketName, int days
) throws RuntimeException {
    try {
        DefaultRetention retention = new DefaultRetention()
            .withMode(ObjectLockRetentionMode.COMPLIANCE)
            .withDays(days);
        ObjectLockRule rule = new ObjectLockRule()
            .withDefaultRetention(retention);
        ObjectLockConfiguration configuration = new ObjectLockConfiguration()
            .withObjectLockEnabled(ObjectLockEnabled.ENABLED)
            .withRule(rule);
        SetObjectLockConfigurationRequest request =
            new SetObjectLockConfigurationRequest()
                .withBucketName(bucketName)
                .withObjectLockConfiguration(configuration);
        s3Client.setObjectLockConfiguration(request);
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

To retrieve the lock configuration, use the <code>getObjectLockConfiguration</code> method.

```java
public ObjectLockConfiguration getObjectLockConfiguration(
    String bucketName
) throws RuntimeException {
    try {
        GetObjectLockConfigurationRequest request =
            new GetObjectLockConfigurationRequest()
                .withBucketName(bucketName);
        return s3Client.getObjectLockConfiguration(request)
            .getObjectLockConfiguration();
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<details>
<summary>Upload an object</summary>

!!! tip "Note"
    The number of part objects is determined by the size of the object to upload and the part size that you set. The default part size is 5 MiB, and you can specify a minimum of 5 MiB. The maximum number of part objects is 10,000.

```java
public void uploadObject(String bucketName, String objectKey, String filePath, long partSize) {
    TransferManager tm = TransferManagerBuilder.standard()
        .withS3Client(s3Client)
        .withMinimumUploadPartSize(partSize)
        .build();
    Upload upload = tm.upload(bucketName, objectKey, new File(filePath));

    try {
        upload.waitForCompletion();
    } catch (AmazonServiceException e) {
        upload.abort();
    } catch (AmazonClientException e) {
        upload.abort();
    } catch (InterruptedException e) {
        upload.abort();
    }
}
```

</details>

<details>
<summary>Download an object</summary>

```java
public String downloadObject(
    String bucketName, String objKeyName, String filePath
) throws RuntimeException {
    try {
        return s3Client.getObject(
            new GetObjectRequest(bucketName, objKeyName),
            new File(filePath)
        ).getETag();
    } catch (NoSuchKeyException e) {
        throw new RuntimeException(e);
    } catch (InvalidObjectStateException e) {
        throw new RuntimeException(e);
    } catch (S3Exception e) {
        throw new RuntimeException(e);
    } catch (AwsServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<details>
<summary>Delete an object</summary>

```java
public void deleteObject(
    String bucketName, String objKeyName
) throws RuntimeException {
    try {
        s3Client.deleteObject(bucketName, objKeyName);
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<details>
<summary>Generate a signed URL</summary>

```java
public String generatePresignedUrl(
    String bucketName, String objKeyName, long expirationMillis
) throws RuntimeException {
    try {
        Date expiration = new Date(System.currentTimeMillis() + expirationMillis);
        GeneratePresignedUrlRequest request =
            new GeneratePresignedUrlRequest(bucketName, objKeyName)
                .withMethod(HttpMethod.GET)          // Use HttpMethod.PUT for uploads
                .withExpiration(expiration);
        return s3Client.generatePresignedUrl(request).toString();
    } catch (AmazonServiceException e) {
        throw new RuntimeException(e);
    } catch (SdkClientException e) {
        throw new RuntimeException(e);
    }
}
```

</details>

<a id="aws-sdk-dotnet"></a>

### .NET SDK { #aws-sdk-dotnet }

!!! tip "Note"
    For more information, see the [AWS SDK for .NET documentation](https://docs.aws.amazon.com/ko_kr/sdk-for-net/?icmpid=docs_homepage_sdktoolkits).

<a id="aws-sdk-dotnet-context"></a>
#### Context

<details>
<summary>.NET SDK client class</summary>

```csharp
class S3SDKExample
{
    private static string endpoint = "{endpoint}";
    private static string regionName = "{region name}";
    private static string accessKey = "{access}";
    private static string secretKey = "{secret}";

    private static AmazonS3Client GetS3Client()
    {
        var amazonS3Config =
            new AmazonS3Config
            {
                ServiceURL = endpoint,
                AuthenticationRegion = regionName,
                ForcePathStyle = true,
            };
        var basicAWSCredentials = new BasicAWSCredentials(accessKey, secretKey);

        return new AmazonS3Client(basicAWSCredentials, amazonS3Config);
    }
}
```

</details>

<details>
<summary>Create a bucket</summary>

```csharp
static async Task<PutBucketResponse> CreateBucketAsync(
    AmazonS3Client s3Client,
    string bucketName)
{
    try
    {
        if (!(await AmazonS3Util.DoesS3BucketExistAsync(s3Client, bucketName)))
        {
            var putBucketRequest =
                new PutBucketRequest
                {
                    BucketName = bucketName,
                    UseClientRegion = true
                };

            return await s3Client.PutBucketAsync(putBucketRequest);
        }
        throw new Exception("Bucket already exist.");
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<details>
<summary>List buckets</summary>

```csharp
static async Task<ListBucketsResponse> ListBucketsAsync(AmazonS3Client s3Client)
{
    try
    {
        return await s3Client.ListBucketsAsync();
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<details>
<summary>View a bucket (list objects)</summary>

```csharp
static async Task<List<ListObjectsV2Response>> ListBucketContentsAsync(
    AmazonS3Client s3Client,
    string bucketName)
{
    try
    {
        List<ListObjectsV2Response> responses =
            new List<ListObjectsV2Response>();
        var request =
            new ListObjectsV2Request
            {
                BucketName = bucketName,
                MaxKeys = 5,
            };
        var response = new ListObjectsV2Response();

        do
        {
            responses.Add(await s3Client.ListObjectsV2Async(request));
            request.ContinuationToken = response.NextContinuationToken;
        }
        while (response.IsTruncated);

        return responses;
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<details>
<summary>Delete a bucket</summary>

```csharp
static async Task<DeleteBucketResponse> DeleteBucketAsync(
    AmazonS3Client s3Client,
    string bucketName)
{
    try
    {
        return await s3Client.DeleteBucketAsync(
            new DeleteBucketRequest
            {
                BucketName = bucketName
            });
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<details>
<summary>Lock bucket</summary>

Setting <code>ObjectLockEnabledForBucket = true</code> in <code>PutBucketRequest</code> creates a lock bucket. The default retention period is set to 0 days.

```csharp
static async Task<PutBucketResponse> CreateBucketWithLockAsync(
    AmazonS3Client s3Client,
    string bucketName)
{
    try
    {
        var putBucketRequest =
            new PutBucketRequest
            {
                BucketName = bucketName,
                UseClientRegion = true,
                ObjectLockEnabledForBucket = true
            };

        return await s3Client.PutBucketAsync(putBucketRequest);
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

To set the default retention period, use the <code>PutObjectLockConfigurationAsync</code> method.

```csharp
static async Task<PutObjectLockConfigurationResponse> PutObjectLockConfigurationAsync(
    AmazonS3Client s3Client,
    string bucketName,
    int days)
{
    try
    {
        var request =
            new PutObjectLockConfigurationRequest
            {
                BucketName = bucketName,
                ObjectLockConfiguration =
                    new ObjectLockConfiguration
                    {
                        ObjectLockEnabled = ObjectLockEnabled.Enabled,
                        Rule =
                            new ObjectLockRule
                            {
                                DefaultRetention =
                                    new DefaultRetention
                                    {
                                        Mode = ObjectLockRetentionMode.Compliance,
                                        Days = days
                                    }
                            }
                    }
            };

        return await s3Client.PutObjectLockConfigurationAsync(request);
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

To retrieve the lock configuration, use the <code>GetObjectLockConfigurationAsync</code> method.

```csharp
static async Task<GetObjectLockConfigurationResponse> GetObjectLockConfigurationAsync(
    AmazonS3Client s3Client,
    string bucketName)
{
    try
    {
        var request =
            new GetObjectLockConfigurationRequest
            {
                BucketName = bucketName
            };

        return await s3Client.GetObjectLockConfigurationAsync(request);
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<details>
<summary>Upload an object</summary>

!!! tip "Note"
    The number of part objects is determined by the size of the uploaded object and the part size you set. The default part size is 5 MiB, and you can specify a minimum of 5 MiB. The maximum number of part objects is 10,000.

```csharp
private static async Task UploadObjectAsync(
    AmazonS3Client s3Client,
    string bucketName,
    string keyName,
    string filePath,
    int partSize)
{
    try
    {
        TransferUtility uploader = new TransferUtility(s3Client);
        TransferUtilityUploadRequest uploadRequest = new TransferUtilityUploadRequest()
        {
            FilePath = filePath,
            BucketName = bucketName,
            Key = keyName,
            PartSize = partSize
        };
        uploader.Upload(uploadRequest);
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<details>
<summary>Download an object</summary>

```csharp
static async Task ReadObjectDataAsync(
    AmazonS3Client s3Client,
    string bucketName,
    string keyName,
    string filePath)
{
    try
    {
        GetObjectRequest request =
            new GetObjectRequest
            {
                BucketName = bucketName,
                Key = keyName
            };

        ResponseHeaderOverrides responseHeaders =
            new ResponseHeaderOverrides();
        responseHeaders.CacheControl = "No-cache";

        request.ResponseHeaderOverrides = responseHeaders;
        var appendToFile = false;

        using (var response = await s3Client.GetObjectAsync(request))
        await response.WriteResponseStreamToFileAsync(
            filePath, appendToFile, CancellationToken.None);
    }
    catch (AmazonS3Exception e)
    {
       throw e;
    }
}
```

</details>

<details>
<summary>Delete an object</summary>

```csharp
static async Task<DeleteObjectResponse> DeleteObjectNonVersionedBucketAsync(
    AmazonS3Client s3Client,
    string bucketName,
    string keyName)
{
    try
    {
        var deleteObjectRequest =
            new DeleteObjectRequest
            {
                BucketName = bucketName,
                Key = keyName
            };

        return await s3Client.DeleteObjectAsync(deleteObjectRequest);
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<details>
<summary>Create a presigned URL</summary>

```csharp
static string GeneratePresignedUrl(
    AmazonS3Client s3Client,
    string bucketName,
    string keyName,
    double durationHours)
{
    try
    {
        var request =
            new GetPreSignedUrlRequest
            {
                BucketName = bucketName,
                Key = keyName,
                Verb = HttpVerb.GET,                 // Use HttpVerb.PUT for uploads
                Expires = DateTime.UtcNow.AddHours(durationHours)
            };

        return s3Client.GetPreSignedURL(request);
    }
    catch (AmazonS3Exception e)
    {
        throw e;
    }
}
```

</details>

<a id="aws-sdk-virtual-hosted-style"></a>

### Use domain-style endpoints { #aws-sdk-virtual-hosted-style }
To use domain-style endpoints in the AWS SDK, disable path-style access in the client configuration. The endpoint URL and credentials remain the same as before, and the SDK combines the bucket name as a subdomain to send requests.

<details>
<summary>Boto3 - Python SDK</summary>

Create a client by setting the <code>s3.addressing_style</code> value of <code>botocore.client.Config</code> to <code>virtual</code>.

```python
from boto3 import client
from botocore.client import Config


class Boto3Example(object):
    _REGION = '{region name}'
    _ENDPOINT = '{endpoint}'
    _ACCESS = '{access}'
    _SECRET = '{secret}'

    def __init__(self):
        config = Config(s3={'addressing_style': 'virtual'})
        self.s3 = client(service_name='s3',
                         region_name=self._REGION,
                         endpoint_url=self._ENDPOINT,
                         aws_access_key_id=self._ACCESS,
                         aws_secret_access_key=self._SECRET,
                         config=config)
```

</details>

<details>
<summary>Java SDK</summary>

Remove the <code>enablePathStyleAccess()</code> call from the client builder.

```java
public AwsSdkExample() {
    BasicAWSCredentials awsCredentials =
        new BasicAWSCredentials(access, secret);
    s3Client = AmazonS3ClientBuilder.standard()
        .withEndpointConfiguration(
            new AwsClientBuilder.EndpointConfiguration(endpoint, region)
        )
        .withCredentials(
            new AWSStaticCredentialsProvider(awsCredentials)
        )
        .disableChunkedEncoding()
        .build();
}
```

</details>

<details>
<summary>.NET SDK</summary>

Remove the <code>ForcePathStyle</code> property setting from <code>AmazonS3Config</code>.

```csharp
private static AmazonS3Client GetS3Client()
{
    var amazonS3Config =
        new AmazonS3Config
        {
            ServiceURL = endpoint,
            AuthenticationRegion = regionName,
        };
    var basicAWSCredentials = new BasicAWSCredentials(accessKey, secretKey);

    return new AmazonS3Client(basicAWSCredentials, amazonS3Config);
}
```

</details>

!!! danger "Caution"
    If the bucket name contains a dot (`.`), using the domain style may cause certificate validation to fail because the bucket name falls outside the valid scope of the wildcard SSL certificate. In this case, use the path style instead.