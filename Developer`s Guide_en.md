Infrastructure &gt; Object Storage &gt; Developer's Guide
---------------------------------------------------------

Preparation
-----------

### API Password Settings

1.  Click \[Infrastructure\] &gt; \[Object Storage\] menu

2.  Click on \[API Endpoint setting\] button at the top

3.  Appoint desired API password in &lt;API Endpoint&gt; chat window

### Check Tenant Name

In case of using API, to obtain Tenant Name which is used as a request parameter, refer to \[Project ID\] item’s value in Project List as shown in \[Figure 1\].

1.  Log in to Web Console

2.  Check Project List

3.  Tenant Name which project ID item value of Project List uses in API

<img src="http://static.toastoven.net/prod_infrastructure/object_storage/img_111.png" />

\[Figure 1\] Check Tenant Name

### Check Account

In case of using API, to obtain Account which is used as a request parameter, refer to Account name in &lt;API Endpoint&gt; chat window as shown in \[Figure 2\].

<img src="http://static.toastoven.net/prod_infrastructure/object_storage/img_112_sc.png" />

\[Figure 2\] Check Account

<img src="http://static.toastoven.net/prod_infrastructure/object_storage/img_113.png" />

\[Figure 3\] Account check chat window

Issue Verification Token
------------------------

Token is a verification key issued to use RESTful API of Object Storage. Container or Object that are not open to public will request with this Token or they cannot access otherwise. Token is managed by each Account.

    POST   https://api-compute.cloud.toast.com/identity/v2.0/tokens

\[Request parameter\]

| Name       | Type          | Property | Description                                                                    |
|------------|---------------|----------|--------------------------------------------------------------------------------|
| TenantName | Body or Plain | String   | Project ID of TOAST Cloud                                                      |
| Username   | Plain         | String   | TOAST Cloud’s user ID, User who is currently registered as a member in Project |
| Password   | Plain         | String   | User’s password, Password appointed in &lt;API Endpoint&gt; chat window        |

\[Request parameter\]

| Name                   | Type          | Property | Description                                                           |
|------------------------|---------------|----------|-----------------------------------------------------------------------|
| access.token.id        | Body or Plain | String   | Verification Token                                                    |
| access.token.tenant.id | Plain         | String   | Tenant ID that corresponds to Project which requested Token           |
| access.token.expires   | Plain         | String   | Expiration time of issued Token. Describe &lt;Time set as default&gt; |

\[Example of a request\]

    $ cat > token.json
    {
        “auth”: {
            “tenantName: “X1k372c9”,
            “passwordCredentials”: {
                “username”: ”…@nhnnet.com”,
                “password”: “toastcloud”
            }
        }
    }
    $ curl -X POST https://api-compute.cloud.toast.com/identity/v2.0/tokens -H "Content-Type:application/json" -d @token.json

\[Response result\]

    HTTP/1.1 200 OK
    Content-Type: application/json

    {
        "access": {
            "token": {
                "expires": "2015-11-23T03:14:23Z",
                "id": "bbefb86f8e7d41cfb40e582ca5ca39a8",
                "tenant": {
                    "description": "",
                    "enabled": true,
                    "id": "70c11b619c3f4a53af8d2466ee1b0234",
                    "name": "X1k372c9",
                    "project_domain": "NORMAL",
                    "swift": true
                },
                "issued_at": "2015-11-23T02:14:23.482540"
            },
            "serviceCatalog": [
                …
            ],
            "user": {
                "username": "…",
                "id": "…",
                "name": "…",
                "roles": [
                    {
                        "name": "project_admin"
                    }
                ],
                "roles_links": []
            },
            "metadata": {
                "roles": [
                    "…"
                ],
                "is_admin": 0
            }
        }
    }

> \[Notice\]
> Token issue response includes “expires” field. This field tells when the issued Token will expire. Without getting new Token every time, use this information to check Token’s expiration date and renew it.

Containers
----------

### Create Container

Container must be created to upload filed on Object Storage.

    PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}

\[Request parameter\]

| Name         | Type   | Property | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| X-Auth-Token | Header | String   | ID of issued Token                                             |
| Account      | URL    | String   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container    | URL    | String   | Name of Container to be created                                |

\[Example of a request\]

    $ curl -i -X PUT -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER

> \[Note\]
> No request contents needed to create Container only. If Container is successfully created, 201 will be returned as a status code.

### Look up Container

Look up information on appointed Container and the list of Object saved inside.

    GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}

\[Request parameter\]

| Name         | Type   | Property | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| X-Auth-Token | Header | String   | ID of issued Token                                             |
| Account      | URL    | String   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container    | URL    | String   | Name of Container to look up                                   |

\[Example of a request\]

    curl -i -X GET -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER

\[Example of a response\]

    TOAST-OBJECT

### Edit Container

Appoint access rules by changing metadata of Container.

    POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}

\[Request parameter\]

| Name              | Type   | Property | Description                                                                                                                                                                                                                                       |
|-------------------|--------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| X-Auth-Token      | Header | String   | ID of issued Token                                                                                                                                                                                                                                |
| X-Container-Read  | Header | String   | Appoint access rules for Container read.r:\* - Allow access of all users.r:example.com,test.com – Allow access of certain addresses, separated by ‘,’.rlistings. – Allow retrieval of Container list AUTH\_.... – Allow access to certain Account |
| X-Container-Write | Header | String   | Appoint access rules for Container write                                                                                                                                                                                                          |
| Account           | URL    | String   | User’s account name, Check in the chat window                                                                                                                                                                                                     |
| Container         | URL    | String   | Name of Container to be edited                                                                                                                                                                                                                    |

\[Example of a request\]

    $ curl -i -X POST -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" -H "X-Container-Read:.r:*" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER

> \[Note\]
> This request does not return response contents. In a normal request, 204 will be returned.

Once read permission is set to public, use commands like wget to check if it can retrieve without Token.

    $ curl -i -X GET https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER/TOAST-OBJECT

    This is test object.

### Delete Container

Delete appointed Container.

    DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}

\[Request parameter\]

| Name         | Type   | Property | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| X-Auth-Token | Header | String   | ID of issued Token                                             |
| Account      | URL    | String   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container    | URL    | String   | Name of Container to be deleted                                |

\[Example of a request\]

    $ curl -i -X DELETE -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER

> \[Note\]
> This request does not return response contents. Container to delete must be an empty Container. Otherwise an Error occurs. If created normally, 204 will be returned as a status code.

Object
------

### Upload Object

Create new Object in appointed Container.

    PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

\[Request parameter\]

| Name         | Type   | Property | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| X-Auth-Token | Header | String   | ID of issued Token                                             |
| Account      | URL    | String   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container    | URL    | String   | Container name                                                 |
| Object       | URL    | String   | Name of Object to create                                       |
| -            | Body   | Plain    | Contents of Object to create Text                              |

\[Example of a request\]

    curl -i -X PUT -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" -H "Content-type:text/plain" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER/TOAST-OBJECT -d "This is a test object."

> \[Note\]
> When requesting, set Header’s Content-type item value to match Object property. In a normal request, 201 will be returned as a status code.

### Edit Object Contents

If Object trying to create already exists, the contents of applicable Object will be modified to reflect.

    PUT   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

\[Request parameter\]

| Name         | Type   | Property | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| X-Auth-Token | Header | String   | ID of issued Token                                             |
| Account      | URL    | String   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container    | URL    | String   | Container name                                                 |
| Object       | URL    | String   | Object name to download                                        |

\[Example of a request\]

    $ curl -i -X PUT -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" -H "Content-type:text/plain" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER/TOAST-OBJECT -d "This object is modified."

> \[Note\]
> When requesting, set Header’s Content-type item value to match Object property. In a normal request, 201 will be returned as a status code.

### Download Object

    GET   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

\[Request parameter\]

| Name         | Type   | Property | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| X-Auth-Token | Header | String   | ID of issued Token                                             |
| Account      | URL    | String   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container    | URL    | String   | Container name                                                 |
| Object       | URL    | String   | Object name to download                                        |

\[Example of a request\]

    $ curl -i -X GET -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER/TOAST-OBJECT

\[Response result\]

    This object is modified

> \[Note\]
> Contents of Object will be changed to stream and returned. In case of a normal request, 200 will be returned as a status code.

### Copy Object

    COPY   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

\[Request parameter\]

|Name|Type|Property|Description| |X-Auth-Token| Header| String| ID of issued Token | |Destination| Header| String| Target to copy Object, Container name + name of copied Object| |Account| URL| String| User’s account name, Check in &lt;API Endpoint&gt; chat window| |Container| URL| String| Container name| |Object| URL| String| Object name to download|

\[Example of a request\]

    curl -i -X COPY -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" -H "Destination:TOAST-CONTAINER/COPIED-OBJECT" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER/TOAST-OBJECT

> \[Note\]
> Contents of Object will be changed to stream and returned. In a normal request, 201 will be returned as a status code.

### Edit Object Property

    POST   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

\[Request parameter\]

| Name                | Type   | Property                 | Description                                                    |
|---------------------|--------|--------------------------|----------------------------------------------------------------|
| X-Auth-Token        | Header | String                   | ID of issued Token                                             |
| Content-Type Header | String | Type of Object to change |                                                                |
| Account             | URL    | String                   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container           | URL    | String                   | Container name                                                 |
| Object              | URL    | String                   | Object name to download                                        |

\[Example of a request\]

    curl -i -X POST -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" -H "Content-Type:text/css" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER/COPIED-OBJECT

> \[Note\]
> This request does not return response contents. In a normal request, 202 will be returned as a status code.

### Delete Object

    DELETE   https://api-storage.cloud.toast.com/v1/{Account}/{Container}/{Object}

\[Request parameter\]

| Name         | Type   | Property | Description                                                    |
|--------------|--------|----------|----------------------------------------------------------------|
| X-Auth-Token | Header | String   | ID of issued Token                                             |
| Account      | URL    | String   | User’s account name, Check in &lt;API Endpoint&gt; chat window |
| Container    | URL    | String   | Container name                                                 |
| Object       | URL    | String   | Object name to download                                        |

\[Example of a request\]

    curl -i -X DELETE -H "X-Auth-Token:bbefb86f8e7d41cfb40e582ca5ca39a8" https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/TOAST-CONTAINER/COPIED-OBJECT

> \[Note\]
> This request does not return response contents. In a normal request, 204 will be turned as a status code.

### References

Swift API v1 - http://developer.openstack.org/api-ref-objectstorage-v1.html Identity API v2 - http://developer.openstack.org/api-ref-identity-v2.html
