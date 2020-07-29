## Storage > Object Storage > AWS S3互換APIガイド
オブジェクトストレージはAWSのオブジェクトストレージS3 APIと互換性のあるAPIを提供します。したがって、AWS S3 APIを使用することを想定して開発されたアプリケーションは、設定を変更するだけで使用できます。

提供するS3互換APIは次のとおりです。

| S3 APIメソッド | 用途 |
| --- | --- |
| [PUT Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUT.html) | バケット作成 |
| [HEAD Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketHEAD.html) | バケット情報を照会 |
| [DELETE Bucket](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketDELETE.html) | バケットを削除 |
| [PUT Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketPUTacl.html) | バケットACLを設定 |
| [GET Bucket ACL](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETacl.html) | バケットACLを照会 |
| [GET Bucket Location](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGETlocation.html) | バケットがあるリージョンを照会 |
| [GET Bucket List Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTBucketGET.html) | バケットのオブジェクトリストを照会 |
| [GET Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectGET.html) | オブジェクトをダウンロード |
| [HEAD Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectHEAD.html) | オブジェクト情報を照会 |
| [PUT Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectPUT.html) | オブジェクトをアップロード |
| [PUT Object Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectCOPY.html) | オブジェクトをコピー |
| [DELETE Object](http://docs.amazonwebservices.com/AmazonS3/latest/API/RESTObjectDELETE.html) | オブジェクトを削除 |
| [Initiate Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadInitiate.html) | マルチパートアップロードを初期化 |
| [Upload Part](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPart.html) | パートをアップロード |
| [Upload Part Copy](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadUploadPartCopy.html) | パートをコピー |
| [Complete Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadComplete.html) | マルチパートアップロード完了 |
| [Abort Multipart Upload](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadAbort.html) | マルチパートアップロードを中断 |
| [List Parts](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | マルチパートオブジェクトのパートオブジェクトリスト |
| [List Multipart Uploads](http://docs.amazonwebservices.com/AmazonS3/latest/API/mpUploadListParts.html) | アップロード進行中のマルチパートオブジェクトのパートオブジェクトリスト |
| [DELETE Multiple Objects](http://docs.amazonwebservices.com/AmazonS3/latest/API/multiobjectdeleteapi.html) | マルチパートオブジェクトを削除 |

この文書はAPI使用方法の基礎的な部分のみを説明します。高度な機能を使用するには[AWS S3 APIガイド](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)を参照するか、[AWS SDK](https://aws.amazon.com/jp/tools)を使用することを推奨します。

## EC2認証情報(EC2 Credential)

### EC2認証情報を登録
S3と互換性のあるAPIを使用するには、先にAWS EC2の形式の認証情報を登録する必要があります。認証情報を登録するには認証トークンが必要です。認証トークンの発行については、[オブジェクトストレージAPIガイド](/Storage/Object%20Storage/ja/api-guide/#tenant-id-api-endpoint)を参照してください。

```
POST    https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{User ID}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 発行されたトークンID |
| user-id | URL | String | O | ユーザーID、認証トークンに含まれている |
| tenant_id | Body | String | O | ユーザーTenant ID。API Endpoint設定ダイアログボックスで確認可能 |

> [注意]
> 認証情報の登録に使用するユーザーIDはメール形式のTOASTアカウントIDではありません。認証トークン発行時に確認できます。

<details>
<summary>例</summary>

```json
{
  "tenant_id": "84c9e9a51aea402e95389c08ac562ac5"
}
```

</details>

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| access | Body | String | 認証情報アクセスキー |
| secret | Body | String | 認証情報シークレットキー |

<details>
<summary>例</summary>

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

### EC2認証情報照会
登録したEC2認証情報を照会します。

**[Method, URL]**
```
GET   https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2

X-Auth-Token: {token-id}
```
#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O |発行されたトークンID |
| user-id | URL | String | O | ユーザーID。認証トークンに含まれている |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| access | Body | String | 認証情報アクセスキー |
| secret | Body | String | 認証情報シークレットキー |

<details>
<summary>例</summary>

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

### EC2認証情報を削除
登録したEC2認証情報を削除します。

**[Method, URL]**
```
DELETE   https://api-identity.infrastructure.cloud.toast.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```
#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 発行されたトークンID |
| user-id | URL | String | O | ユーザーID。認証トークンに含まれている |
| access | URL | String | O | 認証情報アクセスキー |

#### レスポンス
このAPIはレスポンス本文を返しません。リクエストが正しければステータスコード204を返します。

## 署名(signature)作成
S3 APIを使用するには、認証情報を利用して署名を作成する必要があります。署名の作成方法は[AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)文書を参照してください。

署名の作成に必要な情報は次のとおりです。

| 名前 | 値 |
|---|---|
| アルゴリズム | AWS4-HMAC-SHA256 |
| 署名時刻 | YYYYMMDDThhmmssZ形式 |
| サービス名 | s3 |
| リージョン名 | KR1 - 韓国(パンギョ)リージョン<br/>韓国(ピョンチョン)リージョン |
| シークレットキー | 認証情報シークレットキー |

> [参考]
> S3互換APIは、2020年8月現在、韓国(パンギョ)リージョン、韓国(ピョンチョン)リージョンでのみ提供されます。

## バケット(Bucket)
### バケット作成
バケット(コンテナ)を作成します。 버킷 이름은 다음과 같은 AWS S3의 버킷 명명 규칙을 따라야 합니다.

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
> 웹 콘솔 또는 OBS API를 통해 만든 컨테이너의 이름이 버킷 명명 규칙에 위배되면 S3 호환 API로는 접근할 수 없습니다.

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | 認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ResponseMetadata | Body | Object | レスポンスメタデータオブジェクト |
| ResponseMetadata.HTTPStatusCode | Body | Integer | レスポンスステータスコード |
| Location | Body | String | 作成したバケットのパス |

<details>
<summary>例</summary>

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

### バケットリスト照会
バケットリストを照会します。
```
GET /

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | 認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ResponseMetadata | Body | Object | レスポンスメタデータオブジェクト |
| ResponseMetadata.HTTPStatusCode | Body | Integer | レスポンスステータスコード |
| Buckets.Name | Body | String | バケット名 |
| Buckets.CreationDate | Body | String | 作成時刻 |

<details>
<summary>例</summary>

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

### バケット照会
指定したバケットの情報と内部に保存されたオブジェクトリストを照会します。
```
GET /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | 認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ResponseMetadata | Body | Object | レスポンスメタデータオブジェクト |
| ResponseMetadata.HTTPStatusCode | Body | Integer | レスポンスステータスコード |
| Contents | Body | Object | オブジェクトリストオブジェクト |  
| Contents.Key | Body | String | オブジェクト名 |
| Contents.LastModified | Body | String | オブジェクトの最終修正時刻、 YYYY-MM-DDThh:mm:ssZ |
| Contents.ETag | Body | String | オブジェクトのMD5ハッシュ値 |
| Contents.Size | Body | String | オブジェクトのサイズ |
| Contents.StorageClass | Body | String | オブジェクトが保存された場所の種類 |
| Name | Body | String | バケット名 |
| KeyCount | Body | Integer | リストのオブジェクト数 |

<details>
<summary>例</summary>

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

### バケットの削除
指定したバケットを削除します。削除するバケットは空になっている必要があります。
```
DELETE /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | 認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ResponseMetadata | Body | Object | レスポンスメタデータオブジェクト |
| ResponseMetadata.HTTPStatusCode | Body | Integer | レスポンスステータスコード |

<details>
<summary>例</summary>

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

## オブジェクト
### オブジェクトのアップロード
指定したバケットにオブジェクトをアップロードします。
```
PUT /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| obj | URL | String | O | オブジェクト名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | 認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ResponseMetadata | Body | Object | レスポンスメタデータオブジェクト |
| ResponseMetadata.HTTPStatusCode | Body | Integer | レスポンスステータスコード |
| ETag | Body | String | アップロードしたオブジェクトのMD5ハッシュ値 |

<details>
<summary>例</summary>

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

### オブジェクトのダウンロード
オブジェクトをダウンロードします。
```
GET /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| obj | URL | String | O | オブジェクト名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | 認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ResponseMetadata | Body | Object | レスポンスメタデータオブジェクト |
| ResponseMetadata.HTTPStatusCode | Body | Integer | レスポンスステータスコード |
| LastModified | Body | String | オブジェクトの最終修正時刻、 YYYY-MM-DDThh:mm:ssZ |
| ContentLength | Body | String | ダウンロードしたオブジェクトのサイズ |
| ETag | Body | String | オブジェクトのMD5ハッシュ値 |
| ContentType | Body | String | オブジェクトのコンテンツタイプ |
| Metadata | Body | Object | オブジェクトのメタデータオブジェクト |

<details>
<summary>例</summary>

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

### オブジェクトの削除
指定したオブジェクトを削除します。

```
DELETE /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| obj | URL | String | O | オブジェクト名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | 認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ResponseMetadata | Body | Object | レスポンスメタデータオブジェクト |
| ResponseMetadata.HTTPStatusCode | Body | Integer | レスポンスステータスコード |

<details>
<summary>例</summary>

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
| region name | KR1 - 한국(판교)리전<br/>KR2 - 한국(평촌)리전 |

### S3 명령 사용 방법

```
aws --endpoint-url={endpoint} s3 {command} s3://{bucket}
```

| 이름 | 설명 |
|---|---|
| endpoint | https://api-storage.cloud.toast.com - 한국(판교)리전<br/>https://kr2-api-storage.cloud.toast.com - 한국(평촌)리전 |
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
| region name | KR1 - 한국(판교)리전<br/>KR2 - 한국(평촌)리전 |
| endpoint | https://api-storage.cloud.toast.com - 한국(판교)리전<br/>https://kr2-api-storage.cloud.toast.com - 한국(평촌)리전 |


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
