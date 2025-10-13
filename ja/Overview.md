## Storage > Object Storage > 概要
オブジェクトストレージはさまざまなタイプのデータを好きなだけ保存し、必要な時に取り出すことができるオブジェクトストレージサービスです。

<a id="service-characteristics"></a>
## サービスの特徴
<a id="outstanding-scalability"></a>
### 優れた拡張性
無制限に容量を増設できるため、ユーザーはストレージの容量を考慮する必要がなく、必要な時に必要なだけデータを保存できます。

<a id="high-stability-and-restoration-capability"></a>
### 高い安定性と復元力
データを安全に保存するために、オブジェクトを異なる複数のハードウェアに重複して保存します。一部の複製に意図しない問題が発生しても、完全な複製から迅速に復元されます。

<a id="control-accessibility"></a>
### アクセス制御
コンテナごとにアクセス権限を制御できます。誰でも照会できるようにパブリックに設定することもできますし、ロールベースのアクセス制御(RBAC)を利用してNHN Cloudのプロジェクトまたは個別ユーザー単位で読み書き権限を付与できます。

<a id="convenient-web-console"></a>
### 便利なWebコンソール
ユーザーの利便性のため、使い慣れた階層的なディレクトリ構造でデータを管理できるウェブコンソールを提供します。コンテナ内に仮想フォルダを作成し、オブジェクトをグループ化して管理できます。

<a id="rest-api"></a>
### REST API
コンテナとオブジェクトを制御できるHTTPベースのREST APIを提供します。REST APIを利用してユーザーのアプリケーションでオブジェクトストレージを使用できます。

<a id="amazon-s3-compatible-api"></a>
### Amazon S3互換API
Amazon S3と互換性のあるAPIを提供します。S3互換APIを利用して、Amazon Web Serviceソフトウェア開発キット(SDK)ベースのアプリケーションとさまざまなサードパーティのツールを使用できます。

<a id="life-cycle-control"></a>
### ライフサイクル制御
コンテナまたは個別オブジェクト単位でライフサイクル制御機能を提供します。これにより、ストレージ使用量を効率的に管理できます。

<a id="object-lock"></a>
### オブジェクトロック 
ユーザーの不注意による上書き、削除リクエストからデータを保護するためのオブジェクトロック機能(Write Once Read Many、WORM)を提供します。

<a id="version-management"></a>
### バージョン管理
バージョン管理機能を利用して、オブジェクトを更新、削除した履歴を管理し、以前のバージョンに復元できます。

<a id="disaster-recovery"></a>
### 災害復旧(disaster recovery)
コンテナ複製機能でコンテナのオブジェクトを他のリージョンのコンテナに複製できます。複製されたデータを利用して予期せぬ災害状況に備えることができます。

<a id="data-encryption-for-improved-security"></a>
### セキュリティ強化のためのデータ暗号化
サーバー側暗号化機能を利用して、重要なデータを暗号化してセキュリティを強化できます。暗号化に使用される対称キーは、NHN CloudのSecure Key Managerサービスで安全に管理されます。

<a id="convenient-cloud-accessibility"></a>
### 便利なクラウドアクセシビリティ
NHN CloudのService Gatewayサービスを利用して、外部ネットワークと隔離されたユーザーVPC内のインスタンスからプライベートネットワークを通じてオブジェクトストレージにアクセスできます。

<a id="access-history"></a>
### アクセス記録
NHN CloudのCloudTrailサービスによりオブジェクトストレージにアクセスした記録を提供します。

<a id="terminology"></a>
## 用語
<a id="object"></a>
#### オブジェクト(object)
オブジェクトストレージの基本的なデータ単位です。データとユーザーが付与したメタデータ、固有のアドレスで構成されます。

<a id="folder"></a>
#### フォルダ(folder)
オブジェクトをまとめる仮想単位です。WindowsのフォルダやLinuxのディレクトリと同様に階層的にオブジェクトを管理できます。

<a id="container"></a>
#### コンテナ(container)
オブジェクトを管理するための名前空間(namespace)です。さまざまな管理設定の単位になります。全てのオブジェクトは必ずコンテナ内に存在し、コンテナ内で唯一の名前を持ちます。

<a id="storage-account"></a>
#### ストレージアカウント(account)
オブジェクトストレージのユーザーアカウントです。NHN Cloudのオブジェクトストレージはアカウント単位で隔離されます。

<a id="api-endpoint"></a>
#### API Endpoint
オブジェクトストレージにREST APIでアクセスするために提供されるHTTP URLです。
