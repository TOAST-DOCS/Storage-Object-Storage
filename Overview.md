## Infrastructure > Object Storage > Overview

Object Storage 는 다수의 대용량 파일을 저장할 수 있는 저장소 서비스입니다.

## 주요 기능

- 웹 콘솔이나 RESTful API를 통해 파일을 올리거나 내려 받을 수 있습니다.
- 용량 제한을 가진 디스크와 달리 필요할 때 원하는 만큼 사용할 수 있습니다.
- 접근 권한 설정을 통해 파일의 공개 여부를 선택할 수 있습니다.
- 총 3개의 복제본을 유지하므로 안전하게 파일을 보관할 수 있습니다.

## 서비스 구조

![[그림 1] Object Storage 서비스 구조](http://static.toastoven.net/prod_infrastructure/object_storage/img_01.jpg)
<center>[그림 1] Object Storage 서비스 구조</center>


## 서비스 용어

|용어|	설명|
|---|---|
|Object|	실제 Data를 의미합니다. Object Storage의 기본적인 관리 단위로 사용됩니다. 일반적으로 파일을 Object 로 생각할 수 있습니다.|
|Folder|	Object를 묶는 단위 입니다. 계층적으로 Object를 관리할 수 있도록 도와줍니다. Windows의 폴더나 Linux의 디렉터리와 유사합니다.|
|Container|	최상위 Folder를 의미합니다. 기본적으로 접근 권한 설정의 단위가 됩니다. 모든 Object는 반드시 Container 안에 존재해야 합니다.|
|Account|	Object Storage의 사용자 계정 명입니다. TOAST Cloud의 Object Storage는 Account 단위로 격리됩니다.|
|API| Endpoint	Object Storage에 RESTful API로 접근하기 위해 제공되는 HTTP URL입니다. 제공하는 RESTful API를 참고하여 해당 URL로 요청하면 Object Storage에 접근할 수 있습니다. Account 별로 발급되며 Token을 가지고 요청해야 합니다.|
