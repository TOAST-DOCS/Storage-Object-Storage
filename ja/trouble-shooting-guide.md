## Storage > Object Storage > トラブルシューティング
NHN Cloud Object Storageを使用する際に発生する様々な問題を解決する方法を説明します。

<h3>JDK を使用するアプリケーションで Object Storageにアクセスすると、次のような SSLエラーが発生します。</h3>

```
peer not authenticated; nested exception is javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
```

JDK 11.0.2でTLS 1.3を使う時発生する可能性があるJDKのバグです。

JDKを11.0.3以上のバージョンにアップデートするか、アプリケーションを実行する時、次のようにTLS 1.2を使うようにオプションを追加する必要があります。

```
java -Djdk.tls.client.protocols=TLSv1.2 -jar
```
