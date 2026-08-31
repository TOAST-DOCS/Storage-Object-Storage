<!-- machine_translated: true -->

{% include-markdown '../_object-storage-vars.md' %}

<!-- pre-align:aligned sig=13e8165cba5c -->

<a id="storage-object-storage-acl-configuration-guide"></a>
## Storage > Object Storage > ACL Configuration Guide { #storage-object-storage-acl-configuration-guide }

This document describes how to set role-based access policies and IP-based access policies for containers in NHN Cloud Object Storage.

<a id="role-based-access-policies"></a>
## Role-based Access Policies { #role-based-access-policies }

You can use the console or API to grant read/write access to the container to other users.

<a id="role-based-access-console"></a>
### Console { #role-based-access-console }
In the console, you can select a container access policy from the [Create Container](console-guide/#create-container) dialog box or the container access policy settings dialog box in the [Manage Container](console-guide/#manage-container) window. There are two policies that can be selected: `PRIVATE` and `PUBLIC`.

<a id="role-based-access-private"></a>
#### PRIVATE

`PRIVATE` is the default access policy that grants access only to users of the project to which the container belongs. Users can access the container via the console or the API by obtaining an authentication token. It's the same policy as the `Allow read/write only to users in the project to which the container belongs` in the API section.
<br>

<a id="role-based-access-public"></a>
#### PUBLIC

`PUBLIC` is a policy that allows anyone to read and query the object list. If you set the container to PUBLIC, you can get the URL from the console. Anyone can access the container using this URL. It's the same policy as the `Allow read/list query for all users` entry in the API section.
<br>

<a id="role-based-access-api"></a>
### API { #role-based-access-api }

You can use the API to set access policies for different situations by entering role-based access policy elements in the `X-Container-Read`, `X-Container-Write`, and `X-Container-View` properties of a container. The properties are as follows.

| Property | Description                                                                                       |
| --- |------------------------------------------------------------------------------------------|
| X-Container-Read | Allows querying container information and querying and downloading object information within the container. This applies to GET and HEAD requests for containers and objects.           |
| X-Container-Write | Allows object change requests within a container. This includes PUT, POST, DELETE, and COPY requests for objects.                    |
| X-Container-View | Allow viewing a list of objects within a container and viewing information about them. This includes GET and HEAD requests for containers and HEAD requests for objects. |


!!! tip "Tip"
    You can set up to 100 access policy elements per property in `X-Container-Read`, `X-Container-Write`, and `X-Container-View`. This limit also applies when configuring with the [container policy](container-policy-guide/#acl).

<br>

<a id="role-based-access-elements"></a>
#### Role-based access policy elements

The role-based access policy elements that can be set are as follows. All policy elements can be combined by separating them with commas (`,`).

| Policy Element | Description |
| --- | --- |
| `{tenant-id}:{api-user-id}` | The object can be accessed using an authentication token issued to a specific user in a specific project.<br>Both read and write permissions can be granted. |
| `{tenant-id}:*` | The object can be accessed using an authentication token issued to all users in a specific project.<br>Both read and write permissions can be granted. |
| `*:{api-user-id}` | The object can be accessed using an authentication token issued to a specific user, regardless of the project.<br>Both read and write permissions can be granted. |
| `*:*` | Regardless of the project, any user who can obtain an authentication token can access the object.<br>Both read and write permissions can be granted. |

!!! tip "Tips"
    You can find `{api-user-id}` in the **API User ID** field of the API endpoint settings dialog on the console, or in the **access.user.id** field of the authentication token issuance API response body.
    To use the authentication token issuance API, refer to the [Authentication and Authorization](api-guide/#auth) section in the API guide.

!!! tip "Note"
    You cannot use values where one side of the colon is empty (such as `{tenant-id}:` or `:{api-user-id}`) or values that start with `.`.

<a id="common-access-elements"></a>
#### Other Access Policy Elements

In addition to role-based access policy elements, you can also enter the following policy elements in the `X-Container-Read` property.

| Policy Element | Description |
| --- | --- |
| `.r:*` | Allow anyone to access the object without an authentication token. |
| `.r:{referrer}` | Allows access to the HTTP referer set by referring to the request header.<br>No authentication token is required. |
| `.r:-{referrer}` | Restricts the access of the HTTP referer set by referring to the request header.<br>It is set by adding a minus sign (-) in front of the referer. |
| `.rlistings` | Allows a container query (GET or HEAD request) to users who are allowed to read without an authentication token.<br>Without this policy element, the list of objects cannot be queried.<br>This policy element cannot be set alone. |


!!! tip "Note"
    In referrers, `*` can only be used as `.r:*`, which means full public access. Values that combine `*` with other characters, `.r:-*` for blocking all access, and empty values cannot be used.

<br>

<a id="role-based-access-allow-rw-to-project-users"></a>
#### Allow read/write only to users in the project to which the container belongs

This is the default access policy used when no role-based access policy elements are set. A valid authentication token is required to access the container using the API.
If you delete all the `X-Container-Read` and `X-Container-Write` property values of a container, it becomes a `PRIVATE` container that allows access only to users in the project to which the container belongs.

<br>

<details>
<summary>Example of removing all role-based access policy elements</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

!!! tip "Notice"
    When sending a header without a value using curl, a semicolon (;) must be appended to the header name.

If a request is made without a valid authentication token, an error message is responded.

```
$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

You must have a valid authentication token in the request header to get the desired response.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container

[The list of object in the container]
```
</details>
<br>

<a id="role-based-access-allow-read-and-list-for-all-users"></a>
#### Allow read/list query for all users

Setting the container's `X-Container-Read` property to `.r:*, .rlistings` allows all users to read objects and query an object list. No authentication token is required. It is the same policy as the `PUBLIC` entry in the console section.
<br>

<details>
<summary>Example configuration for allowing object read and list query for all users</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]


$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container

[The list of object in the container]
```

If you set only `.r:*`, users can access the object in the container, but they cannot query the object list.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]


$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br>

<a id="role-based-access-allow-or-deny-by-referer"></a>
#### Allow/deny read for requests from a specific HTTP referer

HTTP referer is the address information of a web page that is requested through a hyperlink. It is included in the request header.
If you set a role-based access policy element in the form of `.r:{referrer}` or `.r:-{referrer}` in the `X-Container-Read` property of the container, you can allow or block access requests from specific referers. When setting the HTTP referer with a role-based access policy element, you must enter the domain name without the protocol and sub-path.

The HTTP referer allow/block access policy applies the block policy first, regardless of the input order. Therefore, access requests from an HTTP referer designated as a block target are denied even if the `.r:*` policy element that allows all access is also entered.

!!! danger "Caution"
    The HTTP referer header can be forged or tampered with and is therefore not recommended as an access control measure.

<details>
<summary>Example of allowing read requests from specific HTTP referer</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

Objects can be accessed by specifying the allowed HTTP referer addresses in the API request header.

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]


$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com/some/path' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]
```

If there is no allowed referer address in the API request header or the protocol is not included in the referer address, access is blocked.

```
$ curl -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: https://example.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

As shown below, entering a domain name starting with `.` in your HTTP referer settings allows read to referers including all subdomain addresses of the configured domain.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]


$ curl -O -X GET \
  -H 'Referer: https://qux.baz.foo.com/some/path' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]
```

Requests that do not contain subdomains are blocked.

```
$ curl -X GET \
  -H 'Referer: https://foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

To allow access requests from all referers with a specific domain name, use a comma list as follows.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:foo.com, .r:.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]


$ curl -O -X GET \
  -H 'Referer: https://baz.foo.com/some/path' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]
```

<details>
<summary>Example of blocking read requests from a specific HTTP referer</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

If you set the HTTP referer domain name with a minus sign in front of it, requests from the HTTP referer are blocked.

```
$ curl -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>

<details>
<summary>Example of configuration that allows all access requests except from a specific HTTP referer</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Download Object]


$ curl -X GET \
  -H 'Referer: https://bar.foo.com' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br>

<a id="role-based-access-allow-rw-project-or-user"></a>
#### Allow read/write to specific projects or specific users

If you set a role-based access policy element in the form of `{tenant-id}:{api-user-id}` in the `X-Container-Read` and `X-Container-Write` properties of the container, you can grant read/write permission to a specific project or specific user, respectively. Entering the wildcard character `*` instead of the tenant ID or API User ID grants access to all projects or all users. A valid authentication token is required when making an access request.

!!! tip "Notice"
    The read permission granted by the access policy that requires an authentication token includes the object list query permission.

<details>
<summary>Example of granting read/write permission to a specific user in a specific project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{api-user-id}' \
  -H 'X-Container-Write: {tenant-id}:{api-user-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

When requesting access to an object, a valid authentication token issued by an authorized Tenant ID and API User ID is required.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container

[The list of object in the container]


$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container/object

[Object download]
```

<details>
<summary>Example of granting read/write permission to all users in a specific project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:*' \
  -H 'X-Container-Write: {tenant-id}:*' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

When requesting access to an object, a valid authentication token issued by an authorized Tenant ID and API User ID is required.
<br><br>
</details>

<details>
<summary>Example of granting read/write permission to a specific user regardless of project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{api-user-id}' \
  -H 'X-Container-Write: *:{api-user-id}' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

When requesting access to an object, a valid authentication token issued by an authorized API User ID is required, regardless of the project.
<br><br>
</details>

<details>
<summary>Example of granting read/write permission to all NHN Cloud users</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

A valid authentication token is required when making an access request to an object.
</details>
<br>

<a id="role-based-access-delete-access-policies"></a>
#### Delete Access Policies

By entering an empty header, you can delete all set role-based access policy elements. A container with no role-based access policy element becomes a **PRIVATE** container, accessible only by authorized users. See `Allow read/write only to users in the project to which the container belongs`.

<a id="role-based-access-references"></a>
### References { #role-based-access-references }
Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)

<a id="ip-based-access-policies"></a>
## IP-based Access Policies { #ip-based-access-policies }

You can use the console or API to restrict read/write access to the container from specific IPs by specifying a whitelist and a blacklist. You can configure both a whitelist and a blacklist simultaneously, but in this case, only the whitelist is applied and the blacklist is ignored. IP-based access policies support only IPv4. You can specify separate exceptions for service gateway requests.

!!! danger "Caution"
    IP-based access policies are intended to control access through public IPs. If you register only private IPs in the whitelist, you could end up with an inaccessible container.
    If the container becomes inaccessible due to an incorrect configuration, you will no longer be able to change the policy. If this issue occurs, please contact the Customer Center.

<a id="ip-based-access-console"></a>
### Console { #ip-based-access-console }

In the container management window, select an IP-based container access policy from the Set the Access Policy dialog box.

!!! danger "Caution"
    If you don't have read permission, you cannot manage containers in the console.

<a id="ip-based-access-whitelist"></a>
#### Whitelist

All requests are denied except those from allowed IPs or network ranges. You can specify read and write permissions for the requests to allow.

<a id="ip-based-access-blacklist"></a>
#### Blacklist

Denies requests from the specified IP addresses or network ranges. All other requests are allowed. If configured together with a whitelist, the blacklist is ignored. You can specify the read and write permissions to deny for requests.

<a id="ip-based-access-service-gateway-ip"></a>
#### Service Gateway IP

Controls requests through the service gateway. If not configured, requests may be rejected depending on the whitelist and blacklist settings.

<a id="ip-based-access-api"></a>
### API { #ip-based-access-api }

You can use the API to enable IP-based access policies by entering IP-based access policy elements in the `X-Container-Ip-Acl-Allowed-List` and `X-Container-Ip-Acl-Denied-List` properties of a container. `X-Container-Ip-Acl-Allowed-List` is the allowlist, and `X-Container-Ip-Acl-Denied-List` is the denylist.

To modify the properties of a container with an IP-based access policy configured, a valid authentication token issued by an authorized tenant ID and API User ID is required, and the request must be made from an allowed IP address.

!!! tip "Note"
    `X-Container-Ip-Acl-Allowed-List` (whitelist) and `X-Container-Ip-Acl-Denied-List` (blacklist) can each have a maximum of 100 policy elements. This limit also applies when setting the policy via [Container Policy](container-policy-guide/#ip-acl).

<br>

The IP-based access policy elements consist of access permissions and IP addresses or network ranges, and you can enter multiple values by separating them with commas (`,`). The access permissions are as follows.

| Access Permission | Description |
| --- | --- |
| `r` | Read permission. Applies to GET and HEAD requests. |
| `w` | Write permission. Applies to PUT, POST, DELETE, and COPY requests. |
| `a` | Indicates both read and write permissions. This includes GET, HEAD, PUT, POST, DELETE, and COPY requests. |

To control service gateway requests, set the permissions in the X-Container-Ip-Acl-Service-Gateway-Control property of the container. The permissions that can be set are as follows.

| Permission | Description |
| --- | --- |
| `read` | Allows read permission. This includes GET and HEAD requests. |
| `write` | Allows write requests. This includes PUT, POST, DELETE, and COPY requests. |
| `rw` | Allows all read and write requests. This includes GET, HEAD, PUT, POST, DELETE, COPY requests. |
| `deny` | Does not allow any read or write requests. |


<details>
<summary>Example of whitelist setup</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

192.168.0.1 can make read requests only, 192.168.0.2 can make write requests only, and all IPs in the 172.16.0.0/24 range can make both read and write requests. All other IPs have their requests denied.

<br><br>
</details>

<details>
<summary>Example of Blacklist Settings</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Denied-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

Read requests from 192.168.0.1 are denied, write requests from 192.168.0.2 are denied, and all IP addresses in the 172.16.0.0/24 range cannot make either read or write requests. All other IP addresses are allowed to make requests.

<br><br>
</details>

<details>
<summary>Service Gateway Request Control Examples</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Service-Gateway-Control: rw' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

All Service Gateway requests are allowed regardless of the configured IP-based access policy.

<br><br>
</details>

<a id="ip-based-access-delete-access-policies"></a>
#### Delete access policies

By entering an empty header, you can delete all set IP-based access policy elements. A container with no policy elements is not subject to IP-based access restrictions.

<details>
<summary>Example of removing IP-based access policy elements</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List;' \
  -H 'X-Container-Ip-Acl-Denied-List;' \
  -H 'X-Container-Ip-Acl-Service-Gateway-Control;' \
  $[ object_storage_url ]$/v1/AUTH_*****/container
```

<br><br>
</details>