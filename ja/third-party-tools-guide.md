<!-- machine_translated: true -->

{% include-markdown '../_object-storage-vars.md' %}

<!-- pre-align:aligned sig=3eabcf443966 -->

<a id="third-party-tools-usage-guide"></a>
## サードパーティツールの使用ガイド { #third-party-tools-usage-guide }

この文書ではサードパーティツールでNHN Cloudオブジェクトストレージサービスを使用する方法を説明します。

<a id="cyberduck"></a>
## Cyberduck { #cyberduck }

Cyberduckはオープンソースのクラウドストレージブラウザです。

<a id="install-cyberduck"></a>
### Cyberduckのインストール { #install-cyberduck }

[Cyberduckダウンロードページ](https://cyberduck.io/download/)から使用しているOSに合ったインストールファイルをダウンロードしてインストールします。

<a id="cyberduck-object-storage-connection-settings"></a>

### オブジェクトストレージの接続設定 { #cyberduck-object-storage-connection-settings }

オブジェクトストレージに接続するには、接続情報を保存するブックマークを作成する必要があります。ブラウザー上部の**新規接続**をクリックした後、ドロップダウンリストから**OpenStack Swift(Keystone 2.0)**を選択します。必要な情報を入力して接続をクリックすると、新しいブックマークが作成されます。

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>項目</th>
    <th>説明</th>
  </tr>
  <tr>
    <td>別名</td>
    <td>ブックマークのエイリアスです。好きな値を自由に設定できます。</td>
  </tr>
  <tr>
    <td>サーバー</td>
    <td><b>Identityサービス(identity)</b>のアドレスです。オブジェクトストレージサービスページの<b>APIエンドポイント設定</b>ダイアログボックスで確認できます。</td>
  </tr>
  <tr>
    <td rowspan="2">Tenant ID:Access Key</td>
    <td><b>Tenant ID</b>：ユーザーのプロジェクトIDです。コンソールの <b>プロジェクト管理 > プロジェクト基本情報</b>で確認できます。</td>
  </tr>
  <tr>
    <td><b>Access ID</b>：NHN CloudアカウントID(メール形式)またはIAMアカウントIDを入力します。</td>
  </tr>
  <tr>
    <td>Secret Key</td>
    <td>オブジェクトストレージサービスページの<b>APIエンドポイント設定</b>ダイアログボックスで設定したAPIパスワードを入力します。</td>
  </tr>
</table>

!!! tip "ヒント"
    API パスワードの設定方法については、API ガイドの [認証および権限](api-guide$[ file_suffix ]$/#auth) を参照してください。

<a id="cyberduck-connect-object-storage"></a>
### オブジェクトストレージの接続 { #cyberduck-connect-object-storage }

接続するブックマークをダブルクリックしてオブジェクトストレージに接続します。

<a id="cyberduck-retrieve-container-or-object"></a>
### コンテナ/オブジェクトの照会 { #cyberduck-retrieve-container-or-object }

オブジェクトストレージに接続すると、**すべてのリージョン**のコンテナリストがブラウザに表示されます。コンテナをダブルクリックしてコンテナのオブジェクトリストを照会できます。

!!! tip "ヒント"
    異なるリージョンに同じ名前のコンテナがある場合、同じ名前のコンテナが複数表示されます。
    メニューから**表示 > 列 > 地域**を選択すると、列項目にリージョンを表示できます。

<a id="cyberduck-create-container"></a>
### コンテナの作成 { #cyberduck-create-container }

コンテナリストの空きスペースで右クリックし、**新規フォルダ...**を選択して新しいコンテナを作成できます。コンテナの名前とリージョンを入力し、**作成**をクリックすると、コンテナが作成されます。

!!! tip "ヒント"
    コンテナリストの空きスペースで右クリックし、**再表示**を選択すると、コンテナリストを更新できます。

<a id="cyberduck-upload-object"></a>
### オブジェクトのアップロード { #cyberduck-upload-object }

コンテナを選択した後、ブラウザ上部の**アクション** > **アップロード...**をクリックするか、オブジェクトリストで右クリックした後、**アップロード...**をクリックしてファイルを選択してアップロードします。

!!! tip "ヒント"
    Cyberduckでフォルダをアップロードまたは作成すると、フォルダ名と同じ0バイトのオブジェクトがもう一つ作成されます。これはコンソールまたはオブジェクトストレージAPIで確認でき、削除しても構いません。

<a id="cyberduck-download-object"></a>
### オブジェクトのダウンロード { #cyberduck-download-object }

ダウンロードするオブジェクトを選択した後、右クリックして**ダウンロード**を選択します。ドラッグ＆ドロップでオブジェクトをダウンロードすることもできます。

!!! tip "ヒント"
    オブジェクトをダウンロードすると、基本的にローカルの**ダウンロード**フォルダに保存されます。右クリックして**指定した場所にダウンロード**を選択すると、指定したパスにダウンロードできます。
    オブジェクトをアップロードまたはダウンロードすると、**転送**ウィンドウがポップアップして進行状況を把握できます。

<a id="cyberduck-delete-container"></a>
### コンテナの削除 { #cyberduck-delete-container }

削除するコンテナを選択し、右クリックして**削除**を選択します。

!!! danger "注意"
    コンテナを削除すると、コンテナ内のオブジェクトも全て削除されるので注意が必要です。

<a id="cyberduck-delete-object"></a>
### オブジェクトの削除 { #cyberduck-delete-object }

削除するオブジェクトを選択し、右クリックして**削除**を選択します。

<a id="cyberduck-synchronize-folder"></a>
### フォルダ同期 { #cyberduck-synchronize-folder }

ローカルフォルダをコンテナまたはフォルダと同期できます。コンテナまたはフォルダを選択した後、右クリックして**同期**を選択します。
フォルダ同期は、ダウンロード、アップロード、ミラーリングの3つの方法を提供します。

<a id="cyberduck-synchronize-download"></a>
#### ダウンロード

オブジェクトストレージで変更または追加されたオブジェクトをローカルにダウンロードします。

<a id="cyberduck-synchronize-upload"></a>
#### アップロード

ローカルで変更または追加されたファイルをオブジェクトストレージにアップロードします。

<a id="cyberduck-synchronize-mirror"></a>

#### ミラー

ローカルとオブジェクトストレージを比較して変更または不足しているファイルまたはオブジェクトをアップロードまたはダウンロードします。

!!! tip "ヒント"
    同期の詳細は、[CyberduckのSynchronize Folders](https://docs.cyberduck.io/cyberduck/sync/#synchronize-folders)ドキュメントを参照してください。

{% if terraform_support %}
<a id="terraform"></a>

## Terraform { #terraform }

Terraform は、インフラを簡単に構築し、安全に変更し、効率的に構成を管理できるオープンソースツールです。基本的な使用方法については、[ユーザーガイド > NHN Cloud > Terraform 使用ガイド]($[ terraform_guide_url ]$)を参照してください。

<a id="terraform-resource-dependency"></a>
### リソースの依存関係 { #terraform-resource-dependency }

一般的に各リソースは独立していますが、他の特定のリソースに依存関係を持つ場合もあります。リソースのラベルで他のリソースの情報を参照すると、Terraformは自動的に依存関係を設定します。
たとえば、`container1`コンテナに含まれる`object1`オブジェクトは次のように表現できます。

```hcl
# コンテナリソース
resource "nhncloud_objectstorage_container_v1" "container_1" {
  name = "container1"
}

# オブジェクトリソース
resource "nhncloud_objectstorage_object_v1" "object_1" {
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name           = "object1"
  source       = "/tmp/dummy"
}
```

!!! tip "ヒント"
    明示的なリソース依存関係の指定方法は、[TerraformのResource dependencies](https://developer.hashicorp.com/terraform/tutorials/configuration-language/dependencies)ドキュメントを参照してください。

<a id="terraform-resources-object-storage"></a>
### Resources - Object Storage { #terraform-resources-object-storage }

<a id="terraform-resources-create-container"></a>
#### コンテナの作成

```hcl
# 基本コンテナの作成
resource "nhncloud_objectstorage_container_v1" "container_1" {
  region = "KR1"
  name   = "tf-test-container-1"
}

# オブジェクトバージョン管理コンテナの作成
resource "nhncloud_objectstorage_container_v1" "container_2" {
  region = "KR1"
  name   = "tf-test-container-2"
  versioning_legacy {
    type     = "history"
    location = resource.nhncloud_objectstorage_container_v1.container_1.name
  }
}

# パブリックコンテナの作成
resource "nhncloud_objectstorage_container_v1" "container_3" {
  region = "KR1"
  name   = "tf-test-container-3"
  container_read = ".r:*,.rlistings"
}
```

| 名前 | タイプ | 必須 | 説明 |
| ---- | ---- | ---- | ---- |
| region | String | | NHN Cloudリソースを管理するリージョン |
| name | String | Y | コンテナ名 |
| container_read | String | | コンテナの読み取りに対する役割ベースのアクセスルールを設定 |
| container_write | String | | コンテナの書き込みに対するロールベースアクセスルールを設定 |
| force_destroy | Boolean | | コンテナを強制的に削除するかどうか、 `true`または`false`<br> 一緒に削除されたオブジェクトは復元できません。 |
| versioning_legacy | Object | | オブジェクトバージョン管理設定 |
| versioning_legacy.type | String | | `history`に指定 |
| versioning_legacy.location | String | | オブジェクトの旧バージョンを保管するコンテナ名 |

<a id="terraform-resources-create-object"></a>

#### オブジェクトの作成

```hcl
# オブジェクトの作成
resource "nhncloud_objectstorage_object_v1" "object_1" {
  region         = "KR1"
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name           = "test/test1.json"
  content_type = "application/json"
  content      = <<JSON
               {
                 "key" : "value"
               }
JSON
}

# ファイルアップロード
resource "nhncloud_objectstorage_object_v1" "object_2" {
  region         = "KR2"
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name           = "test/test2.json"
  source       = "./test2.txt"
}
```

<table>
	<thead>
			<tr>
				<th>名前</th>
				<th>タイプ</th>
				<th>必須</th>
				<th>説明</th>
			</tr>
	</thead>
	<tbody>
		<tr>
			<td>name</td>
			<td>String</td>
			<td>O</td>
			<td>オブジェクト名</td>
		</tr>
		<tr>
			<td>container_name</td>
			<td>String</td>
			<td>O</td>
			<td>コンテナ名</td>
		</tr>
		<tr>
			<td>source</td>
			<td>String</td>
			<td>O</td>
			<td> アップロードするローカルファイルシステム上のファイルパス<br><code>content</code>, <code>copy_from</code> または <code>object_manifest</code>と一緒に使用できません。<br><code>source</code>, <code>content</code>, <code>copy_from</code> または <code>object_manifest</code>のいずれかを必ず指定する必要があります。</td>
		</tr>
		<tr>
			<td>content</td>
			<td>String</td>
			<td>O</td>
			<td>作成するオブジェクトの内容<br><code>source</code>, <code>copy_from</code> または <code>object_manifest</code>と一緒に使用できません。</td>
		</tr>
		<tr>
			<td>copy_from</td>
			<td>String</td>
			<td>O</td>
			<td>コピーするソースオブジェクト、 <code>{コンテナ}/{オブジェクト}</code><br><code>source</code>, <code>content</code> または <code>object_manifest</code>と一緒に使用できません。</td>
		</tr>
		<tr>
			<td>object_manifest</td>
			<td>String</td>
			<td>O</td>
			<td>分割したセグメントオブジェクトをアップロードしたパス。<code>{Segment-Container}/{Segment-Object}/</code><br><code>source</code>、<code>content</code> または <code>copy_from</code>と一緒に使用できません。</td>
		</tr>
		<tr>
			<td>content_disposition</td>
			<td>String</td>
			<td></td>
			<td><code>Content-Disposition</code> ヘッダを指定</td>
		</tr>
		<tr>
			<td>content_encoding</td>
			<td>String</td>
			<td></td>
			<td><code>Content-Encoding</code> ヘッダを指定</td>
		</tr>
			<tr>
			<td>content_type</td>
			<td>String</td>
			<td></td>
			<td><code>Content-Type</code> ヘッダを指定</td>
		</tr>
		<tr>
			<td>delete_after</td>
			<td>Integer</td>
			<td></td>
			<td>オブジェクト有効時間、 UNIX時間(秒)</td>
		</tr>
		<tr>
			<td>delete_at</td>
			<td>String</td>
			<td></td>
			<td>オブジェクトの有効期限、 UNIX時間(秒), RFC3339フォーマット文字列</td>
		</tr>
		<tr>
			<td>detect_content_type</td>
			<td>Boolean</td>
			<td></td>
			<td>コンテンツタイプを推論するかどうか<br/>設定すると<code>content_type</code>が無視されます。</td>
		</tr>
	</tbody>
</table>

{% endif %}
<a id="reference"></a>

## 参考サイト { #reference }
Cyberduck - [https://docs.cyberduck.io/cyberduck/](https://docs.cyberduck.io/cyberduck/)
{%- if terraform_support %}
Terraform - [https://www.terraform.io/](https://www.terraform.io/)
Terraform Registry - [https://registry.terraform.io/](https://registry.terraform.io/)
{%- endif %}
