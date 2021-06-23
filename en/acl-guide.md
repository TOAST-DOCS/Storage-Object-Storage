## Storage > Object Storage > Guide on Setting Access Policy

By using the console or the API, you may authorize other people the access rights for reading/writing the containers.

## Console
In the console, you can select the access policy of the container in the [Container Generation](/Storage/Object%20Storage/en/console-guide/#_2) or [Container Settings](/Storage/Object%20Storage/en/console-guide/#_5) window. The selectable policies are limited to two types: `PRIVATE` and `PUBLIC`.

### PRIVATE
`PRIVATE` is a basic access policy that authorizes access rights only to users of a project that the container is affiliated to. The users may use the console or receive an authentication token to access the container with the API. It is the same policy as that of the provision to authorize reading and writing rights to [users of a project that the container of the API section is affiliated to](/Storage/Object%20Storage/en/acl-guide/#_2).
<br/>

### PUBLIC
`PUBLIC` is a policy that authorizes the disclosure of reading and viewing object lists to all. You will receive a URL in the console if you designate the container as PUBLIC. Through this URL, anyone can access the container. It is the same policy as that of the provision to authorize reading and writing rights to [all users of the API section](/Storage/Object%20Storage/en/acl-guide/#_2).
<br/>

## API
If you enter the ACL policy elements in the container's `X-Container-Read` and `X-Container-Write` attribute using the API, you can set access policies for a variety of circumstances.
<br/>

### ACL Policy Elements

The ACL policy elements that you can set are as follows. All policy elements can be separated and combined using a comma (`,`).

| Policy Elements | Description |
| --- | --- |
| `.r:*` | Anyone can access the object without an authentication token. |
| `.rlistings` | The view container (GET or HEAD request) is permitted to users authorized to read.<br/>Without this policy element, the object list cannot be viewed.<br/>This policy element cannot be set independently. |
| `.r:<referrer>` | Refer to the request header to authorize access to the designated (HTTP Referer).<br/>Refer to the request header to authorize access to the designated (HTTP Referer). <br/>Authentication tokens are not required. |
| `.r:-<referrer>` | Refer to the request header to restrict the access of the designated HTTP Referer.<br/>Place a hyphen (-) in front of the referer to designate. |
| `<tenant-id>:<user-uuid>` | An object can be accessed with an authentication token issued to certain users of specific projects.<br/>You may authorize both reading and writing. |
| `<tenant-id>:*` | An object can be accessed by using the authentication token issued to all users participating in specific projects.<br/>You may authorize both reading and writing. |
| `*:<user-uuid>` | An object can be accessed by using the authentication token issued to a specific user regardless of the project.<br/>You may authorize both reading and writing. |
| `*:*` | Any user issued with an authentication token can access an object regardless of the project.<br/>You may authorize both reading and writing. |

> [Notes]
> A user UUID is not a user ID for NHN Cloud. It is included in the response message for the request to issue an authentication token. (access.user.id)
> Please refer to [Authentication Token Issuance ](/Storage/Object%20Storage/en/api-guide/#_2) in the API guide.
<br/>

### Authorize reading and writing only to users who participate in a project that the container is affiliated to.
This is the basic access policy used when the ACL policy elements are not set. You need a valid authentication token to access the container using API.
If you delete all attribute values of the container’s `X-Container-Read` and `X-Container-Write`, it becomes a [PRIVATE](/Storage/Object%20Storage/en/acl-guide/#private) container that authorizes access only to users who participate in a project that the container is affiliated to.

<br/>

<details>
<summary>Examples of deleting all ACL policy elements</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

<blockquote>
<p>[Notes]
If you want to use curl to send out a header with no values, place a semicolon(;) in the name of the header.</p>
</blockquote>

An error message is sent if a request is submitted without a valid authentication token.

```
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

You can receive the desired response only if you have a valid authentication token in the request header.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
[List of objects in the container]
```
</details>
<br/>

### Authorize all users to read and view the list.
If you set the `X-Container-Read` attribute as `.r:*, .rlistings`, all users are authorized the right to read the object and view the list. Authentication tokens are not required. This is the same policy as that of the provision of [PUBLIC](/Storage/Object%20Storage/en/acl-guide/#public) in the console section.
<br/>

<details>
<summary>Examples of authorizing all users the right to read the object and view the list</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
[List of objects in the container]
```

If only <code>.r:*</code>is designated, you may access the object of the container but may not view the object list.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>


### Authorize/block the permission to read for a specific request made by an HTTP referer
An HTTP Referer is the address information of a web page requested through a hyperlink. It is included in the request header.
If you wish to set the ACL policy elements with the `X-Container-Read` attribute of the container in `.r:<referrer>` or `.r:-<referrer>` form, you can authorize or block the permission of access requested by certain referers. When setting HTTP referers with ACL policy elements, you must enter the domain name excluding the protocol and lower-level path.

> [Caution]
> The user may change the HTTP referer at any time by modifying the header. Accessibility policies using HTTP referers are not recommended due to high security risks.
<details>
<summary>Examples of authorizing reading requests to specific HTTP referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

The object can be accessed if the API request header contains the authorized address of the HTTP referer.

```
$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
```

Access is blocked if the API request header does not contain the authorized address of the referer or if the address of the referer does not include the protocol.

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

When you enter a domain name starting with <code>.</code>as in the following, all subdomains of the referer for the set domains are authorized to read.

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
[Download Object]
$ curl -O -X GET \
  -H 'Referer: https://guide.docs.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
```

Requests without subdomains will be blocked.

```
$ curl -X GET \
  -H 'Referer: https://nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

If you wish to authorize the access requests of all referers with certain domain names, you must use the comma list and set it as follows.

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
[Download Object]
$ curl -O -X GET \
  -H 'Referer: https://container.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
```
</details>

<details>
<summary>Examples of blocking reading access to certain HTTP referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

If a hyphen is added in front of the domain name of the HTTP referer, the request of the HTTP referer is blocked.

```
$ curl -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>

The authorize/block access policy concerning HTTP referers is applied according to the order of input. For example, if `.r:*` policy elements that authorize access to all users are entered after the referer block policy elements, the referer block policy is ignored. On the other hand, if policy elements that authorize access to all users are entered first followed by specific referer block policy elements, all access requests are authorized excluding the access request of set referers.
<br/>

<details>
<summary>Examples of wrong policy settings that disregard the blockage of HTTP referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-cloud.nhn.com, .r:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
$ curl -O -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
```
</details>

<details>
<summary>Examples of policy settings that permit all access requests except for that of certain HTTP referers</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
$ curl -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br/>

### Authorizing reading and writing to certain projects or users
If you set ACL policy elements in the form of `<tenant-id>:<user-uuid>` to the `X-Container-Read` and `X-Container-Write` attributes of the container, you may selectively authorize reading and writing to certain projects or users. All users or projects are authorized to access if you enter a wild card script `*` instead of a tenant ID or user UUID. A valid authentication token is required when requesting access.

> [Notes]
> The authority to view the object list is included in the authority to read for the ACL policy that requires an authentication token.
<details>
<summary>Examples of authorizing reading and writing to specific users of a specific project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{user-uuid}' \
  -H 'X-Container-Write: {tenant-id}:{user-uuid}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

Requesting access to objects requires a valid authentication token received through an authorized tenant ID and an NHN Cloud user ID.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
[List of objects in the container]
$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object
[Download Object]
```
</details>

<details>
<summary>Examples of authorizing reading and writing to all users of a specific project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:*' \
  -H 'X-Container-Write: {tenant-id}:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

Requesting access to objects requires a valid authentication token received through an authorized tenant ID and an NHN Cloud user ID of a relevant project.
<br/><br/>
</details>

<details>
<summary>Examples of authorizing reading and writing to specific users regardless of the project</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{user-uuid}' \
  -H 'X-Container-Write: *:{user-uuid}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

Requesting access to objects requires a valid authentication token received through an authorized NHN Cloud user ID.
<br/><br/>
</details>

<details>
<summary>Examples of authorizing reading and writing to all NHN Cloud users</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

Requesting access to objects requires a valid authentication token.
</details>
<br/>

### Deleting Access Policies
Enter the empty headers to delete all ACL policy elements that have been set. Containers without ACL policy elements become a **PRIVATE** container that only authorized users can access. Refer to provision on [authorizing reading and writing to users of a project that the container is affiliated to](/Storage/Object%20Storage/en/acl-guide/#_2).


## References
Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)
