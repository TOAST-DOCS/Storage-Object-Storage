## Storage > Object Storage > CLI 가이드
OpenStack Swift 명령줄 인터페이스(CLI)로 NHN Cloud 오브젝트 스토리지 서비스를 사용하는 방법을 설명합니다.

<a id="python-swiftclient"></a>
## python-swiftclient

<a id="install"></a>
### 설치

python-swiftclient는 Python 패키지로 제공됩니다. pip를 이용해 설치합니다.

```
pip install python-swiftclient python-keystoneclient
```

> [참고]
> Python 3.6 이상이 필요합니다. Python이 설치되어 있지 않다면 [Python 다운로드 페이지](https://www.python.org/downloads/)를 참고하여 설치합니다.

설치가 완료되면 다음 명령으로 확인할 수 있습니다.

```
$ swift --version
python-swiftclient x.x.x
```

<br/>

<a id="configuration"></a>
### 환경 설정

Swift CLI를 사용하려면 인증에 필요한 환경 변수를 설정해야 합니다. 오브젝트 스토리지 서비스 페이지의 **API Endpoint 설정** 버튼을 클릭해 필요한 정보를 확인할 수 있습니다.

```
export OS_AUTH_URL=https://api-identity-infrastructure.nhncloudservice.com/v2.0
export OS_TENANT_ID=<테넌트 ID>
export OS_USERNAME=<NHN Cloud 회원 ID>
export OS_PASSWORD=<API 비밀번호>
export OS_REGION_NAME=<리전 이름>
```

<br/>

| 환경 변수 | 설명 |
| --- | --- |
| OS_AUTH_URL | Identity API URL<br/>https://api-identity-infrastructure.nhncloudservice.com/v2.0 |
| OS_TENANT_ID | 오브젝트 스토리지 서비스 페이지의 **API Endpoint 설정**에서 확인할 수 있는 테넌트 ID |
| OS_USERNAME | NHN Cloud 회원 ID(이메일 형식) 또는 IAM 멤버 ID |
| OS_PASSWORD | **API Endpoint 설정**에서 설정한 API 비밀번호 |
| OS_REGION_NAME | 리전 이름<br/>KR1 - 한국(판교) 리전<br/>KR2 - 한국(평촌) 리전<br/>KR3 - 한국(광주) 리전<br/>JP1 - 일본(도쿄) 리전 |

<br/>

> [주의]
> 오브젝트 스토리지는 기본 인프라 서비스와는 다른 테넌트 ID를 가지고 있습니다. 오브젝트 스토리지 서비스 페이지의 **API Endpoint 설정** 버튼을 클릭해 확인하세요.

<!-- 개행을 위한 주석 -->

> [참고]
> API 비밀번호는 오브젝트 스토리지 서비스 페이지의 **API Endpoint 설정** 버튼을 클릭해 설정할 수 있습니다.

<br/>

설정 파일을 만들면 매번 환경 변수를 설정하지 않아도 편리하게 사용할 수 있습니다.

```
$ cat ~/swiftrc
export OS_AUTH_URL=https://api-identity-infrastructure.nhncloudservice.com/v2.0
export OS_TENANT_ID=<테넌트 ID>
export OS_USERNAME=<NHN Cloud 회원 ID>
export OS_PASSWORD=<API 비밀번호>
export OS_REGION_NAME=<리전 이름>
```

```
$ source ~/swiftrc
```

<br/>

환경 설정이 올바른지 확인하려면 `swift stat` 명령을 실행합니다. 오류 없이 계정 정보가 출력되면 정상적으로 설정된 것입니다.

```
$ swift stat
        Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
     Containers: 0
        Objects: 0
          Bytes: 0
             ...
...
```

<br/>

<a id="basic-usage"></a>
## 기본 사용 방법
기본 사용 방법은 다음과 같습니다.

```
swift <subcommand> [<options>] [<container> [<object>]]
```

<br/>

<a id="subcommands"></a>
### 서브커맨드
주요 서브커맨드는 다음과 같습니다.

| 서브커맨드 | 설명 |
| --- | --- |
| stat | 계정, 컨테이너, 오브젝트의 정보를 조회합니다. |
| list | 컨테이너 또는 오브젝트 목록을 조회합니다. |
| post | 컨테이너를 생성하거나 메타데이터를 설정합니다. |
| upload | 오브젝트를 업로드합니다. |
| download | 오브젝트를 다운로드합니다. |
| copy | 오브젝트를 복사합니다. |
| delete | 컨테이너 또는 오브젝트를 삭제합니다. |
| tempurl | 서명된 URL을 생성합니다. |
| auth | 인증 관련 환경 변수를 출력합니다. |

<br/>

<a id="options"></a>
### 공통 옵션

모든 서브커맨드에 공통으로 사용할 수 있는 옵션입니다.

| 옵션 | 설명 |
| --- | --- |
| --help | 각 서브커맨드의 사용법을 확인합니다. |
| --debug | 실제 API 호출 내역을 확인합니다. |
| --quiet | 상태 출력을 억제합니다. |
| --retries &lt;num&gt; | 요청 실패 시 재시도 횟수를 지정합니다. |

<br/>

<a id="auth-info"></a>
### 동작 방식과 인증 정보 활용

Swift CLI는 기본적으로 명령을 실행할 때마다 Identity API를 호출해 토큰과 서비스 카탈로그를 얻은 다음, 서브커맨드에 해당하는 Swift API를 호출합니다.

<br/>

`auth` 서브커맨드를 사용하면 스토리지 URL과 인증 토큰을 미리 확인할 수 있습니다.

```
$ swift auth
export OS_STORAGE_URL=https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067
export OS_AUTH_TOKEN=gAAAAABi...
```

<br/>

출력된 환경 변수를 셸에 적용하면, 이후 명령 실행 시 인증 토큰 발급과 카탈로그 질의를 생략하므로 더 빠르게 동작합니다.

```
$ eval $(swift auth)
```

> [참고]
> 인증 토큰에는 만료 시간이 있습니다. 토큰이 만료되어 요청이 실패하면 `eval $(swift auth)`를 다시 실행해 갱신해야 합니다.

<br/>

<a id="stat"></a>
## 정보 조회
스토리지 계정, 컨테이너, 오브젝트의 정보를 조회합니다.

```
swift stat [<options>] [<container> [<object>]]
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --lh | 용량을 사람이 읽기 쉬운 단위(KB, MB 등)로 표시합니다. |

<br/>

<a id="stat-account"></a>
### 스토리지 계정 정보 조회
스토리지 계정의 정보를 조회합니다.

```
$ swift stat
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
            Containers: 3
               Objects: 4
                 Bytes: 807711
          Content-Type: application/json; charset=utf-8
           X-Timestamp: 1772338036.37283
                    ...
```

<br/>

<a id="stat-container"></a>
### 컨테이너 정보 조회
컨테이너의 정보를 조회합니다.

```
$ swift stat media
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
               Objects: 3
                 Bytes: 95983
              Read ACL:
             Write ACL:
          Content-Type: application/json; charset=utf-8
           X-Timestamp: 1772338036.37283
         Last-Modified: Sun, 01 Mar 2026 04:07:16 GMT
      X-Storage-Policy: standard
                    ...
```

<br/>

<a id="stat-object"></a>
### 오브젝트 정보 조회
오브젝트의 정보를 조회합니다.

```
$ swift stat media 797619b171a455e9eec8a87f94ee77f4.jpg
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
                Object: 797619b171a455e9eec8a87f94ee77f4.jpg
          Content Type: image/jpeg
        Content Length: 31661
         Last Modified: Sun, 01 Mar 2026 04:07:16 GMT
                  ETag: bd97323f11e8f2fc1655cbb8a1b9382e
           X-Timestamp: 1772338036.37283
                    ...
```

<br/>

<a id="list"></a>
## 목록 조회
컨테이너 또는 오브젝트 목록을 조회합니다.

```
swift list [container] [options]
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --long | 각 자원의 세부 정보를 포함한 목록을 조회합니다. |
| --lh | `--long`과 유사하지만 용량을 사람이 읽기 쉬운 단위로 표시합니다. |
| --totals | `--long` 또는 `--lh`와 함께 사용하면 합계만 출력합니다. |
| --json | JSON 형식으로 목록을 출력합니다. |
| --prefix &lt;prefix&gt; | 지정한 접두사로 시작하는 항목만 조회합니다. |
| --delimiter &lt;delimiter&gt; | 오브젝트 목록 조회에서만 사용할 수 있습니다.<br/>구분자를 기준으로 오브젝트 이름을 묶어 표시합니다.<br/>예를 들어 `--delimiter /`를 사용하면 `pics/9b171a455e9e.png`와 같은 이름의 오브젝트들은 목록에서 제외되고 폴더처럼 `pics/`로 표시됩니다. |

<br/>

<a id="list-containers"></a>
### 컨테이너 목록 조회
컨테이너 목록을 조회합니다.

```
$ swift list
media
```

<br/>

<a id="list-objects"></a>
### 오브젝트 목록 조회
오브젝트 목록을 조회합니다.

```
$ swift list media
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="create-container"></a>
## 컨테이너 생성
신규 컨테이너를 생성합니다.

```
swift post <container>
```

```
$ swift post docs
$ swift list
docs
media
```

<br/>

<a id="upload"></a>
## 오브젝트 업로드
오브젝트를 업로드 또는 업데이트(overwrite)합니다. 여러 오브젝트를 한꺼번에 업로드할 수 있습니다.

```
swift upload <container> <file_or_directory> [<file_or_directory>] [...]
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --object-name &lt;name&gt; | 업로드할 오브젝트의 이름을 지정합니다.<br/>생략하면 파일 이름을 사용합니다. |
| --changed | 컨테이너에 동일한 이름의 오브젝트가 있으면 수정 시간과 크기를 비교해 변경되었을 때만 업로드합니다. |
| --skip-identical | 컨테이너에 동일한 오브젝트가 있으면 업로드를 건너뜁니다. |
| --segment-size &lt;size&gt; | 지정한 크기로 파일을 분할해 SLO 방식으로 멀티파트 업로드합니다. |
| --segment-container &lt;container&gt; | 세그먼트 오브젝트를 저장할 컨테이너를 지정합니다. |
| --leave-segments | 오브젝트를 덮어쓸 때 기존 세그먼트를 삭제하지 않고 유지합니다. |
| --object-threads &lt;threads&gt; | 오브젝트 업로드에 사용할 스레드 수를 지정합니다. 기본값은 10입니다. |
| --segment-threads &lt;threads&gt; | 세그먼트 업로드에 사용할 스레드 수를 지정합니다. 기본값은 10입니다. |
| --meta &lt;name:value&gt; | 메타데이터를 설정합니다. 여러 번 사용할 수 있습니다. |
| --ignore-checksum | 업로드 시 체크섬 검증을 비활성화합니다. |

<br/>

```
$ swift upload media 4287b656254e69e948c3cab237f3fb03.jpg
4287b656254e69e948c3cab237f3fb03.jpg

$ swift list media
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
4287b656254e69e948c3cab237f3fb03.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="multipart-upload"></a>
### 멀티파트 업로드
`--segment-size` 옵션으로 세그먼트 사이즈를 지정하면 파일을 분할해 멀티파트 업로드합니다.

```
swift upload --segment-size <size> [--segment-container <segment-container>] <container> <file>
```

<br/>

`--segment-container`를 지정하지 않으면 `{container}_segments`라는 이름의 컨테이너를 만들고 세그먼트 오브젝트를 업로드합니다.

```
$ swift upload --segment-size 10m media test.mp4
test.mp4 segment 1
test.mp4 segment 0
test.mp4

$ swift list
media
media_segments

$ swift list media
...
test.mp4

$ swift list media_segments
test.mp4/slo/1635487628.192050/20971520/10485760/00000000
test.mp4/slo/1635487628.192050/20971520/10485760/00000001
```

<br/>

<a id="download"></a>
## 오브젝트 다운로드
오브젝트를 현재 경로에 다운로드합니다. 여러 개의 오브젝트를 다운로드할 수 있습니다.

```
swift download <container> [<obj>] [...]
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --output &lt;out_file&gt; | 오브젝트를 저장할 파일 이름을 지정합니다.<br/>`-`를 지정하면 오브젝트 내용을 표준 출력으로 내보냅니다. |
| --output-dir &lt;out_directory&gt; | 다운로드할 경로를 지정합니다. |
| --prefix &lt;prefix&gt; | 지정한 접두사로 시작하는 오브젝트만 다운로드합니다. |
| --remove-prefix | `--prefix`와 함께 사용하면 접두사를 제거한 이름으로 다운로드합니다. |
| --skip-identical | 로컬 경로에 동일한 파일이 있으면 다운로드를 건너뜁니다. |
| --object-threads &lt;threads&gt; | 오브젝트 다운로드에 사용할 스레드 수를 지정합니다. 기본값은 10입니다. |
| --ignore-checksum | 다운로드 시 체크섬 검증을 비활성화합니다. |
| --no-download | 다운로드를 실행하지만 파일을 디스크에 쓰지 않습니다. |

<br/>

```
$ swift download media 153206025029118e96e38b95b78281c8.jpg
153206025029118e96e38b95b78281c8.jpg [auth 0.117s, headers 0.288s, total 0.393s, 2.344 MB/s]
```

<br/>

<a id="download-all"></a>
### 컨테이너 단위 다운로드
다운로드할 오브젝트를 생략하면 지정한 컨테이너의 모든 오브젝트를 다운로드합니다.

```
$ swift download media
300e5522b971c2159e65f95e3c89d981.jpg [auth 0.111s, headers 0.249s, total 0.264s, 0.376 MB/s]
797619b171a455e9eec8a87f94ee77f4.jpg [auth 0.303s, headers 0.451s, total 0.460s, 0.202 MB/s]
...
```

<br/>

<a id="copy"></a>
## 오브젝트 복사
오브젝트를 지정한 경로로 복사합니다. 복사하면서 메타데이터를 추가하거나 변경할 수 있습니다.

```
swift copy [--destination </container/object>] [--fresh-metadata] [--meta <name:value>] <container> <object>
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --destination &lt;/container/object&gt; | 복사할 대상 컨테이너와 오브젝트 이름을 지정합니다.<br/> 오브젝트 이름을 생략하면 원본과 같은 이름으로 복사합니다. |
| --meta &lt;name:value&gt; | 메타데이터를 설정합니다. 여러 번 사용할 수 있습니다. |
| --fresh-metadata | 원본의 메타데이터를 복사하지 않습니다.<br/>`--meta`와 함께 사용하면 새로 지정한 메타데이터만 설정합니다. |

<br/>

<a id="copy-same-container"></a>
### 같은 컨테이너 내에서 복사
대상 오브젝트 이름을 원본과 다르게 지정해야 합니다.

```
$ swift copy --destination "/media/copied_object.jpg" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container media
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /media/copied_object.jpg
```

<br/>

<a id="copy-other-container"></a>
### 다른 컨테이너로 복사
대상 오브젝트 이름을 생략하면 원본과 같은 이름으로 복사합니다.

```
$ swift copy --destination "/pics" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container pics
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /pics/797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="copy-with-metadata"></a>
### 메타데이터 추가 복사
`--meta` 옵션으로 복사하면서 메타데이터를 추가할 수 있습니다.

```
$ swift copy --destination "/media/copied_object.jpg" --meta "Color:Blue" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container media
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /media/copied_object.jpg
```

`--fresh-metadata` 옵션을 함께 사용하면 기존 메타데이터를 제거하고 새로 지정한 메타데이터만 설정합니다.

<br/>

<a id="post"></a>
## 설정 변경
컨테이너 또는 오브젝트의 설정을 변경합니다.

<br/>

<a id="post-container"></a>
### 컨테이너 설정 변경
컨테이너의 설정을 변경합니다. 컨테이너 설정은 [컨테이너 정보 조회 명령](cli-guide/#stat-container)으로 확인할 수 있습니다.

```
swift post [<options>] <container>
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --meta &lt;key:value&gt; | 컨테이너 메타데이터를 설정합니다. 여러 번 사용할 수 있습니다. |
| --read-acl &lt;acl&gt; | 컨테이너의 읽기 ACL을 설정합니다. |
| --write-acl &lt;acl&gt; | 컨테이너의 쓰기 ACL을 설정합니다. |
| --header &lt;key:value&gt; | 커스텀 HTTP 헤더를 추가합니다. Swift CLI가 옵션으로 제공하지 않는 설정도 이 옵션으로 직접 지정할 수 있습니다. |

<br/>

> [참고]
> ACL 설정 값은 [접근 정책 설정 가이드](acl-guide/)를 참고하세요.
>
> `--header` 옵션으로 설정할 수 있는 헤더 목록은 API 가이드의 [컨테이너 설정 변경](api-guide/#change-container-settings) 섹션을 참고하세요.

<br/>

```
$ swift post --meta "Type:photo" media
$ swift post --header "X-Container-Object-Lifecycle: 90" media
$ swift post --read-acl ".r:*" media
```

<br/>

<a id="post-object"></a>
### 오브젝트 메타데이터 변경
오브젝트의 메타데이터를 변경합니다. 오브젝트의 메타데이터는 [오브젝트 정보 조회 명령](cli-guide/#stat-object)으로 확인할 수 있습니다.

```
swift post [<options>] <container> <object>
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --meta &lt;key:value&gt; | 오브젝트 메타데이터를 설정합니다. 여러 번 사용할 수 있습니다. |
| --header &lt;key:value&gt; | 커스텀 HTTP 헤더를 추가합니다. Content-Type 등 오브젝트 속성을 변경할 수 있습니다. |

<br/>

```
$ swift post --meta "Color:Blue" media 797619b171a455e9eec8a87f94ee77f4.jpg
$ swift post --header "Content-Type:image/png" media 797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="delete"></a>
## 삭제
컨테이너 또는 오브젝트를 삭제합니다.

<br/>

<a id="delete-container"></a>
### 컨테이너 삭제
지정한 컨테이너의 내부 오브젝트를 모두 삭제한 뒤 컨테이너를 삭제합니다.

```
swift delete <container>
```

<br/>

| 옵션 | 설명 |
| --- | --- |
| --prefix &lt;prefix&gt; | 지정한 접두사로 시작하는 오브젝트만 삭제합니다. 컨테이너는 삭제하지 않습니다. |
| --object-threads &lt;threads&gt; | 오브젝트 삭제에 사용할 스레드 수를 지정합니다. 기본값은 10입니다. |
| --container-threads &lt;threads&gt; | 컨테이너 삭제에 사용할 스레드 수를 지정합니다. 기본값은 10입니다. |

<br/>

```
$ swift delete media
797619b171a455e9eec8a87f94ee77f4.jpg
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
```

<br/>

<a id="delete-object"></a>
### 오브젝트 삭제
지정한 오브젝트를 삭제합니다.

```
swift delete <container> <object> [<object>] [...]
```

| 옵션 | 설명 |
| --- | --- |
| --leave-segments | 매니페스트 오브젝트의 세그먼트를 삭제하지 않고 유지합니다. |
| --object-threads &lt;threads&gt; | 오브젝트 삭제에 사용할 스레드 수를 지정합니다. 기본값은 10입니다. |

<br/>

```
$ swift delete media 797619b171a455e9eec8a87f94ee77f4.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="tempurl"></a>
## 서명된 URL 생성
토큰 없이 컨테이너 또는 오브젝트에 접근할 수 있는 서명된 URL을 생성합니다.

```
swift tempurl <method> <time> <path> <key>
```

<br/>

| 인자 | 설명 |
| --- | --- |
| method | 허용할 HTTP 메서드입니다. 일반적으로 `GET` 또는 `PUT`을 사용합니다. |
| time | 서명된 URL의 만료 시간입니다.<br/>ISO8601 형식(`2026-03-01`, `2026-03-01T23:59:59`, `2026-03-01T23:59:59Z`)으로 입력합니다.<br/>초 단위로도 입력할 수 있으며, 현재로부터 해당 시간이 지난 후 만료됩니다. |
| path | `/v1/AUTH_*****/<container>` 또는 `/v1/AUTH_*****/<container>/<object>` 형태로 입력합니다. |
| key | 컨테이너에 미리 등록한 Temp-URL-Key입니다. |

<br/>

| 옵션 | 설명 |
| --- | --- |
| --prefix-based | 접두사 기반의 서명된 URL을 생성합니다. |
| --ip-range &lt;ip_range&gt; | 서명된 URL을 통한 접근을 특정 IP 또는 IP 대역으로 제한합니다. |
| --digest &lt;algorithm&gt; | 해시 알고리즘을 지정합니다.<br/>`sha256`(기본값) 또는 `sha512`를 사용할 수 있습니다. |
| --absolute | `<time>`을 초 단위로 입력했을 때, 현재로부터의 상대 시간이 아닌 Unix 타임스탬프로 해석합니다. ISO8601 형식에는 영향을 주지 않습니다. |
| --iso8601 | 생성된 URL에 Unix 타임스탬프 대신 ISO8601 형식의 만료 시간을 포함합니다. |

<br/>

```
$ swift tempurl GET 2026-03-05T23:59:59 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg 398dded8-b1be
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e479&temp_url_expires=1772755199

$ curl https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg\?temp_url_sig\=8244bff5037316dbe8aebcda9cd679c1b331e479\&temp_url_expires\=1772755199 --output 797619b171a455e9eec8a87f94ee77f4.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 31661  100 31661    0     0  1043k      0 --:--:-- --:--:-- --:--:-- 1058k
```

<br/>

<a id="tempurl-key"></a>
### Temp-URL-Key 등록
서명된 URL을 생성하려면 컨테이너에 Temp-URL-Key를 미리 등록해야 합니다. 설정된 키는 [컨테이너 정보 조회 명령](cli-guide/#stat-container)으로 확인할 수 있습니다.

```
swift post --meta "Temp-URL-Key: <key>" <container>
```

```
$ swift post --meta "Temp-URL-Key: 398dded8-b1be" media
$ swift stat media
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
               Objects: 3
                    ...
     Meta Temp-Url-Key: 398dded8-b1be
                    ...
```

<a id="reference"></a>
## References
Object Storage service (swift) command-line client - [https://docs.openstack.org/ocata/cli-reference/swift.html](https://docs.openstack.org/ocata/cli-reference/swift.html)


