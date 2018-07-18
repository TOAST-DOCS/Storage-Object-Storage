## Storage > Object Storage > API Guide 

## Prerequisites 

### Check Tenant Name 

To use API, enter tenant name in parameters. Tenant name refers to **Project ID** which is available on the Project Setting page.  

1. Click **Project Setting** under the organization name on the left of the web console. 
2. Check **Project ID**. 

### Check API Endpoint 

To check API endpoint, click **API Endpoint Setting** on the Object Storage service page. 

| Item | API Endpoint | Usage |
|---|---|---|
| Object-Store | https://api-storage.cloud.toast.com/v1/{Account} | Control object storage |
| Identity | https://api-compute.cloud.toast.com/identity/v2.0 | Issue authentication token |

> [Note]  
> User account for API is composed of character strings in the `AUTH_***` format: included in the Object-Store API endpoint.

### Set API Passwords 

To set API password, click **API Endpoint Setting** on the Object Storage service page. 

1. Click **API Endpoint Setting**. 
2. Enter password to issue a token in **API Password Setting** under **API Endpoint Setting**. 
3. Click **Save**. 

## Issue Authentication Tokens 

An authentication token is an authentication key required to use REST API of object storage. To access containers or entities which are not set public, token is a necessity. Token is managed by each account.   

**Method, URL**
```
POST    https://api-compute.cloud.toast.com/identity/v2.0/tokens
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|tenantName|	Body or Plain|	String| TOAST Project ID |
|username|	Plain|	String| Enter TOAST account ID (email) |
|password|	Plain|	String| Password saved in **API Endpoint Setting** |

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

|Name| Type | Attribute | Description |
|---|---|---|---|
|access.token.id|	Body or Plain|	String| ID of issued token |
|access.token.tenant.id|	Plain|	String| Tenant ID in response to the project requesting for a token |
|access.token.expires|	Plain|	String| Time when issued token is expired, <br/>one hour after token is issued |

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

> [Note]  
> Token expires after valid period. "expires" included in the response to request for token issuance refers to token expiration time. If a token is expired, a new token needs to get issued.


## Containers 

### Create Containers 
To upload files to object storage, containers must be created. 

**Method, URL**
```
PUT https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name|Type|Attribute|Description|
|---|---|---|---|
|X-Auth-Token|Header|String|ID of issued token|
|Account|URL|String|User account name: included in API Endpoint|
|Container|URL|String|Container name to create|

> [Note]
> The API does not return response body: return status code 201, if a container has been created.

### Retrieve Containers 
Retrieve information of a specified container along with the list of objects saved in it.

**Method, URL**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name|Type|Attribute|Description|
|---|---|---|---|
|X-Auth-Token|Header|String|ID of issued token|
|Container|URL|String|Container name to retrieve|

**Response Body Example**
```
[List of objects included in a specified container]
```

### Modify Containers 

Specify access rules by changing meta data of a container. 

**Method, URL**
```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: {Token ID}
X-Container-Read: {Read Container Policy}
X-Container-Write: {Write Container Policy}
```

**Request Parameter**

|Name|Type|Attribute|Description|
|---|---|---|---|
|X-Auth-Token|Header|String|ID of issued token|
|X-Container-Read|Header|String|Specify access rules to read container <br/>.r:* - Allow access for all users <br/>.r:example.com,test.com – Allow access for particular address, delimited by ',' <br/>.rlistings. – Allow retrieval of container list <br/>AUTH_.... – Allow access for particular accounts|
|X-Container-Write|Header|String|Specify access rules for write containers|
|Account|URL|String|User account name: included in API Endpoint|
|Container|URL|String|Container name to modify|

**Request Example**
```
POST  https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
X-Container-Read: .r:*
```

> [Note]
> This API does not return response body: return status code 204 if properly requested.

After read authority is set public, check if retrieval without token is available by using tools, like `curl`and `wget`, or browser. 

**Verification Example**
```
$ curl https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

{content of object}
```

### Delete Containers 

Delete containers as specified. 

**Method, URL**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name to delete |

> [Note]
> This request does not return response body. Container to delete must be empty. Return status code 204 if properly requested. 

## Object 

### Upload Objects 

Upload new objects to containers as specified. 

**Method, URL**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to create |
|-|	Body|	Plain| Content of object to which text is to be created |

> [Note]
> Set appropriate content-type items for parameter attributes, at the request header. Return status code 201, if properly requested.   


### Upload Multiple Parts 

Objects exceeding 5GB must be uploaded in segments with less than 5GB . 

**Method, URL**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}/{Count}
X-Auth-Token: [Token ID]
```

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint.|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to create |
|Count| URL| String| Order of segmented objects |
|-|	Body|	Plain| Content of objects of which text is segmented |


Upload segments of all objects and create a manifest object, so as to apply as one object. Manifest object refers to the route where segments are saved. 

**Method, URL**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
X-Object-Manifest: {Container}/{Object}/
```

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|X-Object-Manifest| Header| String | Uploaded route for segmented objects, `{Container}/{Object}/` |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to create |
|-|	Body|	Plain| Content of object of which text is segmented Text |


**Example**
```
// Upload Segmented Objects 
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/001 --data-binary '.....'
$ curl -X PUT -H 'X-Auth-Token: *****' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg/002 --data-binary '.....'

// Upload Manifest Objects 
$ curl -X PUT -H 'X-Auth-Token: *****' -H 'X-Object-Manifest: con/sample.jpg/' http://10.162.50.125/v1/AUTH_*****/con/sample.jpg --data-binary ''

// Download Objects
$ curl http://10.162.50.125/v1/AUTH_*****/con/sample.jpg > sample.jpg
```

### Modify Objects 

If an object is already in a container, although its upload API is the same, the content shall change. 

**Method, URL**
```
PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Name of object of which content is to be modified |

> [Note]
> Content-type item appropriate for an object attribute must be set at the request header: return status code 201 if properly requested.  

### Download Objects 

**Method, URL**
```
GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to download |

> [Note]
> Object content is returned to stream: return status code 200 if properly requested. 

### Copy Objects 

**Method, URL**
```
COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Destination|	Header|	String| Target of copying for object, `{Container name} / {Name of copied object}` |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to copy |


> [Note]
> This request does not return response body: return status code 201 if properly requested. 

### Modify Object Meta Data

**Method, URL**
```
POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Content-Type|	Header|	String| Object format to change |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name of which attribute is to be modified |

> [Note]
> This request does not return response body: return status code 202 if properly requested.

### Delete Objects 

**Method, URL**
```
DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}
X-Auth-Token: [Token ID]
```

**Request Parameter**

|Name| Type | Attribute | Description |
|---|---|---|---|
|X-Auth-Token|	Header|	String| ID of issued token |
|Account|URL|String|User account name: included in API Endpoint|
|Container|	URL|	String| Container name |
|Object|	URL|	String| Object name to delete |

> [Note]
> This request does not return response body: return status code 204 if properly requested. 

## Reference 

Swift API v1 - [http://developer.openstack.org/api-ref-objectstorage-v1.html](http://developer.openstack.org/api-ref-objectstorage-v1.html)
