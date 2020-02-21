## Storage > Object Storage > AWS S3 호환 API 가이드
오브젝트 스토리지는 AWS의 오브젝트 스토리지 S3가 제공하는 API 형태의 호환 API를 제공합니다. AWS S3 API를 사용하도록 개발된 애플리케이션을 설정만 변경하여 그대로 사용할 수 있습니다. 또한 AWS SDK를 사용할 수 도 있습니다.

제공하는 S3 호환 API는 다음과 같습니다.

| S3 API Method | 용도 |
| --- | --- |
| [PUT Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUT.html) | 버킷(컨테이너) 생성 |
| [HEAD Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketHEAD.html) | 버킷 정보 조회 |
| [DELETE Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketDELETE.html) | 버킷 삭제 |
| [PUT Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUTacl.html) | 버킷 ACL 설정 |
| [GET Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETacl.html) | 버킷 ACL 조회 |
| [GET Bucket Location](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETlocation.html) | 버킷이 있는 리전 조회 |
| [GET Bucket List Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGET.html) | 버킷의 개체 목록 조회 |
| [GET Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectGET.html) | 개체 다운로드 |
| [HEAD Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectHEAD.html) | 개체 정보 조회 |
| [PUT Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectPUT.html) | 개체 업로드 |
| [PUT Object Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectCOPY.html) | 개체 복사 |
| [DELETE Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectDELETE.html) | 개체 삭제 |
| [Initiate Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadInitiate.html) | 멀티 파트 업로드 초기화 |
| [Upload Part](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPart.html) | 파트 업로드 |
| [Upload Part Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPartCopy.html) | 파트 복사 |
| [Complete Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadComplete.html) | 멀티 파트 업로드 완료 |
| [Abort Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadAbort.html) | 멀티 파트 업로드 중단 |
| [List Parts](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | 멀티 파트 개체의 파트 개체 리스트 |
| [List Multipart Uploads](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | 업로드 진행 중인 멀티 파트 개체의 파트 개체 리스트 |
| [DELETE Multiple Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/multiobjectdeleteapi.html) | 멀티 파트 개체 삭제 |

이 문서는 기본적인 API 사용 방법만을 설명합니다. 고급 기능을 사용하기 위해서는 [AWS S3 API 가이드](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)를 참고하거나, [AWS SDK](https://aws.amazon.com/ko/tools) 사용을 권장합니다.

## EC2 자격 증명(EC2 Credential)

### 자격 증명 등록
S3 호환 API를 사용하기 위해서는 먼저 AWS EC2 형태의 자격 증명을 등록해야 합니다. 자격 증명 등록은 인증 토큰이 필요합니다. 인증 토큰 발급은 [오브젝트 스토리지 API 가이드](/Storage/Object%20Storage/ko/api-guide/#tenant-id-api-endpoint)를 참조하십시오.

**[Method, URL]**
```
POST    https://api-compute.cloud.toast.com/identity/v2.0/users/{User ID}/credentials/OS-EC2
Content-Type: application/json
X-Auth-Token: {token-id}
```

**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| X-Auth-Token | Header | String | 발급받은 토큰 ID |
| user-id | URL | String | 사용자 ID, 인증 토큰에 포함되어 있음 |
| tenant_id | Body | String | Tenant ID. API Endpoint 설정 대화창에서 확인 가능 |

> [주의]
> 자격 증명 등록에 사용하는 사용자 UUID는 이메일 형태의 TOAST 계정 ID가 아닙니다. 인증 토큰 발급시 확인할 수 있습니다.

**[Request Body]**
```
{
  "tenant_id": "{tenant_id}"
}
```

**[Response Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| access | Plain | String | 자격 증명 접근 키 |
| secret | Plain | String | 자격 증명 비밀 키 |

**[Response Body]**
```
{
  "credentials": [
    {
      "access": "{access}",
      "tenant_id": "{tenant_id}",
      "secret": "{secret}",
      "user_id": "{user_id}",
      "trust_id": null
    }
  ]
}
```

### EC2 자격 증명 조회
등록한 EC2 자격 증명을 조회합니다.

**[Method, URL]**
```
GET https://gov-api-compute.cloud.toast.com/identity/v2.0/users/{user-id}/credentials/OS-EC2
X-Auth-Token: {token-id}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| X-Auth-Token | Header | String | 발급받은 토큰 ID |
| user-id | URL | String | 사용자 ID, 인증 토큰에 포함되어 있음 |

**[Response Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| access | Plain | String | 자격 증명 접근 키 |
| secret | Plain | String | 자격 증명 비밀 키 |

**[Response Body]**
```
{
  "credentials": [
    {
      "access": "{access}",
      "tenant_id": "{tenant_id}",
      "secret": "{secret}",
      "user_id": "{user_id}",
      "trust_id": null
    }
  ]
}
```

### EC2 자격 증명 삭제
등록한 EC2 자격 증명을 삭제합니다.

**[Method, URL]**
```
GET https://gov-api-compute.cloud.toast.com/identity/v2.0/users/{user-id}/credentials/OS-EC2/{access}
X-Auth-Token: {token-id}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| X-Auth-Token | Header | String | 발급받은 토큰 ID |
| user-id | URL | String | 사용자 ID, 인증 토큰에 포함되어 있음 |
| access | Plain | String | 자격 증명 접근 키 |

> [참고]
> 이 요청은 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## 서명(signature) 생성
S3 API를 사용하기 위해서는 자격 증명 키를 이용해 서명을 생성해야 합니다. 서명 방법은 [AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) 문서를 참조하십시오.

서명 생성에 필요한 정보는 다음과 같습니다.

| 이름 | 값 |
|---|---|
| 알고리즘 | AWS4-HMAC-SHA256 |
| 서명 시각 | YYYYMMDDThhmmssZ 형태 |
| 서비스 이름 | s3 |
| 리전 이름 | KR1 - 한국(판교)리전 |
| 비밀 키 | 자격 증명 비밀 키 |

> [참고]
> S3 호환 API는 현재 한국(판교)리전에서만 제공되고 있습니다.

## 버킷(Bucket)
### 버킷 생성
버킷(컨테이너)을 생성합니다.

**[Method, URL]**
```
PUT /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| bucket | URL | String | 버킷 이름 |
| Date | Header | String | 요청 시각 |
| Authorization | Header | String | 자격 증명 접근 키와 서명으로 구성 |

**[Response Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| Location | Plain | String | 생성한 버킷 경로 |

**[Response Body]**
```json
{
    "ResponseMetadata": {
        "RequestId": "{request_id}",
        "HostId": "{host_id}",
        "HTTPStatusCode": 200,
        "HTTPHeaders": {
            "x-amz-id-2": "{request_id}",
            "content-length": "{content-length}",
            "x-amz-request-id": "{request_id}",
            "content-type": "application/xml",
            "x-trans-id": "{request_id}",
            "x-openstack-request-id": "{request_id}",
            "date": "{request_datetime}"
        },
        "RetryAttempts": 0
    },
    "Location": "/{bucket_name}"
}
```

### 버킷 목록 조회
버킷 목록을 조회합니다.
```
GET /

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| Date | Header | String | 요청 시각 |
| Authorization | Header | String | 자격 증명 접근 키와 서명으로 구성 |

**[Response Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| Buckets.Name | Plain | String | 버킷 이름 |
| Buckets.CreationDate | Plain | String | 생성 시각 |

```json
{
    "ResponseMetadata": {
        "HTTPStatusCode": 200,
        "...": "..."
    },
    "Buckets": [
        {
            "Name": "{bucket_name}",
            "CreationDate": "{creation_date}"
        }
    ],
    "Owner": {}
}
```

### 버킷 조회
지정한 버킷의 정보와 내부에 저장된 개체 목록을 조회합니다.
```
GET /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| bucket | URL | String | 버킷 이름 |
| Date | Header | String | 요청 시각 |
| Authorization | Header | String | 자격 증명 접근 키와 서명으로 구성 |

**[Response Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| Contents.Key | Plain | String | 개체 이름 |
| Contents.LastModified | Plain | String | 개체의 최근 수정 시각, YYYY-MM-DDThh:mm:ssZ |
| Contents.ETag | Plain | String | 개체의 MD5 hash 값 |
| Contents.Size | Plain | String | 개체의 데이터 크기 |
| Contents.StorageClass | Plain | String | 개체가 저장된 저장소 종류 |
| Name | Plain | String | 버킷 이름 |
| KeyCount | Plain | String | 목록의 개체 수 |

```json
{
    "ResponseMetadata": {
        "HTTPStatusCode": 200,
        "...": "..."
    },
    "IsTruncated": false,
    "Contents": [
        {
            "Key": "{object_name}",
            "LastModified": "{last_modified_date}",
            "ETag": "{etag}",
            "Size": 0,
            "StorageClass": "{storage_class}"
        }
    ],
    "Name": "{bucket_name}",
    "Prefix": "",
    "MaxKeys": 0,
    "EncodingType": "{encoding_type}",
    "KeyCount": 0
}
```

### 버킷 삭제
지정한 버킷을 삭제합니다. 삭제할 버킷은 비어있어야 합니다.
```
DELETE /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| bucket | URL | String | 버킷 이름 |
| Date | Header | String | 요청 시각 |
| Authorization | Header | String | 자격 증명 접근 키와 서명으로 구성 |

**[Response Parameters]**

```json
{
    "ResponseMetadata": {
        "HTTPStatusCode": 204,
        "...": "..."
    }
}
```

## 개체
### 개체 업로드
```
PUT /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| bucket | URL | String | 버킷 이름 |
| obj | URL | String | 개체 이름 |
| Date | Header | String | 요청 시각 |
| Authorization | Header | String | 자격 증명 접근 키와 서명으로 구성 |

**[Response Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| Contents.Key | Plain | String | 개체 이름 |
| Contents.LastModified | Plain | String | 개체의 최근 수정 시각, YYYY-MM-DDThh:mm:ssZ |
| Contents.ETag | Plain | String | 개체의 MD5 hash 값 |
| Contents.Size | Plain | String | 개체의 데이터 크기 |
| Contents.StorageClass | Plain | String | 개체가 저장된 저장소 종류 |
| Name | Plain | String | 버킷 이름 |
| KeyCount | Plain | String | 목록의 개체 수 |

```json
{
    "ResponseMetadata": {
        "HTTPStatusCode": 200,
        "...": "..."
    },
    "IsTruncated": false,
    "Contents": [
        {
            "Key": "{object_name}",
            "LastModified": "{last_modified_date}",
            "ETag": "{etag}",
            "Size": 0,
            "StorageClass": "{storage_class}"
        }
    ],
    "Name": "{bucket_name}",
    "Prefix": "",
    "MaxKeys": 0,
    "EncodingType": "{encoding_type}",
    "KeyCount": 0
}
```

### 개체 다운로드
```
PUT /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| bucket | URL | String | 버킷 이름 |
| obj | URL | String | 개체 이름 |
| Date | Header | String | 요청 시각 |
| Authorization | Header | String | 자격 증명 접근 키와 서명으로 구성 |

**[Response Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| LastModified | Plain | String | 개체의 최근 수정 시각, YYYY-MM-DDThh:mm:ssZ |
| ContentLength | Plain | String | 다운로드 한 개체의 데이터 크기 |
| ETag | Plain | String | 개체의 MD5 hash 값 |
| ContentType | Plain | String | 개체의 콘텐츠 타입 |

```json
{
    "ResponseMetadata": {
        "HTTPStatusCode": 200,
        "...": "..."
    },
    "LastModified": "{last_modified_date}",
    "ContentLength": 0,
    "ETag": "{etag}",
    "ContentType": "{content_type}",
    "Metadata": {}
}
```

### 개체 삭제
지정한 개체를 삭제합니다.

```
DELETE /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```
**[Request Parameters]**

| 이름 | 종류 | 속성 | 설명 |
|---|---|---|---|
| bucket | URL | String | 버킷 이름 |
| obj | URL | String | 개체 이름 |
| Date | Header | String | 요청 시각 |
| Authorization | Header | String | 자격 증명 접근 키와 서명으로 구성 |

**[Response Parameters]**

```json
{
    "ResponseMetadata": {
        "HTTPStatusCode": 204,
        "...": "..."
    }
}
```
