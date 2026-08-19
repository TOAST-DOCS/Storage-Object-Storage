<!-- machine_translated: true -->

<!-- pre-align:aligned sig=acedf0f45de3 -->

<a id="storage-object-storage-amazon-s3-compatible-api-guide"></a>
## Storage > Object Storage > Amazon S3互換APIガイド { #storage-object-storage-amazon-s3-compatible-api-guide }
NHN Cloud オブジェクトストレージは、AWS のオブジェクトストレージ S3 API と互換性のある API を提供します。そのため、Amazon S3 API を使用するように開発されたアプリケーションは、設定を変更するだけでそのまま使用できます。

提供する Amazon S3互換 API は次のとおりです。

| S3 API メソッド | 用途 |
| --- | --- |
| PUT Bucket | バケットの作成 |
| HEAD Bucket | バケット情報の照会 |
| DELETE Bucket | バケットの削除 |
| PUT Bucket Object Lock | ロックバケットの作成 |
| PUT Object Lock Configuration | ロックバケット保管期間の設定 |
| GET Object Lock Configuration | ロックバケット保管期間の照会 |
| PUT Bucket ACL | バケット ACL の設定 |
| GET Bucket ACL | バケット ACL の照会 |
| GET Bucket Location | バケットが存在するリージョンの照会 |
| GET Bucket List Objects | バケット内オブジェクト一覧の照会 |
| GET Object | オブジェクトのダウンロード |
| HEAD Object | オブジェクト情報の照会 |
| PUT Object | オブジェクトのアップロード |
| PUT Object Copy | オブジェクトのコピー |
| DELETE Object | オブジェクトの削除 |
| Initiate Multipart Upload | マルチパートアップロードの初期化 |
| Upload Part | パートのアップロード |
| Upload Part Copy | パートのコピー |
| Complete Multipart Upload | マルチパートアップロードの完了 |
| Abort Multipart Upload | マルチパートアップロードの中断 |
| List Parts | マルチパートオブジェクトのパートオブジェクトリスト |
| List Multipart Uploads | アップロード進行中のマルチパートオブジェクトのパートオブジェクトリスト |
| DELETE Multiple Objects | 2つ以上のオブジェクトの削除 |

この文書では、基本的な API の使用方法のみを説明します。高度な機能を使用するには、[Amazon S3 API ガイド](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/API/API_Operations_Amazon_Simple_Storage_Service.html)を参照するか、[AWS SDK](https://aws.amazon.com/ko/tools) の使用をお勧めします。

<a id="s3-api-credential"></a>
## S3 API認証情報(S3 API Credential) { #s3-api-credential }

<a id="obtain-s3-api-credentials"></a>
### S3 API認証情報の発行 { #obtain-s3-api-credentials }
Amazon S3互換APIを使用するには、まず AWS EC2 形式の S3 API認証情報を発行する必要があります。認証情報はコンソールまたは API を使用して発行できます。コンソールを使用した認証情報の発行については、[S3 API認証情報](console-guide/#s3-api-credentials) を参照してください。

API を使用して認証情報を発行するには、認証トークンが必要です。認証トークンの発行については、[オブジェクトストレージ API ガイド](api-guide/#auth) を参照してください。

```
POST https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

<a id="obtain-s3-api-credentials-request"></a>
#### 要求

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | 発行済みトークン ID |
| api-user-id | URL | String | Y | APIユーザーID。API エンドポイント設定ダイアログボックスで確認可能 |
| tenant_id | Body | String | Y | テナントID。API エンドポイント設定ダイアログボックスで確認可能 |

!!! tip "ヒント"
    `{api-user-id}` は、コンソールの API エンドポイント設定ダイアログボックスで **[API ユーザー ID]** 項目を参照するか、認証トークン発行 API レスポンス本文の **access.user.id** フィールドで確認できます。
    認証トークン発行 API を使用するには、API ガイドの[認証と権限](api-guide/#auth) を参照してください。


    S3 API認証情報には有効期限がなく、ユーザーごとにプロジェクトあたり最大 3 つまで発行できます。

<!-- 改行のためのコメントのため、必ず含める必要があります。 -->

!!! danger "注意"
    S3 API認証情報キーが漏洩すると、誰でも漏洩したキーを使用してオブジェクトにアクセスできます。キーが漏洩した場合は、漏洩した認証情報を削除して新しく発行して使用することをお勧めします。

    S3 API認証情報を発行したユーザーアカウントがプロジェクトへのアクセス権限を失った場合、または NHN Cloud を退会して削除された場合、認証情報は即座に失効して使用できなくなります。

<details>
<summary>例</summary>

```json
{
  "tenant_id": "84c9e9a51aea402e95389c08ac562ac5"
}
```

</details>

<a id="obtain-s3-api-credentials-response"></a>
#### 応答

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| access | Body | String | S3 API認証情報アクセスキー |
| secret | Body | String | S3 API認証情報シークレットキー |
| user_id | Body | String | APIユーザーID |
| tenant_id | Body | String | テナントID |
| created_at | Body | String | S3 API認証情報の作成日時 |
| accessed_at | Body | String | S3 API認証情報の最終アクセス日時 |

<details>
<summary>例</summary>

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
### S3 API認証情報の照会 { #get-s3-api-credentials }
発行済みの S3 API認証情報を照会します。

**[Method, URL]**

```
GET https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2

X-Auth-Token: {token-id}
```

<a id="get-s3-api-credentials-request"></a>
#### 要求
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | 発行済みトークン ID |
| user-id | URL | String | Y | ユーザー ID。認証トークンに含まれています |

<a id="get-s3-api-credentials-response"></a>
#### 応答

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| access | Body | String | S3 API認証情報アクセスキー |
| secret | Body | String | S3 API認証情報シークレットキー |
| user_id | Body | String | APIユーザーID |
| tenant_id | Body | String | テナントID |
| created_at | Body | String | S3 API認証情報の作成日時 |
| accessed_at | Body | String | S3 API認証情報の最終アクセス日時 |

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
      "created_at": "2024-10-19T08:24:46.000000Z",
      "accessed_at": "2024-10-19T08:30:42.000000Z"
    }
  ]
}
```

</details>

<a id="delete-s3-api-credentials"></a>
### S3 API認証情報の削除 { #delete-s3-api-credentials }
発行済みの S3 API認証情報を削除します。

**[Method, URL]**

```
DELETE https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```

<a id="delete-s3-api-credentials-request"></a>
#### 要求
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | Y | 発行済みトークン ID |
| user-id | URL | String | Y | ユーザー ID。認証トークンに含まれています |
| access | URL | String | Y | S3 API認証情報アクセスキー |

<a id="delete-s3-api-credentials-response"></a>
#### 応答
この API はレスポンス本文を返しません。リクエストが正しい場合、ステータスコード 204 を返します。

<a id="create-signature"></a>
## 署名(signature)の生成 { #create-signature }
S3 API を使用するには、認証情報を使用して署名を生成する必要があります。署名の生成方法については、[AWS signature V4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) のドキュメントを参照してください。


署名の生成に必要な情報は次のとおりです。

| 名前 | 値 |
|---|---|
| アルゴリズム | AWS4-HMAC-SHA256 |
| 署名日時 | YYYYMMDDThhmmssZ 形式 |
| サービス名 | s3 |
| リージョン名 | KR1 - 韓国(板橋)リージョン<br>KR2 - 韓国(平村)リージョン<br>KR3 - 韓国(光州)リージョン<br>JP1 - 日本(東京)リージョン |
| シークレットキー | S3 API認証情報シークレットキー |

AWS signature V4 の署名生成時に `x-amz-content-sha256` ヘッダーが必要です。このヘッダーは正規リクエスト(Canonical Request)に含まれて署名の計算に使用され、ヘッダーの値によってペイロードの処理方式が決まります。使用可能な値は次のとおりです。

| x-amz-content-sha256 値 | 説明 |
|---|---|
| `<ペイロードハッシュ>` | リクエストペイロード全体の SHA-256 ハッシュ値を使用する基本方式 |
| `UNSIGNED-PAYLOAD` | ペイロードの署名を省略 |
| `STREAMING-AWS4-HMAC-SHA256-PAYLOAD` | AWS Chunked Upload 方式(各チャンクに署名を含む) |
| `STREAMING-UNSIGNED-PAYLOAD-TRAILER` | AWS Chunked Upload 方式(チャンク署名なしでトレーラーヘッダーを使用) |
| `STREAMING-AWS4-HMAC-SHA256-PAYLOAD-TRAILER` | AWS Chunked Upload 方式(各チャンクに署名を含む + トレーラーヘッダーを使用) |

!!! tip "ヒント"
    詳細については、[Authenticating Requests: Using the Authorization Header(AWS Signature Version 4)](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-auth-using-authorization-header.html) のドキュメントを参照してください。

x-amz-content-sha256 の値が `STREAMING-UNSIGNED-PAYLOAD-TRAILER` または `STREAMING-AWS4-HMAC-SHA256-PAYLOAD-TRAILER` の場合、`x-amz-trailer` リクエストヘッダーでトレーラーに送信するチェックサムアルゴリズムを宣言する必要があります。サポートしているアルゴリズムは次のとおりです。

| x-amz-trailer 値 | アルゴリズム |
|---|---|
| `x-amz-checksum-crc32` | CRC-32 |
| `x-amz-checksum-crc32c` | CRC-32C |
| `x-amz-checksum-crc64nvme` | CRC-64/NVME |
| `x-amz-checksum-sha1` | SHA-1 |
| `x-amz-checksum-sha256` | SHA-256 |

!!! tip "ヒント"
    トレーラーヘッダーを使用した署名計算方法の詳細については、[Signature calculations for trailing headers(chunked uploads)](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-streaming-trailers.html) のドキュメントを参照してください。

<a id="bucket"></a>
## Bucket { #bucket }

<a id="create-bucket"></a>
### バケットの作成 { #create-bucket }
バケットを作成します。バケット名は、次の Amazon S3 の命名規則に従う必要があります。

* バケット名は 3〜63 文字である必要があります。
* バケット名は小文字、数字、ピリオド (.) およびハイフン (-) のみで構成できます。
* バケット名は文字または数字で始まり、文字または数字で終わる必要があります。
* バケット名は IP アドレス形式 (例: 192.168.5.4) を使用できません。
* バケット名は xn-- で始めることができません。

詳細については、[Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html) ドキュメントを参照してください。


```
PUT /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

!!! tip "ヒント"
    コンソールまたはオブジェクトストレージ API で作成したコンテナの名前がバケットの命名規則に違反している場合、S3互換API ではアクセスできません。

<a id="create-bucket-request"></a>
#### リクエスト
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| Date | Header | String | Y | リクエスト日時 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="create-bucket-response"></a>
#### レスポンス
この API はレスポンス本文を返しません。リクエストが正しければ、ステータスコード 200 を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| Location | Header | String | 作成したバケットのパス |

<a id="list-buckets"></a>
### バケット一覧の照会 { #list-buckets }
バケットの一覧を照会します。

```
GET /

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="list-buckets-request"></a>
#### リクエスト
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| Date | Header | String | Y | リクエスト日時 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="list-buckets-response"></a>
#### レスポンス
リクエストが正しければ、ステータスコード 200 と XML 形式のバケット一覧を返します。

<details>
<summary>例</summary>

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
### バケットの照会 { #get-bucket }
指定したバケットの情報と、内部に保存されているオブジェクトの一覧を照会します。

```
GET /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

!!! tip "ヒント"
    コンソールまたはオブジェクトストレージ API を使用して作成したコンテナの名前がバケットの命名規則に違反している場合、S3互換API ではアクセスできません。

<a id="get-bucket-request"></a>
#### リクエスト
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| Date | Header | String | Y | リクエスト日時 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="get-bucket-response"></a>
#### レスポンス
リクエストが正しければ、ステータスコード 200 と XML 形式のオブジェクト一覧を返します。

<details>
<summary>例</summary>

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
### バケットの削除 { #delete-bucket }
指定したバケットを削除します。削除するバケットは空である必要があります。

```
DELETE /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="delete-bucket-request"></a>
#### リクエスト
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| Date | Header | String | Y | リクエスト日時 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="delete-bucket-response"></a>
#### レスポンス
この API はレスポンス本文を返しません。リクエストが正しければ、ステータスコード 204 を返します。

<a id="create-lock-bucket"></a>
### ロックバケットの作成 { #create-lock-bucket }
オブジェクトロックが有効なバケットを作成します。バケットを作成する際に `x-amz-bucket-object-lock-enabled` ヘッダーを `true` に設定します。デフォルトの保管期間は 0 日に設定されます。

```
PUT /{bucket}

x-amz-bucket-object-lock-enabled: true
Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="create-lock-bucket-request"></a>
#### リクエスト
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| x-amz-bucket-object-lock-enabled | Header | Boolean | Y | オブジェクトロックの有効化 (`true`) |
| Date | Header | String | Y | リクエスト日時 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="create-lock-bucket-response"></a>
#### レスポンス
この API はレスポンス本文を返しません。リクエストが正しければ、ステータスコード 200 を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| Location | Header | String | 作成したバケットのパス |

<a id="put-object-lock-configuration"></a>
### ロックバケットの保管期間の設定 { #put-object-lock-configuration }
ロックバケットのデフォルトの保管期間を設定します。

```
PUT /{bucket}?object-lock

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="put-object-lock-configuration-request"></a>
#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| Date | Header | String | Y | リクエスト日時 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

リクエスト本文に JSON 形式のオブジェクトロック設定を含める必要があります。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| ObjectLockEnabled | Body | String | Y | オブジェクトロックの有効化状態。`Enabled` のみ許可 |
| Rule | Body | Object | N | デフォルトの保管ルール |
| Rule.DefaultRetention | Body | Object | Conditional | デフォルトの保管期間の設定。Rule 設定時は必須 |
| Rule.DefaultRetention.Mode | Body | String | Conditional | 保管モード。`COMPLIANCE` のみ許可 |
| Rule.DefaultRetention.Days | Body | Integer | Conditional | 保管期間 (日)。正の整数。Days または Years のいずれか一方が必須 |
| Rule.DefaultRetention.Years | Body | Integer | Conditional | 保管期間 (年)。正の整数。Days または Years のいずれか一方が必須 |

!!! tip "ヒント"
    `Rule` を省略すると、デフォルトの保管期間が 0 日に設定されます。
    `Years` で設定した場合でも、照会時は常に `Days` に変換して返します。

<details>
<summary>例</summary>

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
#### レスポンス
この API はレスポンス本文を返しません。リクエストが正しければ、ステータスコード 200 を返します。

<a id="get-object-lock-configuration"></a>
### ロックバケットの保管期間の照会 { #get-object-lock-configuration }
ロックバケットのオブジェクトロック設定を照会します。

```
GET /{bucket}?object-lock

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="get-object-lock-configuration-request"></a>
#### リクエスト
この API はリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| Date | Header | String | Y | リクエスト日時 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="get-object-lock-configuration-response"></a>
#### レスポンス
リクエストが正しければ、ステータスコード 200 と JSON 形式のオブジェクトロック設定を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ObjectLockEnabled | Body | String | オブジェクトロックの有効化状態 |
| Rule | Body | Object | デフォルトの保管ルール |
| Rule.DefaultRetention | Body | Object | デフォルトの保管期間の設定 |
| Rule.DefaultRetention.Mode | Body | String | 保管モード |
| Rule.DefaultRetention.Days | Body | Integer | 保管期間 (日) |

!!! tip "ヒント"
    `Years` で設定した保管期間も、照会時は `Days` に変換して返します。

<details>
<summary>例</summary>

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
## オブジェクト { #object }

<a id="upload-object"></a>
### オブジェクトのアップロード { #upload-object }
指定したバケットにオブジェクトをアップロードします。

```
PUT /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="upload-object-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| obj | URL | String | Y | オブジェクト名 |
| Date | Header | String | Y | リクエスト時刻 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="upload-object-response"></a>
#### レスポンス
このAPIはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード 200 を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ETag | Header | String | オブジェクトの MD5 ハッシュ値 |
| Last-Modified | Header | String | オブジェクトの最終更新日時(e.g. Wed, 01 Mar 2006 12:00:00 GMT) |

<a id="download-object"></a>
### オブジェクトのダウンロード { #download-object }
オブジェクトをダウンロードします。

```
GET /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="download-object-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| obj | URL | String | Y | オブジェクト名 |
| Date | Header | String | Y | リクエスト時刻 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="download-object-response"></a>
#### レスポンス
リクエストが正しい場合、ステータスコード 200 を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| Last-Modified | Header | String | オブジェクトの最終更新日時(e.g. Wed, 01 Mar 2006 12:00:00 GMT) |
| ETag | Header | String | オブジェクトの MD5 ハッシュ値 |

<a id="delete-object"></a>
### オブジェクトの削除 { #delete-object }
指定したオブジェクトを削除します。

```
DELETE /{bucket}/{obj}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

<a id="delete-object-request"></a>
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | Y | バケット名 |
| obj | URL | String | Y | オブジェクト名 |
| Date | Header | String | Y | リクエスト時刻 |
| Authorization | Header | String | Y | S3 API認証情報のアクセスキーと署名で構成 |

<a id="delete-object-response"></a>
#### レスポンス
このAPIはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード 204 を返します。

<a id="presigned-url"></a>
## 署名付き URL の生成 { #presigned-url }
**AWS Signature Version 4 (SigV4)** の署名をクエリパラメータに含めることで、認証トークン (Authorization ヘッダー) なしに一定時間オブジェクトにアクセスできる URL です。ダウンロードは `GET`、アップロードは `PUT` でリクエストします。

<a id="presigned-url-format"></a>
### 署名付き URL の形式 { #presigned-url-format }

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
#### リクエスト
このAPIはリクエスト本文を必要としません。

| 名前 | 種類 | 形式 | 必須 | 説明                                                                             |
|---|---|---|---|--------------------------------------------------------------------------------|
| bucket | URL | String | Y | バケット名                                                                          |
| obj | URL | String | Y | オブジェクト名                                                                        |
| X-Amz-Algorithm | Query | String | Y | 署名アルゴリズム。AWS4-HMAC-SHA256 に設定                                                 |
| X-Amz-Credential | Query | String | Y | アクセスキーと署名スコープ。`{access}/{date}/{region}/s3/aws4_request` 形式 (`/` は `%2F` にエンコード) |
| X-Amz-Date | Query | String | Y | 署名時刻。ISO 8601 `yyyyMMddTHHmmssZ` (UTC)                                        |
| X-Amz-Expires | Query | String | Y | 有効期間 (秒)。最小 `1`、最大 `604800` (7日)                                              |
| X-Amz-SignedHeaders | Query | String | Y | 署名に含めるヘッダーのリスト。最小 `host` を含む                                                    |
| X-Amz-Signature | Query | String | Y | リクエストを認証する HMAC 署名値                                                             |

<a id="presigned-url-format-response"></a>
#### レスポンス
リクエストが正しい場合、ステータスコード 200 を返します。

!!! tip "ヒント"
    Swift TempURL 方式や言語別の直接署名の例など、詳細については「[署名付き URL ガイド](presigned-url-guide/)」を参照してください。


<a id="aws-command-line-interface"></a>
## AWS Command Line Interface (CLI) { #aws-command-line-interface }
S3互換APIを使用して、[AWS Command Line Interface](https://aws.amazon.com/ko/cli/)でNHN Cloudオブジェクトストレージを使用できます。

<a id="aws-command-line-interface-installation"></a>
### インストール { #aws-command-line-interface-installation }
[Installing past releases of the AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-version.html) ドキュメントを参照して、AWS Command Line Interfaceをインストールします。

!!! tip "注記"
    NHN CloudオブジェクトストレージはAWS CLIバージョン2.34.38までサポートしています。

<a id="aws-command-line-interface-configuration"></a>
### 設定 { #aws-command-line-interface-configuration }
AWS Command Line Interfaceを使用するには、まずS3 API認証情報と環境を設定する必要があります。

```shell
$ aws configure
AWS Access Key ID [None]: {access}
AWS Secret Access Key [None]: {secret}
Default region name [None]: {region name}
Default output format [None]: json
```

| 名前 | 説明 |
|---|---|
| access | S3 API認証情報のアクセスキー |
| secret | S3 API認証情報のシークレットキー |
| region name | KR1 - 韓国(板橋)リージョン<br>KR2 - 韓国(坪村)リージョン<br>KR3 - 韓国(光州)リージョン<br>JP1 - 日本(東京)リージョン |

<a id="how-to-use-the-s3-commands"></a>
### S3コマンドの使用方法 { #how-to-use-the-s3-commands }

```shell
aws --endpoint-url={endpoint} s3 {command} s3://{bucket}
```

| 名前 | 説明 |
|---|---|
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - 韓国(板橋)リージョン<br>https://kr2-api-object-storage.nhncloudservice.com - 韓国(坪村)リージョン<br>https://kr3-api-object-storage.nhncloudservice.com - 韓国(光州)リージョン<br>https://jp1-api-object-storage.nhncloudservice.com - 日本(東京)リージョン |
| command | AWS Command Line Interfaceコマンド |
| bucket | バケット名 |

!!! tip "注記"
    AWS Command Line Interfaceは、デフォルトで AWS ドメインを使用するように設定されたツールです。そのため、NHN Cloudオブジェクトストレージを使用するには、すべてのコマンドでエンドポイントを必ず指定する必要があります。
    AWS Command Line Interfaceコマンドについては、[AWS CLIでの上位レベル(s3)コマンドの使用](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-services-s3-commands.html) ドキュメントを参照してください。

<details>
<summary>バケットの作成</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 mb s3://example-bucket
make_bucket: example-bucket
```

</details>

<details>
<summary>バケット一覧の照会</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls
2020-07-13 10:07:13 example-bucket
```

</details>

<details>
<summary>バケットの照会</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls s3://example-bucket
2020-07-13 10:08:49     104389 0428b9e3e419d4fb7aedffde984ba5b3.jpg
2020-07-13 10:09:09      74448 6dd6d48eef889a5dab5495267944bdc6.jpg
```

</details>

<details>
<summary>バケットの削除</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rb s3://example-bucket
remove_bucket: example-bucket
```

</details>

<details>
<summary>ロックバケット</summary>

ロックバケットは <code>aws s3api</code> サブコマンドで管理します。
<br>
<code>create-bucket</code> コマンドに <code>--object-lock-enabled-for-bucket</code> オプションを使用すると、オブジェクトロックが有効なバケットを作成します。
デフォルトの保持期間は0日に設定されます。

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3api create-bucket \
  --bucket example-bucket \
  --object-lock-enabled-for-bucket
```

デフォルトの保持期間を設定するには、<code>put-object-lock-configuration</code> コマンドを使用します。

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

ロック設定を照会するには、<code>get-object-lock-configuration</code> コマンドを使用します。

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
<summary>オブジェクトのアップロード</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp ./3b5ab489edffdea7bf4d914e3e9b8240.jpg s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
upload: ./3b5ab489edffdea7bf4d914e3e9b8240.jpg to s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

!!! tip "注記"
    オブジェクトのサイズが8MB以上の場合、AWS Command Line Interfaceはオブジェクトを複数のパートに分割してアップロードします。パートオブジェクトは <code style="display: inline;">{bucket}+segments</code> というバケットに <code style="display: inline;">{object-name}/{upload-id}/{part-number}</code> 形式の名前で保存され、すべてのパートのアップロードが完了すると、アップロードをリクエストしたバケットにパートオブジェクトを結合したオブジェクトが作成されます。

    パートオブジェクトが保存される <code style="display: inline;">{bucket}+segments</code> バケットはS3互換APIではアクセスできず、Object Storage APIまたはコンソールからアクセスできます。

    マルチパートオブジェクトのETagは、各パートオブジェクトのETag値をバイナリデータに変換し、順番に連結(concatenate)してMD5ハッシュした値です。

!!! danger "警告"
    マルチパートでアップロードしたオブジェクトの一部または全パートオブジェクトを削除すると、オブジェクトにアクセスできなくなります。

</details>

<details>
<summary>オブジェクトのダウンロード</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg ./3b5ab489edffdea7bf4d914e3e9b8240.jpg
download: s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg to ./0428b9e3e419d4fb7aedffde984ba5b3.jpg
```

</details>

<details>
<summary>オブジェクトの削除</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rm s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
delete: s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

</details>

<details>
<summary>署名付きURLの生成</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 presign s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg --expires-in 3600
https://kr1-api-object-storage.nhncloudservice.com/example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Date=...&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=...
```

</details>

<a id="aws-command-line-interface-virtual-hosted-style"></a>
### ドメインスタイルエンドポイントの使用 { #aws-command-line-interface-virtual-hosted-style }
S3互換APIは、バケットへのアクセス方式としてパススタイル(Path-style)とドメインスタイル(Virtual Hosted-style)の両方をサポートしています。ドメインスタイルはバケット名をエンドポイントのサブドメインとして使用します。

| 方式 | 形式 |
|---|---|
| パススタイル(Path-style) | `https://{endpoint}/{bucket}/{object}` |
| ドメインスタイル(Virtual Hosted-style) | `https://{bucket}.{endpoint}/{object}` |

<br>

AWS Command Line Interfaceでドメインスタイルエンドポイントを使用するには、`addressing_style` オプションを `virtual` に設定します。この設定を適用すると、AWS Command Line Interfaceがエンドポイントとバケット名を組み合わせて自動的にドメインスタイルURLでリクエストを送信します。

```shell
$ aws configure set default.s3.addressing_style virtual
```

または `~/.aws/config` ファイルで使用中のプロファイルセクションに次の設定を追加します。

```ini
[default]
s3 =
  addressing_style = virtual
```

| 名前 | 説明 |
|---|---|
| addressing_style | `virtual` - ドメインスタイルを使用<br>`path` - パススタイルを使用<br>`auto` - 自動選択(デフォルト値。NHN Cloud Object Storageのようなカスタムエンドポイントを使用する場合はパススタイルで動作) |

!!! danger "警告"
    バケット名にピリオド(`.`)が含まれている場合、ドメインスタイルを使用するとワイルドカードSSL証明書の有効範囲外となり、証明書の検証に失敗する可能性があります。この場合はパススタイルを使用してください。

<a id="aws-sdk"></a>
## AWS SDK { #aws-sdk }
AWSは複数のプログラミング言語向けのSDKを提供しています。S3互換APIを使用して、AWS SDKでNHN Cloudオブジェクトストレージを使用できます。

!!! tip "注記"
    詳細については、[AWS SDK](https://aws.amazon.com/ko/tools) ドキュメントを参照してください。


AWS SDKを使用するために必要な主なパラメータは次のとおりです。

| 名前 | 説明 |
|---|---|
| access | S3 API認証情報のアクセスキー |
| secret | S3 API認証情報のシークレットキー |
| region name | KR1 - 韓国(板橋)リージョン<br>KR2 - 韓国(坪村)リージョン<br>KR3 - 韓国(光州)リージョン<br>JP1 - 日本(東京)リージョン |
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - 韓国(板橋)リージョン<br>https://kr2-api-object-storage.nhncloudservice.com - 韓国(坪村)リージョン<br>https://kr3-api-object-storage.nhncloudservice.com - 韓国(光州)リージョン<br>https://jp1-api-object-storage.nhncloudservice.com - 日本(東京)リージョン |

<a id="aws-sdk-boto3-python"></a>
### Boto3 - Python SDK { #aws-sdk-boto3-python }

!!! tip "ヒント"
    詳細については、[AWS SDK for Python(Boto3) 説明書](https://docs.aws.amazon.com/ko_kr/pythonsdk/?icmpid=docs_homepage_sdktoolkits)を参照してください。

<a id="aws-sdk-boto3-python-context"></a>
#### Context

<details>
<summary>Boto3 クライアントクラス</summary>

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
<summary>バケットの作成</summary>

```python
def create_bucket(self, bucket_name):
    try:
        return self.s3.create_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>バケット一覧の照会</summary>

```python
def list_buckets(self):
    try:
        return self.s3.list_buckets().get('Buckets')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>バケットの照会（オブジェクト一覧の照会）</summary>

```python
def list_objs(self, bucket_name):
    try:
        return self.s3.list_objects_v2(Bucket=bucket_name).get('Contents')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>バケットの削除</summary>

```python
def delete_bucket(self, bucket_name):
    try:
        return self.s3.delete_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>ロックバケット</summary>

<code>create_bucket</code> メソッドに <code>ObjectLockEnabledForBucket=True</code> を設定すると、ロックバケットを作成します。デフォルトの保管期間は 0 日に設定されます。

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

デフォルトの保管期間を設定するには、<code>put_object_lock_configuration</code> メソッドを使用します。

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

ロック設定を照会するには、<code>get_object_lock_configuration</code> メソッドを使用します。

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
<summary>オブジェクトのアップロード</summary>

!!! tip "ヒント"
    パートオブジェクトの数は、アップロードするオブジェクトのサイズと設定したパートサイズによって決まります。デフォルトのパートサイズは 8MiB で、最小 5MiB から指定できます。パートオブジェクトの最大数は 10,000 個です。

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
<summary>オブジェクトのダウンロード</summary>

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
<summary>オブジェクトの削除</summary>

```python
def delete(self, bucket_name, key):
    try:
        return self.s3.delete_object(Bucket=bucket_name, Key=key)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>署名付き URL の生成</summary>

```python
def generate_presigned_url(self, bucket_name, key, expires_in):
    try:
        # アップロード用は 'put_object'
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

!!! tip "ヒント"
    詳細については、[AWS SDK for Java 説明書](https://docs.aws.amazon.com/ko_kr/sdk-for-java/index.html)を参照してください。

<a id="aws-sdk-java-context"></a>
#### Context

<details>
<summary>Java SDK クライアントクラス</summary>

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
<summary>バケットの作成</summary>

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
<summary>バケットリストの照会</summary>

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
<summary>バケットの照会（オブジェクトリストの照会）</summary>

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
<summary>バケットの削除</summary>

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
<summary>ロックバケット</summary>

<code>CreateBucketRequest</code> に <code>withObjectLockEnabledForBucket(true)</code> を設定すると、ロックバケットを作成します。デフォルトの保管期間は 0 日に設定されます。

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

デフォルトの保管期間を設定するには、<code>setObjectLockConfiguration</code> メソッドを使用します。

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

ロック設定を照会するには、<code>getObjectLockConfiguration</code> メソッドを使用します。

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
<summary>オブジェクトのアップロード</summary>

!!! tip "ヒント"
    パートオブジェクトの数は、アップロードするオブジェクトのサイズと設定したパートサイズによって決まります。デフォルトのパートサイズは 5MiB で、最小 5MiB から指定できます。パートオブジェクトの最大数は 10,000 個です。

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
<summary>オブジェクトのダウンロード</summary>

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
<summary>オブジェクトの削除</summary>

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
<summary>署名付き URL の作成</summary>

```java
public String generatePresignedUrl(
    String bucketName, String objKeyName, long expirationMillis
) throws RuntimeException {
    try {
        Date expiration = new Date(System.currentTimeMillis() + expirationMillis);
        GeneratePresignedUrlRequest request =
            new GeneratePresignedUrlRequest(bucketName, objKeyName)
                .withMethod(HttpMethod.GET)          // アップロード用は HttpMethod.PUT
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

!!! tip "ヒント"
    詳細については、[AWS SDK for .NET 説明書](https://docs.aws.amazon.com/ko_kr/sdk-for-net/?icmpid=docs_homepage_sdktoolkits)を参照してください。

<a id="aws-sdk-dotnet-context"></a>
#### Context

<details>
<summary>.NET SDK クライアントクラス</summary>

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
<summary>バケットの作成</summary>

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
<summary>バケット一覧の照会</summary>

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
<summary>バケットの照会（オブジェクト一覧の照会）</summary>

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
<summary>バケットの削除</summary>

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
<summary>ロックバケット</summary>

<code>PutBucketRequest</code> に <code>ObjectLockEnabledForBucket = true</code> を設定すると、ロックバケットを作成します。デフォルトの保持期間は 0 日に設定されます。

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

デフォルトの保持期間を設定するには、<code>PutObjectLockConfigurationAsync</code> メソッドを使用します。

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

ロック設定を照会するには、<code>GetObjectLockConfigurationAsync</code> メソッドを使用します。

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
<summary>オブジェクトのアップロード</summary>

!!! tip "ヒント"
    パートオブジェクトの数は、アップロードするオブジェクトのサイズと設定したパートサイズによって決まります。デフォルトのパートサイズは 5MiB で、最小 5MiB から指定できます。パートオブジェクトの最大数は 10,000 個です。

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
<summary>オブジェクトのダウンロード</summary>

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
<summary>オブジェクトの削除</summary>

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
<summary>署名付き URL の生成</summary>

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
                Verb = HttpVerb.GET,                 // アップロード用は HttpVerb.PUT
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
### ドメインスタイルエンドポイントの使用 { #aws-sdk-virtual-hosted-style }
AWS SDK でドメインスタイルエンドポイントを使用するには、クライアント設定でパススタイルアクセスを無効にします。エンドポイント URL と認証情報は従来と同様に使用し、SDK がバケット名をサブドメインとして組み合わせてリクエストを送信します。

<details>
<summary>Boto3 - Python SDK</summary>

<code>botocore.client.Config</code> の <code>s3.addressing_style</code> の値を <code>virtual</code> に指定してクライアントを作成します。

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

クライアントビルダーで <code>enablePathStyleAccess()</code> の呼び出しを削除します。

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

<code>AmazonS3Config</code> で <code>ForcePathStyle</code> プロパティの設定を削除します。

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

!!! danger "注意"
    バケット名にドット (`.`) が含まれている場合、ドメインスタイルを使用するとワイルドカード SSL 証明書の有効範囲外となり、証明書の検証に失敗する可能性があります。この場合は、パススタイルを使用してください。