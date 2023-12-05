## Storage > Object Storage > Amazon S3 호환 API 가이드
NHN Cloud 오브젝트 스토리지는 AWS의 오브젝트 스토리지 S3 API와 호환되는 API를 제공합니다. 따라서 Amazon S3 API를 사용하도록 개발된 애플리케이션을 설정만 변경하여 그대로 사용할 수 있습니다.

제공하는 Amazon S3 호환 API는 다음과 같습니다.

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
| [Initiate Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadInitiate.html) | 멀티파트 업로드 초기화 |
| [Upload Part](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPart.html) | 파트 업로드 |
| [Upload Part Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPartCopy.html) | 파트 복사 |
| [Complete Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadComplete.html) | 멀티파트 업로드 완료 |
| [Abort Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadAbort.html) | 멀티파트 업로드 중단 |
| [List Parts](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | 멀티파트 오브젝트의 파트 오브젝트 리스트 |
| [List Multipart Uploads](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | 업로드 진행 중인 멀티파트 오브젝트의 파트 오브젝트 리스트 |
| [DELETE Multiple Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/multiobjectdeleteapi.html) | 멀티파트 오브젝트 삭제 |

이 문서는 기본적인 API 사용 방법만을 설명합니다. 고급 기능을 사용하려면 [Amazon S3 API 가이드](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)를 참고하거나, [AWS SDK](https://aws.amazon.com/ko/tools) 사용을 권장합니다.

## S3 API 자격 증명(S3 API Credential)

### S3 API 자격 증명 발급
Amazon S3 호환 API를 사용하려면 먼저 AWS EC2 형태의 S3 API 자격 증명을 발급 받아야 합니다. 자격 증명은 API를 이용하여 발급 받을 수 있습니다.

API를 이용하여 자격 증명을 발급 받으려면 인증 토큰이 필요합니다. 인증 토큰 발급은 [오브젝트 스토리지 API 가이드](api-guide-ngsc/#tenant-id-api-endpoint)를 참조하세요.

```
POST    https://api-identity-infrastructure.gncloud.go.kr/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

#### 요청

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 발급 받은 토큰 ID |
| api-user-id | URL | String | O | API 사용자 ID, API Endpoint 설정 대화 상자에서 확인 가능 |
| tenant_id | Body | String | O | 테넌트 ID. API Endpoint 설정 대화 상자에서 확인 가능 |

> [참고]
> `{api-user-id}`는 콘솔의 API Endpoint 설정 대화 상자에서 **API 사용자 ID** 항목을 참조하거나 인증 토큰 발급 API 응답 본문의 **access.user.id** 필드에서 확인할 수 있습니다.
> 인증 토큰 발급 API를 이용하려면 API 가이드의 [인증 토큰 발급](api-guide-ngsc/#_2) 항목을 참조하세요.
>
> S3 API 자격 증명은 유효 기간이 없습니다.

<!-- 개행을 위한 주석이므로 필수로 포함되어야 합니다. -->

> [주의]
> S3 API 자격 증명 키가 유출되면 누구나 유출된 키를 이용하여 오브젝트에 접근할 수 있습니다. 키가 유출되었다면 유출된 자격 증명을 삭제하고 새로 발급 받아 사용하는 것을 권장합니다.


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
| access | Body | String | S3 API 자격 증명 접근 키 |
| secret | Body | String | S3 API 자격 증명 비밀 키 |

<details>
<summary>예시</summary>

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

### S3 API 자격 증명 조회
발급 받은 S3 API 자격 증명을 조회합니다.

**[Method, URL]**
```
GET   https://api-identity-infrastructure.gncloud.go.kr/v2.0/users/{user-id}/credentials/OS-EC2

X-Auth-Token: {token-id}
```
#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 발급 받은 토큰 ID |
| user-id | URL | String | O | 사용자 ID, 인증 토큰에 포함되어 있음 |

#### 응답

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| access | Body | String | S3 API 자격 증명 접근 키 |
| secret | Body | String | S3 API 자격 증명 비밀 키 |

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

### S3 API 자격 증명 삭제
발급 받은 S3 API 자격 증명을 삭제합니다.

**[Method, URL]**
```
DELETE   https://api-identity-infrastructure.gncloud.go.kr/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```
#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 발급 받은 토큰 ID |
| user-id | URL | String | O | 사용자 ID, 인증 토큰에 포함되어 있음 |
| access | URL | String | O | S3 API 자격 증명 접근 키 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## 서명(signature) 생성
S3 API를 사용하려면 자격 증명을 이용하여 서명을 생성해야 합니다. 서명 방법은 [AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) 문서를 참고하십시오.

서명 생성에 필요한 정보는 다음과 같습니다.

| 이름 | 값 |
|---|---|
| 알고리즘 | AWS4-HMAC-SHA256 |
| 서명 시각 | YYYYMMDDThhmmssZ 형태 |
| 서비스 이름 | s3 |
| 리전 이름 | gov |
| 비밀 키 | S3 API 자격 증명 비밀 키 |

## 버킷(Bucket)
### 버킷 생성
버킷을 생성합니다. 버킷 이름은 다음과 같은 Amazon S3의 명명 규칙을 따라야 합니다.

* 버킷 이름은 3자에서 63자 사이여야 합니다.
* 버킷 이름은 소문자, 숫자, 점(.) 및 하이픈(-)으로만 구성될 수 있습니다.
* 버킷 이름은 문자 또는 숫자로 시작하고 끝나야 합니다.
* 버킷 이름은 IP 주소 형식(예: 192.168.5.4)을 사용하지 않습니다.
* 버킷 이름은 xn--으로 시작할 수 없습니다.

자세한 내용은 [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) 문서를 참조하세요.

```
PUT /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

> [참고]
> 웹 콘솔 또는 오브젝트 스토리지 API를 통해 만든 컨테이너의 이름이 버킷 명명 규칙에 위배되면 S3 호환 API로는 접근할 수 없습니다.

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | S3 API 자격 증명 접근 키와 서명으로 구성 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 200을 반환합니다.

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| Location | Header | String | 생성한 버킷 경로 |

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
| Authorization | Header | String | O | S3 API 자격 증명 접근 키와 서명으로 구성 |

#### 응답
요청이 올바르면 상태 코드 200과 XML 형식으로 구성된 버킷 목록을 반환합니다.

<details>
<summary>예시</summary>

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

### 버킷 조회
지정한 버킷의 정보와 내부에 저장된 오브젝트 목록을 조회합니다.
```
GET /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

> [참고]
> 웹 콘솔 또는 오브젝트 스토리지 API를 통해 생성한 버킷의 이름이 버킷 명명 규칙에 위배되면 S3 호환 API로는 접근할 수 없습니다.

#### 요청
이 API는 요청 본문을 요구하지 않습니다.

| 이름 | 종류 | 형식 | 필수 | 설명 |
|---|---|---|---|---|
| bucket | URL | String | O | 버킷 이름 |
| Date | Header | String | O | 요청 시각 |
| Authorization | Header | String | O | S3 API 자격 증명 접근 키와 서명으로 구성 |

#### 응답
요청이 올바르면 상태 코드 200과 XML 형식으로 구성된 오브젝트 목록을 반환합니다.

<details>
<summary>예시</summary>

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
| Authorization | Header | String | O | S3 API 자격 증명 접근 키와 서명으로 구성 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

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
| Authorization | Header | String | O | S3 API 자격 증명 접근 키와 서명으로 구성 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 200을 반환합니다.

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| ETag | Header | String | 오브젝트의 MD5 해시값 |
| Last-Modified | Header | String | 오브젝트의 마지막 수정 일시(e.g. Wed, 01 Mar 2006 12:00:00 GMT) |

### 오브젝트 다운로드
오브젝트를 다운로드합니다.
```
GET /{bucket}/{obj}

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
| Authorization | Header | String | O | S3 API 자격 증명 접근 키와 서명으로 구성 |

#### 응답
요청이 올바르면 상태 코드 200을 반환합니다.

| 이름 | 종류 | 형식 | 설명 |
|---|---|---|---|
| Last-Modified | Header | String | 오브젝트의 마지막 수정 일시(e.g. Wed, 01 Mar 2006 12:00:00 GMT) |
| ETag | Header | String | 오브젝트의 MD5 해시값 |

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
| Authorization | Header | String | O | S3 API 자격 증명 접근 키와 서명으로 구성 |

#### 응답
이 API는 응답 본문을 반환하지 않습니다. 요청이 올바르면 상태 코드 204를 반환합니다.

## AWS 명령줄 인터페이스(CLI)
S3 호환 API를 이용하여 [AWS 명령줄 인터페이스](https://aws.amazon.com/ko/cli/)로 NHN Cloud 오브젝트 스토리지를 사용할 수 있습니다.

### 설치
AWS 명령줄 인터페이스는 파이썬 패키지로 제공됩니다. 파이썬 패키지 관리자(pip)를 이용하여 설치합니다.

```shell
$ sudo pip install awscli
```

### 설정
AWS 명령줄 인터페이스를 사용하기 위해서는 먼저 S3 API 자격 증명과 환경을 설정해야 합니다.

```shell
$ aws configure
AWS Access Key ID [None]: {access}
AWS Secret Access Key [None]: {secret}
Default region name [None]: {region name}
Default output format [None]: json
```

| 이름 | 설명 |
|---|---|
| access | S3 API 자격 증명 접근 키 |
| secret | S3 API 자격 증명 비밀 키 |
| region name | gov |

### S3 명령 사용 방법

```shell
aws --endpoint-url={endpoint} s3 {command} s3://{bucket}
```

| 이름 | 설명 |
|---|---|
| endpoint | https://api-object-storage.gncloud.go.kr |
| command | AWS 명령줄 인터페이스 명령 |
| bucket | 버킷 이름 |


> [참고]
> AWS 명령줄 인터페이스는 AWS를 사용하기 위해 제공되는 도구이기 때문에 AWS 도메인을 사용하도록 설정되어 있습니다. 따라서 NHN Cloud 오브젝트 스토리지를 사용하려면 반드시 매 명령마다 엔드포인트를 지정해야합니다.
> AWS 명령줄 인터페이스 명령은 [AWS CLI에서 상위 수준(s3) 명령 사용](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-services-s3-commands.html) 문서를 참조하세요.

<details>
<summary>버킷 생성</summary>

```shell
$ aws --endpoint-url=https://api-object-storage.gncloud.go.kr s3 mb s3://example-bucket
make_bucket: example-bucket
```

</details>

<details>
<summary>버킷 목록 조회</summary>

```shell
$ aws --endpoint-url=https://api-object-storage.gncloud.go.kr s3 ls
2020-07-13 10:07:13 example-bucket
```

</details>


<details>
<summary>버킷 조회</summary>

```shell
$ aws --endpoint-url=https://api-object-storage.gncloud.go.kr s3 ls s3://example-bucket
2020-07-13 10:08:49     104389 0428b9e3e419d4fb7aedffde984ba5b3.jpg
2020-07-13 10:09:09      74448 6dd6d48eef889a5dab5495267944bdc6.jpg
```

</details>

<details>
<summary>버킷 삭제</summary>

```shell
$ aws --endpoint-url=https://api-object-storage.gncloud.go.kr s3 rb s3://example-bucket
remove_bucket: example-bucket
```

</details>

<details>
<summary>오브젝트 업로드</summary>

```shell
$ aws --endpoint-url=https://api-object-storage.gncloud.go.kr s3 cp ./3b5ab489edffdea7bf4d914e3e9b8240.jpg s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
upload: ./3b5ab489edffdea7bf4d914e3e9b8240.jpg to s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

<blockquote>
[참고]
</br>
오브젝트의 용량이 8MB 이상이면 AWS 명령줄 인터페이스는 오브젝트를 여러 개의 파트로 나누어 업로드합니다. 파트 오브젝트는 <code style="display: inline;">{bucket}+segments</code>라는 버킷에 <code style="display: inline;">{object-name}/{upload-id}/{part-number}</code> 형태의 이름으로 저장되고, 모든 파트 업로드가 끝나면 업로드 요청한 버킷에 파트 오브젝트를 연결한 오브젝트가 만들어집니다.
</br></br>
파트 오브젝트가 저장되는 <code style="display: inline;">{bucket}+segments</code> 버킷은 S3 호환 API로는 접근할 수 없고, Object Storage API 또는 콘솔을 통해 접근할 수 있습니다.
</br></br>
멀티파트 오브젝트의 ETag는 각 파트 오브젝트의 ETag 값을 이진 데이터로 변환하고 순서대로 연결해(concatenate) MD5 해시한 값입니다.
</blockquote>

<blockquote>
[주의]
</br>
멀티파트로 업로드한 오브젝트의 일부 또는 전체 파트 오브젝트를 삭제하면 오브젝트에 접근할 수 없습니다.
</blockquote>

</details>

<details>
<summary>오브젝트 다운로드</summary>

```shell
$ aws --endpoint-url=https://api-object-storage.gncloud.go.kr s3 cp s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg ./3b5ab489edffdea7bf4d914e3e9b8240.jpg
download: s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg to ./0428b9e3e419d4fb7aedffde984ba5b3.jpg
```

</details>

<details>
<summary>오브젝트 삭제</summary>

```shell
$ aws --endpoint-url=https://api-object-storage.gncloud.go.kr s3 rm s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
delete: s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

</details>


## AWS SDK
AWS는 여러가지 프로그래밍 언어를 위한 SDK를 제공하고 있습니다. S3 호환 API를 이용하여 AWS SDK로 NHN Cloud 오브젝트 스토리지를 사용할 수 있습니다.

> [참고]
> 보다 자세한 내용은 [AWS SDK](https://aws.amazon.com/ko/tools) 문서를 참조하세요.

AWS SDK를 사용하기 위해 필요한 주요 파라미터는 다음과 같습니다.

| 이름 | 설명 |
|---|---|
| access | S3 API 자격 증명 접근 키 |
| secret | S3 API 자격 증명 비밀 키 |
| region name | gov |
| endpoint | https://api-object-storage.gncloud.go.kr |

### Boto3 - Python SDK

> [참고]
> 보다 자세한 내용은 [AWS SDK for Python (Boto3) 설명서](https://docs.aws.amazon.com/ko_kr/pythonsdk/?icmpid=docs_homepage_sdktoolkits) 문서를 참조하세요.

#### Context

<details>
<summary>Boto3 클라이언트 클래스</summary>

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
<summary>버킷 생성</summary>

```python
def create_bucket(self, bucket_name):
    try:
        return self.s3.create_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>버킷 목록 조회</summary>

```python
def list_buckets(self):
    try:
        return self.s3.list_buckets().get('Buckets')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>버킷 조회(오브젝트 목록 조회)</summary>

```python
def list_objs(self, bucket_name):
    try:
        return self.s3.list_objects_v2(Bucket=bucket_name).get('Contents')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>버킷 삭제</summary>

```python
def delete_bucket(self, bucket_name):
    try:
        return self.s3.delete_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>오브젝트 업로드</summary>

<blockquote>
<p>[참고]
파트 오브젝트의 개수는 업로드할 오브젝트의 용량과 설정한 파트 크기에 의해 결정됩니다. 기본 파트 크기는 8MiB이며, 파트 오브젝트의 최대 개수는 1000개입니다.</p>
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
<summary>오브젝트 다운로드</summary>

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
<summary>오브젝트 삭제</summary>

```python
def delete(self, bucket_name, key):
    try:
        return self.s3.delete_object(Bucket=bucket_name, Key=key)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

### Java SDK

> [참고]
> 보다 자세한 내용은 [AWS SDK for Java 설명서](https://docs.aws.amazon.com/ko_kr/sdk-for-java/index.html) 문서를 참조하세요.

#### Context

<details>
<summary>Java SDK 클라이언트 클래스</summary>

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
<summary>버킷 생성</summary>

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
<summary>버킷 목록 조회</summary>

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
<summary>버킷 조회(오브젝트 목록 조회)</summary>

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
<summary>버킷 삭제</summary>

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
<summary>오브젝트 업로드</summary>

<blockquote>
<p>[참고]
파트 오브젝트의 개수는 업로드할 오브젝트의 용량과 설정한 파트 크기에 의해 결정됩니다. 기본 파트 크기는 5MiB이며, 파트 오브젝트의 최대 개수는 1000개입니다.</p>
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
<summary>오브젝트 다운로드</summary>

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
<summary>오브젝트 삭제</summary>

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

> [참고]
> 보다 자세한 내용은 [AWS SDK for .NET 설명서](https://docs.aws.amazon.com/ko_kr/sdk-for-net/?icmpid=docs_homepage_sdktoolkits) 문서를 참조하세요.

#### Context

<details>
<summary>.NET SDK 클라이언트 클래스</summary>

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
<summary>버킷 생성</summary>

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
<summary>버킷 목록 조회</summary>

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
<summary>버킷 조회(오브젝트 목록 조회)</summary>

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
<summary>버킷 삭제</summary>

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
<summary>오브젝트 업로드</summary>

<blockquote>
<p>[참고]
파트 오브젝트의 개수는 업로드할 오브젝트의 용량과 설정한 파트 크기에 의해 결정됩니다. 기본 파트 크기는 5MiB이며, 파트 오브젝트의 최대 개수는 1000개입니다.</p>
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
<summary>오브젝트 다운로드</summary>

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
<summary>오브젝트 삭제</summary>

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

