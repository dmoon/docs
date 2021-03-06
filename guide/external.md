# 라이브리 회원 연동 API 명세 
``rev.1.201510071500`` ``Tamm``

## 1. Overview

- 라이브리 회원 연동 API는 고객사의 일반 회원 계정을 라이브리 소셜 계정과 연동하는 역할을 합니다.
- 고객사의 회원 정보를 읽어오기 위해 `인증 및 유효성 검사` 과정이 필요하며, 이를 구현해 주셔야 합니다.

### 준비해 주셔야 하는 내용들

1. 고객사 BI 아이콘 스프라이트 2종(일반, HiDPI)
2. 로그인, 로그아웃 페이지 URL 혹은 API
3. 기본 프로필 이미지(선택)
4. 개발 사항
	- 클라이언트: 로그인, 로그아웃 시 라이브리 서버로 해당 사용자의 토큰을 전달하는 함수 작성
	- 서버: 해당 사용자의 유효성 검사를 위한 API 작성

## 2. 회원 로그인

1. 사용자가 로그인 할 때 서버에서 해시를 생성 하십시오.
	- 생성한 해시는(이하 ``CODE``) 세션 혹은 인증이 유지되고 있을 때 해당 사용자를 식별할 수 있는 고유한 값 입니다.
	- 해당 `CODE`는 사용자 ID를 기반으로 난독화 하시거나 완전한 랜덤 해시를 생성하는 것을 권장 드립니다.
	- 해당 ``CODE``에 사용자 정보를 입력하지 마십시오.(예: ``davidphil0912``)
		- 적절한 ``CODE``의 예 : ``ef55562fc6aa943ce34``

2. 사용자의 로그인 응답을 전송할 때 해당 ``CODE``를 클라이언트로 전달 하십시오.
3. 클라이언트는 서버로부터 응답을 받았다면 다음 API에 해당 토큰을 전송 하십시오.

**[Request]**

```
GET /v1/auth/external HTTP/1.1
Host: passport.livere.com
```	
I
아래 파라메터들을 클라이언트에서 ``window.open`` 함수를 통해 요청 합니다.

| 키  | 설명 | 필수         |
| :-------- | :-------------------- | :--: |
| code     | 위 과정에서 생성한 사용자 토큰  | O   |
| target   | 전달된 고객사 ID      | O   |

예)

```
curl -v -X GET https://passport.livere.com/v1/auth/external \
 -d 'target=sbs' \m
 -d 'code={CODE}'
```

- 다음으로 라이브리 서버는 해당 토큰의 유효성을 검사하기 위해 유효성 검사 API를 호출 합니다.
	- 따라서 유효성을 검사할 수 있는 API를 작성해 주셔야 합니다.
	- 사용자의 ID는 자체적으로 암호화 혹은 난독화해서 전달해 주시는 것을 권장 드립니다.

- 라이브리는 다음과 같이 유효성 검사 API를 호출 합니다.

**[Request]**

```
curl -v -X GET {{VALID_API_URI}} \
 -d 'code={CODE}'
```

- 해당 API의 응답 값은 다음과 같이 설정해 주십시오.
	
**[Response]**

```
HTTP/1.1 200 OK
{
	status: 200,
	id: {ENCRYPTED_ID}, // ef55562fc6aa943ce34
    username: {USERNAME}, // Angry Neeson
	profile: {PROFILE_IMAGE} // https://.../2015727/10613-fzftvw.jpg
}
```

* 프로필 이미지가 없는 경우 생략해 주시기 바랍니다.

해당 과정이 완료되면 회원 연동이 완료됩니다.

## 3. 회원 로그아웃

- 사용자가 로그아웃 할 때 클라이언트에서 다음 API를 비동기로 호출 하십시오.

**[Request]**

```
GET /v1/logout HTTP/1.1
Host: passport.livere.com
```	

```
curl -v -X GET https://passport.livere.com/v1/logout
```