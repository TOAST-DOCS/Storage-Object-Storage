## 서드 파티 도구 사용 가이드

이 문서는 서드 파티 도구로 NHN Cloud 오브젝트 스토리지 서비스를 사용하는 방법을 설명합니다.

<a id="cyberduck"></a>
## Cyberduck

Cyberduck은 오픈소스 클라우드 스토리지 브라우저입니다.

<a id="install-cyberduck"></a>
### Cyberduck 설치

[Cyberduck 다운로드 페이지](https://cyberduck.io/download/)에서 사용 중인 운영 체제에 맞는 설치 파일을 다운로드해 설치합니다.

<a id="cyberduck-object-storage-connection-settings"></a>
### 오브젝트 스토리지 연결 설정

오브젝트 스토리지에 연결하기 위해서는 연결 정보를 저장하는 책갈피를 만들어야 합니다. 브라우저 상단의 **새 연결**을 클릭한 뒤 드롭다운 메뉴에서 **OpenStack Swift(Keystone 2.0)**를 선택합니다. 필요한 정보를 입력한 뒤 연결을 클릭하면 새 책갈피가 생성됩니다.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>항목</th>
    <th>설명</th>
  </tr>
  <tr>
    <td>별명</td>
    <td>책갈피의 별칭입니다. 원하는 값으로 자유롭게 설정할 수 있습니다.</td>
  </tr>
  <tr>
    <td>서버</td>
    <td><b>신원 서비스(identity)</b>의 주소입니다. 오브젝트 스토리지 서비스 페이지의 <b>API Endpoint 설정</b> 대화 상자에서 확인할 수 있습니다.</td>
  </tr>
  <tr>
    <td rowspan="2">Tenant ID:Access Key</td>
    <td><b>Tenant ID</b>: 사용자의 프로젝트 ID입니다. 웹 콘솔의 <b>프로젝트 관리 > 프로젝트 기본 정보</b>에서 확인할 수 있습니다.</td>
  </tr>
  <tr>
    <td><b>Access ID</b>: NHN Cloud 회원 ID(이메일 형식) 또는 IAM 멤버 ID를 입력합니다.</td>
  </tr>
  <tr>
    <td>Secret Key</td>
    <td>오브젝트 스토리지 서비스 페이지의 <b>API Endpoint 설정</b> 대화 상자에서 설정한 API 비밀번호를 입력합니다.</td>
  </tr>
</table>

> [참고]
> API 비밀번호 설정 방법은 API 가이드의 [API 비밀번호 설정](api-guide-gov/#set-the-api-password) 항목을 참조하세요.

<a id="cyberduck-connect-object-storage"></a>
### 오브젝트 스토리지 연결

연결할 책갈피를 더블클릭하여 오브젝트 스토리지에 접속합니다.

<a id="cyberduck-retrieve-container-or-object"></a>
### 컨테이너/오브젝트 조회

오브젝트 스토리지에 접속하면 **모든 리전**의 컨테이너 목록이 브라우저에 나타납니다. 원하는 컨테이너를 더블클릭하여 컨테이너의 오브젝트 목록을 조회할 수 있습니다.

> [참고]
> 서로 다른 리전에 같은 이름의 컨테이너가 있다면 동일한 이름의 컨테이너가 여러 개 표시됩니다. 
> 메뉴에서 **보기 > 열 > 지역**을 선택하면 열 항목에 리전을 표시할 수 있습니다.

<a id="cyberduck-create-container"></a>
### 컨테이너 생성

컨테이너 목록의 빈 공간에서 마우스 오른쪽 버튼을 클릭한 뒤 **새 폴더...**를 선택해 새 컨테이너를 만들 수 있습니다. 컨테이너의 이름과 리전을 입력한 뒤 **생성**을 클릭하면 컨테이너가 생성됩니다.

> [참고]
> 컨테이너 목록의 빈 공간에서 마우스 오른쪽 버튼을 클릭한 뒤 **다시보기**를 선택해 컨테이너 목록을 새로 고침할 수 있습니다.

<a id="cyberduck-upload-object></a>
### 오브젝트 업로드

컨테이너를 선택한 뒤 브라우저 상단의 **동작** > **업로드...**를 클릭하거나, 오브젝트 목록에서 마우스 오른쪽 버튼을 클릭한 뒤 **업로드...**를 클릭해 파일을 선택하고 업로드합니다.

> [참고]
> Cyberduck으로 폴더를 업로드하거나 생성하면 폴더 이름과 동일한 0 바이트 오브젝트가 하나 더 생성됩니다. 이는 콘솔 또는 오브젝트 스토리지 API로 확인할 수 있으며 삭제해도 무방합니다.

<a id="cyberduck-download-object></a>
### 오브젝트 다운로드

다운로드할 오브젝트를 선택한 뒤 마우스 오른쪽 버튼을 클릭하고 **내려받기**를 선택합니다. 끌어다 놓기로 오브젝트를 다운로드할 수도 있습니다.

> [참고]
> 오브젝트를 다운로드하면 기본적으로 로컬의 **다운로드** 폴더에 저장됩니다. 마우스 오른쪽 버튼을 클릭해 **지정된 위치로 내려받기**를 선택하면 지정한 경로에 다운로드할 수 있습니다.
> 오브젝트를 업로드 또는 다운로드하면 **전송** 창이 팝업되어 진행 상태를 파악할 수 있습니다.

<a id="cyberduck-delete-container></a>
### 컨테이너 삭제

삭제할 컨테이너를 선택한 뒤 마우스 오른쪽 버튼을 클릭하고 **삭제**를 선택합니다.

> [주의]
> 컨테이너를 삭제할 경우 컨테이너 내부의 오브젝트도 모두 삭제되므로 주의해야 합니다.

<a id="cyberduck-delete-object></a>
### 오브젝트 삭제

삭제할 오브젝트를 선택한 뒤 마우스 오른쪽 버튼을 클릭하고 **삭제**를 선택합니다.

<a id="cyberduck-synchronize-folder></a>
### 폴더 동기화

로컬 폴더를 컨테이너 또는 폴더와 동기화할 수 있습니다. 컨테이너 또는 폴더를 선택한 뒤 마우스 오른쪽 버튼을 클릭하고 **동기화**를 선택합니다.
폴더 동기화는 다운로드, 업로드, 미러와 같은 세 가지 방식을 제공합니다.

<a id="cyberduck-synchronize-download></a>
#### 다운로드

오브젝트 스토리지에서 변경되거나 추가된 오브젝트를 로컬에 다운로드합니다.

<a id="cyberduck-synchronize-upload></a>
#### 업로드

로컬에서 변경되거나 추가된 파일을 오브젝트 스토리지에 업로드합니다.

<a id="cyberduck-synchronize-mirror></a>
#### 미러

로컬과 오브젝트 스토리지를 비교하여 변경되거나 누락된 파일 또는 오브젝트를 업로드하거나 다운로드합니다.

> [참고]
> 동기화에 대한 보다 자세한 내용은 [Cyberduck의 Synchronize Folders](https://docs.cyberduck.io/cyberduck/sync/#synchronize-folders) 문서를 참고하십시오.

<a id="terraform></a>
## Terraform

Terraform은 인프라를 손쉽게 구축하고 안전하게 변경하고, 효율적으로 인프라의 형상을 관리할 수 있는 오픈 소스 도구입니다. 기본적인 사용법은 [사용자 가이드 > Compute > Instance > 테라폼 사용 가이드](https://docs.nhncloud.com/ko/Compute/Instance/ko/terraform-guide/)를 참고하십시오.

<a id="terraform-resource-dependency></a>
### 리소스 의존성

일반적으로 각 리소스는 독립적이지만 다른 특정 리소스에 의존성을 가질 수도 있습니다. 리소스의 레이블을 통해 다른 리소스의 정보를 참조하면 Terraform은 자동으로 의존성을 설정합니다.
예를 들어, `conatiner1` 컨테이너에 포함되는 `object1` 오브젝트는 다음과 같이 표현될 수 있습니다.

```hcl
# 컨테이너 리소스
resource "nhncloud_objectstorage_container_v1" "container_1" {
  name = "container1"
}

# 오브젝트 리소스
resource "nhncloud_objectstorage_object_v1" "object_1" {
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name           = "object1"
  source       = "/tmp/dummy"
}
```

> [참고]
> 명시적인 리소스 의존성 지정 방법은 [Terraform의 Resource dependencies](https://developer.hashicorp.com/terraform/tutorials/configuration-language/dependencies) 문서를 참고하십시오.

<a id="terraform-resources-object-storage></a>
### Resources - Object Storage

<a id="terraform-resources-create-container></a>
#### 컨테이너 생성

```hcl
# 기본 컨테이너 생성
resource "nhncloud_objectstorage_container_v1" "container_1" {
  region = "KR1"
  name   = "tf-test-container-1"
}

# 오브젝트 버전 관리 컨테이너 생성
resource "nhncloud_objectstorage_container_v1" "container_2" {
  region = "KR1"
  name   = "tf-test-container-2"
  versioning_legacy {
    type     = "history"
    location = resource.nhncloud_objectstorage_container_v1.container_1.name
  }
}

# 퍼블릭 컨테이너 생성
resource "nhncloud_objectstorage_container_v1" "container_3" {
  region = "KR1"
  name   = "tf-test-container-3"
  container_read = ".r:*,.rlistings"
}
```

| 이름 | 타입 | 필수 | 설명 |
| ---- | ---- | ---- | ---- |
| region | String | | NHN Cloud 리소스를 관리할 리전 |
| name | String | O | 컨테이너 이름 |
| container_read | String | | 컨테이너 읽기에 대한 역할 기반 접근 규칙 설정 |
| container_write | String | | 컨테이너 쓰기에 대한 역할 기반 접근 규칙 |
| force_destroy | Boolean | | 컨테이너 강제 삭제 여부, `true` 또는 `false`<br>함께 삭제된 오브젝트는 복구할 수 없습니다. |
| versioning_legacy | Object | | 오브젝트 버전 관리 설정 |
| versioning_legacy.type | String | | `history`로 지정 |
| versioning_legacy.location | String | | 오브젝트의 이전 버전을 보관할 컨테이너 이름 |

<a id="terraform-resources-create-object></a>
#### 오브젝트 생성

```hcl
# 오브젝트 생성
resource "nhncloud_objectstorage_object_v1" "object_1" {
  region         = "KR1"
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name           = "test/test1.json"
  content_type = "application/json"
  content      = <<JSON
               {
                 "key" : "value"
               }
JSON
}

# 파일 업로드
resource "nhncloud_objectstorage_object_v1" "object_2" {
  region         = "KR2"
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name           = "test/test2.json"
  source       = "./test2.txt"
}
```

<table>
	<thead>
			<tr>
				<th>이름</th>
				<th>타입</th>
				<th>필수</th>
				<th>설명</th>
			</tr>
	</thead>
	<tbody>
		<tr>
			<td>name</td>
			<td>String</td>
			<td>O</td>
			<td>오브젝트 이름</td>
		</tr>
		<tr>
			<td>container_name</td>
			<td>String</td>
			<td>O</td>
			<td>컨테이너 이름</td>
		</tr>
		<tr>
			<td>source</td>
			<td>String</td>
			<td>O</td>
			<td> 업로드할 로컬 파일시스템상의 파일 경로<br><code>content</code>, <code>copy_from</code> 또는 <code>object_manifest</code>와 함께 사용할 수 없습니다.<br><code>source</code>, <code>content</code>, <code>copy_from</code> 또는 <code>object_manifest</code> 중 하나를 반드시 지정해야 합니다.</td>
		</tr>
		<tr>
			<td>content</td>
			<td>String</td>
			<td>O</td>
			<td>생성할 오브젝트의 내용<br><code>source</code>, <code>copy_from</code> 또는 <code>object_manifest</code>와 함께 사용할 수 없습니다.</td>
		</tr>
		<tr>
			<td>copy_from</td>
			<td>String</td>
			<td>O</td>
			<td>복사할 원본 오브젝트, <code>{컨테이너}/{오브젝트}</code><br><code>source</code>, <code>content</code> 또는 <code>object_manifest</code>와 함께 사용할 수 없습니다.</td>
		</tr>
		<tr>
			<td>object_manifest</td>
			<td>String</td>
			<td>O</td>
			<td>분할한 세그먼트 오브젝트를 업로드한 경로, <code>{Segment-Container}/{Segment-Object}/</code><br><code>source</code>, <code>content</code> 또는 <code>copy_from</code>과 함께 사용할 수 없습니다.</td>
		</tr>
		<tr>
			<td>content_disposition</td>
			<td>String</td>
			<td></td>
			<td><code>Content-Disposition</code> 헤더를 지정</td>
		</tr>
		<tr>
			<td>content_encoding</td>
			<td>String</td>
			<td></td>
			<td><code>Content-Encoding</code> 헤더를 지정</td>
		</tr>
			<tr>
			<td>content_type</td>
			<td>String</td>
			<td></td>
			<td><code>Content-Type</code> 헤더를 지정</td>
		</tr>
		<tr>
			<td>delete_after</td>
			<td>Integer</td>
			<td></td>
			<td>오브젝트 유효 시간, 유닉스 시간(초)</td>
		</tr>
		<tr>
			<td>delete_at</td>
			<td>String</td>
			<td></td>
			<td>오브젝트 만료 날짜, 유닉스 시간(초), RFC3339 포매팅 문자열</td>
		</tr>
		<tr>
			<td>detect_content_type</td>
			<td>Boolean</td>
			<td></td>
			<td>콘텐츠 타입 추론 여부</br>설정 시 <code>content_type</code>이 무시됩니다.</td>
		</tr>
	</tbody>
</table>

<a id="reference></a>
## 참고 사이트
Cyberduck - [https://docs.cyberduck.io/cyberduck/](https://docs.cyberduck.io/cyberduck/)
Terraform - [https://www.terraform.io/](https://www.terraform.io/)
Terraform Registry - [https://registry.terraform.io/](https://registry.terraform.io/)

