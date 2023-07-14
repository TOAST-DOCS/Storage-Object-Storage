## Storage > Object Storage > アクセスポリシー設定ガイド
## 役割基盤アクセスポリシー

コンソールまたはAPIを使用して、他のユーザーにコンテナの読み取り/書き取りアクセス権限を付与できます。

### コンソール
コンソールでは[コンテナ作成](console-guide/#_2)ダイアログボックスまたは[コンテナ管理](console-guide/#_5)ウィンドウのコンテナアクセスポリシー設定ダイアログボックスでコンテナアクセスポリシーを選択できます。選択できるポリシーは`PRIVATE`と`PUBLIC`の2つに制限されます。

#### PRIVATE
`PRIVATE`は、コンテナが属すプロジェクトのユーザーにのみアクセス権限を付与する基本アクセスポリシーです。コンソールを利用するか、認証トークンを発行してAPIでコンテナにアクセスできます。 APIセッションの`コンテナが属すプロジェクトのユーザーにのみ読み取り/書き込み許可`項目と同じポリシーです。
<br/>

#### PUBLIC
`PUBLIC`は、誰でも読み取りとオブジェクトリスト照会を許可するポリシーです。コンテナをPUBLICに設定すると、コンソールでURLを取得できます。このURLを利用して誰でもコンテナにアクセスできます。 APIセッションの`すべてのユーザーに読み取り許可`項目と同じポリシーです。
<br/>

### API
APIを使用してコンテナの`X-Container-Read`, `X-Container-Write`プロパティにACLポリシー要素を入力すると、さまざまな状況に合わせてアクセスポリシーを設定できます。
<br/>

#### ACLポリシー要素

設定することができるACLポリシー要素は次のとおりです。すべてのポリシー要素はカンマ(,)で区切って組み合わせることができます。

| ポリシー要素 | 説明 |
| --- | --- |
| `.r:*` | 誰でも認証トークンなしでオブジェクトにアクセスできます。 |
| `.rlistings` | 読み取り権限があるユーザーにコンテナ照会(GETまたはHEADリクエスト)を許可します。<br/>このポリシー要素がなければオブジェクトリストを照会できません。<br/>がポリシー要素は単独で設定できません。 |
| `.r:<referrer>` | リクエストヘッダを参照して設定されたHTTPリファラー(HTTP Referer)にアクセスを許可します。<br/>認証トークンは必要ありません。 |
| `.r:-<referrer>` | リクエストヘッダを参照して設定されたHTTPリファラーのアクセスを制限します。<br/>リファラーの前にハイフン(-)をつけて設定します。 |
| `<tenant-id>:<api-user-id>>` | 特定プロジェクトに属す特定ユーザーに発行された認証トークンでオブジェクトにアクセスできます。<br/>書き込み、読み取り権限を付与できます。 |
| `<tenant-id>:*` | 特定プロジェクトに属すすべてのユーザーに発行された認証トークンでオブジェクトにアクセスできます。<br/>書き込み、読み取り権限を全て付与できます。 |
| `*:<api-user-id>` | プロジェクトに関係なく、特定ユーザーに発行された認証トークンでオブジェクトにアクセスできます。<br/>書き込み、読み取り権限を付与できます。 |
| `*:*` | プロジェクトに関係なく、認証トークンを発行できるユーザーなら誰でもオブジェクトにアクセスできます。<br/>書き込み、読み取り権限を付与できます。 |

> [参考]
> `<api-user-id>`は、コンソールのAPI Endpoint設定ダイアログボックスで**APIユーザーID**項目を参照するか、認証トークン発行APIレスポンス本文の**access.user.id**フィールドで確認できます。
> 認証トークン発行APIを利用するには、APIガイドの[認証トークン発行](api-guide/#_2)項目を参照してください。

<br/>

#### コンテナが属するプロジェクトのユーザーにのみ書き込み/読み取りを許可
ACLポリシー要素を設定しなかった時に使用される基本アクセスポリシーです。 APIを使用してコンテナにアクセスするには、必ず有効な認証トークンが必要です。
コンテナの`X-Container-Read`、`X-Container-Write`プロパティ値を全て削除すると、コンテナが属すプロジェクトのユーザーにのみアクセスを許可する`PRIVATE`コンテナになります。

<br/>

<details>
<summary>すべてのACLポリシー要素を削除する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

<blockquote>
<p>[参考]
curlを利用して値がないヘッダを送る時は、ヘッダ名にセミコロン(`;`)をつける必要があります。</p>
</blockquote>

有効な認証トークンなしでリクエストするとエラーメッセージを返します。

```
$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

リクエストヘッダに有効な認証トークンがなければレスポンスを受け取れません。

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[コンテナのオブジェクトリスト]
```
</details>
<br/>

#### すべてのユーザーに読み取り/リスト照会を許可
コンテナの`X-Container-Read`プロパティを`.r:*, .rlistings`に設定すると、すべてのユーザーにオブジェクトの読み取りとリスト照会を許可します。認証トークンは必要ありません。コンソールセッションの`PUBLIC`項目と同じポリシーです。
<br/>

<details>
<summary>全てのユーザーにオブジェクトの読み取りおよびリスト照会を許可する設定例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[コンテナのオブジェクトリスト]
```

<code>.r:*</code>だけ設定すると、コンテナのオブジェクトにはアクセスできますが、オブジェクトリストは照会できません。

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>


#### 特定HTTPリファラーのリクエストに読み取り許可/拒否
HTTPリファラー(HTTP Referer)は、ハイパーリンクを介してリクエストするWebページのアドレス情報です。リクエストヘッダに含まれています。
コンテナの`X-Container-Read`プロパティに`.r:<referrer>`または`.r:-<referrer>`形式のACLポリシー要素を設定すると、特定リファラーのアクセスリクエストを許可またはブロックできます。 ACLポリシー要素でHTTPリファラーを設定する時は、プロトコルとサブパスを除くドメイン名を入力する必要があります。

> [注意]
> HTTPリファラーはヘッダを編集してユーザーがいつでも変更できます。 HTTPリファラーを利用したアクセスポリシーはセキュリティに脆弱であるため推奨しません。

<details>
<summary>特定HTTPリファラーの読み取りリクエストを許可する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

APIリクエストヘッダに、許可されたHTTPリファラーアドレスを明示してリクエストすると、オブジェクトにアクセスできます。

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```

APIリクエストヘッダに許可されたリファラーアドレスがないか、リファラーアドレスにプロトコルが含まれていない場合は、アクセスがブロックされます。

```
$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: https://example.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

HTTPリファラー設定に、次のように<code>.</code>で始まるドメイン名を入力すると、設定されたドメインのすべてのサブドメインアドレスを含むリファラーに読み取りを許可します。

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -O -X GET \
  -H 'Referer: https://qux.baz.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```

サブドメインが含まれていないリクエストはブロックされます。

```
$ curl -X GET \
  -H 'Referer: https://foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

特定ドメイン名を持つすべてのリファラーのアクセスリクエストを許可するには、次のようにカンマで区切ったリストを利用して設定します。

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:foo.com, .r:.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -O -X GET \
  -H 'Referer: https://baz.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```
</details>

<details>
<summary>特定HTTPリファラーの読み取りリクエストをブロックする例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

HTTPリファラードメイン名の前にハイフンをつけて設定すると、設定されたHTTPリファラーリクエストがブロックされます。

```
$ curl -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>

HTTPリファラーのアクセス許可/ブロックポリシーは、入力した順序で適用されます。例えば、リファラーブロックポリシー要素の後ろに全てのアクセスを許可する`.r:*`ポリシー要素を入力すると、リファラーブロックポリシーは無視されます。反対に、全てのアクセスを許可するポリシー要素を先に入力して特定リファラーブロックポリシー要素を後ろに入力すると、設定されたリファラーのアクセスリクエストを除くすべてのアクセスリクエストが許可されます。
<br/>

<details>
<summary>HTTPリファラーブロックが無視される無効なポリシー設定例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com, .r:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -O -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```
</details>

<details>
<summary>特定HTTPリファラーのアクセスリクエストを除くすべてのアクセスリクエストを許可するポリシー設定例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br/>

#### 特定プロジェクトまたは特定ユーザーに書き込み/読み取りを許可
コンテナの`X-Container-Read`と`X-Container-Write`プロパティに`<tenant-id>:<api-user-id>`形式のACLポリシー要素を設定すると、特定プロジェクトまたは特定ユーザーに書き込み/読み取り権限をそれぞれ付与できます。テナントIDまたはAPIユーザーIDの代わりにワイルドカード文字`*`を入力すると、すべてのプロジェクトまたはすべてのユーザーにアクセス権限を付与します。アクセスリクエストを行う時は必ず有効な認証トークンが必要です。

> [参考]
> 認証トークンが必要なACLポリシーで付与された読み取り権限にはオブジェクトリスト照会権限が含まれています。

<details>
<summary>特定プロジェクトの特定ユーザーに書き込み/読み取り権限を付与する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{api-user-id}' \
  -H 'X-Container-Write: {tenant-id}:{api-user-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

オブジェクトにアクセスリクエストを行う時は、必ず許可されたテナントIDと、NHN CloudユーザーIDに発行された有効な認証トークンが必要です。

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[コンテナのオブジェクトリスト]


$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```
</details>

<details>
<summary>特定プロジェクトのすべてのユーザーに書き込み/読み取り権限を付与する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:*' \
  -H 'X-Container-Write: {tenant-id}:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

オブジェクトにアクセスリクエストを行う時は、必ず許可されたテナントIDと、該当するプロジェクトに属すNHN CloudユーザーIDに発行された有効な認証トークンが必要です。
<br/><br/>
</details>

<details>
<summary>プロジェクトに関係なく特定ユーザーに書き込み/読み取り権限を付与する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{api-user-id}' \
  -H 'X-Container-Write: *:{api-user-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

オブジェクトにアクセスリクエストを行う時は、必ず許可されたNHN CloudユーザーIDに発行された有効な認証トークンが必要です。
<br/><br/>
</details>

<details>
<summary>すべてのNHN Cloudユーザーに書き込み/読み取り権限を付与する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

オブジェクトにアクセスリクエストを行う時は、必ず有効な認証トークンが必要です。
</details>
<br/>

#### アクセスポリシーの削除
空のヘッダを入力すると、設定されたACLポリシー要素を全て削除できます。ACLポリシー要素がないコンテナは、許可されたユーザーのみアクセスできる**PRIVATE**コンテナになります。`コンテナが属すプロジェクトのユーザーにのみ書き込み/読み取り許可`項目を参照してください。


### References
Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)

## IPベースのアクセスポリシー

コンソールまたはAPIを使用してホワイトリストとブラックリストを指定して特定IPからコンテナの読み取り/書き込みアクセス権限を制限できます。ホワイトリストとブラックリストは同時に使用できません。ホワイトリストとブラックリストの両方を入力した場合、ホワイトリストのみ使用されます。 IPベースのアクセスポリシーはIPv4のみサポートします。


> [注意]
> 誤った設定により、書き込み権限がないコンテナが発生する可能性があります。この場合、設定を変更することはできません。このような問題が発生した場合はサポートにお問い合わせください。


### コンソール

コンテナ管理ウィンドウのコンテナアクセスポリシー設定ダイアログボックスでIPベースのコンテナアクセスポリシーを選択します。 

> [注意]
> 読み取り権限がない場合、コンソールで該当コンテナの操作はできなくなります。

#### ホワイトリスト
許可されたIPまたはネットワーク帯域を除く全てのリクエストを拒否します。リクエストを許可する読み取り、書き込み権限を指定できます。

#### ブラックリスト
指定されたIPまたはネットワーク帯域からのリクエストを拒否します。それ以外のすべてのリクエストは許可されます。許可ポリシーと一緒に使用する場合、拒否ポリシーは無視されます。リクエストを拒否する読み取り、書き込み権限を指定できます。


### API

APIを使用してコンテナの`X-Container-Ip-Acl-Allowed-List`, `X-Container-Ip-Acl-Denied-List`プロパティにACLポリシー要素を入力するとIPベースのACLを有効化できます。 `X-Container-Ip-Acl-Allowed-List`はホワイトリスト、`X-Container-Ip-Acl-Denied-List`はブラックリストを意味します。
<br>

ポリシー要素はアクセス権限と、IPまたはネットワーク帯域で構成されており、コンマ(`,`)で区切って複数の値を入力できます。アクセス権限は下記の表の通りです。

| アクセス権限 | 説明 |
| --- | --- |
| `r` | 読み取り権限、GET、HEADリクエストが該当します。 |
| `w` | 書き込み権限、PUT、POST、DELETE、COPYリクエストが該当します。|
| `a` | 読み取りと書き込み権限の両方を意味します。 GET、HEAD、PUT、POST、DELETE、COPYリクエストが該当します。 |


<details>
<summary>ホワイトリスト設定</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

192.168.0.1 IPは読み取りリクエストのみ、192.168.0.2 IPは書き込みリクエストのみ可能で、172.16.0.0/24帯域のすべてのIPはすべてのリクエストが可能です。それ以外のすべてのIPはリクエストが拒否されます。<br><br>

コンテナを変更するには、許可されたテナントIDと該当プロジェクトに属するNHN CloudユーザーIDで発行された有効な認証トークンが必要で、必ず許可されたIPからリクエストする必要があります。

<br/><br/>
</details>

<details>
<summary>ブラックリスト設定</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Denied-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

192.168.0.1 IPは読み取りリクエストが、 192.168.0.2 IPは書き込みリクエストが拒否され、172.16.0.0/24帯域のすべてのIPはすべてのリクエストができません。その他のすべてのIPはリクエストが許可されます。<br><br>

コンテナを変更するには、許可されたテナントIDと該当プロジェクトに属するNHN CloudユーザーIDで発行された有効な認証トークンが必要で、必ず許可されたIPからリクエストする必要があります。

<br/><br/>
</details>

<details>
<summary>ACL設定の削除</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List;' \
  -H 'X-Container-Ip-Acl-Denied-List;' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

コンテナを変更するには、許可されたテナントIDと該当プロジェクトに属するNHN CloudユーザーIDで発行された有効な認証トークンが必要で、必ず許可されたIPからリクエストする必要があります。

<br/><br/>
</details>
