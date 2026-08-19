<!-- machine_translated: true -->

<!-- pre-align:aligned sig=a08a3e30075c -->

<a id="storage-object-storage-troubleshooting-guide"></a>
## Storage > Object Storage > Troubleshooting Guide { #storage-object-storage-troubleshooting-guide }
This document describes how to resolve various issues you may encounter while using NHN Cloud Object Storage.

<a id="jdk-ssl-error"></a>
### SSL Error When an Application Using JDK Accesses Object Storage { #jdk-ssl-error }

```
peer not authenticated; nested exception is javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
```

A JDK bug that can occur when using TLS 1.3 with JDK 11.0.2.

1. Update the JDK to version 11.0.3 or later.
2. Or, add an option to use TLS 1.2 when running the application, as follows.

```
java -Djdk.tls.client.protocols=TLSv1.2 -jar
```


<a id="multipart-upload-korean-filename-error"></a>
### Multipart Upload Failure for Files with Korean Filenames in Windows Environments { #multipart-upload-korean-filename-error }

The default encoding for Korean characters on Windows OS is CP949 format, not Unicode. When multipart uploading files with names that contain Korean characters, you may receive an error response during the manifest object configuration step due to the difference in encoding, especially when using third-party tools such as Cyberduck that follow the system's default encoding settings.

The Windows OS provides encoding settings for languages for programs that do not support Unicode. You need to enable the **Use Unicode UTF-8** option through Settings or Control Panel, as follows

* **Settings** > **Time & Language** > **Date, Time, and Regional Languages** > **Additional Date, Time, and Regional Settings** > **Country or Region** > **Administrator Options** > **Languages for Programs that don't support Unicode** > **Change System Locale** > Check **Use Unicode UTF-8 for international language support** 

* **Control Panel** > **Clock and Country** > **Country or Region** > **Administrator Options** > **Languages for programs that do not support Unicode** > **Change System Locale** > Check **Use Unicode  UTF-8 for international language support** 