<!-- machine_translated: true -->

{% include-markdown '../_object-storage-vars.md' %}

<!-- pre-align:aligned sig=4d6c14c57aab -->

<a id="storage-object-storage-cli-guide"></a>
## Storage > Object Storage > CLIガイド { #storage-object-storage-cli-guide }

このドキュメントでは、OpenStack Swift コマンドラインインターフェース(CLI)でNHN Cloud Object Storageを使用する方法を説明します。

<a id="python-swiftclient"></a>
## python-swiftclient { #python-swiftclient }

<a id="install"></a>
### インストール { #install }

python-swiftclientはPythonパッケージとして提供されます。pipを利用してインストールします。

```bash
pip install python-swiftclient python-keystoneclient
```

!!! tip "ヒント"
    Python 3.7 以上が必要です。Python がインストールされていない場合は、[Python ダウンロードページ](https://www.python.org/downloads/)を参照してインストールします。

インストールが完了すると、次のコマンドで確認できます。

```bash
$ swift --version
python-swiftclient x.x.x
```

<br>

<a id="configuration"></a>

### 環境設定 { #configuration }

Swift CLIを使用するには、認証に必要な環境変数を設定する必要があります。Object Storageサービスページの**APIエンドポイント設定**ボタンをクリックして必要な情報を確認できます。

```bash
export OS_AUTH_URL=$[ identity_url ]$/v2.0
export OS_TENANT_ID=<テナントID>
export OS_USERNAME=<NHN Cloud アカウントID>
export OS_PASSWORD=<APIパスワード>
export OS_REGION_NAME=<リージョン名>
```

<br>

| 環境変数 | 説明 |
| --- | --- |
| OS_AUTH_URL | Identity API URL<br>$[ identity_url ]$/v2.0 |
| OS_TENANT_ID | Object Storageサービスページの**APIエンドポイント設定**で確認できるテナントID |
| OS_USERNAME | NHN CloudアカウントID(メール形式)またはIAMアカウントID |
| OS_PASSWORD | **APIエンドポイント設定**で設定したAPIパスワード |
| OS_REGION_NAME | リージョン名<br>{% for region in regions %}$[ region.code ]$ - $[ region.name ]${% if not loop.last %}<br>{% endif %}{% endfor %} |

<br>

!!! danger "注意"
    Object Storageは、基本インフラサービスとは異なるテナントIDを持っています。Object Storageサービスページの**APIエンドポイント設定**ボタンをクリックして確認してください。

<!-- 改行のためのコメント -->

!!! tip "ヒント"
    API パスワードは、Object Storage サービスページの **[API エンドポイント設定]** ボタンをクリックして設定できます。

<br>

設定ファイルを作成すると、毎回環境変数を設定しなくても便利に使用できます。

```bash
$ cat ~/swiftrc
export OS_AUTH_URL=$[ identity_url ]$/v2.0
export OS_TENANT_ID=<テナントID>
export OS_USERNAME=<NHN Cloud アカウント ID>
export OS_PASSWORD=<API パスワード>
export OS_REGION_NAME=<リージョン名>
```

```bash
$ source ~/swiftrc
```

<br>

環境設定が正しいか確認するには、`swift stat`コマンドを実行します。エラーなしでアカウント情報が出力されれば、正常に設定されています。

```bash
$ swift stat
        Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
     Containers: 0
        Objects: 0
          Bytes: 0
             ...
...
```

<br>

<a id="basic-usage"></a>
## 基本的な使用方法 { #basic-usage }

基本的な使用方法は次のとおりです。

```bash
swift <subcommand> [<options>] [<container> [<object>]]
```

<br>

<a id="subcommands"></a>
### サブコマンド { #subcommands }

主要なサブコマンドは次のとおりです。

| サブコマンド | 説明 |
| --- | --- |
| stat | アカウント、コンテナ、オブジェクトの情報を照会します。 |
| list | コンテナまたはオブジェクト一覧を照会します。 |
| post | コンテナを作成するか、コンテナおよびオブジェクトの設定を変更します。 |
| upload | オブジェクトをアップロードします。 |
| download | オブジェクトをダウンロードします。 |
| copy | オブジェクトをコピーします。 |
| delete | コンテナまたはオブジェクトを削除します。 |
| tempurl | 署名付きURLを作成します。 |
| auth | 認証関連の環境変数を出力します。 |

<br>

<a id="options"></a>
### 共通オプション { #options }

全てのサブコマンドに共通で使用できる主要なオプションです。

| オプション | 説明 |
| --- | --- |
| --help | 各サブコマンドの使用方法を確認します。 |
| --debug | 実際のAPI呼び出し履歴を確認します。 |
| --quiet | 進行状況を出力しません。 |
| --retries &lt;num&gt; | リクエスト失敗時の再試行回数を指定します。 |

<br>

<a id="auth-info"></a>

### 動作方式と認証情報の活用 { #auth-info }

Swift CLIは基本的にコマンドを実行するたびにIdentity APIを呼び出してトークンとサービスカタログを取得した後、サブコマンドに該当するSwift APIを呼び出します。

<br>

`auth`サブコマンドを使用すると、ストレージURLと認証トークンを事前に確認できます。

```bash
$ swift auth
export OS_STORAGE_URL=$[ object_storage_url ]$/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067
export OS_AUTH_TOKEN=gAAAAABi...
```

<br>

出力された環境変数をシェルに適用すると、以後のコマンド実行時に認証トークン発行とカタログ照会を省略するため、より高速に動作します。

```bash
$ eval $(swift auth)
```

!!! tip "ヒント"
    認証トークンには有効期限があります。トークンが期限切れになってリクエストが失敗した場合は、`eval $(swift auth)` を再度実行して更新する必要があります。

<br>

<a id="stat"></a>
## 情報照会 { #stat }

ストレージアカウント、コンテナ、オブジェクトの情報を照会します。

```bash
swift stat [<options>] [<container> [<object>]]
```

<br>

| オプション | 説明 |
| --- | --- |
| --lh | 容量を人間が読みやすい単位(KB、MBなど)で表示します。 |

<br>

<a id="stat-account"></a>
### ストレージアカウント情報照会 { #stat-account }

ストレージアカウントの情報を照会します。

```bash
$ swift stat
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
            Containers: 3
               Objects: 4
                 Bytes: 807711
          Content-Type: application/json; charset=utf-8
           X-Timestamp: 1772338036.37283
                    ...
```

<br>

<a id="stat-container"></a>
### コンテナ情報照会 { #stat-container }

コンテナの情報を照会します。

```bash
$ swift stat media
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
               Objects: 3
                 Bytes: 95983
              Read ACL:
             Write ACL:
          Content-Type: application/json; charset=utf-8
           X-Timestamp: 1772338036.37283
         Last-Modified: Sun, 01 Mar 2026 04:07:16 GMT
      X-Storage-Policy: standard
                    ...
```

<br>

<a id="stat-object"></a>
### オブジェクト情報照会 { #stat-object }

オブジェクトの情報を照会します。

```bash
$ swift stat media 797619b171a455e9eec8a87f94ee77f4.jpg
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
                Object: 797619b171a455e9eec8a87f94ee77f4.jpg
          Content Type: image/jpeg
        Content Length: 31661
         Last Modified: Sun, 01 Mar 2026 04:07:16 GMT
                  ETag: bd97323f11e8f2fc1655cbb8a1b9382e
           X-Timestamp: 1772338036.37283
                    ...
```

<br>

<a id="list"></a>
## 一覧照会 { #list }

コンテナまたはオブジェクト一覧を照会します。

```bash
swift list [<options>] [<container>]
```

<br>

| オプション | 説明 |
| --- | --- |
| --long | 各リソースの詳細情報を含む一覧を照会します。 |
| --lh | `--long`と似ていますが、容量を人間が読みやすい単位で表示します。 |
| --totals | `--long`または`--lh`と一緒に使用すると、合計のみ出力します。 |
| --json | JSON形式で一覧を出力します。 |
| --prefix &lt;prefix&gt; | 指定したプレフィックスで始まる項目のみを照会します。 |
| --delimiter &lt;delimiter&gt; | オブジェクト一覧照会でのみ使用できます。<br>区切り文字を基準にオブジェクト名をグループ化して表示します。<br>たとえば`--delimiter /`を使用すると、`pics/9b171a455e9e.png`のような名前のオブジェクトは一覧から除外され、フォルダのように`pics/`として表示されます。 |

<br>

<a id="list-containers"></a>
### コンテナ一覧照会 { #list-containers }

コンテナ一覧を照会します。

```bash
$ swift list
media
```

<br>

<a id="list-objects"></a>
### オブジェクト一覧照会 { #list-objects }

オブジェクト一覧を照会します。

```bash
$ swift list media
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br>

<a id="create-container"></a>
## コンテナ作成 { #create-container }

新規コンテナを作成します。

```bash
swift post <container>
```

```bash
$ swift post docs
$ swift list
docs
media
```

<br>

<a id="upload"></a>
## オブジェクトのアップロード { #upload }

オブジェクトをアップロードまたは上書き(overwrite)します。複数のオブジェクトを一括でアップロードできます。

```bash
swift upload <container> <file_or_directory> [<file_or_directory>] [...]
```

<br>

| オプション | 説明 |
| --- | --- |
| --object-name &lt;name&gt; | アップロードするオブジェクトの名前を指定します。<br>省略するとファイル名を使用します。 |
| --changed | コンテナに同じ名前のオブジェクトがある場合、変更時刻とサイズを比較して変更された場合のみアップロードします。 |
| --skip-identical | コンテナに同じオブジェクトがある場合はアップロードをスキップします。 |
| --segment-size &lt;size&gt; | 指定したサイズでファイルを分割し、SLO方式でマルチパートアップロードします。 |
| --segment-container &lt;container&gt; | セグメントオブジェクトを保存するコンテナを指定します。 |
| --leave-segments | オブジェクトを上書きする際、既存のセグメントを削除せずに維持します。 |
| --object-threads &lt;threads&gt; | オブジェクトアップロードに使用するスレッド数を指定します。デフォルト値は10です。 |
| --segment-threads &lt;threads&gt; | セグメントアップロードに使用するスレッド数を指定します。デフォルト値は10です。 |
| --meta &lt;name:value&gt; | メタデータを設定します。複数回使用できます。 |
| --ignore-checksum | アップロード時のチェックサム検証を無効にします。 |

<br>

```bash
$ swift upload media 4287b656254e69e948c3cab237f3fb03.jpg
4287b656254e69e948c3cab237f3fb03.jpg

$ swift list media
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
4287b656254e69e948c3cab237f3fb03.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br>

<a id="multipart-upload"></a>
### マルチパートアップロード { #multipart-upload }

`--segment-size`オプションでセグメントサイズを指定すると、ファイルを分割してマルチパートアップロードします。

```bash
swift upload --segment-size <size> [--segment-container <segment-container>] <container> <file>
```

<br>

`--segment-container`を指定しないと、`<container>_segments`という名前のコンテナを作成し、セグメントオブジェクトをアップロードします。

```bash
$ swift upload --segment-size 10m media test.mp4
test.mp4 segment 1
test.mp4 segment 0
test.mp4

$ swift list
media
media_segments

$ swift list media
...
test.mp4

$ swift list media_segments
test.mp4/slo/1635487628.192050/20971520/10485760/00000000
test.mp4/slo/1635487628.192050/20971520/10485760/00000001
```

<br>

<a id="download"></a>
## オブジェクトのダウンロード { #download }

オブジェクトを現在のパスにダウンロードします。複数のオブジェクトをダウンロードできます。

```bash
swift download <container> [<obj>] [...]
```

<br>

| オプション | 説明 |
| --- | --- |
| --output &lt;out_file&gt; | オブジェクトを保存するファイル名を指定します。<br>`-`を指定すると、オブジェクトの内容を標準出力に出力します。 |
| --output-dir &lt;out_directory&gt; | ダウンロードするパスを指定します。 |
| --prefix &lt;prefix&gt; | 指定したプレフィックスで始まるオブジェクトのみダウンロードします。 |
| --remove-prefix | `--prefix`と一緒に使用すると、プレフィックスを削除した名前でダウンロードします。 |
| --skip-identical | ローカルパスに同じファイルがある場合はダウンロードをスキップします。 |
| --object-threads &lt;threads&gt; | オブジェクトダウンロードに使用するスレッド数を指定します。デフォルト値は10です。 |
| --ignore-checksum | ダウンロード時のチェックサム検証を無効にします。 |
| --no-download | ダウンロードを実行しますが、ファイルをディスクに書き込みません。 |

<br>

```bash
$ swift download media 153206025029118e96e38b95b78281c8.jpg
153206025029118e96e38b95b78281c8.jpg [auth 0.117s, headers 0.288s, total 0.393s, 2.344 MB/s]
```

<br>

<a id="download-all"></a>
### コンテナ単位のダウンロード { #download-all }

ダウンロードするオブジェクトを省略すると、指定したコンテナの全てのオブジェクトをダウンロードします。

```bash
$ swift download media
300e5522b971c2159e65f95e3c89d981.jpg [auth 0.111s, headers 0.249s, total 0.264s, 0.376 MB/s]
797619b171a455e9eec8a87f94ee77f4.jpg [auth 0.303s, headers 0.451s, total 0.460s, 0.202 MB/s]
...
```

<br>

<a id="copy"></a>
## オブジェクトのコピー { #copy }

オブジェクトを指定したパスにコピーします。コピー時にメタデータを追加または変更できます。

```bash
swift copy [--destination </container/object>] [--fresh-metadata] [--meta <name:value>] <container> <object>
```

<br>

| オプション | 説明 |
| --- | --- |
| --destination &lt;/container/object&gt; | コピー先のコンテナとオブジェクト名を指定します。<br> オブジェクト名を省略すると、元の名前と同じ名前でコピーします。 |
| --meta &lt;name:value&gt; | メタデータを設定します。複数回使用できます。 |
| --fresh-metadata | 元のメタデータをコピーしません。<br>`--meta`と一緒に使用すると、新しく指定したメタデータのみを設定します。 |

<br>

<a id="copy-same-container"></a>
### 同じコンテナ内でのコピー { #copy-same-container }

対象オブジェクト名を元と異なるように指定する必要があります。

```bash
$ swift copy --destination "/media/copied_object.jpg" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container media
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /media/copied_object.jpg
```

<br>

<a id="copy-other-container"></a>
### 別のコンテナへのコピー { #copy-other-container }

対象オブジェクト名を省略すると、元と同じ名前でコピーします。

```bash
$ swift copy --destination "/pics" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container pics
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /pics/797619b171a455e9eec8a87f94ee77f4.jpg
```

<br>

<a id="copy-with-metadata"></a>
### メタデータの追加コピー { #copy-with-metadata }

`--meta`オプションを使用して、コピー時にメタデータを追加できます。

```bash
$ swift copy --destination "/media/copied_object.jpg" --meta "Color:Blue" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container media
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /media/copied_object.jpg
```

`--fresh-metadata`オプションを一緒に使用すると、既存のメタデータを削除し、新しく指定したメタデータのみを設定します。

<br>

<a id="post"></a>
## 設定変更 { #post }

コンテナまたはオブジェクトの設定を変更します。

<br>

<a id="post-container"></a>

### コンテナ設定の変更 { #post-container }

コンテナの設定を変更します。コンテナ設定は[コンテナ情報照会コマンド](#stat-container)で確認できます。

```bash
swift post [<options>] <container>
```

<br>

| オプション | 説明 |
| --- | --- |
| --meta &lt;key:value&gt; | コンテナメタデータを設定します。複数回使用できます。 |
| --read-acl &lt;acl&gt; | コンテナの読み取りACLを設定します。 |
| --write-acl &lt;acl&gt; | コンテナの書き込みACLを設定します。 |
| --header &lt;key:value&gt; | ユーザー定義HTTPヘッダを追加します。Swift CLIがオプションとして提供していない設定も、このオプションで直接指定できます。 |

<br>

!!! tip "ヒント"
    ACL 設定値については、[アクセスポリシー設定ガイド](acl-guide$[ file_suffix ]$/)を参照してください。

`--header` オプションで設定できるヘッダーの一覧については、API ガイドの [コンテナ設定の変更](api-guide$[ file_suffix ]$/#change-container-settings) セクションを参照してください。

<br>

```bash
$ swift post --meta "Type:photo" media
$ swift post --header "X-Container-Object-Lifecycle: 90" media
$ swift post --read-acl ".r:*" media
```

<br>

<a id="post-object"></a>
### オブジェクトメタデータの変更 { #post-object }

オブジェクトのメタデータを変更します。オブジェクトのメタデータは[オブジェクト情報照会コマンド](#stat-object)で確認できます。

```bash
swift post [<options>] <container> <object>
```

<br>

| オプション | 説明 |
| --- | --- |
| --meta &lt;key:value&gt; | オブジェクトメタデータを設定します。複数回使用できます。 |
| --header &lt;key:value&gt; | ユーザー定義HTTPヘッダを追加します。`Content-Type`などのオブジェクトプロパティを変更できます。 |

<br>

```bash
$ swift post --meta "Color:Blue" media 797619b171a455e9eec8a87f94ee77f4.jpg
$ swift post --header "Content-Type:image/png" media 797619b171a455e9eec8a87f94ee77f4.jpg
```

<br>

<a id="delete"></a>
## 削除 { #delete }

コンテナまたはオブジェクトを削除します。

<br>

<a id="delete-container"></a>
### コンテナの削除 { #delete-container }

指定したコンテナ内のオブジェクトを全て削除した後、コンテナを削除します。

```bash
swift delete <container>
```

<br>

| オプション | 説明 |
| --- | --- |
| --prefix &lt;prefix&gt; | 指定したプレフィックスで始まるオブジェクトのみ削除します。コンテナは削除しません。 |
| --object-threads &lt;threads&gt; | オブジェクト削除に使用するスレッド数を指定します。デフォルト値は10です。 |
| --container-threads &lt;threads&gt; | コンテナ削除に使用するスレッド数を指定します。デフォルト値は10です。 |

<br>

```bash
$ swift delete media
797619b171a455e9eec8a87f94ee77f4.jpg
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
```

<br>

<a id="delete-object"></a>
### オブジェクトの削除 { #delete-object }

指定したオブジェクトを削除します。

```bash
swift delete <container> <object> [<object>] [...]
```

<br>

| オプション | 説明 |
| --- | --- |
| --leave-segments | マニフェストオブジェクトのセグメントを削除せずに維持します。 |
| --object-threads &lt;threads&gt; | オブジェクト削除に使用するスレッド数を指定します。デフォルト値は10です。 |

<br>

```bash
$ swift delete media 797619b171a455e9eec8a87f94ee77f4.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br>

<a id="tempurl"></a>

## 署名付きURLの作成 { #tempurl }

トークンなしでコンテナまたはオブジェクトにアクセスできる署名付きURLを作成します。

```bash
swift tempurl <method> <time> <path> <key>
```

<br>

| 引数 | 説明 |
| --- | --- |
| method | 許可するHTTPメソッドです。一般的に`GET`または`PUT`を使用します。 |
| time | 署名付きURLの有効期限です。<br>ISO8601形式(`2026-03-01`、`2026-03-01T23:59:59`、`2026-03-01T23:59:59Z`)で入力します。<br>秒単位でも入力でき、現在から該当する時間が経過した後に有効期限が切れます。 |
| path | `/v1/AUTH_*****/<container>`または`/v1/AUTH_*****/<container>/<object>`の形式で入力します。 |
| key | コンテナに事前に登録したTemp-URL-Keyです。 |

<br>

| オプション | 説明 |
| --- | --- |
| --prefix-based | プレフィックスベースの署名付きURLを作成します。 |
| --ip-range &lt;ip_range&gt; | 署名付きURL経由のアクセスを特定のIPまたはIP範囲に制限します。 |
| --digest &lt;algorithm&gt; | ハッシュアルゴリズムを指定します。<br>`sha256`(デフォルト値)または`sha512`を使用できます。 |
| --absolute | `<time>`を秒単位で入力した際、現在からの相対時間ではなくUnixタイムスタンプとして解釈します。ISO8601形式には影響しません。 |
| --iso8601 | 作成されたURLに、Unixタイムスタンプの代わりにISO8601形式の有効期限を含めます。 |

<br>

```bash
$ swift tempurl GET 2026-03-05T23:59:59 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg 398dded8-b1be
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e479&temp_url_expires=1772755199

$ curl $[ object_storage_url ]$/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg\?temp_url_sig\=8244bff5037316dbe8aebcda9cd679c1b331e479\&temp_url_expires\=1772755199 --output 797619b171a455e9eec8a87f94ee77f4.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 31661  100 31661    0     0  1043k      0 --:--:-- --:--:-- --:--:-- 1058k
```

<br>

<a id="tempurl-key"></a>
### Temp-URL-Keyの登録 { #tempurl-key }

署名付きURLを作成するには、コンテナにTemp-URL-Keyを事前に登録する必要があります。設定されたキーは[コンテナ情報照会コマンド](#stat-container)で確認できます。

```bash
swift post --meta "Temp-URL-Key:<key>" <container>
```

```bash
$ swift post --meta "Temp-URL-Key:398dded8-b1be" media
$ swift stat media
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
               Objects: 3
                    ...
     Meta Temp-Url-Key: 398dded8-b1be
                    ...
```

<a id="reference"></a>
## References { #reference }

Object Storage service (swift) command-line client - [https://docs.openstack.org/ocata/cli-reference/swift.html](https://docs.openstack.org/ocata/cli-reference/swift.html)