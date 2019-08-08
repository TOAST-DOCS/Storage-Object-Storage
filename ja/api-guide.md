## Storage > Object Storage > APIガイド

## 事前準備

### Tenant Nameの確認

APIを利用する時、Tenant Nameをパラメータとして入力する必要があります。Tenant Nameはプロジェクト設定ページで確認できる[プロジェクトID]です。

1. WEBコンソールのプロジェクト設定ボタンをクリック
2. プロジェクトのID値を確認

### API Endpointの確認

APIのエンドポイントは、Object Storageサービスのページの`[API Endpoint設定]`ボタンをクリックして確認できます。

| 項目 | API Endpoint | 用途 |
|---|---|---|
| Identity | https://api-compute.cloud.toast.com/identity/v2.0 | 認証トークンの発行 |
| Object-Store | https://api-storage.cloud.toast.com/v1/{Account} | オブジェクトストレージの制御 |

> [参考]  
> APIで使用されるユーザーのAccountは`AUTH_***`形式の文字列です。Object-Store APIエンドポイントに含まれています。

### APIのパスワードの設定

APIのパスワードはObject Storageサービスページの`[API Endpoint設定]`ボタンをクリックして設定できます。

1. [API Endpoint設定]ボタンをクリック
2. API Endpoint設定 > APIパスワード設定項目に、トークン発行時に使用するパスワードを入力
3. パスワード入力後、保存ボタンをクリック

## 認証トークンの発行

認証トークンは、オブジェクトストレージのRESTful APIを使用する時に必要な認証キーです。外部公開に設定されていないコンテナやオブジェクトにアクセスするには、必ずトークンが必要です。トークンはアカウントごとに管理されます。

**[Method、 URL]**
```
POST    https://api-compute.cloud.toast.com/identity/v2.0/tokens
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|tenantId|	Body or Plain|	String|	Tenant ID. API Endpoint設定ダイアログウィンドウで確認可能。|
|username|	Plain|	String|	TOASTアカウントID(Email)を入力|
|password|	Plain|	String|	API Endpoint設定会話ウィンドウで保存したパスワード|

**[Request Body Example]**
```
{
  "auth": {
    "tenantId": "{Tenant ID}"、
    "passwordCredentials": {
      "username": "{TOAST ID}"、
      "password": "{API Password}"
    }
  }
}
```

**[Response Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|access.token.id|	Body or Plain|	String|	発行されたトークンID|
|access.token.tenant.id|	Plain|	String|	トークンをリクエストしたプロジェクトに対応するTenant ID|
|access.token.expires|	Plain|	String|	発行したTokenの満了時間。 yyyy-mm-ddTHH：MM：ssZの形式。例) 2017-05-16T03：17：50Z |

**[Response Body Example]**
```
{
    "access": {
        "token": {
            "expires": "2018-01-15T08:05:05Z"、
            "id": "b587ae461278419da6ecd21a2344c8aa"、
            "tenant": {
                "description": ""、
                "enabled": true、
                "id": "c6439197d5e74e4593ca37a62bbc09a6"、
                "name": "9AD5KRlH"、
                "groupId": "XEj2zkHrbA7modGU"、
                "project_domain": "NORMAL"、
                "swift": true
            }、
            "issued_at": "2018-01-15T07:05:05.719672"
        }、
        …
    }
}
```

> [注意]  
> トークンは有効時間があります。トークン発行リクエストのレスポンスに含まれる"expires"項目は発行されたトークンが満了する時間です。トークンが満了したら、新しいトークンの発行を受ける必要があります。


## コンテナ

### コンテナ作成
オブジェクトストレージにファイルをアップロードするには、必ずコンテナを作成する必要があります。

**[Method、 URL]**
```
PUT https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|種類|属性|説明|
|---|---|---|---|
|X-Auth-Token|Header|String|発行されたトークンID|
|Account|URL|String|ユーザーアカウント名、API Endpointに含まれている|
|Container|URL|String|作成するコンテナの名前|

> [参考]
> このAPIはレスポンス本文を返しません。コンテナが作成されると、状態コード201を返します。

### コンテナの照会
指定したコンテナの情報と、内部に保存されたオブジェクトのリストを照会します。

**[Method、 URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|種類|属性|説明|
|---|---|---|---|
|X-Auth-Token|Header|String|発行されたトークンのID|
|Container|URL|String|照会するコンテナの名前|

[Response Body Example]
```
[指定したコンテナに属するオブジェクトのリスト]
```

### コンテナの修正

コンテナのメタデータを変更して、アクセス規則を指定できます。

**[Method、 URL]**
```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {トークンID}
X-Container-Read: {コンテナ読み取りポリシー}
X-Container-Write: {コンテナ書き込みポリシー}
```

**[Request Parameters]**

|名前|種類|属性|説明|
|---|---|---|---|
|X-Auth-Token|Header|String|発行されたトークンのID|
|X-Container-Read|Header|String|コンテナの読み取りに対するアクセス規則指定<br/>.r:* - 全てのユーザーに対してアクセス許容<br/>.r:example.com、test.com – 特定アドレスに対してアクセス許容、 ‘,’で区分<br/>.rlistings. – コンテナリストの照会を許容<br/>AUTH_.... – 特定アカウントに対してアクセス許容|
|X-Container-Write|Header|String|コンテナの書き込みに対するアクセス規則指定|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|URL|String|修正するコンテナの名前|

**[Request Example]**
```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [トークンID]
X-Container-Read: .r:*
```

> [参考]
> このAPIはレスポンス本文を返しません。リクエストが正しければ、状態コード204を返します。

読み取り権限を公開に設定した後は`curl`、 `wget`などのツールを使用したり、ブラウザを通してトークンなしで照会されるか確認できます。

**[Verification Example]**
```
$ curl https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{オブジェクトの内容}
```

### コンテナの削除

指定したコンテナを削除します。

**[Method、 URL]**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンのID|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	削除するコンテナの名前|

> [参考]
> このリクエストはレスポンス本文を返しません。削除するコンテナは必ず空になっている必要があります。リクエストが正しければ、状態コード204を返します。

## オブジェクト

### オブジェクトのアップロード

指定したコンテナに新しいオブジェクトをアップロードします。

**[Method、 URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンID|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	作成するオブジェクトの名前|
|-|	Body|	Plain| Text	作成するオブジェクトの内容|

> [参考]
> リクエストヘッダに、オブジェクトの属性に合うContent-type項目を設定する必要があります。リクエストが正しければ、状態コード201を返します。


### マルチパートアップロード

5GBを超える容量のオブジェクトは5GB以下のセグメントに分割してアップロードする必要があります。

**[Method、 URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: [トークンID]
```

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたークンID|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	作成するオブジェクトの名前|
|Count| URL| String| 分割したオブジェクトの順番|
|-|	Body|	Plain| Text分割したオブジェクトの内容|


全てのオブジェクトのセグメントをアップロードした後、マニフェストオブジェクトを作成すると、1つのオブジェクトのように使用できます。マニフェストオブジェクトは、セグメントが保存されたパスを指します。

**[Method、 URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [トークンID]
X-Object-Manifest: {Container}/{Object}/
```

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンのID|
|X-Object-Manifest| Header| String | 分割したオブジェクトをアップロードしたパス、 `{Container}/{Object}/` |
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	作成するオブジェクトの名前|
|-|	Body|	Plain| Text分割したオブジェクトの内容|


**[Example]**
```
// 分割されたオブジェクトのアップロード
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/001 --data-binary '.....'
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/002 --data-binary '.....'

// マニフェストオブジェクトアップロード
$ curl -X PUT -H 'X-Auth-Token: *****' -H 'X-Object-Manifest: con/sample.jpg/' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg --data-binary ''

// オブジェクトのダウンロード
$ curl http://10.162.50.125/v1/AUTH_*****/con/sample.jpg > sample.jpg
```

### オブジェクトの内容の修正

オブジェクトアップロードAPIと同じですが、オブジェクトが既にコンテナに存在する場合、そのオブジェクトの内容が修正されます。

**[Method、 URL]**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンのID|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	内容を修正するオブジェクトの名前|

> [参考]
> リクエストヘッダにオブジェクト属性に合うContent-type項目を設定する必要があります。リクエストが正しければ、状態コード201を返します。

### オブジェクトのダウンロード

**[Method、 URL]**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンのID|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	ダウンロードするオブジェクトの名前|

> [参考]
> オブジェクトの内容がストリームで返されます。リクエストが正しければ、状態コード200を返します。

### オブジェクトのコピー

**[Method、 URL]**
```
COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンのID|
|Destination|	Header|	String|	オブジェクトをコピーする対象、 `{コンテナの名前} / {コピーされたオブジェクトの名前}`|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	コピーするオブジェクトの名前|


> [参考]
> このリクエストはレスポンス本文を返しません。リクエストが正しければ状態コード201を返します。

### オブジェクトのメタデータの修正

**[Method、 URL]**
```
POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンのID|
|Content-Type|	Header|	String|	変更するオブジェクトの形式|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	属性を修正するオブジェクトの名前|

> [参考]
> このリクエストはレスポンス本文を返しません。リクエストが正しければ、状態コード202を返します。

### オブジェクトの削除

**[Method、 URL]**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [トークンID]
```

**[Request Parameters]**

|名前|	種類|	属性|	説明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	発行されたトークンのID|
|Account|URL|String|ユーザーアカウント名、 API Endpointに含まれている|
|Container|	URL|	String|	コンテナの名前|
|Object|	URL|	String|	削除するオブジェクトの名前|

> [参考]
> このリクエストはレスポンス本文を返しません。リクエストが正しければ、状態コード204を返します。

## References

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
