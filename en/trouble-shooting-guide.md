## Storage > Object Storage > 문제 해결 가이드
NHN Cloud Object Storage를 사용하면서 겪을 수 있는 다양한 문제들을 해결하는 방법을 설명합니다.

<h3>JDK를 사용하는 애플리케이션에서 Object Storage에 접근하면 다음과 같은 SSL 오류가 발생합니다. </h3>

```
peer not authenticated; nested exception is javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
```

JDK 11.0.2에서 TLS 1.3을 사용할 때 발생할 수 있는 JDK 버그입니다. 

JDK를 11.0.3 이상 버전으로 업데이트하거나, 애플리케이션을 실행할 때 다음과 같이 TLS 1.2를 사용하도록 옵션을 추가해야 합니다.

```
java -Djdk.tls.client.protocols=TLSv1.2 -jar
```
