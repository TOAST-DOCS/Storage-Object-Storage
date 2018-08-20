## Storage > Object Storage > 控制台使用指南


### 创建Bucket

至少有一个以上的Bucket才能将对象上传到对象存储。

* **访问Bucket规则**
    * Private: 只有授权用户才能访问Bucket中的对象。
    * Public: 任何人都可以通过公开URL访问Bucket中的的对象。
* **Storage Class**
    * Standard : 默认值。

> [参考]
> Bucket名称限制为255个字符，中语85个字符。


### 删除Bucket
删除Bucket之前必须确保Bucket为空。如果对象保留在Bucket中，则无法删除。

> [参考]
> 删除文件夹也同样，如果对象仍然存在则无法删除。

### 创建文件夹

文件夹是管理对象存储中包含多个对象的虚拟单位。它类似于Windows的文件夹或Linux的目录，帮助用户分层管理对象。

> [参考]
> 文件夹名称限制为255个字符，中语85个字符。斜杠(/)作为分隔符来分隔文件夹。


### 上传对象

所有的对象要上传到Bucket里。一个对象的最大容量限制为5GB。

> [参考]
> 无法从Web控制台上传超过5GB的文件。
> 单个对象文件，容量大小不能超过5GB。超过的，则必须使用“split”等命令行工具对其进行分割，或在APP上编写程序把容量分割为5GB以下的大小后，分段上传。
> 详细使用方法请参考API指南的[上传multipart](api-guide/#_10).

### 下载对象

创建Bucket时如果访问Bucket的权限设置为Private，则只有授权用户才能访问对象。如果设置为Public，
则可以通过点击**Actions > Public URL查看**来确认对象的公开URL。
使用此URL可以创建对象的链接或可以直接下载对象。

**例**

* 创建网页

```
# cat > index.html
<html>
<body> hello world!
<a href="https://api-storage.cloud.toast.com/v1/{account}/{container}/{object}">Download</a>
</body>
</html>
```

* 运行Web服务器

```
# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

连接到Web浏览器后点击**Download**以确认下载文件。


### 复制对象
通过复制来创建新的对象。在目标Bucket中创建新的对象，也可以从其他Bucket直接复制。
