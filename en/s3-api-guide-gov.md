## Storage > Object Storage > API Guide for AWS S3 Compatibility

Object storage provides APIs that are compatible with S3 API of AWS object storage. To enable the service, you can only change settings for applications developed for AWS S3 API.

APIs that are compatible with S3 are provided as follows.  

| S3 API Method                                                | Usage                                   |
| ------------------------------------------------------------ | --------------------------------------- |
| [PUT Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUT.html) | Create bucket                           |
| [HEAD Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketHEAD.html) | Query bucket information                |
| [DELETE Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketDELETE.html) | Delete bucket                           |
| [PUT Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUTacl.html) | Set bucket ACL                          |
| [GET Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETacl.html) | Get bucket ACL                          |
| [GET Bucket Location](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETlocation.html) | Get region with bucket                  |
| [GET Bucket List Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGET.html) | List bucket objects                     |
| [GET Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectGET.html) | Download object                         |
| [HEAD Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectHEAD.html) | Query object information                |
| [PUT Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectPUT.html) | Upload object                           |
| [PUT Object Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectCOPY.html) | Copy object                             |
| [DELETE Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectDELETE.html) | Delete object                           |
| [Initiate Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadInitiate.html) | Initialize multi-part upload            |
| [Upload Part](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPart.html) | Upload part                             |
| [Upload Part Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPartCopy.html) | Copy part                               |
| [Complete Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadComplete.html) | Complete multi-part upload              |
| [Abort Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadAbort.html) | Abort multi-part upload                 |
| [List Parts](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | List multi-part objects                 |
| [List Multipart Uploads](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | List multi-part objects under uploading |
| [DELETE Multiple Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/multiobjectdeleteapi.html) | Delete multi-part objects               |

This document describes only the basic usage of API. To use advanced features, see [API Guide for AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html), or [AWS SDK](https://aws.amazon.com/ko/tools) is recommended.

## EC2 Credentials

### Register

To use APIs compatible with S3, register AWS EC2-type credential first. To that end, an authentication token is required. To get a token, see [API Guide for Object Storage](/Storage/Object%20Storage/ko/api-guide/#tenant-id-api-endpoint).

```
POST    https://gov-api-identity.infrastructure.cloud.toast.com/v2.0/users/{User ID}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

#### Request

| Name         | Type   | Format | Required | Description                                                  |
| ------------ | ------ | ------ | -------- | ------------------------------------------------------------ |
| X-Auth-Token | Header | String | O        | Issued token ID                                              |
| user-id      | URL    | String | O        | User ID, included in authentication token                    |
| tenant_id    | Body   | String | O        | User Tenant ID, to be found on the setup box for API Endpoint |

> [Caution]
> User ID for credential registration is not an email-type TOAST account. Find this, when you get a token issued.

<details>
<summary>Example</summary>

```json
{
  "tenant_id": "84c9e9a51aea402e95389c08ac562ac5"
}
```

</details>

#### Response

| Name   | Type | Format | Description           |
| ------ | ---- | ------ | --------------------- |
| access | Body | String | Credential access key |
| secret | Body | String | Credential secret key |

<details>
<summary>Example</summary>

```json
{
  "credential": {
    "access": "253a3c7ca27f4731a9c757addfac29ca",
    "tenant_id": "84c9e9a51aea402e95389c08ac562ac5",
    "secret": "be057f235abf45ee8e2ba14edc5fb253",
    "user_id": "84db0c80-3c39-11e7-b29c-005056ac1497",
    "trust_id": null
  }
}
```

</details>

### Get

Get registered EC2 credential.

**[Method, URL]**

```
GET   https://gov-api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2

X-Auth-Token: {token-id}
```

#### Request

This API does not require a request body.

| Name         | Type   | Format | Required | Description                               |
| ------------ | ------ | ------ | -------- | ----------------------------------------- |
| X-Auth-Token | Header | String | O        | Issued token ID                           |
| user-id      | URL    | String | O        | User ID, included in authentication token |

#### Response

| Name   | Type | Format | Description           |
| ------ | ---- | ------ | --------------------- |
| access | Body | String | Credential access key |
| secret | Body | String | Credential secret key |

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
      "trust_id": null
    }
  ]
}
```

</details>

### Delete

Delete registered EC2 credential.

**[Method, URL]**

```
DELETE   https://gov-api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```

#### Request

This API does not require a request body.

| Name         | Type   | Format | Required | Description                               |
| ------------ | ------ | ------ | -------- | ----------------------------------------- |
| X-Auth-Token | Header | String | O        | Issued token ID                           |
| user-id      | URL    | String | O        | User ID, included in authentication token |
| access       | URL    | String | O        | Credential access key                     |

#### Response

This API does not return request body. When the request is appropriate, return status code 204.

## Signatures

To enable S3 APIs, use credential to create a signature. See [AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) regarding how to sign.

Following information is required to create a signature.

| Name          | Value                          |
| ------------- | ------------------------------ |
| Algorithm     | AWS4-HMAC-SHA256               |
| Signed Time   | In the ZssmmhhTDDMMYYYY format |
| Service Name  | s3                             |
| Region Name   | KR1 - Korea (Pangyo) region<br/>KR2 - KOREA (Pyeongchon) Region<br/>JP1 - JAPAN (Tokyo) Region<br/>US1 - USA (California) Region |
| Secret Key    | Credential secret key          |

> [참고]
> 2020년 8월 현재 공공 클라우드에서는 S3 호환 API를 제공하고 있지 않습니다.

## Buckets

### Create  

Create a bucket (container). 버킷 이름은 다음과 같은 AWS S3의 버킷 명명 규칙을 따라야 합니다.

* 버킷 이름은 3자에서 63자 사이여야 합니다.
* 버킷 이름은 소문자, 숫자, 점(.) 및 하이픈(-)으로만 구성될 수 있습니다.
* 버킷 이름은 문자 또는 숫자로 시작하고 끝나야 합니다.
* 버킷 이름은 IP 주소 형식(예: 192.168.5.4)을 사용하지 않습니다.
* 버킷 이름은 xn--으로 시작할 수 없습니다.

자세한 내용은 [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) 문서를 참조하세요.

```
PUT /{bucket}

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

> [참고]
> 웹 콘솔 또는 OBS API를 통해 만든 컨테이너의 이름이 버킷 명명 규칙에 위배되면 S3 호환 API로는 접근할 수 없습니다.

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of credential access key and signature |

#### Response

| Name                            | Type | Format  | Description                 |
| ------------------------------- | ---- | ------- | --------------------------- |
| ResponseMetadata                | Body | Object  | Object of response metadata |
| ResponseMetadata.HTTPStatusCode | Body | Integer | Response status code        |
| Location                        | Body | String  | Created bucket path         |

<details>
<summary>Example</summary>

```json
{
  "ResponseMetadata": {
    "RequestId": "txfad4e17792b1432fb106f-005e5ef0e4",
    "HostId": "txfad4e17792b1432fb106f-005e5ef0e4",
    "HTTPStatusCode": 200,
    "HTTPHeaders": {
      "x-amz-id-2": "txfad4e17792b1432fb106f-005e5ef0e4",
      "content-length": "0",
      "x-amz-request-id": "txfad4e17792b1432fb106f-005e5ef0e4",
      "content-type": "text/html; charset=UTF-8",
      "location": "/new-container",
      "x-trans-id": "txfad4e17792b1432fb106f-005e5ef0e4",
      "x-openstack-request-id": "txfad4e17792b1432fb106f-005e5ef0e4",
      "date": "22:22:22 GMT, Sat, 22 Feb 2020"
    },
    "RetryAttempts": 0
  },
  "Location": "/new-container"
}
```

</details>

### List  

List buckets.

```
GET /

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| Date          | Header | String | O        | Request Time                                     |
| Authorization | Header | String | O        | Comprised of credential access key and signature |

#### Response

| Name                            | Type | Format  | Description                 |
| ------------------------------- | ---- | ------- | --------------------------- |
| ResponseMetadata                | Body | Object  | Object of response metadata |
| ResponseMetadata.HTTPStatusCode | Body | Integer | Response status code        |
| Buckets.Name                    | Body | String  | Bucket name                 |
| Buckets.CreationDate            | Body | String  | Created time                |

<details>
<summary>Example</summary>

```json
{
  "ResponseMetadata": {
    "RequestId": "txbf73f4d73ad34344a21bb-005e5ef141",
    "HostId": "txbf73f4d73ad34344a21bb-005e5ef141",
    "HTTPStatusCode": 200,
    "HTTPHeaders": {
      "x-amz-id-2": "txbf73f4d73ad34344a21bb-005e5ef141",
      "content-length": "750",
      "x-amz-request-id": "txbf73f4d73ad34344a21bb-005e5ef141",
      "content-type": "application/xml",
      "x-trans-id": "txbf73f4d73ad34344a21bb-005e5ef141",
      "x-openstack-request-id": "txbf73f4d73ad34344a21bb-005e5ef141",
      "date": "22:22:22 GMT, 22 Feb 2020"
    },
    "RetryAttempts": 0
  },
  "Buckets": [
    {
      "Name": "new-container",
      "CreationDate": "T22:22:22+00:00,22 Feb 2020"
    }
  ],
  "Owner": {}
}
```

</details>

### Get

Get bucket information as specified and list objects that are saved within.

```
GET /{bucket}

Date: 22:22:22 +0000, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of credential access key and signature |

#### Response

| Name                            | Type | Format  | Description                                         |
| ------------------------------- | ---- | ------- | --------------------------------------------------- |
| ResponseMetadata                | Body | Object  | Object of response metadata                         |
| ResponseMetadata.HTTPStatusCode | Body | Integer | Response status code                                |
| Contents                        | Body | Object  | Object on object list                               |
| Contents.Key                    | Body | String  | Object name                                         |
| Contents.LastModified           | Body | String  | The latest object update time, ssZ:mm:hhTDD-MM-YYYY |
| Contents.ETag                   | Body | String  | MD5 hash of object                                  |
| Contents.Size                   | Body | String  | Size of object                                      |
| Contents.StorageClass           | Body | String  | Type of storage for object                          |
| Name                            | Body | String  | Bucket name                                         |
| KeyCount                        | Body | Integer | Object count on list                                |

<details>
<summary>Example</summary>

```json
{
  "ResponseMetadata": {
    "RequestId": "tx75a3242dac55411fac69b-005e5ef1f1",
    "HostId": "tx75a3242dac55411fac69b-005e5ef1f1",
    "HTTPStatusCode": 200,
    "HTTPHeaders": {
      "x-amz-id-2": "tx75a3242dac55411fac69b-005e5ef1f1",
      "content-length": "1273",
      "x-amz-request-id": "tx75a3242dac55411fac69b-005e5ef1f1",
      "content-type": "application/xml",
      "x-trans-id": "tx75a3242dac55411fac69b-005e5ef1f1",
      "x-openstack-request-id": "tx75a3242dac55411fac69b-005e5ef1f1",
      "date": "22:22:22 GMT, Sat, 22 Feb 2020"
    },
    "RetryAttempts": 0
  },
  "IsTruncated": false,
  "Contents": [
    {
       "Key": "benjamin-ashton-Af9X4A8qMtM-unsplash.jpg",
       "LastModified": "2020-02-22T22:22:22.222222+00:00",
       "ETag": "\"2e95b028564c14371939358d3e88a771\"",
       "Size": 3267226,
       "StorageClass": "STANDARD"
    }
  ],
  "Name": "new-container",
  "Prefix": "",
  "MaxKeys": 1000,
  "EncodingType": "url",
  "KeyCount": 1
}
```

</details>

### Delete

Delete buckets as specified. To be deleted, buckets must be empty.

```
DELETE /{bucket}

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of credential access key and signature |

#### Response

| Name                            | Type | Format  | Description                 |
| ------------------------------- | ---- | ------- | --------------------------- |
| ResponseMetadata                | Body | Object  | Object of response metadata |
| ResponseMetadata.HTTPStatusCode | Body | Integer | Response status code        |

<details>
<summary>Example</summary>

```json
{
    "ResponseMetadata": {
        "RequestId": "tx9b01c2e650e746ecba298-005e5ef28b",
        "HostId": "tx9b01c2e650e746ecba298-005e5ef28b",
        "HTTPStatusCode": 204,
        "HTTPHeaders": {
            "x-amz-id-2": "tx9b01c2e650e746ecba298-005e5ef28b",
            "content-length": "0",
            "x-amz-request-id": "tx9b01c2e650e746ecba298-005e5ef28b",
            "content-type": "text/html; charset=UTF-8",
            "x-trans-id": "tx9b01c2e650e746ecba298-005e5ef28b",
            "x-openstack-request-id": "tx9b01c2e650e746ecba298-005e5ef28b",
            "date": "22:22:22 GMT, Sat, 22 Feb 2020"
        },
        "RetryAttempts": 0
    }
}
```

</details>

## Objects

### Upload

Upload objects to a specified bucket.

```
PUT /{bucket}/{obj}

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| obj           | URL    | String | O        | Object name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of credential access key and signature |

#### Response

| Name                            | Type | Format  | Description                 |
| ------------------------------- | ---- | ------- | --------------------------- |
| ResponseMetadata                | Body | Object  | Object of response metadata |
| ResponseMetadata.HTTPStatusCode | Body | Integer | Response status code        |
| ETag                            | Body | String  | MD5 hash of uploaded object |

<details>
<summary>Example</summary>

```json
{
  "ResponseMetadata": {
    "RequestId": "tx1d914107987d4bd98b7f3-005e5ef3ef",
    "HostId": "tx1d914107987d4bd98b7f3-005e5ef3ef",
    "HTTPStatusCode": 200,
    "HTTPHeaders": {
      "content-length": "0",
      "x-amz-id-2": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "last-modified": "22:22:22 GMT, Sat, 22 Feb 2020",
      "etag": "\"01463f775ef4f4dbbc7525f88120df09\"",
      "x-amz-request-id": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "content-type": "text/html; charset=UTF-8",
      "x-trans-id": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "x-openstack-request-id": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "date": "22:22:22 GMT, Sat, 22 Feb 2020"
    },
    "RetryAttempts": 0
  },
  "ETag": "\"01463f775ef4f4dbbc7525f88120df09\""
}
```

</details>

### Download

Download objects.

```
GET /{bucket}/{obj}

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| obj           | URL    | String | O        | Object name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of credential access key and signature |

#### Response

| Name                            | Type | Format  | Description                                             |
| ------------------------------- | ---- | ------- | ------------------------------------------------------- |
| ResponseMetadata                | Body | Object  | Object of response metadata                             |
| ResponseMetadata.HTTPStatusCode | Body | Integer | Response status code                                    |
| LastModified                    | Body | String  | The latest update time of object, ssZ:mm:Thh DD-MM-YYYY |
| ContentLength                   | Body | String  | Size of downloaded object                               |
| ETag                            | Body | String  | MD5 hash of object                                      |
| ContentType                     | Body | String  | Content type of object                                  |
| Metadata                        | Body | Object  | Metadata of object                                      |

<details>
<summary>Example</summary>

```json
{
    "ResponseMetadata": {
        "RequestId": "tx637d5de3c27f4b0a9664e-005e5ef491",
        "HostId": "tx637d5de3c27f4b0a9664e-005e5ef491",
        "HTTPStatusCode": 200,
        "HTTPHeaders": {
            "content-length": "124352",
            "x-amz-id-2": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "last-modified": "22:22:22 GMT, Sat, 22 Feb 2020",
            "etag": "\"01463f775ef4f4dbbc7525f88120df09\"",
            "x-amz-request-id": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "content-type": "image/jpeg",
            "x-trans-id": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "x-openstack-request-id": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "date": "22:22:22 GMT, Sat, 22 Feb 2020"
        },
        "RetryAttempts": 0
    },
    "LastModified": "T22:22:22+00:00 2020-02-22",
    "ContentLength": 124352,
    "ETag": "\"01463f775ef4f4dbbc7525f88120df09\"",
    "ContentType": "image/jpeg",
    "Metadata": {}
}
```

</details>

### Delete

Delete objects as specified.

```
DELETE /{bucket}/{obj}

Date: 22:22:22 +0000 Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| obj           | URL    | String | O        | Object name                                      |
| Date          | Header | String | O        | Requested time                                   |
| Authorization | Header | String | O        | Comprised of credential access key and signature |

#### Response

| Name                            | Type | Format  | Description                 |
| ------------------------------- | ---- | ------- | --------------------------- |
| ResponseMetadata                | Body | Object  | Object of response metadata |
| ResponseMetadata.HTTPStatusCode | Body | Integer | Response status code        |

<details>
<summary>Example</summary>

```json
{
    "ResponseMetadata": {
        "RequestId": "tx04a548072068487a8a5be-005e5ef648",
        "HostId": "tx04a548072068487a8a5be-005e5ef648",
        "HTTPStatusCode": 204,
        "HTTPHeaders": {
            "x-amz-id-2": "tx04a548072068487a8a5be-005e5ef648",
            "content-length": "0",
            "x-amz-request-id": "tx04a548072068487a8a5be-005e5ef648",
            "content-type": "text/html; charset=UTF-8",
            "x-trans-id": "tx04a548072068487a8a5be-005e5ef648",
            "x-openstack-request-id": "tx04a548072068487a8a5be-005e5ef648",
            "date": "22:22:22 GMT, Sat, 22 Feb 2020"
        },
        "RetryAttempts": 0
    }
}
```

</details>

## AWS 명령줄 인터페이스(CLI)
S3 호환 API를 이용해 [AWS 명령줄 인터페이스](https://aws.amazon.com/ko/cli/)로 TOAST OBS를 사용할 수 있습니다.

### 설치
AWS 명령줄 인터페이스는 파이썬 패키지로 제공됩니다. 파이썬 패키지 관리자(pip)를 이용해 설치합니다.

```
$ sudo pip install awscli
```

### 설정
AWS 명령줄 인터페이스를 사용하기 위해서는 먼저 자격 증명과 환경을 설정해야 합니다.

```
$ aws configure
AWS Access Key ID [None]: {access}
AWS Secret Access Key [None]: {secret}
Default region name [None]: {region name}
Default output format [None]: json
```

| 이름 | 설명 |
|---|---|
| access | 자격 증명 접근 키 |
| secret | 자격 증명 비밀 키 |
| region name | KR1 - 한국(판교)리전<br/>KR2 - 한국(평촌)리전<br/>JP1 - 일본(도쿄)리전<br/>US1 - 미국(캘리포니아)리전 |


### S3 명령 사용 방법

```
aws --endpoint-url={endpoint} s3 {command} s3://{bucket}
```

| 이름 | 설명 |
|---|---|
| endpoint | https://api-storage.cloud.toast.com - 한국(판교)리전<br/>https://kr2-api-storage.cloud.toast.com - 한국(평촌)리전<br/>https://jp1-api-storage.cloud.toast.com - 일본(도쿄)리전<br/>https://us1-api-storage.cloud.toast.com - 미국(캘리포니아)리전 |
| command | AWS 명령줄 인터페이스 명령 |
| bucket | 버킷 이름 |


> [참고]
> AWS 명령줄 인터페이스는 AWS를 사용하기 위해 제공되는 도구이기 때문에 AWS 도메인을 사용하도록 설정되어 있습니다. 따라서 TAOST OBS를 사용하려면 반드시 매 명령마다 엔드포인트를 지정해야합니다.
> AWS 명령줄 인터페이스 명령은 [AWS CLI에서 상위 수준(s3) 명령 사용](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-services-s3-commands.html) 문서를 참조하세요.

<details>
<summary>버킷 생성</summary>

```
$ aws --endpoint-url=https://api-storage.cloud.toast.com s3 mb s3://example-bucket
make_bucket: example-bucket
```

</details>

<details>
<summary>버킷 목록 조회</summary>

```
$ aws --endpoint-url=https://api-storage.cloud.toast.com s3 ls
2020-07-13 10:07:13 example-bucket
```

</details>


<details>
<summary>버킷 조회</summary>

```
$ aws --endpoint-url=https://api-storage.cloud.toast.com s3 ls s3://example-bucket
2020-07-13 10:08:49     104389 0428b9e3e419d4fb7aedffde984ba5b3.jpg
2020-07-13 10:09:09      74448 6dd6d48eef889a5dab5495267944bdc6.jpg
```

</details>

<details>
<summary>버킷 삭제</summary>

```
$ aws --endpoint-url=https://api-storage.cloud.toast.com s3 ls s3://example-bucket
2020-07-13 10:08:49     104389 0428b9e3e419d4fb7aedffde984ba5b3.jpg
2020-07-13 10:09:09      74448 6dd6d48eef889a5dab5495267944bdc6.jpg
```

</details>

<details>
<summary>오브젝트 업로드</summary>

```
$  aws --endpoint-url=https://api-storage.cloud.toast.com s3 cp ./3b5ab489edffdea7bf4d914e3e9b8240.jpg s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
upload: ./3b5ab489edffdea7bf4d914e3e9b8240.jpg to s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

</details>

<details>
<summary>오브젝트 다운로드</summary>

```
$ aws --endpoint-url=https://api-storage.cloud.toast.com s3 cp s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg ./3b5ab489edffdea7bf4d914e3e9b8240.jpg
download: s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg to ./0428b9e3e419d4fb7aedffde984ba5b3.jpg
```

</details>

<details>
<summary>오브젝트 삭제</summary>

```
$ aws --endpoint-url=https://api-storage.cloud.toast.com s3 rm s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
delete: s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

</details>


## AWS SDK
AWS는 여러가지 프로그래밍 언어를 위한 SDK를 제공하고 있습니다. S3 호환 API를 이용해 AWS SDK로 TOAST OBS를 사용할 수 있습니다.

> [참고]
> 이 문서에서는 Python과 Java SDK의 간단한 사용 예시만 설명합니다. 자세한 내용은 [AWS SDK](https://aws.amazon.com/ko/tools) 문서를 참조하세요.


AWS SDK를 사용하기 위해 필요한 주요 파라미터는 다음과 같습니다.

| 이름 | 설명 |
|---|---|
| access | 자격 증명 접근 키 |
| secret | 자격 증명 비밀 키 |
| region name | KR1 - 한국(판교)리전<br/>KR2 - 한국(평촌)리전<br/>JP1 - 일본(도쿄)리전<br/>US1 - 미국(캘리포니아)리전 |
| endpoint | https://api-storage.cloud.toast.com - 한국(판교)리전<br/>https://kr2-api-storage.cloud.toast.com - 한국(평촌)리전<br/>https://jp1-api-storage.cloud.toast.com - 일본(도쿄)리전<br/>https://us1-api-storage.cloud.toast.com - 미국(캘리포니아)리전 |


### Boto3 - Python SDK

<details>
<summary>Boto3 클라이언트 클래스</summary>

```python
# boto3example.py
import boto3

class Boto3Example(object):
    _REGION = '{region name}'
    _ENDPOINT = '{endpoint}'
    _ACCESS = '{access}'
    _SECRET = '{secret}'

    def __init__(self):
        self.s3 = boto3.client(service_name='s3',
                               region_name=self._REGION,
                               endpoint_url=self._ENDPOINT,
                               aws_access_key_id=self._ACCESS,
                               aws_secret_access_key=self._SECRET)

```

</details>

<details>
<summary>버킷 생성</summary>

```python
    def create_bucket(self, bucket_name):
        return self.s3.create_bucket(Bucket=bucket_name)
```

</details>

<details>
<summary>버킷 목록 조회</summary>

```python
    def list_buckets(self):
        response = self.s3.list_buckets()
        return response.get('Buckets')
```

</details>

<details>
<summary>버킷 조회(오브젝트 목록 조회)</summary>

```python
    def list_objs(self, bucket_name):
        response = self.s3.list_objects_v2(Bucket=bucket_name)
        return response.get('Contents')
```

</details>

<details>
<summary>버킷 삭제</summary>

```python
    def delete_bucket(self, bucket_name):
        return self.s3.delete_bucket(Bucket=bucket_name)
```

</details>

<details>
<summary>오브젝트 업로드</summary>

```python
    def upload(self, bucket_name, key, filename):
        with open(filename, 'rb') as fd:
            return self.s3.put_object(Bucket=bucket_name, Key=key, Body=fd)
```

</details>

<details>
<summary>오브젝트 다운로드</summary>

```python
    def download(self, bucket_name, key, filename):
        response = self.s3.get_object(Bucket=bucket_name, Key=key)

        with io.FileIO(filename, 'w') as fd:
            for chunk in response['Body']:
                fd.write(chunk)
        response.pop('Body')

        return response
```

</details>

<details>
<summary>오브젝트 삭제</summary>

```python
    def delete(self, bucket_name, key):
        return self.s3.delete_object(Bucket=bucket_name, Key=keys)
```

</details>


### Java SDK

<details>
<summary>Java SDK 클라이언트 클래스</summary>

```java
// AwsSdkExapmple.java
public class AwsSdkExapmple {
    private static final String access = "{access}";
    private static final String secret = "{secret}";
    private static final String region = "{region name}";
    private static final String ednpoint = "{endpoint}";

    private AmazonS3 s3Client;

    public AwsSdkExapmple() {
        BasicAWSCredentials awsCredentials = new BasicAWSCredentials(access, secret);
        s3Client = AmazonS3ClientBuilder.standard()
                .withEndpointConfiguration(new AwsClientBuilder.EndpointConfiguration(ednpoint, region))
                .withCredentials(new AWSStaticCredentialsProvider(awsCredentials))
                .enablePathStyleAccess()
                .disableChunkedEncoding()
                .build();
    }
}
```

</details>

<details>
<summary>버킷 생성</summary>

```java
    public String createBucket(String bucketName) {
        Bucket bucket = s3Client.createBucket(bucketName);
        return bucket.toString();
    }
```

</details>

<details>
<summary>버킷 목록 조회</summary>

```java
    public List<Bucket> listBuckets() {
        return s3Client.listBuckets();
    }
```

</details>

<details>
<summary>버킷 조회(오브젝트 목록 조회)</summary>

```java
    public ListObjectsV2Result listObjects(String bucketName) {
        return s3Client.listObjectsV2(bucketName);
    }
```

</details>

<details>
<summary>버킷 삭제</summary>

```java
    public void deleteBucket(String bucketName) {
        s3Client.deleteBucket(bucketName);
    }
```

</details>

<details>
<summary>오브젝트 업로드</summary>

```java
    public String uploadObject(String bucketName, String objKeyName, String filePath) {
        PutObjectResult result = s3Client.putObject(bucketName, objKeyName, new File(filePath));
        return result.getETag();
    }
```

</details>

<details>
<summary>오브젝트 다운로드</summary>

```java
    public String downloadObject(String bucketName, String objKeyName, String filePath) {
        GetObjectRequest request = new GetObjectRequest(bucketName, objKeyName);
        ObjectMetadata metadata = s3Client.getObject(request, new File(filePath));
        return metadata.getETag();
    }
```

</details>

<details>
<summary>오브젝트 삭제</summary>

```java
    public void deleteObject(String bucketName, String objKeyName) {
        s3Client.deleteObject(bucketName, objKeyName);
    }
```

</details>
