<!-- pre-align:aligned sig=eba0aa9ce90d -->

<a id="storage-object-storage-troubleshooting-guide"></a>
## Storage > Object Storage > 문제 해결 가이드 { #storage-object-storage-troubleshooting-guide }
이 문서는 NHN Cloud 오브젝트 스토리지를 사용하면서 겪을 수 있는 다양한 문제를 해결하는 방법을 설명합니다.

<a id="jdk-ssl-error"></a>
### JDK를 사용하는 애플리케이션의 오브젝트 스토리지 접근 시 SSL 오류 발생 { #jdk-ssl-error }

```
peer not authenticated; nested exception is javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
```

JDK 11.0.2에서 TLS 1.3을 사용할 때 발생할 수 있는 JDK 버그입니다.

1. JDK를 11.0.3 이상 버전으로 업데이트합니다.
2. 또는 애플리케이션을 실행할 때 다음과 같이 TLS 1.2를 사용하도록 옵션을 추가합니다.

```
java -Djdk.tls.client.protocols=TLSv1.2 -jar
```

<a id="multipart-upload-korean-filename-error"></a>
### Windows 환경에서 한글 파일명의 멀티파트 업로드 실패 { #multipart-upload-korean-filename-error }

Windows OS의 기본 한글 인코딩 방식은 유니코드가 아닌 CP949 형식입니다. 한글이 포함된 이름을 가진 파일을 멀티파트 업로드할 때 인코딩 방식의 차이로 인해 매니페스트 오브젝트를 구성하는 단계에서 오류 응답을 받을 수 있습니다. 특히 시스템의 기본 인코딩 설정을 따르는 Cyberduck 등의 서드파티 도구를 사용할 때 발생할 수 있습니다.

Windows OS는 유니코드를 지원하지 않는 프로그램용 언어를 위한 인코딩 설정을 제공합니다. 다음과 같이 설정 또는 제어판을 통해 **Unicode UTF-8 사용** 옵션을 활성화해야 합니다.

* **설정** > **시간 및 언어** > **날짜, 시간 및 사용지역 언어** > **추가 날짜, 시간 및 국가별 설정** > **국가 또는 지역** > **관리자 옵션** > **유니코드를 지원하지 않는 프로그램용 언어** > **시스템 로캘 변경** > **세계 언어 지원을 위해 Unicode UTF-8 사용** 체크

* **제어판** > **시계 및 국가** > **국가 또는 지역** > **관리자 옵션** > **유니코드를 지원하지 않는 프로그램용 언어** > **시스템 로캘 변경** > **세계 언어 지원을 위해 Unicode UTF-8 사용** 체크
