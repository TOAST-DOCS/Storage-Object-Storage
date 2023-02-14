## SwiftStack 사용 가이드
이 문서는 SwiftStack 을 통해 NHN Cloud의 Object Storage 서비스를 사용하는 방법을 설명합니다.

## SwiftStack

## SwiftStack 클라이언트 다운로드

[SwiftStack 다운로드 페이지](https://platform.swiftstack.com/docs/install/swiftstack_client.html)에서 본인의 OS에 맞는 클라이언트를 다운로드 할 수 있습니다.

## Object Storage 연동 설정

### Account 생성

처음 화면의 `Add account` 를 통해서 어카운트를 생성합니다. 생성 시 입력해야 하는 정보는 아래와 같습니다.

- Auth Type : `V2`
- Username
  - NHN Cloud 사용자 Email 혹은 IAM 사용자 ID
- Password
  - 오브젝트 스토리지 서비스 페이지의 `API Endpoint 설정` 버튼을 클릭해 설정할 수 있습니다.
    - `API 비밀번호 설정`에서 원하는 비밀번호로 설정합니다.
    - 설정한 값을 입력합니다.
- Auth URL
  - 오브젝트 스토리지 서비스 페이지의 `API Endpoint 설정` 버튼을 클릭하여 확인할 수 있습니다.
  - 콘솔 > `Storage` > `Object Storage` > `API 엔드포인트 설정` > `신원 서비스(identity)` 의 주소 끝에 `/tokens`를 붙여서 입력합니다.
- Project Name
  - Project_id : 사용자의 Project ID 입니다.
  - 콘솔 상단 프로젝트 우측의 톱니바퀴 클릭 > `프로젝트 기본 정보` - `프로젝트 ID`에서 확인할 수 있습니다.
- Region
  - 사용할 리전을 지정할 수 있습니다.

> [참고]
> 입력값이 올바르지 않은 경우 Region을 지정할 때 오류 팝업창이 나타납니다.

입력값을 올바르게 입력한 뒤, 하단의 `Save`을 클릭해 새로운 어카운트를 생성할 수 있습니다.

## Object Storage 사용 방법

생성한 어카운트를 더블 클릭하여 Object Storage에 접속할 수 있습니다.

### 컨테이너 / 오브젝트 조회

Object Storage에 접속하면 설정한 리전의 컨테이너 리스트가 브라우저에 나타납니다.

원하는 컨테이너를 더블 클릭하여 컨테이너 내 오브젝트를 조회할 수 있습니다.

### 컨테이너 생성

브라우저 상단의 `Add Container` - `Edit Before Creation` 을 통해서 생성할 컨테이너의 이름을 지정할 수 있습니다.

이름을 입력한 후 하단의 `Create`을 클릭해서 컨테이너 생성을 요청할 수 있습니다.

> [참고]
> 생성 요청 후 브라우저 상단의 `Refresh` 를 진행하여 리스트를 업데이트할 수 있습니다.

### 객체 업로드

브라우저 상단의 `Upload` - `Choose File(s)` 버튼을 클릭하여 업로드할 파일 / 폴더를 선택할 수 있습니다.

또한 브라우저 밖에서 드래그 앤 드롭을 통해 파일을 업로드할 수 있습니다.

> [참고]
> `Upload` - `Choose File(s)`을 통해서 파일을 업로드하는 경우 하단의 `Object Name Prefix`을 통해 업로드할 파일들 이름 앞에 접두사를 붙일 수 있습니다.

### 객체 다운로드

#### 폴더 다운로드

폴더 리스트 우측의 `Actions`에 있는 다운로드 버튼을 통해서 원하는 폴더를 다운로드받을 수 있습니다.

또한, 원하는 폴더들을 리스트 좌측 체크박스에서 체크한 뒤, 브라우저 상단의 `Selection` > `Download` 를 통해서 다운로드받을 수 있습니다.

#### 파일 다운로드

원하는 파일들을 리스트 좌측 체크박스에서 체크한 뒤, 브라우저 상단의 `Selection` > `Download` 를 통해서 다운로드받을 수 있습니다.

### 삭제

파일 / 폴더 리스트 우측의 `Actions`에 있는 삭제 버튼을 통해서 원하는 파일 / 폴더를 삭제할 수 있습니다.

또한 원하는 파일 / 폴더들을 리스트 좌측 체크박스에서 체크한 뒤, 브라우저 상단의 `Selection` > `Delete` 를 통해서 삭제할 수 있습니다.

> [주의]
콘솔과 달리 컨테이너 내부에 파일이 있어도 전부 삭제됩니다.