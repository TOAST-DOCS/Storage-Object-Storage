## Storage > Object Storage > Troubleshooting Guide
This section describes how to resolve various issues you may encounter while using NHN Cloud Object Storage.

<h3>When an application using the JDK accesses Object Storage, the following SSL errors will occur. </h3>

```
peer not authenticated; nested exception is javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
```

A JDK bug that can occur when using TLS 1.3 with JDK 11.0.2. 

You must update the JDK to version 11.0.3 or later, or add an option to use TLS 1.2 when running the application, as follows

```
java -Djdk.tls.client.protocols=TLSv1.2 -jar
```


<h3>Failure in multipart uploading of files with Korean characters in the name in Windows environments. </h3>

The default encoding for Korean characters on Windows OS is CP949 format, not Unicode. When multipart uploading files with names that contain Korean characters, you may receive an error response during the manifest object configuration step due to the difference in encoding, especially when using third-party tools such as Cyberduck that follow the system's default encoding settings.

The Windows OS provides encoding settings for languages for programs that do not support Unicode. You need to enable the **Use Unicode UTF-8** option through Settings or Control Panel, as follows

* **Settings** > **Time & Language** > **Date, Time, and Regional Languages** > **Additional Date, Time, and Regional Settings** > **Country or Region** > **Administrator Options** > **Languages for Programs that don't support Unicode** > **Change System Locale**>** Check **Use Unicode UTF-8 for international language support** 

* **Control Panel** > **Clock and Country** > **Country or Region** > **Administrator Options** > **Languages for programs that do not support Unicode** > **Change System Locale** > Check **Use Unicode  UTF-8 for international language support** 