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
POST    https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{User ID}/credentials/OS-EC2

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

### Get

Get registered EC2 credential.

**[Method, URL]**

```
GET   https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2

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
DELETE   https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

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
| Region Name   | KR1 - Korea (Pangyo) region    |
| Secret Key 키 | Credential secret key          |

> [Note]
> APIs compatible with S3 are provided only within Korea (Pangyo) region as of March 2020.

## Buckets

### Create  

Create a bucket (container).

```
PUT /{bucket}

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request

This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | O      | String   | Comprised of credential access key and signature |

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
