<!-- machine_translated: true -->

{% include-markdown '../_object-storage-vars.md' %}

<!-- pre-align:aligned sig=13e8165cba5c -->

<a id="storage-object-storage-acl-configuration-guide"></a>
## Storage > Object Storage > アクセスポリシー設定ガイド { #storage-object-storage-acl-configuration-guide }

NHN Cloud オブジェクトストレージのコンテナに、ロールベースのアクセスポリシーおよび IP ベースのアクセスポリシーを設定する方法を説明します。

<a id="role-based-access-policies"></a>
## ロールベースのアクセスポリシー { #role-based-access-policies }

コンソールまたはAPIを使用して、他のユーザーにコンテナの読み取り/書き取りアクセス権限を付与できます。

<a id="role-based-access-console"></a>
### コンソール { #role-based-access-console }
コンソールでは[コンテナ作成](console-guide/#create-container)ダイアログボックスまたは[コンテナ管理](console-guide/#manage-container)ウィンドウのコンテナアクセスポリシー設定ダイアログボックスでコンテナアクセスポリシーを選択できます。選択できるポリシーは`PRIVATE`と`PUBLIC`の2つに制限されます。

コンソールでは、[コンテナの作成](console-guide$[ file_suffix ]$/#create-container)ダイアログボックス、または[コンテナ管理](console-guide$[ file_suffix ]$/#manage-container)ウィンドウのコンテナアクセスポリシー設定ダイアログボックスから、コンテナアクセスポリシーを選択できます。選択可能なポリシーは `PRIVATE` と `PUBLIC` の2種類に限られます。

<a id="role-based-access-private"></a>
#### PRIVATE

`PRIVATE` は、コンテナが属するプロジェクトのユーザーにのみアクセス許可を付与するデフォルトのアクセスポリシーです。コンソールを使用するか、認証トークンを発行して API でコンテナにアクセスできます。API セクションの `コンテナが属するプロジェクトのユーザーにのみ読み取り/書き込みを許可` 項目と同じポリシーです。
<br>

<a id="role-based-access-public"></a>
#### PUBLIC

`PUBLIC` はすべてのユーザーに読み取りとオブジェクトのリスト照会を許可するポリシーです。コンテナを `PUBLIC` に設定すると、コンソールで URL を取得できます。この URL を使用して、誰でもコンテナにアクセスできます。API セクションの `すべてのユーザーへの読み取り/リスト照会の許可` 項目と同じポリシーです。
<br>

<a id="role-based-access-api"></a>
### API { #role-based-access-api }

APIを使用して、コンテナの `X-Container-Read`、`X-Container-Write`、`X-Container-View` 属性にロールベースのアクセスポリシー要素を設定すると、さまざまな状況に応じてアクセスポリシーを設定できます。各属性は次のとおりです。

| 属性 | 説明                                                                                       |
| --- |------------------------------------------------------------------------------------------|
| X-Container-Read | コンテナ情報の照会と、コンテナ内オブジェクトの情報照会およびダウンロードを許可します。コンテナおよびオブジェクトの GET、HEAD リクエストが対象です。           |
| X-Container-Write | コンテナ内のオブジェクト変更リクエストを許可します。オブジェクトに対する PUT、POST、DELETE、COPY リクエストが該当します。                    |
| X-Container-View | コンテナ内のオブジェクト一覧の照会及びオブジェクトの情報照会を許可します。コンテナに対するGET、HEADリクエスト及びオブジェクトに対するHEADリクエストが該当します。 |


!!! tip "ヒント"
    `X-Container-Read`、`X-Container-Write`、`X-Container-View` に設定できるアクセスポリシー要素は、各属性につき最大 100 個です。この制限は[コンテナポリシー](container-policy-guide$[ file_suffix ]$/#acl)で設定する場合にも同様に適用されます。

<br>

<a id="role-based-access-elements"></a>
#### ロールベースのアクセスポリシー要素

設定できるロールベースのアクセスポリシー要素は次のとおりです。すべてのポリシー要素は、カンマ（`,`）で区切って組み合わせることができます。

| ポリシー要素 | 説明 |
| --- | --- |
| `{tenant-id}:{api-user-id}` | 特定のプロジェクトに属する特定のユーザーに発行された認証トークンを使用して、オブジェクトにアクセスできます。<br>読み取りおよび書き込みアクセス許可をどちらも付与できます。 |
| `{tenant-id}:*` | 特定のプロジェクトに属するすべてのユーザーに発行された認証トークンで、オブジェクトにアクセスできます。<br>読み取りおよび書き込み権限をどちらも付与できます。 |
| `*:{api-user-id}` | プロジェクトに関係なく、特定のユーザーに発行された認証トークンでオブジェクトにアクセスできます。<br>読み取りおよび書き込み権限をすべて付与できます。 |
| `*:*` | プロジェクトに関係なく、認証トークンを発行できるユーザーであれば誰でもオブジェクトにアクセスできます。<br>読み取りと書き込みのアクセス許可をどちらも付与できます。|

!!! tip "ヒント"
    `{api-user-id}` は、コンソールの API エンドポイント設定ダイアログの **[APIユーザーID]** 項目、または認証トークン発行 API のレスポンスボディの **access.user.id** フィールドで確認できます。
    認証トークン発行 API を使用するには、API ガイドの [認証および権限](api-guide$[ file_suffix ]$/#auth) を参照してください。

!!! tip "覚えておくこと"
    `{tenant-id}:` や `:{api-user-id}` のようにコロンの片側が空の値、`.` で始まる値は使用することはできません。

<a id="common-access-elements"></a>
#### その他のアクセスポリシー要素

`X-Container-Read` 属性には、ロールベースのアクセスポリシー要素以外に、次のポリシー要素も入力できます。

| ポリシー要素 | 説明 |
| --- | --- |
| `.r:*` | 誰でも認証トークンなしでオブジェクトにアクセスできます。 |
| `.r:{referrer}` | リクエストヘッダーを参照して設定された HTTP リファラーのアクセスを許可します。<br>認証トークンは必要ありません。 |
| `.r:-{referrer}` | リクエストヘッダーを参照して設定された HTTP リファラーのアクセスを制限します。<br>リファラーの前にマイナス記号（-）を付けて設定します。 |
| `.rlistings` | 認証トークンなしで読み取りが許可されたユーザーに対して、コンテナの照会（GET または HEAD リクエスト）を許可します。<br>このポリシー要素がない場合、オブジェクト一覧を照会することはできません。<br>このポリシー要素は単独で設定することはできません。 |


!!! tip "ヒント"
    リファラーで `*` は全体公開を意味する `.r:*` としてのみ使用できます。`*` を他の文字と組み合わせた値、全体をブロックする `.r:-*`、空の値は使用できません。

<br>

<a id="role-based-access-allow-rw-to-project-users"></a>
#### コンテナが属するプロジェクトのユーザーにのみ書き込み/読み取りを許可

コンテナの `X-Container-Read`、`X-Container-Write` 属性値をすべて削除すると、コンテナが属するプロジェクトのユーザーのみアクセスを許可する `PRIVATE` コンテナになります。

<br>

<details>
<summary>すべてのロールベースのアクセスポリシー要素を削除する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

!!! tip "ヒント"
    curl で値のないヘッダーを送信する際は、ヘッダー名にセミコロン（;）を付ける必要があります。

有効な認証トークンなしでリクエストすると、エラーメッセージが返されます。

```
$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

リクエストヘッダに有効な認証トークンがなければレスポンスを受け取れません。

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container

[コンテナのオブジェクトリスト]
```
</details>
<br>

<a id="role-based-access-allow-read-and-list-for-all-users"></a>
#### すべてのユーザーに読み取り/リスト照会を許可

コンテナの `X-Container-Read` 属性を `.r:*, .rlistings` に設定すると、すべてのユーザーにオブジェクトの読み取りと一覧の照会を許可します。認証トークンは必要ありません。コンソールセクションの `PUBLIC` 項目と同じポリシーです。
<br>

<details>
<summary>すべてのユーザーにオブジェクトの読み取りおよびリスト照会を許可する設定例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container

[コンテナのオブジェクト一覧]
```

`.r:*` のみを設定した場合、コンテナのオブジェクトにはアクセスできますが、オブジェクトの一覧は照会することはできません。

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br>

<a id="role-based-access-allow-or-deny-by-referer"></a>
#### 特定HTTPリファラーのリクエストに読み取り許可/拒否

HTTP リファラー (HTTP Referer) は、ハイパーリンクでリクエストされたウェブページのアドレス情報であり、リクエストヘッダーに含まれます。
コンテナの `X-Container-Read` 属性に `.r:{referrer}` または `.r:-{referrer}` 形式のロールベースアクセスポリシー要素を設定すると、特定のリファラーからのアクセスリクエストを許可またはブロックできます。ロールベースアクセスポリシー要素として HTTP リファラーを設定する場合は、プロトコルとサブパスを除いたドメイン名を入力する必要があります。

HTTP リファラーのアクセス許可/拒否ポリシーは、入力順序に関係なく拒否ポリシーが優先して適用されます。そのため、拒否対象として指定された HTTP リファラーのアクセスリクエストは、すべてのアクセスを許可する `.r:*` ポリシー要素を同時に入力した場合でも拒否されます。

!!! danger "注意"
    HTTP リファラーヘッダーは改ざんされる可能性があるため、アクセス制御の手段としてお勧めしません。

<details>
<summary>特定HTTPリファラーの読み取りリクエストを許可する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

APIリクエストヘッダに、許可されたHTTPリファラーアドレスを明示してリクエストすると、オブジェクトにアクセスできます。

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com/some/path' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```

API リクエストヘッダーに許可されたリファラーアドレスがない場合、またはリファラーアドレスにプロトコルが含まれていない場合は、アクセスがブロックされます。

```
$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: https://example.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

次のように HTTP リファラー設定に `.` で始まるドメイン名を入力すると、設定されたドメインのすべてのサブドメインアドレスを含むリファラーへの読み取りを許可します。

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -O -X GET \
  -H 'Referer: https://qux.baz.foo.com/some/path' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```

サブドメインが含まれていないリクエストはブロックされます。

```
$ curl -X GET \
  -H 'Referer: https://foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

特定のドメイン名を持つすべてのリファラーからのアクセスリクエストを許可するには、次のようにカンマ区切りリストを使用して設定します。

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:foo.com, .r:.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -O -X GET \
  -H 'Referer: https://baz.foo.com/some/path' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]
```
</details>

<details>
<summary>特定HTTPリファラーの読み取りリクエストをブロックする例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

HTTP リファラーのドメイン名の前にマイナス記号を付けて設定すると、該当する HTTP リファラーのリクエストをブロックします。

```
$ curl -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>

<details>
<summary>特定の HTTP リファラーを除くすべてのアクセスリクエストを許可する設定例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[オブジェクトのダウンロード]


$ curl -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br>

<a id="role-based-access-allow-rw-project-or-user"></a>
#### 特定プロジェクトまたは特定ユーザーに書き込み/読み取りを許可

コンテナの `X-Container-Read` と `X-Container-Write` 属性に `{tenant-id}:{api-user-id}` 形式のロールベースのアクセスポリシー要素を設定すると、特定のプロジェクトまたは特定のユーザーに読み取り/書き込みアクセス許可をそれぞれ付与できます。テナントIDまたはAPIユーザーIDの代わりにワイルドカード文字 `*` を入力すると、すべてのプロジェクトまたはすべてのユーザーにアクセス許可を付与します。アクセスをリクエストする際は、必ず有効な認証トークンが必要です。

!!! tip "知っておくこと"
    認証トークンが必要なアクセスポリシーで付与した読み取り権限には、オブジェクトリストの参照権限が含まれます。

<details>
<summary>特定プロジェクトの特定ユーザーに書き込み/読み取り権限を付与する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{api-user-id}' \
  -H 'X-Container-Write: {tenant-id}:{api-user-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

オブジェクトへのアクセスをリクエストする際は、許可されたテナントIDおよびAPIユーザーIDで発行された有効な認証トークンが必要です。

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container

[コンテナのオブジェクト一覧]


$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

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
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

オブジェクトへのアクセスをリクエストする際は、許可されたテナントIDとAPIユーザーIDで発行した有効な認証トークンが必要です。
<br><br>
</details>

<details>
<summary>プロジェクトに関係なく特定ユーザーに書き込み/読み取り権限を付与する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{api-user-id}' \
  -H 'X-Container-Write: *:{api-user-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

オブジェクトへのアクセスをリクエストする際は、プロジェクトに関係なく、許可されたAPIユーザーIDで発行した有効な認証トークンが必要です。
<br><br>
</details>

<details>
<summary>すべてのNHN Cloudユーザーに書き込み/読み取り権限を付与する例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

オブジェクトへのアクセスをリクエストする際は、有効な認証トークンが必要です。
</details>
<br>

<a id="role-based-access-delete-access-policies"></a>
#### アクセスポリシーの削除

空のヘッダーを入力すると、設定されているロールベースのアクセスポリシー要素をすべて削除できます。ロールベースのアクセスポリシー要素がないコンテナは、許可されたユーザーのみがアクセスできる **PRIVATE** コンテナになります。`컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용` の項目を参照してください。

<a id="role-based-access-references"></a>
### References { #role-based-access-references }
Swift Access Control Lists(ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)

<a id="ip-based-access-policies"></a>
## IPベースのアクセスポリシー { #ip-based-access-policies }

コンソールまたは API を使用してホワイトリストとブラックリストを指定し、特定の IP からコンテナの読み取り/書き込みアクセス権限を制限できます。ホワイトリストとブラックリストを同時に設定できますが、この場合はホワイトリストのみが適用され、ブラックリストは無視されます。IP ベースのアクセスポリシーは IPv4 のみをサポートします。サービスゲートウェイリクエストには、別途例外を指定できます。

!!! danger "注意"
    IPベースのアクセスポリシーは、パブリック IP を介したアクセスを制御するためのものです。ホワイトリストにプライベート IP のみを登録すると、アクセスできないコンテナになる可能性があります。
    誤った設定によりアクセス権限のないコンテナになった場合、ポリシーを変更することはできません。このような問題が発生した場合は、カスタマーセンターにお問い合わせください。

<a id="ip-based-access-console"></a>
### コンソール { #ip-based-access-console }

**[コンテナ管理]** ウィンドウの **[コンテナアクセスポリシー設定]** ダイアログボックスで、**[IPベースのコンテナアクセスポリシー]** を選択します。

!!! danger "注意"
    読み取り権限がない場合は、コンソールでコンテナを操作することはできません。

<a id="ip-based-access-whitelist"></a>
#### ホワイトリスト

許可された IP またはネットワーク帯域を除くすべてのリクエストを拒否します。リクエストを許可する読み取り、書き込みのアクセス許可を指定できます。

<a id="ip-based-access-blacklist"></a>
#### ブラックリスト

指定された IP またはネットワーク帯域のリクエストを拒否します。それ以外のすべてのリクエストは許可されます。ホワイトリストと併せて設定した場合、ブラックリストは無視されます。拒否するリクエストの読み取り、書き込み権限を指定できます。

<a id="ip-based-access-service-gateway-ip"></a>
#### サービスゲートウェイ IP

サービスゲートウェイを経由するリクエストを制御します。設定しない場合、ホワイトリストとブラックリストの設定に従ってリクエストが拒否される場合があります。

<a id="ip-based-access-api"></a>
### API { #ip-based-access-api }

API を使用してコンテナの `X-Container-Ip-Acl-Allowed-List`、`X-Container-Ip-Acl-Denied-List` 属性に IP ベースのアクセスポリシー要素を入力すると、IP ベースのアクセスポリシーを有効にできます。`X-Container-Ip-Acl-Allowed-List` はホワイトリスト、`X-Container-Ip-Acl-Denied-List` はブラックリストを意味します。

IPベースのアクセスポリシーが設定されたコンテナの属性を変更するには、許可されたテナントIDとAPIユーザーIDで発行した有効な認証トークンが必要であり、許可されたIPからリクエストする必要があります。

!!! tip "ヒント"
    `X-Container-Ip-Acl-Allowed-List`（ホワイトリスト）と `X-Container-Ip-Acl-Denied-List`（ブラックリスト）に設定できるポリシー要素は、それぞれ最大 100 個です。この制限は、[コンテナポリシー](container-policy-guide$[ file_suffix ]$/#ip-acl)で設定する場合にも同様に適用されます。

<br>

IPベースのアクセスポリシー要素は、アクセス権限とIPまたはネットワーク帯域で構成されており、カンマ(`,`)で区切って複数の値を入力できます。アクセス権限は次のとおりです。

| アクセス権限 | 説明 |
| --- | --- |
| `r` | 読み取り権限です。GET、HEAD リクエストが該当します。 |
| `w` | 書き込み権限です。PUT、POST、DELETE、COPY リクエストが対象です。 |
| `a` | 読み取りと書き込み権限の両方を意味します。 GET、HEAD、PUT、POST、DELETE、COPYリクエストが該当します。 |

サービスゲートウェイリクエストを制御するには、コンテナの X-Container-Ip-Acl-Service-Gateway-Control 属性にアクセス許可を設定します。設定できるアクセス許可は次のとおりです。

| 権限 | 説明 |
| --- | --- |
| `read` | 読み取りリクエストを許可します。 GET, HEADリクエストが該当します。 |
| `write` | 書き込みリクエストを許可します。PUT、POST、DELETE、COPY リクエストが該当します。 |
| `rw` | 読み書き全てのリクエストを許可します。 GET, HEAD, PUT, POST, DELETE, COPYリクエストが該当します。 |
| `deny` | 読み取りと書き込みのすべてのリクエストを許可しません。|

<details>
<summary>ホワイトリスト設定例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

192.168.0.1 は読み取りリクエストのみ、192.168.0.2 は書き込みリクエストのみ実行でき、172.16.0.0/24 帯域のすべての IP は読み取りと書き込みリクエストの両方を実行できます。それ以外のすべての IP はリクエストが拒否されます。

<br><br>
</details>

<details>
<summary>ブラックリスト設定の例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Denied-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

192.168.0.1は読み取りリクエストが、192.168.0.2は書き込みリクエストが拒否され、172.16.0.0/24帯域のすべてのIPは読み取りと書き込みリクエストをいずれも実行することはできません。その他のすべてのIPはリクエストが許可されます。

<br><br>
</details>

<details>
<summary>サービスゲートウェイリクエスト制御の例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Service-Gateway-Control: rw' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

設定された IP ベースのアクセスポリシーに関係なく、サービスゲートウェイからのリクエストはすべて許可されます。

<br><br>
</details>

<a id="ip-based-access-delete-access-policies"></a>
#### アクセスポリシーの削除

空のヘッダーを入力すると、設定されたIPベースのアクセスポリシー要素をすべて削除できます。ポリシー要素がないコンテナは、IPによるアクセス制限を受けません。

<details>
<summary>IPベースのアクセスポリシー要素の削除例</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List;' \
  -H 'X-Container-Ip-Acl-Denied-List;' \
  -H 'X-Container-Ip-Acl-Service-Gateway-Control;' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

<br><br>
</details>
