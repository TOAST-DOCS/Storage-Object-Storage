## Storage > Object Storage > Amazon S3互換APIガイド
NHN CloudオブジェクトストレージはAWSのオブジェクトストレージS3 APIと互換性のあるAPIを提供します。したがって、AWS S3 APIを使用することを想定して開発されたアプリケーションは、設定を変更するだけで使用できます。

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

この文書はAPI使用方法の基礎的な部分のみを説明します。高度な機能を使用するには[Amazon S3 APIガイド](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)を参照するか、[AWS SDK](https://aws.amazon.com/jp/tools)を使用することを推奨します。

## S3 API認証情報(S3 API Credential)

### S3 API認証情報の発行
Amazon S3互換APIを使用するには、まずAWS EC2形式のS3 API認証情報を発行する必要があります。認証情報はWebコンソールまたはAPIを利用して発行できます。Webコンソールを利用した認証情報の発行は[S3 API認証情報](console-guide/#s3-api)項目を参照してください。

APIを利用して認証情報を発行するには認証トークンが必要です。認証トークンの発行は[オブジェクトストレージAPIガイド](api-guide/#tenant-id-api-endpoint)を参照してください。

```
POST    https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{api-user-id}/credentials/OS-EC2

Content-Type: application/json
X-Auth-Token: {token-id}
```

#### リクエスト

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 発行されたトークンID |
| api-user-id | URL | String | O | APIユーザーID、API Endpoint設定ダイアログボックスで確認可能 |
| tenant_id | Body | String | O | ユーザーテナントID。API Endpoint設定ダイアログボックスで確認可能 |

> [参考]
> `{api-user-id}`は、コンソールのAPI Endpoint設定ダイアログボックスで**APIユーザーID**項目を参照するか、認証トークン発行APIレスポンス本文の**access.user.id**フィールドで確認できます。
> 認証トークン発行APIを利用するにはAPIガイドの[認証トークン発行](api-guide/#_2)項目を参照してください。
>
> S3 API認証情報は有効期限がなく、ユーザーごとにプロジェクトあたり最大3個まで発行できます。

<!-- 改行用。 削除しないでください。 -->

> [注意]
> S3 API認証情報キーが漏洩すると、誰でも漏洩したキーを利用してオブジェクトにアクセスできます。キーが漏洩した場合は漏洩した認証情報を削除して、新しく発行して使用することを推奨します。

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
| access | Body | String | S3 API認証情報アクセスキー |
| secret | Body | String | S3 API認証情報シークレットキー |

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

### S3 API認証情報照会
発行されたS3 API認証情報を照会します。

**[Method, URL]**
```
GET   https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2

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
DELETE   https://api-identity-infrastructure.nhncloudservice.com/v2.0/users/{user-id}/credentials/OS-EC2/{access}

X-Auth-Token: {token-id}
```
#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| X-Auth-Token | Header | String | O | 発行されたトークンID |
| user-id | URL | String | O | ユーザーID。認証トークンに含まれている |
| access | URL | String | O | S3 API認証情報アクセスキー |

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
| リージョン名 | KR1 - 韓国(パンギョ)リージョン<br/>KR2 - 韓国(ピョンチョン)リージョン<br/>KR3 - 韓国(光州)リージョン<br/>JP1 - 日本(東京)リージョン<br/>US1 - 米国(カリフォルニア)リージョン |
| シークレットキー | 認証情報シークレットキー |


## バケット(Bucket)
### バケット作成
バケットを作成します。バケット名は次のようにAmazon S3の命名ルールに従う必要があります。

* バケット名は3文字から63文字にする必要があります。
* バケット名は英字(小文字)、数字、ドット(.)およびハイフン(-)のみ使用できます。
* バケット名の最初と最後の文字は英数字のみ使用できます。
* バケット名はIPアドレス形式(例：192.168.5.4)を使用しません。
* バケット名の最初の文字にxn--は使用できません。

詳細な内容は[Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html)文書を参照してください。

```
PUT /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

> [参考]
> WebコンソールまたはオブジェクトストレージAPIを通して作ったコンテナの名前がバケット命名ルールに違反している場合、S3互換APIにアクセスできません。

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | S3 API認証情報アクセスキーと署名で構成 |

#### レスポンス
このAPIはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード200を返します。

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| Location | Header | String | 作成したバケットパス |

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
| Authorization | Header | String | O | S3 API認証情報アクセスキーと署名で構成 |

#### レスポンス
リクエストが正しい場合、ステータスコード200とXML形式で構成されたバケットリストを返します。

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

### バケット照会
指定したバケットの情報と内部に保存されたオブジェクトリストを照会します。

```
GET /{bucket}

Date: Sat, 22 Feb 2020 22:22:22 +0000
Authorization: AWS {access}:{signature}
```

> [参考]
> WebコンソールまたはオブジェクトストレージAPIで作成したバケットの名前がバケット命名規則に違反する場合、S3互換APIではアクセスできません。

#### リクエスト
このAPIはリクエスト本文を要求しません。

| 名前 | 種類 | 形式 | 必須 | 説明 |
|---|---|---|---|---|
| bucket | URL | String | O | バケット名 |
| Date | Header | String | O | リクエスト時刻 |
| Authorization | Header | String | O | S3 API認証情報アクセスキーと署名で構成 |

#### レスポンス
リクエストが正しい場合、ステータスコード200とXML形式で構成されたオブジェクトリストトを返します。

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
| Authorization | Header | String | O | S3 API認証情報アクセスキーと署名で構成 |

#### レスポンス
このAPIはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード204を返します。

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
| Authorization | Header | String | O | S3 API認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ETag | Header | String | オブジェクトのMD5ハッシュ値 |
| Last-Modified | Header | String | オブジェクトの最後の修正日時(e.g. Wed, 01 Mar 2006 12:00:00 GMT) |

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
| Authorization | Header | String | O | S3 API認証情報アクセスキーと署名で構成 |

#### レスポンス

| 名前 | 種類 | 形式 | 説明 |
|---|---|---|---|
| ETag | Header | String | オブジェクトのMD5ハッシュ値 |
| Last-Modified | Header | String | オブジェクトの最後の修正日時(e.g. Wed, 01 Mar 2006 12:00:00 GMT) |

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
| Authorization | Header | String | O | S3 API認証情報アクセスキーと署名で構成 |

#### レスポンス
このAPIはレスポンス本文を返しません。リクエストが正しい場合、ステータスコード204を返します。

## AWSコマンドラインインターフェイス(CLI)
S3互換APIを利用して[AWSコマンドラインインターフェイス](https://aws.amazon.com/jp/cli/)でNHN Cloudオブジェクトストレージを使用できます。

### インストール
AWSコマンドラインインターフェイスはPythonパッケージで提供されます。Pythonパッケージ管理者(pip)を利用してインストールします。

```shell
$ sudo pip install awscli
```

### 設定
AWSコマンドラインインターフェイスを使用するには、先にS3 API認証情報と環境を設定する必要があります。

```shell
$ aws configure
AWS Access Key ID [None]: {access}
AWS Secret Access Key [None]: {secret}
Default region name [None]: {region name}
Default output format [None]: json
```

| 名前 | 説明 |
|---|---|
| access | S3 API認証情報アクセスキー |
| secret | S3 API認証情報シークレットキー |
| region name | KR1 - 韓国(パンギョ)リージョン<br/>KR2 - 韓国(ピョンチョン)リージョン<br/>KR3 - 韓国(光州)リージョン<br/>JP1 - 日本(東京)リージョン<br/>US1 - 米国(カリフォルニア)リージョン |

### S3コマンド使用方法

```shell
aws --endpoint-url={endpoint} s3 {command} s3://{bucket}
```

| 名前 | 説明 |
|---|---|
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - 韓国(パンギョ)リージョン<br/>https://kr2-api-object-storage.nhncloudservice.com - 韓国(ピョンチョン)リージョン<br/>https://kr3-api-object-storage.nhncloudservice.com - 韓国(光州)リージョン<br/>https://jp1-api-object-storage.nhncloudservice.com - 日本(東京)リージョン<br/>https://us1-api-object-storage.nhncloudservice.com - 米国(カリフォルニア)リージョン |
| command | AWSコマンドラインインターフェイスコマンド |
| bucket | バケット名 |


> [参考]
> AWSコマンドラインインターフェイスはAWSを使用するために提供されるツールのため、AWSドメインを使用するように設定されています。したがってTAOSTオブジェクトストレージを使用するには必ずコマンドごとにエンドポイントを指定する必要があります。
> AWSコマンドラインインターフェイスコマンドは[AWS CLIで高レベル(s3)コマンド使用](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-services-s3-commands.html)文書を参照してください。

<details>
<summary>バケット作成</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 mb s3://example-bucket
make_bucket: example-bucket
```

</details>

<details>
<summary>バケットリスト照会</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls
2020-07-13 10:07:13 example-bucket
```

</details>


<details>
<summary>バケット照会</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 ls s3://example-bucket
2020-07-13 10:08:49     104389 0428b9e3e419d4fb7aedffde984ba5b3.jpg
2020-07-13 10:09:09      74448 6dd6d48eef889a5dab5495267944bdc6.jpg
```

</details>

<details>
<summary>バケット削除</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rb s3://example-bucket
remove_bucket: example-bucket
```

</details>

<details>
<summary>オブジェクトアップロード</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp ./3b5ab489edffdea7bf4d914e3e9b8240.jpg s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
upload: ./3b5ab489edffdea7bf4d914e3e9b8240.jpg to s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

<blockquote>
[参考]
</br>
オブジェクトの容量が8MB以上の場合、AWSコマンドラインインタフェースはオブジェクトを複数のパートに分けてアップロードします。パートオブジェクトは <code style="display: inline;">{bucket}+segments</code>というバケットに <code style="display: inline;">{object-name}/{upload-id}/{part-number}</code>の形の名前で保存され、すべてのパートアップロードが終わったらアップロードをリクエストしたバケットにパートオブジェクトを接続したオブジェクトが作られます。
</br></br>
パートオブジェクトが保存される<code style="display: inline;">{bucket}+segments</code> バケットはS3互換APIではアクセスできず、Object Storage APIまたはコンソールからアクセスできます。
</br></br>
マルチパートオブジェクトのEtagは、各パートオブジェクトのETag値をバイナリデータに変換し、順番に接続して(concatenate) MD5ハッシュした値です。
</blockquote>

<blockquote>
[注意]
</br>
マルチパートでアップロードしたオブジェクトの一部または全体のパートオブジェクトを削除すると、オブジェクトにアクセスできなくなります。
</blockquote>

</details>

<details>
<summary>オブジェクトダウンロード</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 cp s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg ./3b5ab489edffdea7bf4d914e3e9b8240.jpg
download: s3://example-bucket/0428b9e3e419d4fb7aedffde984ba5b3.jpg to ./0428b9e3e419d4fb7aedffde984ba5b3.jpg
```

</details>

<details>
<summary>オブジェクト削除</summary>

```shell
$ aws --endpoint-url=https://kr1-api-object-storage.nhncloudservice.com s3 rm s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
delete: s3://example-bucket/3b5ab489edffdea7bf4d914e3e9b8240.jpg
```

</details>


## AWS SDK
AWSは多くのプログラミング言語用のSDKを提供しています。S3互換APIを利用してAWS SDKでNHN Cloudオブジェクトストレージを使用できます。

> [参考]
> 詳細については[AWS SDK](https://aws.amazon.com/ko/tools)文書を参照してください。


AWS SDKを使用するために必要な主要パラメータは次のとおりです。

| 名前 | 説明 |
|---|---|
| access | S3 API認証情報アクセスキー |
| secret | S3 API認証情報シークレットキー |
| region name | KR1 - 韓国(パンギョ)リージョン<br/>KR2 - 韓国(ピョンチョン)リージョン<br/>KR3 - 韓国(光州)リージョン<br/>JP1 - 日本(東京)リージョン<br/>US1 - 米国(カリフォルニア)リージョン |
| endpoint | https://kr1-api-object-storage.nhncloudservice.com - 韓国(パンギョ)リージョン<br/>https://kr2-api-object-storage.nhncloudservice.com - 韓国(ピョンチョン)リージョン<br/>https://kr3-api-object-storage.nhncloudservice.com - 韓国(光州)リージョン<br/>https://jp1-api-object-storage.nhncloudservice.com - 日本(東京)リージョン<br/>https://us1-api-object-storage.nhncloudservice.com - 米国(カリフォルニア)リージョン | |


### Boto3 - Python SDK

> [参考]
> 詳細については[AWS SDK for Python (Boto3)説明書](https://docs.aws.amazon.com/ko_kr/pythonsdk/?icmpid=docs_homepage_sdktoolkits)文書を参照してください。
#### Context

<details>
<summary>Boto3クライアントクラス</summary>

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
<summary>バケット作成</summary>

```python
    def create_bucket(self, bucket_name):
        return self.s3.create_bucket(Bucket=bucket_name)
```

</details>

<details>
<summary>バケットリスト照会</summary>

```python
def create_bucket(self, bucket_name):
    try:
        return self.s3.create_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>バケット照会(オブジェクトリスト照会)</summary>

```python
def list_buckets(self):
    try:
        return self.s3.list_buckets().get('Buckets')
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>バケット削除</summary>

```python
def delete_bucket(self, bucket_name):
    try:
        return self.s3.delete_bucket(Bucket=bucket_name)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>

<details>
<summary>オブジェクトアップロード</summary>

<blockquote>
<p>[参考]
パーツオブジェクトの数は、アップロードするオブジェクトの容量と設定したパーツサイズによって決定されます。基本パーツサイズは8MiBで、パーツオブジェクトの最大数は1000個です。</p>
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
<summary>オブジェクトダウンロード</summary>

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
<summary>オブジェクト削除</summary>

```python
def delete(self, bucket_name, key):
    try:
        return self.s3.delete_object(Bucket=bucket_name, Key=key)
    except ClientError as e:
        raise RuntimeError(e)
```

</details>


### Java SDK

> [参考]
> 詳細については[AWS SDK for Java説明書](https://docs.aws.amazon.com/ko_kr/sdk-for-java/index.html)文書を参照してください。
#### Context

<details>
<summary>Java SDKクライアントクラス</summary>

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
<summary>バケット作成</summary>

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
<summary>バケットリスト照会</summary>

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
<summary>バケット照会(オブジェクトリスト照会)</summary>

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
    }
```

</details>

<details>
<summary>バケット削除</summary>

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
    }
```

</details>

<details>
<summary>オブジェクトアップロード</summary>

<blockquote>
<p>[参考]
パーツオブジェクトの数は、アップロードするオブジェクトの容量と設定したパーツサイズによって決定されます。基本パーツサイズは5MiBで、パーツオブジェクトの最大数は1000個です。</p>
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
<summary>オブジェクトダウンロード</summary>

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
<summary>オブジェクト削除</summary>

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

> [参考]
> 詳細については[AWS SDK for .NET説明書](https://docs.aws.amazon.com/ko_kr/sdk-for-net/?icmpid=docs_homepage_sdktoolkits)文書を参照してください。
#### Context

<details>
<summary>.NET SDKクライアントクラス</summary>

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
<summary>バケットリストの照会</summary>

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
<summary>バケットの照会(オブジェクトリスト照会)</summary>

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
<summary>オブジェクトのアップロード</summary>

<blockquote>
<p>[参考]
パーツオブジェクトの数は、アップロードするオブジェクトの容量と設定したパーツサイズによって決定されます。基本パーツサイズは5MiBで、パーツオブジェクトの最大数は1000個です。</p>
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
