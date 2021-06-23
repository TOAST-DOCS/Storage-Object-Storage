## Storage > Object Storage > API向导

## 事前准备

### 确认用户(tenant name)名称

使用API时，必须输入用户名作为参数。用户的名称是“项目ID”，可以在项目页面确认。

1. 在Web控制台左侧区域的组织(ORGANIZATION)名下方，点击齿轮形状的按钮，再点击“项目设置”按钮.
2. 确认“项目ID”值。

### 确认API Endpoint

API的Endpoint（地域节点）可以在Object Storage页面上点击“API Endpoint设置”按钮来确认。

| 项目 | API Endpoint | 用途 |
|---|---|---|
| Identity |  https://gov-api-identity.infrastructure.cloud.toast.com/v2.0 | 颁发身份验证令牌。|
| Object-Store | https://gov-api-storage.cloud.toast.com/v1/{Account} | 控制对象存储。|

> [参考]
> 用在API的用户帐户(account)是“AUTH_***”形式的字符串。它包含在Object-Store API Endpoint中。

### 设置API密码

可以通过单击Object Storage服务页面上的“API Endpoint设置”按钮来设置API密码。

1. 点击 “API Endpoint 设置” 按钮。
2. 在 “API Endpoint 设置” 下方的“设置API密码”的输入框中输入发出令牌时使用的密码。
3. 点击“保存”按钮。


## 颁发身份验证令牌

身份验证令牌是使用对象存储的REST API时所需的身份验证密钥。如需访问尚未设置为“Public”的Bucket或对象文件时，则需要令牌。令牌按帐户类别来管理。

**Method, URL**
```
POST    https://gov-api-identity.infrastructure.cloud.toast.com/v2.0/tokens
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|tenantName|	Body or Plain|	String|	NHN Cloud 项目ID|
|username|	Plain|	String|	输入NHN Cloud帐户 ID(邮箱) |
|password|	Plain|	String|	在**API Endpoint设置**保存的密码|

**Request Body Example**
```
{
  "auth": {
    "tenantName": "{Project ID}",
    "passwordCredentials": {
      "username": "{TOAST ID}",
      "password": "{API Password}"
    }
  }
}
```

**Response Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|access.token.id|	Body or Plain|	String|	颁发的令牌ID|
|access.token.tenant.id|	Plain|	String|	与请求令牌的项目相对应的Tenant ID |
|access.token.expires|	Plain|	String|	所发布令牌的到期时间。YYYY-MM-DDThh:mm:ssZ格式。例如) 2017-05-16T03:17:50Z |

**Response Body Example**
```
{
    "access": {
        "token": {
            "expires": "2018-01-15T08:05:05Z",
            "id": "b587ae461278419da6ecd21a2344c8aa",
            "tenant": {
                "description": "",
                "enabled": true,
                "id": "c6439197d5e74e4593ca37a62bbc09a6",
                "name": "9AD5KRlH",
                "groupId": "XEj2zkHrbA7modGU",
                "project_domain": "NORMAL",
                "swift": true
            },
            "issued_at": "2018-01-15T07:05:05.719672"
        },
        …
    }
}
```

> [注意]  
> 令牌具有效期。对响应令牌颁发请求中包含的“expires”栏是令牌到期的时间。令牌过期后，必须颁发新令牌。


## Bucket

### 创建Bucket
为了上传对象文件到对象存储中，需要您先要创建一个Bucket。

> [参考]
> 若容器或对象名中存在特殊字符 `! * ' ( ) ; : @ & = + $ , / ? # [ ]`时， 则使用API时必须进行URL编码（百分号编码）。   这些字符是URL中有重要作用的备用字符。若不对含有这些字符的路径进行URL编码就发送API请求，则无法收到所需响应。


**Method, URL**
```
PUT https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|种类|属性|说明|
|---|---|---|---|
|X-Auth-Token|Header|String|颁发的令牌ID|
|Account|URL|String|用户帐户名称, 包含在API Endpoint中|
|Container|URL|String|需创建的Bucket名称|

> [参考]
> 此API不会返回响应正文。如创建Bucket，则返回状态码201。

### 查询Bucket
查询指定的Bucket信息和保存在Bucket中的对象列表。

**Method, URL**
```
GET   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|种类|属性|说明|
|---|---|---|---|
|X-Auth-Token|Header|String|颁发的令牌ID|
|Container|URL|String|要查询的Bucket名称|

**Response Body Example**
```
[属于指定Bucket的对象列表]
```

###编辑Bucket

可以通过变更Bucket的元数据来指定访问的规则。

**Method, URL**
```
POST  https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {令牌ID}
X-Container-Read: {Bucket读规则}
X-Container-Write: {Bucket写规则}
```

**Request Parameter**

|名称|种类|属性|说明|
|---|---|---|---|
|X-Auth-Token|Header|String|颁发的令牌ID|
|X-Container-Read|Header|String|对Bucket读的访问规则指定<br/>.r:* - 允许所有用户的访问<br/>.r:example.com,test.com – 允许特定地址的访问, 以‘,’来分隔<br/>.rlistings. – 允许查询Bucket目录<br/>AUTH_.... – 允许特定账户的访问|
|X-Container-Write|Header|String|指定对写Bucket的访问规则|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|URL|String|要修改的Bucket名称|

**Request Example**
```
POST  https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [令牌ID]
X-Container-Read: .r:*
```

> [参考]
>此API不会返回响应正文。如果请求正确，则返回状态码204。

将读权限设置为“Public”后，可以使用`curl`, `wget`等道具，或通过浏览器可以在没有令牌的情况下直接访问。

**Verification Example**
```
$ curl https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{对象的内容}
```

### 删除Bucket

删除指定的Bucket。

**Method, URL**
```
DELETE   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|颁发的令牌ID|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|要删除的Bucket名称|

> [参考]
> 此请求不会返回响应正文。要删除的Bucket必须为空。如果请求正确，则返回状态码204。

## 对象

### 上传对象

在指定的Bucket中上传新对象文件。

**Method, URL**
```
PUT   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|颁发的令牌ID|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	要创建的对象的名称|
|-|	Body|	Plain| Text	要创建的对象的内容|

> [参考]
> 在请求header中设置符合对象属性的Content-type条目。如果请求正确，则返回状态码201。


### 分段上传

单个对象文件，容量大小不能超过5GB。超过的必须要分割成5GB以下，分段上传。

**Method, URL**
```
PUT   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: [令牌ID]
```

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	颁发的令牌ID|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	要创建的对象名称|
|Count| URL| String| 要分割对象的序列号|
|-|	Body|	Plain| Text 已分割对象的内容|


上传对象的所有段后，创建manifest对象，可作为一个对象来使用。
manifest是指保存这些段的路径。

**Method, URL**
```
PUT   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [令牌ID]
X-Object-Manifest: {Container}/{Object}/
```

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|颁发的令牌ID|
|X-Object-Manifest| Header| String | 上传已分割对象的路径, `{Container}/{Object}/` |
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	要创建的对象名称|
|-|	Body|	Plain| Text 已分割对象的内容|


**Example**
```
// 上传已分割对象
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/001 --data-binary '.....'
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/002 --data-binary '.....'

// 上传manifest对象
$ curl -X PUT -H 'X-Auth-Token: *****' -H 'X-Object-Manifest: con/sample.jpg/' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg --data-binary ''

// 下载对象
$ curl http://10.162.50.125/v1/AUTH_*****/con/sample.jpg > sample.jpg
```

### 修改对象内容

与对象上传API一样，Bucket中如果已经存在的对象，则修改该对象的内容。

**Method, URL**
```
PUT   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	颁发的令牌ID|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	需要修改内容的对象名称|

> [参考]
> 在请求header中设置符合对象属性的Content-type条目。如果请求正确，则返回状态码201。

### 下载对象

**Method, URL**
```
GET   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	颁发的令牌ID|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	要下载的对象名称|

> [参考]
> 对象的内容以stream的形式返还。如果请求正确，则返回状态代码200。
### 复制对象

**Method, URL**
```
COPY   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	颁发的令牌ID|
|Destination|	Header|	String|	复制对象到哪里, `{Bucket名称} / {已复制的对象的名称}`|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	要复制的对象名称|


> [参考]
>此请求不会返回响应正文。要删除的Bucket必须为空。如果请求正确，则返回状态码201。

### 更改对象元数据

**Method, URL**
```
POST   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	颁发的令牌ID|
|Content-Type|	Header|	String|	要更改的对象形式|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	要更改属性的对象名称|

> [参考]
>此请求不会返回响应正文。如果请求正确，则返回状态码202。

### 删除对象

**Method, URL**
```
DELETE   https://gov-api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [令牌ID]
```

**Request Parameter**

|名称|	种类|	属性|	说明|
|---|---|---|---|
|X-Auth-Token|	Header|	String|	颁发的令牌ID|
|Account|URL|String|用户账户名，包含在API Endpoint中|
|Container|	URL|	String|	Bucket名称|
|Object|	URL|	String|	要删除的对象名称|

> [参考]
> 此请求不会返回响应正文。如果请求正确，则返回状态码204。

## 参考事项

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
