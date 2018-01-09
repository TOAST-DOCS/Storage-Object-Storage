## Infrastructure > Object Storage > Getting Started

Object Storage는 웹 콘솔을 이용해 파일 업/다운로드가 가능하여 확장 가능한 파일 서비스 구성이 가능합니다. 예를 들어 대용량 파일을 Object Storage에 저장하고 직접 파일을 배포하면, 웹 서버의 네트워크 부하와 디스크 용량 문제가 해결됩니다.

이 문서에서는 예제로서 다음과 같은 내용을 다룹니다.

- Container 생성
- Object 올리기, 내려 받기

## Container 생성

Object Storage에서는 파일을 담기 위해 Container를 지원합니다. Object Storage에 파일을 올리기 위해서는 반드시 하나 이상의 Container가 필요합니다. Object Storage 상품을 활성화 한 뒤 Container를 생성합니다.

```
[Infrastructure] > [Object Storage] > [상품 이용] 버튼 클릭
[Infrastructure] > [Object Storage] > [Containers] > [+ Container 생성] 버튼 클릭
```

![[그림 1] Container 생성](http://static.toastoven.net/prod_infrastructure/object_storage/img_101_sc.png)
<center>[그림 1] Container 생성</center>

[그림 2]와 같은 <Container 생성> 대화창이 나타납니다. 외부에서 다운로드 가능해야 하므로 Public으로 생성합니다.
Storage Class는 기본값인 Standard를 선택합니다. (차후 신규 항목 추가 예정)
```
[이름] 항목에 Container명 입력
[Container Access] 항목에 “Public”선택
[Storage Class] 항목에 "Standard" 선택 (기본값)
```

![[그림 2] Container 생성](http://static.toastoven.net/prod_infrastructure/object_storage/img_03_sc.png)
<center>[그림 2] Container 생성</center>

Container 목록에서 생성한 Container명을 확인하고 클릭합니다.

```
[Container 생성] 버튼 클릭
```

![[그림 3] Container 생성 확인](http://static.toastoven.net/prod_infrastructure/object_storage/img_103_sc.png)
<center>[그림 3] Container 생성 확인</center>

## Object 올리기

링크를 걸 파일을 업로드 합니다.

```
[Upload Object] 버튼 클릭
```

![[그림 4] Object 올리기](http://static.toastoven.net/prod_infrastructure/object_storage/img_104.png)
<center>[그림 4] Object 올리기</center>

파일을 선택합니다.

```
[파일 선택] 버튼 클릭
대용량 파일 선택 > [열기] 버튼 클릭
[Upload Object] 버튼 클릭
```

![[그림 5] Object 업로드 대화창](http://static.toastoven.net/prod_infrastructure/object_storage/img_105.png)
<center>[그림 5] Object 업로드 대화창</center>

## Object 내려 받기

웹 서버에 링크를 연결하기 위한 Public URL을 [그림 6], [그림 7]과 같이 확인합니다.

```
내려 받을 Object 선택
[Actions] > [Public URL 보기] 메뉴 클릭
```

![[그림 6] 내려 받을 Object 선택](http://static.toastoven.net/prod_infrastructure/object_storage/img_106.png)
<center>[그림 6] 내려 받을 Object 선택</center>

![[그림 7] Public URL 확인](http://static.toastoven.net/prod_infrastructure/object_storage/img_08.jpg)
<center>[그림 7] Public URL 확인</center>

[그림 7]의 <Public URL> 대화창의 주소를 복사해서 index.html에 반영합니다.

```
[root@host-192-168-0-4 ~]# cat > index.html
<html>
<body> hello world!
<a href="https://api-storage.cloud.toast.com/v1/AUTH_70c11b619c3f4a53af8d2466ee1b0234/web_static/CentOS-7-x86_64-Minimal-1503-01.iso">Download</a>
</body>
</html>
```

웹 서버 실행

```
[root@host-192-168-0-4 ~]# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

웹 브라우저로 접속한 뒤 "Download" 링크를 클릭하여 정상적으로 파일이 다운로드 되는 것을 확인합니다.

![[그림 8] 다운로드 링크 확인](http://static.toastoven.net/prod_infrastructure/object_storage/img_09.jpg)
<center>[그림 8] 다운로드 링크 확인</center>
