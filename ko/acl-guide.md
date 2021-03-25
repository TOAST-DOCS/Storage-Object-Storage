## Storage > Object Storage > ACL 가이드

## ACL
Object Storage에서는 컨테이너 단위의 읽기/쓰기 정책을 제공합니다.

## 콘솔
콘솔에서는 Public과 Private 읽기 정책으로 설정이 가능합니다.
Public으로 컨테이너의  설정을 하면 공개 URL을 통해 누구나 컨테이너 내부의 오브젝트에 접근할 수 있습니다.
이는 `X-Container-Read: .r*, .rlisting` 과 같은 설정이며, 자세한 내용은 아래 API 섹션에서 확인이 가능합니다.


## API
API를 이용하여 ACL을 지정할 시 HTTP 리퍼러 이용한 정책과 토큰을 활용한 정책이 있습니다.
HTTP 리퍼러를 이용한 정책은 읽기 정책에서만 사용할 수 있습니다.

컨테이너 읽기 정책은 컨테이너의 `X-Container-Read` 속성에서, 쓰기 정책은 `X-Container-Write` 속성에서 설정할 수 있으며 `,` 로 구분하여 여러 정책을 입력 가능합니다.

### HTTP 리퍼러를 사용한 정책
HTTP 리퍼러를 이용한 정책의 경우 `.r:<referer>` 형식이 사용됩니다.
단 이는 오브젝트 읽기 권한이며, 컨테이너의 목록 보기를 허용하기 위해선 `.rlisting` 정책이 함께 사용되어야 합니다. <br>
HTTP 리퍼러를 이용한 정책은 쓰기 정책에서 사용이 불가능합니다.

referer의 경우 호스트명을 입력해야 하고, `.`으로 시작하는경우 호스트명이 동일하게 끝나면 되며, `-`로 시작하면 허용하지 않겠다는 의미입니다.

| 예시 | 설명 | 
| --- | --- | 
| `.rlistings` | 레퍼러를 통해 오브젝트 읽기권한이 있는 경우 컨테이너 목록 조회를 허용합니다. |  
| `.r:*` | 모든 사용자에게 컨테이너 내의 오브젝트 읽기를 허용합니다. | 
| `.r:<referer>` | 리퍼러의 호스트명이 `$referer`인 경우 오브젝트 읽기를 허용합니다. | 
| `.r:.<suffix>` | 리퍼러의 호스트명이 `.$suffix`로 끝나는 경우 오브젝트 읽기를 허용합니다. | 
| `.r:-<referer>`| 리퍼러의 호스트명이 `$referer`인 경우 읽기를 허용하지 않습니다. | 
| `.r:-.<suffix>` | 리퍼러의 호스트명이 `.$suffix`로 끝나는 경우 오브젝트 읽기를 허용합니다. | 

### 토큰을 사용한 정책
토큰을 이용한 정책은 `<tenant-id>:<user-id>` 형식을 따르며, 전체를 허용한단 의미로 `*`을 허용합니다. 테넌트 아이디, 유저 아이디는 토큰 발급 결과를 통해 조회 가능합니다. HTTP 리퍼러를 사용할때와는 다르게, 읽기 권한이 있는경우 목록 조회 권한도 같이 부여됩니다.

| 예시 | 설명 | 
| --- | --- | 
| `<tenant-id>:<user-id>` | 토큰의 테넌트 아이디와 유저 아이디가 같은 경우 허용합니다. |
| `<tenant-id>:*` | 토큰의 테넌트 아이디가 같은 경우 허용합니다. |
| `*:<user-id>` | 토큰의 유저 아이디가 같은 경우 허용합니다. |
| `*:*` | 유효한 토큰을 사용한경우 모두 허용합니다. |


### 주의할 점 
HTTP 리퍼러를 이용한 ACL은 순서가 적용됩니다.
```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:*, .r:-www.example.com' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object
<정상 응답>
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object -e "http://www.example.com"
<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

위와 같은경우 HTTP 리퍼러가 www.example.com인 경우를 제외한 모든 클라이언트는 오브젝트 조회가 가능합니다.

```
curl -i -X POST \
    -H X-Auth-Token:${TOKEN} \
    -H 'X-Container-Read: .r:-www.example.com, .r:*' \
    https://api-storage.cloud.toast.com/v1/AUTH_****/container
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object
<정상 응답>
curl https://api-storage.cloud.toast.com/v1/AUTH_****/container/object -e "http://www.example.com"
<정상 응답>
```

위와같은경우 `.r:-www.example.com` 정책이 있지만, 이후 `.r:*`이 있으므로 오브젝트 읽기가 가능합니다. 이처럼 정책은 영향을 받는경우에 대해 순서대로 적용됩니다.

## References

Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)
