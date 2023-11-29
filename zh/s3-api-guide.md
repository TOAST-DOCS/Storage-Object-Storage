## Storage > Object Storage > Amazon S3 Compatible API Guide
NHN Cloud Object Storage provides API compatible with S3 API of AWS object storage. Therefore, you can use applications developed to use the Amazon S3 API as is, with only a few configuration changes.

The following Amazon S3 compatible API is provided.

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
| [Initiate Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadInitiate.html) | Initialize multipart upload            |
| [Upload Part](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPart.html) | Upload part                             |
| [Upload Part Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPartCopy.html) | Copy part                               |
| [Complete Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadComplete.html) | Complete multipart upload              |
| [Abort Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadAbort.html) | Abort multipart upload                 |
| [List Parts](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | List multipart objects                 |
| [List Multipart Uploads](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | List multipart objects under uploading |
| [DELETE Multiple Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/multiobjectdeleteapi.html) | Delete multipart objects               |

This document describes only the basic usage of API. To use advanced features, it is recommended that you see [Amazon S3 API Guide](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html) or use [AWS SDK](https://aws.amazon.com/tools).

## S3 API Credentials

### Obtain S3 API Credentials
To use Amazon S3 compatible API, you must first obtain S3 API credentials in the form of AWS EC2. Credentials can be issued using the web console or API. To obtain credentials using the web console, refer to [S3 API Credentials](console-guide/#s3-api-credentials).

To obtain credentials using the API, an authentication token is required. To obtain the authentication token, refer to [Object Storage API Guide](api-guide/#tenant-id-api-endpoint).

```
POST    https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

#### Request

| Name         | Type   | Format | Required | Description                                                  |
| ------------ | ------ | ------ | -------- | ------------------------------------------------------------ |
| X-Auth-Token | Header | String | O        | Issued token ID                                              |
| api-user-id      | URL    | String | O        | API user ID, which can be found on the API Endpoint setting dialog box   |
| tenant_id    | Body   | String | O        | Tenant ID, which can be found on the API Endpoint setting dialog box |

> [Note]
> `<api-user-id>` can be found in the **API User ID** item in the API Endpoint settings dialog box on the console or in the **access.user.id** field in the response body of the Authentication Token Issuance API.
> To use the Authentication Token Issuance API, refer to [Authentication Token Issuance](api-guide/#authentication-token-issuance) in the API guide.
> 
> S3 API credentials have no expiration date, and up to 3 credentials can be issued per project for each user.

<!-- This is a comment for line break, so it must be included. -->

> [Caution]
> If the S3 API credentials key is leaked, anyone can access the object using the leaked key. If the key is leaked, it is recommended to delete the leaked credentials and obtain a new one.

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
| access | Body | String | S3 API credentials access key |
| secret | Body | String | S3 API credentials secret key |

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

### Get S3 API Credentials
Retrieves the issued S3 API credentials.

**[Method, URL]**

```
GET   https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2

X-Auth-Token: {token-id}
```
#### Request
This API does not require a request body.

| Name         | Type   | Format | Required | Description                               |
| ------------ | ------ | ------ | -------- | ----------------------------------------- |
| X-Auth-Token | Header | String | O        | Issued token ID                           |
| user-id      | URL    | String | O        | User ID, included in the authentication token |

#### Response

| Name   | Type | Format | Description           |
| ------ | ---- | ------ | --------------------- |
| access | Body | String | S3 API credentials access key |
| secret | Body | String | S3 API credentials secret key |

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

### Delete S3 API Credentials
Deletes the issued S3 API credentials.

**[Method, URL]**

```
DELETE   https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```
#### Request
This API does not require a request body.

| Name         | Type   | Format | Required | Description                               |
| ------------ | ------ | ------ | -------- | ----------------------------------------- |
| X-Auth-Token | Header | String | O        | Issued token ID                           |
| user-id      | URL    | String | O        | User ID, included in the authentication token |
| access       | URL    | String | O        | S3 API credentials access key                     |

#### Response
This API does not return request body. When the request is appropriate, return status code 204.

## Create Signature
To use S3 API, you must create a signature use credentials. Regarding how to sign, see [AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html).

The following information is required to create a signature.

| Name          | Value                          |
| ------------- | ------------------------------ |
| Algorithm     | AWS4-HMAC-SHA256               |
| Signed Time   | In the ZssmmhhTDDMMYYYY format |
| Service Name  | s3                             |
| Region Name   | KR1 - Korea (Pangyo) region<br/>KR2 - KOREA (Pyeongchon) Region<br/>KR3 - KOREA (Gwangju) Region<br/>JP1 - JAPAN (Tokyo) Region<br/>US1 - USA (California) Region   |
| Secret Key    | S3 API credentials secret key          |


## Bucket
### Create Bucket
Creates a bucket. Bucket names must follow Amazon S3's naming rules.

* Bucket names must be between 3 and 63 characters long.
* Bucket names can consist only of lowercase letters, numbers, dots (.), and hyphens (-).
* Bucket names must begin and end with a letter or number.
* Bucket names must not be formatted as an IP address (for example, 192.168.5.4).
* Bucket names must not start with the prefix xn--.

For more details, see [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html).

```
PUT /{bucket}

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

> [Note]
> If a container name made via web console or object storage API violates any bucket naming rules, it cannot be accessed with S3 compatible API.

#### Request
This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of S3 API credentials access key and signature |

#### Response
This API does not return a response body. It returns a status code of 200 if the request is valid.

| Name                            | Type | Format  | Description                 |
| ------------------------------- | ---- | ------- | --------------------------- |
| Location | Header | String | Path for created bucket |

### List Buckets
Retrieves bucket lists.

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
| Authorization | Header | String | O        | Comprised of S3 API credentials access key and signature |

#### Response
If the request is valid, returns a status code of 200 and a bucket list in XML format.

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

### Get Bucket
Retrieves the information of the specified bucket and the list of objects that are stored in the bucket.

```
GET /{bucket}

Date: 22:22:22 +0000, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

> [Note]
> If a bucket name made via web console or object storage API violates any bucket naming rules, it cannot be accessed with S3 compatible API.

#### Request
This API does not require a request body.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of S3 API credentials access key and signature |

#### Response
If the request is valid, returns a status code of 200 and a object list in XML format.

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

### Delete Bucket
Deletes the specified bucket. The bucket to be deleted must be empty.

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
| Authorization | Header | String | O        | Comprised of S3 API credentials access key and signature |

#### Response
This API does not return a response body; it returns status code 204 if the request is valid.

## Object
### Upload Object
Uploads an object to the specified bucket.

```
PUT /{bucket}/{obj}

Date: 22:22:22 +0000, Sat, 22 Feb 2020
Authorization: AWS {access}:{signature}
```

#### Request
This API does not return a response body. It returns status code 204 if the request is valid.

| Name          | Type   | Format | Required | Description                                      |
| ------------- | ------ | ------ | -------- | ------------------------------------------------ |
| bucket        | URL    | String | O        | Bucket name                                      |
| obj           | URL    | String | O        | Object name                                      |
| Date          | Header | String | O        | Request time                                     |
| Authorization | Header | String | O        | Comprised of S3 API credentials access key and signature |

#### Response
This API does not return a response body. It returns a status code of 200 if the request is valid.

| Name                            | Type | Format  | Description                 |
| ------------------------------- | ---- | ------- | --------------------------- |
| ETag | Header | String | MD5 hash value of the object|
| Last-Modified | Header | String | The object's last modified date (e.g. Wed, 01 Mar 2006 12:00:00 GMT) |

### Download Object
Downloads an object.

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
| Authorization | Header | String | O        | Comprised of S3 API credentials access key and signature |

#### Response
If the request is valid, returns the status code of 200.

| Name                            | Type | Format  | Description                                             |
| ------------------------------- | ---- | ------- | ------------------------------------------------------- |
| Last-Modified | Header | String | The object's last modified date (e.g. Wed, 01 Mar 2006 12:00:00 GMT) |
| ETag | Header | String | MD5 hash value of the obejct |

### Delete Object
Delete the specified object.

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
| Authorization | Header | String | O        | Comprised of S3 API credentials access key and signature |

#### Response
This API does not return a response body. It returns status code 204 if the request is valid.

## AWS Command Line Interface (CLI)
You can use NHN Cloud Object Storage with [AWS Command Line Interface](https://aws.amazon.com/cli/) using the S3 compatible API.

### Installation
The AWS Command Line Interface (CLI) is provided as a Python package, which can be installed by using the Python package manager (pip).  

```shell
$ sudo pip install awscli
```

### Configuration
To use AWS CLI, you must set up S3 API credentials and environment first.

```shell
$ aws configure
AWS Access Key ID [None]: {access}
AWS Secret Access Key [None]: {secret}
Default region name [None]: {region name}
Default output format [None]: json
```

| Name | Description |
|---|---|
| access | S3 API credentials access key |
| secret | S3 API credentials secret key |
| region name | KR1 - Korea (Pangyo) Region <br/>KR2 - Korea (Pyeongchon) Region <br/>KR3 - KOREA (Gwangju) Region<br/>JP1 - Japan (Tokyo) Region <br/>US1 - US (California) Region |

### How to Use the S3 Commands

```shell
aws --endpoint-url={endpoint} s3 {command} s3://{bucket}
```

| Name | Description |
|---|---|
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - Korea (Pangyo) region <br/>https://kr2-api-object-storage.nhncloudservice.com - Korea (Pyeongcheon) region<br/>https://kr3-api-object-storage.nhncloudservice.com - Korea(Gwangju) region<br/>https://jp1-api-object-storage.nhncloudservice.com - Japan (Tokyo) region <br/>https://us1-api-object-storage.nhncloudservice.com - US (California) region |
| command | Command for AWS Command Line Interface |
| bucket | Bucket name |


> [Note]
> Since AWS CLI is provided to use AWS, it is configured to use AWS domain. Therefore, to use NHN Cloud Object Storage, it is required to specify endpoint for every command.
> Regarding commands for AWS Command Line Interface, see [Using high-level (s3) commands with the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html).

<details>
<summary>Create Bucket</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 mb s3://example-bucket
make_bucket: example-bucket
```

</details>

<details>
<summary>List Buckets</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls
2020-07-13 10:07:13 example-bucket
```

</details>


<details>
<summary>Get Bucket</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls s3://example-bucket
2020-07-13 10:08:49     104389 0428b9e3e419d4fb7aedffde984ba5b3.jpg
2020-07-13 10:09:09      74448 6dd6d48eef889a5dab5495267944bdc6.jpg
```

</details>

<details>
<summary>Delete Bucket</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rb s3://example-bucket
remove_bucket: example-bucket
```

</details>

<details>
<summary>Upload Object</summary>

```shell
$  aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp ./3b5ab489edffdea7bf4d914e3e9b8240.jpg s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
upload: ./3b5ab489edffdea7bf4d914e3e9b8240.jpg to s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

<blockquote>
[Note]
</br>
If the object is larger than 8 MB, the AWS Command Line Interface splits the object into multiple parts and uploads them. The part object is stored in a bucket called <code style="display: inline;">{bucket}+segments</code> It is saved in the form of part-number<code style="display: inline;">{object-name}/{upload-id}/{part-number}</code>, and when all parts are uploaded, an object linked to the part object is created in the bucket requested for upload.
</br></br>
The <code style="display: inline;">{bucket}+segments</code> bucket where the part object is stored cannot be accessed through the S3 compatible API, but can be accessed through the Object Storage API or the console.
</br></br>
The ETag of a multipart object is an MD5 hashed value by converting the ETag values of each part object into binary data and concatenating them in order.
</blockquote>

<blockquote>
[Caution]
</br>
If you delete some or all parts of an object uploaded as multipart, the object cannot be accessed.
</blockquote>

</details>

<details>
<summary>Download Object</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg ./3b5ab489edffdea7bf4d914e3e9b8240.jpg
download: s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg to ./0428b9e3e419d4fb7aedffde984ba5b3.jpg
```

</details>

<details>
<summary>Delete Object</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rm s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
delete: s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

</details>


## AWS SDK
AWS provides SDKs for many types of programming languages. By using the S3 compatible API, you can use NHN Cloud Object Storage with AWS SDK.

> [Note]
> For more information, see [AWS SDK](https://aws.amazon.com/ko/tools).

The following are the major parameters required to use AWS SDK.

| Name | Description |
|---|---|
| access | S3 API credentials access key |
| secret | S3 API credentials secret key |
| region name | KR1 - Korea (Pangyo) region <br/>KR2 - Korea (Pyeongchon) region<br/>KR3 - Korea (Gwangju) region<br/>JP1 - Japan (Tokyo) region <br/>US1 - US (California) region |
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - Korea (Pangyo) region<br/>https://kr2-api-object-storage.nhncloudservice.com - Korea (Pyeongchon) region<br/>https://kr3-api-object-storage.nhncloudservice.com - Korea (Gwangju) region<br/>https://jp1-api-object-storage.nhncloudservice.com - Japan (Tokyo) region<br/>https://us1-api-object-storage.nhncloudservice.com - US (California) region | |

### Boto3 - Python SDK

> [Note]
> For more information, see [AWS SDK for Python (Boto3)](https://docs.aws.amazon.com/ko_kr/pythonsdk/?icmpid=docs_homepage_sdktoolkits).

#### Context

<details>
<summary>Boto3 Client Class</summary>

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
<summary>Create Bucket</summary>

```python
def create_bucket(self, bucket_name):
  try:
      return self.s3.create_bucket(Bucket=bucket_name)
  except ClientError as e:
      raise RuntimeError(e)
```

</details>

<details>
<summary>List Buckets</summary>

```python
def list_buckets(self):
    try:
        return self.s3.list_buckets().get('Buckets')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Get Bucket (List Objects)</summary>

```python
def list_objs(self, bucket_name):
    try:
        return self.s3.list_objects_v2(Bucket=bucket_name).get('Contents')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Delete Bucket</summary>

```python
def delete_bucket(self, bucket_name):
    try:
        return self.s3.delete_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>Upload Object</summary>

<blockquote>
<p>[Note]
The number of part objects is determined by the size of the object being uploaded and the part size you set. The default part size is 8MiB, and the maximum number of part objects is 1000.</p>
</blockquote>

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
<summary>Download Object</summary>

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
<summary>Delete Object</summary>

```python
def delete(self, bucket_name, key):
    try:
        return self.s3.delete_object(Bucket=bucket_name, Key=key)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

### Java SDK

> [Note]
> For more information, see [AWS SDK for Java](https://docs.aws.amazon.com/ko_kr/sdk-for-java/index.html).

#### Context

<details>
<summary>Java SDK Client Class</summary>

```java
// AwsSdkExample.java
public class AwsSdkExample {
    private static final String access = "{access}";
    private static final String secret = "{secret}";
    private static final String region = "{region name}";
    private static final String ednpoint = "{endpoint}";

    private AmazonS3 s3Client;

    public AwsSdkExample() {
        BasicAWSCredentials awsCredentials =
            new BasicAWSCredentials(access, secret);
        s3Client = AmazonS3ClientBuilder.standard()
                    .withEndpointConfiguration(
                new AwsClientBuilder.EndpointConfiguration(ednpoint, region)
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
<summary>Create Bucket</summary>

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
<summary>List Buckets</summary>

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
<summary>Get Bucket (List Objects)</summary>

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
<summary>Delete Bucket</summary>

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
<summary>Upload Object</summary>

<blockquote>
<p>[Note]
The number of part objects is determined by the size of the object being uploaded and the part size you set. The default part size is 5MiB, and the maximum number of part objects is 1000.</p>
</blockquote>

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
<summary>Download Object</summary>

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

</details>
<summary>Delete Object</summary>

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

### .NET SDK

> [Note]
> For more information, see [AWS SDK for .NET](https://docs.aws.amazon.com/ko_kr/sdk-for-net/?icmpid=docs_homepage_sdktoolkits).

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
<summary>Create Bucket</summary>

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
<summary>List Buckets</summary>

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
<summary>List Bucket (List objects)</summary>

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
<summary>Delete Bucket</summary>

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
<summary>Upload Object</summary>

<blockquote>
<p>[Note]
The number of part objects is determined by the size of the object being uploaded and the part size you set. The default part size is 5MiB, and the maximum number of part objects is 1000.</p>
</blockquote>

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
<summary>Download Object</summary>

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
<summary>Delete Object</summary>

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

