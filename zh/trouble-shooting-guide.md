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
