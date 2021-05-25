## Storage > Object Storage > Guide to setting up access policy

By using the Console or the API, you can give other people the access rights for reading/writing the containers.

## Console
In the Console, [Container Generation](/Storage/Object%20Storage/ko/console-guide/#_2) or [Container Setup](http://localhost:8080/Storage/Object%20Storage/ko/console-guide/#_5) window, you can select the Container access policy. The selectable policies are limited to 2 types: `PRIVATE`and `PUBLIC`.

### PRIVATE
`PRIVATE`is a basic access policy that gives access rights only to users who are part of the project wherein the container belong to. They can use the Console and access the container with the API by receiving authentication token. It is the same policy as that of the API section’s[ permission of reading/writing only to users who are part of the project wherein the container belong to ](/Storage/Object%20Storage/ko/acl-guide/#_2).
<br/>

### PUBLIC
`PUBLIC`is a policy that allows Read or View Object Lists by all people. If the container is set up as PUBLIC, you can receive an URL in the Console. Through this URL, anybody can access the container. It is the same policy as that of the API Section's [permission of reading to all users ](/Storage/Object%20Storage/ko/acl-guide/#_2).
<br/>

## API
If you enter the ACL policy elements in the Container's `X-Container-Read` and `X-Container-Write` using the API, you can set up the accessibility policies for a variety of circumstances.
<br/>

### ACL 정책 요소

The ACL policy elements that you can set up are as follows. All policy elements can be combined using the comma (`,`).

| Policy Elements | Description |
| --- | --- |
| `.r:*` | Anybody can access the Object without an authentication token. |
| `.rlistings` | To all users with reading rights, the View Container (GET or HEAD Request)is allowed. <br/>If this policy element does not exist, the Object List cannot be checked.<br/> This policy element cannot be set up alone. |
| `.r:<referrer>` | By referring to the request header, access is permitted to the set-up (HTTP Referer).<br/>Authentication token is not necessary. |
| `.r:-<referrer>` | By referring to the request header, the set-up HTTP Referer’s access is restricted.<br/>In front of the referer, the minus symbol (-)is added for setup. |
| `<tenant-id>:<user-uuid>` | An authentication token issued to certain users of certain Projects to access Objects.<br/>You can assign both reading and writing rights. |
| `<tenant-id>:*` | An authentication token issued to all users of certain Projects to access Objects.<br/>You can assign both reading and writing rights. |
| `*:<user-uuid>` | Regardless of a project, certain user can access the Object with an issued authentication tokens.<br/>You can assign both reading and writing rights. |
| `*:*` | Regardless of a project, any user with an issued authentication token can access the Objects.<br/>You can assign both reading and writing rights. |

[Note]
User UUID is not the ID of NHN Cloud users. It is included in the reply statement of the issue request for authentication tokens. (access.user.id)
Please refer to the API Guide's [Authentication Token Issuance ](/Storage/Object%20Storage/ko/api-guide/#_2) provision.
<br/>

### Permitting reading/writing only to users who are part of the project wherein the container belong to
If you delete all the Container’s `X-Container-Read` and `X-Container-Write` attribute values, the container permits access to users who are part of the project wherein the container belong to [PRIVATE](/Storage/Object%20Storage/ko/acl-guide/#private).

<br/>

<details>
<summary>An example of the deletion of all ACL policy elements</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

<blockquote>
<p>[Note]
If you want to send a header without values using curl, you must include in the header name the semi-colon (;).</p>
</blockquote>

If you submit request without a valid authentication token, an error message will appear.

```
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

You can only receive the wanted reply if you have a valid authentication token that is included in the request header.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
[The list of Object in the container]
```
</details>
<br/>

### Allow read/view list to all Users.
If you set up the `X-Container-Read` attribute to `.r:*, .rlistings`, Read and View List are permitted to all Users. Authentication tokens are not necessary. This is the same policy as the Console Section’s [PUBLIC](/Storage/Object%20Storage/ko/acl-guide/#public) provision.
<br/>

<details>
<summary>An example of permitting the Object Read and View List to all Users</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
[The list of Object in the container]
```

If only <code>.r:*</code>is set up, you can access the Container's object, but may not view the Object lists.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>


### Allowing/rejecting reading to certain HTTP Referers
HTTP Referer is the web page's address information requested through Hyperlink. It is included in the Request Header.
If you wish to set up the ACL Policy Elements with the Container’s `X-Container-Read` attributes in `.r:<referrer>` or in `.r:-<referrer>` form, you can permit or reject certain Referers who made accessibility requests. When setting up the HTTP Referers in the ACL policy elements, you must enter the domain name exluding the Protocol and lower-level path.

> [Caution]
> The HTTP Referer can always be changed by the User through Header tampering. Accessibility policies using HTTP Referers are weak in security, and thus not recommended.
<details>
<summary>An example of permitting reading requests to certain HTTP Referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

If the API Request Header contains the permitted HTTP Referer’s address, access to Object is allowed.

```
$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
```

If the API Request Header does not contain the permitted Referer’s address, or if the Referer’s address does not include the Protocol, access is blocked.

```
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
$ curl -X GET \
  -H 'Referer: https://example.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
$ curl -X GET \
  -H 'Referer: cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

When the domain name starting with <code>.</code>is entered as below, reading is permitted to all Referers’ subdomains in specified domains.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
$ curl -O -X GET \
  -H 'Referer: https://guide.docs.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
```

Requests without including subdomains will be blocked.

```
$ curl -X GET \
  -H 'Referer: https://nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

If you wish to permit the access requests of all Referers with certain domain names, you must use the comma list and set up as follows.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:nhn.com, .r:.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
$ curl -O -X GET \
  -H 'Referer: https://container.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
```
</details>

<details>
<summary>An example of blocking reading access to certain HTTP Referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

If a minus symbol is added in front of the HTTP Referer’s domain name, the HTTP Referer's Request is blocked.

```
$ curl -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>

The permit/reject policy concerning HTTP Referers is applied according to the order of input. For example, if Referer policy elements allowing `.r:*` are entered after policy elements blocking Referers, Referer blockage policies are ignored. On the other hand, if policy elements allowing requests to all are entered first and certain Referers are entered as not allowed to access, all access requests excluding the specified exceptions are allowed.
<br/>

<details>
<summary>An example of a wrong policy setup that ignores blockage of HTTP Referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-cloud.nhn.com, .r:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
$ curl -O -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
```
</details>

<details>
<summary>An example of a policy setup that allows all access requests except for that of certain HTTP Referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
$ curl -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br/>

### Allowing reading/writing to certain projects or certain Users
If you wish to set up ACL Policy Elements with the Container’s `X-Container-Read`and `X-Container-Write` attribute’s `<tenant-id>:<user-uuid>` form, you can assign reading/writing rights to certain project or certain users. If you enter wildcard script`*` instead of Tenant ID or User UUID, all Projects or all Users are allowed to access. When making access requests, a valid authentication token is required.

> [Note]
> The Reading Right assigned through the ACL Policy with the authentication token includes the Object List View rights.
<details>
<summary>An example of reading/writing rights assigned to certain Users of a certain Project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{user-uuid}' \
  -H 'X-Container-Write: {tenant-id}:{user-uuid}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

In making access requests to Objects, a valid authentication token received through a Tenant ID and NHN Cloud User’s ID is required.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
[The list of Object in the container]
$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Object Download]
```
</details>

<details>
<summary>An example of allowing reading/writing rights to all users of a certain Project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:*' \
  -H 'X-Container-Write: {tenant-id}:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

In making access requests to Objects, a valid authentication token received through a Tenant ID and an NHN Cloud User's ID issued from a related project is required.
<br/><br/>
</details>

<details>
<summary>An example of allowing reading/writing rights to certain users regardless of Projects</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{user-uuid}' \
  -H 'X-Container-Write: *:{user-uuid}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

In making an access request to an Object, a valid authentication token issued with a permitted NHN Cloud User's ID is required.
<br/><br/>
</details>

<details>
<summary>An example of allowing reading/writing rights to all NHN Cloud Users</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

When making an access request to an Object, a valid authentication token is necessary.
</details>
<br/>

### Deleting Access Policies
If you enter the empty headers, you can delete all set-up ACL Policy Elements. Containers without ACL Policy Elements become a**PRIVATE** Container that only authorized users may access. Check the provision of [authorizing reading/writing rights to users of Containers belonging to certain Projects](/Storage/Object%20Storage/ko/acl-guide/#_2).


## References
Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)
