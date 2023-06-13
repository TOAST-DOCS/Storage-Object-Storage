## Storage > Object Storage > ACL Configuration Guide
## Role-based Access Policies

You can use the console or API to grant read/write access to the container to other users.

### Console
In the console, you can select a container access policy from the [Create Container](console-guide/#create-container) dialog box or the container access policy settings dialog box within [Manage Container](console-guide/#manage-container) window. There are two policies that can be selected: `PRIVATE` and `PUBLIC`.

#### PRIVATE
`PRIVATE` is the default access policy that grants access only to users of the project to which the container belongs. Users can access the container through the console or through the API by getting an authentication token. It’s the same policy as the `Allow read/write only to users in the project to which the container belongs` in the API section.
<br/>

#### PUBLIC
`PUBLIC` is a policy that allows anyone to read and query the object list. If you set the container to PUBLIC, you can get the URL from the console. Anyone can access the container using this URL. It's the same policy as the `Allow read for all users` in the API section.
<br/>

### API
You can use the API to set access policies for different situations by entering ACL policy elements in the `X-Container-Read` and `X-Container-Write` properties of a container.
<br/>

#### ACL Policy Elements

The ACL policy elements that can be set are as follows. All policy elements can be combined by separating them with commas (`,`).

| Policy Element | Description |
| --- | --- |
| `.r:*` | Anyone can access the object without an authentication token. |
| `.rlistings` | Allows a container query (GET or HEAD request) to users with read permission.<br/>Without this policy element, the list of objects cannot be queried.<br/>This policy element cannot be set alone. |
| `.r:<referrer>` | Allows access to the HTTP referer set by referring to the request header.<br/>No authentication token is required. |
| `.r:-<referrer>` | Restricts the access of the HTTP referer set by referring to the request header.<br/>It is set by adding a minus sign (-) in front of the referer. |
| `<tenant-id>:<api-user-id>` | The object can be accessed with an authentication token issued to a specific user belonging to a specific project.<br/>Both read and write permissions can be granted. |
| `<tenant-id>:*` | The object can be accessed with an authentication token issued to all users belonging to a specific project.<br/>Both read and write permissions can be granted. |
| `*:<api-user-id>` | The object can be accessed with an authentication token issued to a specific user, regardless of the project.<br/>Both read and write permissions can be granted. |
| `*:*` | Regardless of the project, any user who can obtain an authentication token can access the object.<br/>Both read and write permissions can be granted. |

> [Note]
>  `<api-user-id>` can be found in the **API User ID** item in the API Endpoint Settings dialog box on the console or in the **access.user.id** field in the response body of the Authentication Token Issuance API.
> To use the Authentication Token Issuance API, see [Authentication Token Issuance](api-guide/#authentication-token-issuance) in the API Guide.

<br/>

### Allow read/write only to users in the project to which the container belongs
This is the default access policy used when no ACL policy elements are set. A valid authentication token is required to access the container using the API.
If you delete all the `X-Container-Read` and `X-Container-Write` property values of a container, it becomes a `PRIVATE` container that allows access only to users in the project to which the container belongs.

<br/>

<details>
<summary>Example of removing all ACL policy elements</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

<blockquote>
<p>[Note]
When sending a header without a value using curl, a semicolon (;) must be appended to the header name.</p>
</blockquote>

If a request is made without a valid authentication token, an error message is responded.

```
$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

You must have a valid authentication token in the request header to get the desired response.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[List of objects in the container]
```
</details>
<br/>

#### Allow read/list query for all users
Setting the container's `X-Container-Read` property to `.r:*, .rlistings` allows all users to read objects and query an object list. No authentication token is required. It is the same policy as the `PUBLIC` entry in the console section.
<br/>

<details>
<summary>Example of setting to allow object read and list query for all users</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]


$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[List of objects in the container]
```

If you set only <code>.r:*</code>, users can access the object in the container, but they cannot query the object list.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]


$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>


#### Allow/deny read for requests from a specific HTTP referer
HTTP referer is the address information of a web page that is requested through a hyperlink. It is included in the request header.
If you set an ACL policy element in the form of `.r:<referrer>` or `.r:-<referrer>` in the `X-Container-Read` property of the container, you can allow or block access requests from specific referers. When setting the HTTP referer with an ACL policy element, you must enter the domain name without the protocol and sub-path.

> [Caution]
The HTTP referer can be changed at any time by the user through header tampering. Access policies using the HTTP referer are not recommended because they are vulnerable to security attacks.

<details>
<summary>Example of allowing read requests from specific HTTP referer</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

Objects can be accessed by specifying the allowed HTTP referer addresses in the API request header.

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]


$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]
```

If there is no authorized referer address in the API request header or the protocol is not included in the referer address, access is blocked.

```
$ curl -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: https://example.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

As shown below, entering a domain name starting with <code>.</code> in your HTTP referer settings allows read to referers including all subdomain addresses of the configured domain.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]


$ curl -O -X GET \
  -H 'Referer: https://qux.baz.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]
```

Requests that do not contain subdomains are blocked.

```
$ curl -X GET \
  -H 'Referer: https://foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

To allow access requests from all referers with a specific domain name, use a comma list as follows.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:foo.com, .r:.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]


$ curl -O -X GET \
  -H 'Referer: https://baz.foo.com/some/path' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]
```
</details>

<details>
<summary>Example of blocking read requests from a specific HTTP referer</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

If you set the HTTP referer domain name with a minus sign in front of it, requests from the set HTTP referer is blocked.

```
$ curl -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>

The policy to allow/block access for the HTTP referer is applied according to the entered order. For example, if you enter an `.r:*` policy element that allows access to all users after the block referer policy element, the block referer policy is ignored. Conversely, if a policy element that allows access to all users is entered first and a block policy element for a specific referer is entered later, all access requests are allowed except for the set referer's access request.
<br/>

<details>
<summary>Example of incorrect policy setting that ignores HTTP referer blocking</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-bar.foo.com, .r:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]


$ curl -O -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]
```
</details>

<details>
<summary>Example of policy setting that allows all access requests except for access requests from a specific HTTP referer</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]


$ curl -X GET -H 'Referer: https://bar.foo.com' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br/>

### Allow read/write to specific projects or specific users
If you set an ACL policy element in the form of `<tenant-id>:<api-user-id>` in the `X-Container-Read` and `X-Container-Write` properties of the container, you can grant read/write permission to a specific project or specific user, respectively. Entering the wildcard character `*` instead of the tenant ID or API user ID grants access to all projects or all users. A valid authentication token is required when making an access request.

> [Note]
> The read permission granted by the ACL policy that requires an authentication token includes the object list query permission.

<details>
<summary>Example of granting read/write permission to a specific user in a specific project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{api-user-id}' \
  -H 'X-Container-Write: {tenant-id}:{api-user-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

When requesting access to an object, a valid authentication token issued by an authorized tenant ID and NHN Cloud user ID is required.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container

[List of objects in the container]


$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container/object

[Object download]
```
</details>

<details>
<summary>Example of granting read/write permission to all users in a specific project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:*' \
  -H 'X-Container-Write: {tenant-id}:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

When requesting access to an object, a valid authentication token issued by an authorized tenant ID and NHN Cloud user ID belonging to the corresponding project is required.
<br/><br/>
</details>

<details>
<summary>Example of granting read/write permission to a specific user regardless of project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{api-user-id}' \
  -H 'X-Container-Write: *:{api-user-id}' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

When requesting access to an object, a valid authentication token issued by an authorized NHN Cloud user ID is required.
<br/><br/>
</details>

<details>
<summary>Example of granting read/write permission to all NHN Cloud users</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

A valid authentication token is required when making an access request to an object.
</details>
<br/>

#### Delete Access Policies
By entering an empty header, you can delete all set ACL policy elements. A container with no ACL policy element becomes a **PRIVATE** container, accessible only by authorized users. See `Allow read/write only to users in the project to which the container belongs`.


### References
Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)

## IP-based Access Policies

You can use the console or API to specify whitelists and blacklists to restrict read/write access to containers from specific IPs. You can't use whitelists and blacklists at the same time. If you enter both a whitelist and a blacklist, only the whitelist is used. IP-based access policies support IPv4 only.


> [Caution]
> Incorrect settings can result in containers without write access. In this case, you cannot change the settings. If this causes any problem, please contact the Customer Center.


### Console

Select IP-based container access policy from the container access policy setting dialog box in the container management windows.

> [Caution]
> If you don't have read permission, you can no longer handle that container from the console.

#### Whitelist
Denies all requests except from allowed IPs or network bands. You can specify read and write permissions to allow requests.

#### Blacklist
Denies requests from the specified IP or network band. All other requests are permitted. When used with an allow policy, the deny policy is ignored. You can specify read and write permissions to deny requests.


### API

You can use the API to enable IP-based ACLs by entering ACL policy elements in the container's `X-Container-Ip-Acl-Allowed-List` and `X-Container-Ip-Acl-Denied-List` properties.`X-Container-Ip-Acl-Allowed-List` indicates whitelist, `X-Container-Ip-Acl-Denied-List` indicates blacklist.
<br>

A policy element consists of access permission and an IP or network band, and you can enter multiple values separated by commas (`, `). The access permissions are shown in the table below.

| Access Permission | Description |
| --- | --- |
| `r` | Includes read permission, GET, and HEAD requests. |
| `w` | Includes Write permission, PUT, POST, DELETE, and COPY requests.|
| `a` | Indicates both read and write permissions. This includes GET, HEAD, PUT, POST, DELETE, and COPY requests. |


<details>
<summary>Whitelist Setting</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

The IP of 192.168.0.1 can only make read requests, the IP of 192.168.0.2 can only make write requests, and all IPs in the 172.16.0.0/24 band can make all requests. All other IPs will have their requests denied.<br><br>

Changes to a container require a valid authentication token issued with an authorized tenant ID and NHN Cloud user ID belonging to the project, and must be requested from an allowed IP.

<br/><br/>
</details>

<details>
<summary>Blacklist Setting</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Denied-List: r192.168.0.1,w192.168.0.2,a172.16.0.0/24' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

Read requests are denied from the IP of 192.168.0.1, write requests are denied from the IP of 192.168.0.2, and all IPs in the 172.16.0.0/24 band are denied all requests. All other IPs are allowed requests.<br><br>

Changes to a container require a valid authentication token issued with an authorized tenant ID and NHN Cloud user ID belonging to the project, and must be requested from an allowed IP.

<br/><br/>
</details>

<details>
<summary>Remove ACL Setting</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Ip-Acl-Allowed-List;' \
  -H 'X-Container-Ip-Acl-Denied-List;' \
  https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_*****/container
```

Changes to a container require a valid authentication token issued with an authorized tenant ID and NHN Cloud user ID belonging to the project, and must be requested from an allowed IP.

<br/><br/>
</details>