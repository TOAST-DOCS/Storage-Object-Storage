## Storage > Object Storage > AWS S3 호환 API 가이드
오브젝트 스토리지는 AWS의 오브젝트 스토리지 S3 API와 호환되는 API를 제공합니다. 따라서 AWS S3 API를 사용하도록 개발된 애플리케이션을 설정만 변경하여 그대로 사용할 수 있습니다.

제공하는 S3 호환 API는 다음과 같습니다.

| S3 API 메서드 | 용도 |
| --- | --- |
| [PUT Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUT.html) | 버킷 생성 |
| [HEAD Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketHEAD.html) | 버킷 정보 조회 |
| [DELETE Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketDELETE.html) | 버킷 삭제 |
| [PUT Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUTacl.html) | 버킷 ACL 설정 |
| [GET Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETacl.html) | 버킷 ACL 조회 |
| [GET Bucket Location](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETlocation.html) | 버킷이 있는 리전 조회 |
| [GET Bucket List Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGET.html) | 버킷의 오브젝트 목록 조회 |
| [GET Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectGET.html) | 오브젝트 다운로드 |
| [HEAD Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectHEAD.html) | 오브젝트 정보 조회 |
| [PUT Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectPUT.html) | 오브젝트 업로드 |
| [PUT Object Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectCOPY.html) | 오브젝트 복사 |
| [DELETE Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectDELETE.html) | 오브젝트 삭제 |
| [Initiate Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadInitiate.html) | 멀티 파트 업로드 초기화 |
| [Upload Part](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPart.html) | 파트 업로드 |
| [Upload Part Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPartCopy.html) | 파트 복사 |
| [Complete Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadComplete.html) | 멀티 파트 업로드 완료 |
| [Abort Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadAbort.html) | 멀티 파트 업로드 중단 |
| [List Parts](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | 멀티 파트 오브젝트의 파트 오브젝트 리스트 |
| [List Multipart Uploads](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | 업로드 진행 중인 멀티 파트 오브젝트의 파트 오브젝트 리스트 |
| [DELETE Multiple Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/multiobjectdeleteapi.html) | 멀티 파트 오브젝트 삭제 |

이 문서는 기본적인 API 사용 방법만을 설명합니다. 고급 기능을 사용하려면 [AWS S3 API 가이드](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)를 참고하거나, [AWS SDK](https://aws.amazon.com/ko/tools) 사용을 권장합니다.

## EC2 자격 증명(EC2 Credential)

### 자격 증명 등록
S3 호환 API를 사용하려면 먼저 AWS EC2 형태의 자격 증명을 등록해야 합니다. 자격 증명을 등록하려면 인증 토큰이 필요합니다. 인증 토큰 발급은 [오브젝트 스토리지 API 가이드](/Storage/Object%20Storage/ko/api-guide/#tenant-id-api-endpoint)를 참고하십시오.

```
POST    https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{User ID}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 발급받은 토큰 ID |
| user-id | URL | String | O | 사용자 ID, 인증 토큰에 포함되어 있음 |
| tenant_id | Body | String | O | 사용자 Tenant ID. API Endpoint 설정 대화 상자에서 확인 가능 |

> [주의]
> 자격 증명 등록에 사용하는 사용자 ID는 이메일 형태의 TOAST 계정 ID가 아닙니다. 인증 토큰 발급 시 확인할 수 있습니다.

<details>
<summary>예시</summary>

```json
{
  "tenant_id": "84c9e9a51aea402e95389c08ac562ac5"
}
```

</details>

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| access | Body | String | 자격 증명 접근 키 |
| secret | Body | String | 자격 증명 비밀 키 |

<details>
<summary>예시</summary>

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

### EC2 자격 증명 조회
등록한 EC2 자격 증명을 조회합니다.

**[Method, URL]**
```
GET   https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2

X-Auth-Token: {token-id}
```
#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O |발급받은 토큰 ID |
| user-id | URL | String | O | 사용자 ID, 인증 토큰에 포함되어 있음 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| access | Body | String | 자격 증명 접근 키 |
| secret | Body | String | 자격 증명 비밀 키 |

<details>
<summary>예시</summary>

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

### EC2 자격 증명 삭제
등록한 EC2 자격 증명을 삭제합니다.

**[Method, URL]**
```
DELETE   https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```
#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 발급받은 토큰 ID |
| user-id | URL | String | O | 사용자 ID, 인증 토큰에 포함되어 있음 |
| access | URL | String | O | 자격 증명 접근 키 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## 서명(signature) 생성
S3 API를 사용하려면 자격 증명 키를 이용해 서명을 생성해야 합니다. 서명 방법은 [AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) 문서를 참고하십시오.

서명 생성에 필요한 정보는 다음과 같습니다.

| 이름 | 값 |
|---|---|
| 알고리즘 | AWS4-HMAC-SHA256 |
| 서명 시각 | YYYYMMDDThhmmssZ 형태 |
| 서비스 이름 | s3 |
| 리전 이름 | KR1 - 한국(판교)리전 |
| 비밀 키 | 자격 증명 비밀 키 |

> [참고]
> S3 호환 API는 2020년 3월 현재 한국(판교)리전에서만 제공됩니다.

## 버킷(Bucket)
### 버킷 생성
버킷(컨테이너)을 생성합니다.

```
PUT /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | O | String | 자격 증명 접근 키와 서명으로 구성 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ResponseMetadata | Body | Object | 응답 메타데이터 객체 |
| ResponseMetadata.HTTPStatusCode | Body | Integer | 응답 상태 코드 |
| Location | Body | String | 생성한 버킷 경로 |

<details>
<summary>예시</summary>

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
      "date": "Sat, 22 Feb 2020 22:22:22 GMT"
    },
    "RetryAttempts": 0
  },
  "Location": "/new-container"
}
```

</details>

### 버킷 목록 조회
버킷 목록을 조회합니다.
```
GET /

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | 자격 증명 접근 키와 서명으로 구성 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ResponseMetadata | Body | Object | 응답 메타데이터 객체 |
| ResponseMetadata.HTTPStatusCode | Body | Integer | 응답 상태 코드 |
| Buckets.Name | Body | String | 버킷 이름 |
| Buckets.CreationDate | Body | String | 생성 시각 |

<details>
<summary>예시</summary>

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
      "date": "Sat, 22 Feb 2020 22:22:22 GMT"
    },
    "RetryAttempts": 0
  },
  "Buckets": [
    {
      "Name": "new-container",
      "CreationDate": "2020-02-22T22:22:22+00:00"
    }
  ],
  "Owner": {}
}
```

</details>

### 버킷 조회
지정한 버킷의 정보와 내부에 저장된 오브젝트 목록을 조회합니다.
```
GET /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | 자격 증명 접근 키와 서명으로 구성 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ResponseMetadata | Body | Object | 응답 메타데이터 객체 |
| ResponseMetadata.HTTPStatusCode | Body | Integer | 응답 상태 코드 |
| Contents | Body | Object | 오브젝트 목록 객체 |  
| Contents.Key | Body | String | 오브젝트 이름 |
| Contents.LastModified | Body | String | 오브젝트의 최근 수정 시각, YYYY-MM-DDThh:mm:ssZ |
| Contents.ETag | Body | String | 오브젝트의 MD5 해시값 |
| Contents.Size | Body | String | 오브젝트의 크기 |
| Contents.StorageClass | Body | String | 오브젝트가 저장된 저장소 종류 |
| Name | Body | String | 버킷 이름 |
| KeyCount | Body | Integer | 목록의 오브젝트 수 |

<details>
<summary>예시</summary>

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
      "date": "Sat, 22 Feb 2020 22:22:22 GMT"
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

### 버킷 삭제
지정한 버킷을 삭제합니다. 삭제할 버킷은 비어 있어야 합니다.
```
DELETE /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | 자격 증명 접근 키와 서명으로 구성 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ResponseMetadata | Body | Object | 응답 메타데이터 객체 |
| ResponseMetadata.HTTPStatusCode | Body | Integer | 응답 상태 코드 |

<details>
<summary>예시</summary>

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
            "date": "Sat, 22 Feb 2020 22:22:22 GMT"
        },
        "RetryAttempts": 0
    }
}
```

</details>

## 오브젝트
### 오브젝트 업로드
지정한 버킷에 오브젝트를 업로드합니다.
```
PUT /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| obj | URL | String | O | 오브젝트 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | 자격 증명 접근 키와 서명으로 구성 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ResponseMetadata | Body | Object | 응답 메타데이터 객체 |
| ResponseMetadata.HTTPStatusCode | Body | Integer | 응답 상태 코드 |
| ETag | Body | String | 업로드한 오브젝트의 MD5 해시값 |

<details>
<summary>예시</summary>

```json
{
  "ResponseMetadata": {
    "RequestId": "tx1d914107987d4bd98b7f3-005e5ef3ef",
    "HostId": "tx1d914107987d4bd98b7f3-005e5ef3ef",
    "HTTPStatusCode": 200,
    "HTTPHeaders": {
      "content-length": "0",
      "x-amz-id-2": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "last-modified": "Sat, 22 Feb 2020 22:22:22 GMT",
      "etag": "\"01463f775ef4f4dbbc7525f88120df09\"",
      "x-amz-request-id": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "content-type": "text/html; charset=UTF-8",
      "x-trans-id": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "x-openstack-request-id": "tx1d914107987d4bd98b7f3-005e5ef3ef",
      "date": "Sat, 22 Feb 2020 22:22:22 GMT"
    },
    "RetryAttempts": 0
  },
  "ETag": "\"01463f775ef4f4dbbc7525f88120df09\""
}
```

</details>

### 오브젝트 다운로드
오브젝트를 다운로드합니다.
```
PUT /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| obj | URL | String | O | 오브젝트 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | 자격 증명 접근 키와 서명으로 구성 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ResponseMetadata | Body | Object | 응답 메타데이터 객체 |
| ResponseMetadata.HTTPStatusCode | Body | Integer | 응답 상태 코드 |
| LastModified | Body | String | 오브젝트의 최근 수정 시각, YYYY-MM-DDThh:mm:ssZ |
| ContentLength | Body | String | 다운로드한 오브젝트의 크기 |
| ETag | Body | String | 오브젝트의 MD5 해시값 |
| ContentType | Body | String | 오브젝트의 콘텐츠 타입 |
| Metadata | Body | Object | 오브젝트의 메타데이터 객체 |

<details>
<summary>예시</summary>

```json
{
    "ResponseMetadata": {
        "RequestId": "tx637d5de3c27f4b0a9664e-005e5ef491",
        "HostId": "tx637d5de3c27f4b0a9664e-005e5ef491",
        "HTTPStatusCode": 200,
        "HTTPHeaders": {
            "content-length": "124352",
            "x-amz-id-2": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "last-modified": "Sat, 22 Feb 2020 22:22:22 GMT",
            "etag": "\"01463f775ef4f4dbbc7525f88120df09\"",
            "x-amz-request-id": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "content-type": "image/jpeg",
            "x-trans-id": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "x-openstack-request-id": "tx637d5de3c27f4b0a9664e-005e5ef491",
            "date": "Sat, 22 Feb 2020 22:22:22 GMT"
        },
        "RetryAttempts": 0
    },
    "LastModified": "2020-02-22T22:22:22+00:00",
    "ContentLength": 124352,
    "ETag": "\"01463f775ef4f4dbbc7525f88120df09\"",
    "ContentType": "image/jpeg",
    "Metadata": {}
}
```

</details>

### 오브젝트 삭제
지정한 오브젝트를 삭제합니다.

```
DELETE /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| obj | URL | String | O | 오브젝트 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | 자격 증명 접근 키와 서명으로 구성 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ResponseMetadata | Body | Object | 응답 메타데이터 객체 |
| ResponseMetadata.HTTPStatusCode | Body | Integer | 응답 상태 코드 |

<details>
<summary>예시</summary>

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
            "date": "Sat, 22 Feb 2020 22:22:22 GMT"
        },
        "RetryAttempts": 0
    }
}
```

</details>
