<!-- machine_translated: true -->

<!-- pre-align:aligned sig=a08a3e30075c -->

<a id="storage-object-storage-troubleshooting-guide"></a>
## Storage > Object Storage > トラブルシューティング { #storage-object-storage-troubleshooting-guide }
このドキュメントは、NHN Cloud オブジェクトストレージを使用する際に発生するさまざまな問題を解決する方法を説明します。

<a id="jdk-ssl-error"></a>
### JDKを使用するアプリケーションのオブジェクトストレージアクセス時にSSLエラーが発生 { #jdk-ssl-error }

```
peer not authenticated; nested exception is javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
```

JDK 11.0.2でTLS 1.3を使う時発生する可能性があるJDKのバグです。

1. JDKを11.0.3以上のバージョンにアップデートします。
2. またはアプリケーションを実行する時、次のようにTLS 1.2を使うようにオプションを追加します。

```
java -Djdk.tls.client.protocols=TLSv1.2 -jar
```


<a id="multipart-upload-korean-filename-error"></a>
### Windows環境でハングルファイル名のマルチパートアップロード失敗 { #multipart-upload-korean-filename-error }

Windows OSの基本的なハングルエンコーディング方式は、UnicodeではなくCP949形式です。ハングルが含まれている名前のファイルをマルチパートアップロードする場合、エンコーディング方式の違いにより、マニフェストオブジェクトを構成する段階でエラーが発生することがあります。特に、システムのデフォルトのエンコーディング設定に従うCyberduckなどのサードパーティツールを使用する場合に発生する可能性があります。

Windows OSは、Unicodeをサポートしていないプログラム用の言語のエンコーディング設定を提供しています。次のように設定またはコントロールパネルから**Unicode UTF-8を使用する**オプションを有効にする必要があります。

* **設定** > **時間及び言語** > **日付、時間及び使用地域言語** > **追加日付、時間及び国別設定** > **国家または地域** > **管理者オプション** > **Unicodeをサポートしないプログラム用言語** > **システムロケールの変更** > **世界言語サポートするためにUnicode UTF-8使用**にチェック

* **コントロールパネル** > **時計及び国** > **国または地域** > **管理者オプション** > **Unicodeをサポートしないプログラム用言語** > **システムロケールの変更** > **世界言語サポートするためにUnicode UTF-8使用**にチェック
