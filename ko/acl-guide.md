## Storage > Object Storage > ACL 가이드

## ACL
Object Storage에서는 컨테이너 단위의 읽기/쓰기 정책을 제공합니다.

## 콘솔
콘솔에서 `컨테이너 설정` 화면에서 접근 정책을 Public과 Private으로 설정할 수 있습니다. 컨테이너를 Public으로 설정하면 공개 URL을 통해 누구나 컨테이너 내부의 오브젝트에 접근할 수 있습니다. 이는 `X-Container-Read: .r*, .rlisting` 과 같은 설정이며, 자세한 내용은 아래 API 섹션에서 확인할 수 있습니다.


## API
API를 이용하여 ACL을 지정할 시 일반적인 읽기 정책과 토큰을 활용한 정책이 있습니다. 일반적인 읽기 정책은 전체 대상과 HTTP 리퍼러 헤더를 이용하여 설정 할 수 있습니다. HTTP 리퍼러를 이용한 정책은 헤더 변조의 위험이 있어 권장하지 않습니다. <br>

컨테이너 읽기 정책은 컨테이너의 `X-Container-Read` 속성에서, 쓰기 정책은 `X-Container-Write` 속성에서 설정할 수 있으며 `,`로 구분하여 여러 정책을 입력할 수 있습니다. 빈 값이라면 어떠한 정책도 사용하지 않습니다.

### 일반적인 읽기 정책

일반적인 읽기 정책은 오브젝트에 접근하는 전체 사용자 또는 HTTP 리퍼러 헤더를 이용한 설정이 가능합니다. 이 권한은 읽기 정책에서만 사용할 수 있으며, 쓰기 정책으로는 사용이 불가능합니다. <br>

`.r:*`은 전체 사용자를 의미하며, 주소를 아는 모든 사용자가 오브젝트에 대해 접근할 수 있습니다. <br>

`.r:<referer>`는 HTTP 리퍼러에 대한 접근 정책입니다. referer의 경우 스킴과 경로를 제외한 `cloud.toast.com`과 같은 호스트명 또는 `.com`, `.toast.com`과 같이 `.` 이후의 값으로 규칙을 넣을 수 있습니다. <br>

`.r:-<referer>`는 입력된 리퍼러에 대해 접근을 허용하지 않습니다. <br>

`.rlisting` 정책은 컨테이너 목록 조회에 대한 정책입니다. 위에서 설명한 `.r:*`과 같은 정책은 모두 오브젝트 읽기에 대한 권한이며, `.rlisting`은 오브젝트 읽기 권한이 있는 모든 사용자에게 컨테이너 목록 조회 권한을 부여합니다.

| 정책 | 설명 | 
| --- | --- | 
| `.rlistings` | 오브젝트 읽기 권한이 있는 경우 컨테이너 목록을 조회할 수 있습니다.  |  
| `.r:*` | 모든 사용자가 컨테이너 내의 오브젝트를 읽을 수 있습니다. | 
| `.r:<referer>` | 입력된 리퍼러 값인 경우 오브젝트를 읽을 수 있습니다. | 
| `.r:-<referer>`| 입력된 리퍼러 값인 경우 오브젝트를 읽을 수 없습니다. | 


#### 예시
<details>
<summary>전체 사용자에 대해 오브젝트 읽기 허용</summary>

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:*' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```

위와같이 설정시 오브젝트 접근은 성공하지만, 컨테이너 목록을 읽을 수 없습니다.
```
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object
<정상 응답>

curl https://api-storage.cloud.toast.com/v1/AUTH_****/container
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>

<details>
<summary>전체 사용자에 대해 오브젝트 읽기 및 컨테이너 목록 조회 허용</summary>

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:*, .rlistings' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```

```
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object
<정상 응답>

curl https://api-storage.cloud.toast.com/v1/AUTH_****/container
<정상 응답>
```
</details>


<details>
<summary>특정 리퍼러에 대해 읽기 권한 부여</summary>

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:cloud.toast.com, .rlistings' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```

```
curl -H 'Referer: cloud.toast.com' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
<정상 응답>

curl -H 'Referer: cloud.toast.com/some/path' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
<정상 응답>

curl https://api-storage.cloud.toast.com/v1/AUTH_****/container
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>


<details>
<summary>`.`으로 시작하는 리퍼러에 대해 읽기 권한 부여</summary>

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:.com, .rlistings' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```

```
curl -H 'Referer: cloud.toast.com' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
<정상 응답>

curl -H 'Referer: www.example.com' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
<정상 응답>

curl -H 'Referer: www.example.net' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>


<details>
<summary>특정 리퍼러에 대해 읽기 권한 제거</summary>

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .rlistings, .r:*, .r:-.toast.com' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```

```
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container
<정상 응답>

curl -H 'Referer: cloud.toast.com' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>


### 토큰을 사용한 정책
토큰을 이용한 정책은 `<tenant-id>:<user-id>` 형식을 따르며, 전체를 허용한다는 의미로 `*`을 사용할 수 있습니다. 테넌트 아이디, 유저 아이디는 토큰 발급 결과를 통해 조회할 수 있습니다. 일반적인 읽기 정책과는 다르게, 읽기 권한이 있는경우 별다른 설정없이 컨테이너 목록조회도 가능하며, 쓰기 권한을 부여할 수 있습니다.

| 예시 | 설명 | 
| --- | --- | 
| `<tenant-id>:<user-id>` | 토큰의 테넌트 아이디와 유저 아이디가 같은 경우 컨테이너를 읽거나 쓰기가 가능합니다. |
| `<tenant-id>:*` | 토큰의 테넌트 아이디가 같은 경우 컨테이너를 읽거나 쓰기가 가능합니다. |
| `*:<user-id>` | 토큰의 유저 아이디가 같은 컨테이너를  가능합니다. |
| `*:*` | 유효한 토큰을 사용한경우 컨테이너를 읽거나 쓰기가 가능합니다. |

#### 예시

<details>
<summary>특정 테넌트에 대해 읽기/쓰기 권한 추가</summary>

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: {tenantId}:*' \
    -H 'X-Container-Write: {tenantId}:*' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```

</details>



### ACL 순서 
일반적인 읽기 정책은 순서가 적용됩니다.
```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:*, .r:-www.example.com' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```
위와 같은 설정은 HTTP 리퍼러가 www.example.com인 경우를 제외한 모든 클라이언트는 오브젝트 조회할 수 있습니다.
```
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object
<정상 응답>

curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object -e "http://www.example.com"
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:-www.example.com, .r:*' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
```
위와같은 설정은 `.r:-www.example.com` 정책이 있지만, 이후 `.r:*`이 있으므로 오브젝트 읽기가 가능합니다. 이처럼 정책은 영향을 받는경우에 대해 순서대로 적용됩니다.

```
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object
<정상 응답>
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object -e "http://www.example.com"
<정상 응답>
```

## References

Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)
