# WEB2 - OAuth 2.0

## 1. 수업 소개

유튜브 생활코딩 WEB2 - OAuth 2.0 강의를 기반으로, 개인적으로 내용을 추가.<br/>

나의 서비스가 사용자의 다른 서비스 API에 접근하기 위해서는 사용자로부터 권한을 위임 받아야 한다.<br/>
이 때, 사용자의 패스워드 없이도 권한을 위임 받을 수 있는 방법이 필요.<br/>
이를 위해 고안된 기술이 OAuth.<br/>
오늘날 많은 API들이 OAuth를 통해서 상호 연동을 지원하고 있다.

구글, 페이스북과 같은 서비스의 API에 사용자 대신 접근하는 방법을 담고 있다.<br/>
또한 대표적으로 많이 사용되는 '다른 서비스로 로그인 하기' 기능을 구현하는데도 필수적으로 필요한 기능.

### 배경

- 관련 항목

1. 나의 서비스 (Mine)<br/>
2. 해당 서비스 사용자 (User)<br/>
3. 나의 서비스가 연동하고자 하는 서비스 (Their)

Mine은 Their에서 사용되는 User의 정보 일부가 필요.<br/>
정보를 가져오기 위해 Their 서비스에서 사용하는 User의 ID/PW가 필요.

사용자가 third party Application에 ID/PW를 제공하기 싫은 요구가 기반이다.<br/>
해당 ID/PW는 일반적으로 대부분의 서비스에서 공통적으로 사용되기 때문에, 유출된다면 굉장한 보안 사고.

Mine이 OAuth통해 accessToken을 부여받고 이 accessToken을 통해 데이터를 핸들링할 수 있게 된다.<br/>
또한, OAuth의 특징을 이용하면 회원들의 ID/PW 보관 할 필요 없이 회원 식별 기능을 구현할 수도 있다.

<br/>

## 2. 역할

### OAuth

OAuth 2.0은 다양한 플랫폼 환경에서 권한 부여를 위한 산업 표준 프로토콜.

간단하게 인증(Authentication)과 권한(Authorization)을 획득하는 것으로 볼 수 있다.

인증은 시스템 접근을 확인하는 것 (로그인). -> 인증만 하는 것은 openID<br/>
권한은 행위의 권리를 검증하는 것.

### OAuth 1.0a

구현이 복잡하고 웹이 아닌 어플리케이션에서의 지원이 부족.<br/>
HMAC을 통해 암호화를 하는 번거로운 과정을 겪으며, 인증토큰(access token)이 만료되지 않는다.

### OAuth 2.0

OAuth 1.0a보다 기능의 단순화, 기능과 규모의 확장성 등을 지원하기 위해 만들어 졌다.<br/>
1.0a는 만들어진 다음 표준이 된 반면 2.0은 처음부터 표준 프로세스로 만들어짐.<br/>
https가 필수여서 간단해 졌다(암호화는 https에 맡김).<br/>
1.0a는 인증방식이 한가지였지만 2는 다양한 인증방식을 지원한다.<br/>
api서버에서 인증서버를 분리 할 수 있도록 해 놓았다.

위에서 언급된 3개의 주체 명칭을 정정 해보도록 한다.<br/>

1. 나의 서비스 (Client)<br/>
2. 해당 서비스 사용자 (Resource Owner)<br/>
3. 나의 서비스가 연동하고자 하는 서비스 (Resource Server (+ Authorization Server))

<br/>

## 3. 등록

Client가 Resource Server를 이용하기 위해서는 Resource Server에게 사전에 승인을 받아놓아야 한다.<br/>
이 과정을 register(등록)이라고 하는 것.

서비스마다 등록하는 방법은 모두 다르다.<br/>
하지만 공통적으로 Client ID / Client Secret / Authorized redirect URIs를 필요로 한다.

Client ID : Client Service를 식별하는 식별자<br/>
Client Secret : Client Service를 식별하는데 추가로 사용되는 비밀번호(노출 금지)<br/>
Authorized redirect URIs : 해당 주소에서 accessToken을 요청

<br/>

## 4. Resource Owner의 승인

Client는 redirect URIs에 해당하는 페이지를 제작해놓고 기다려야 한다.

<img src="https://developers.payco.com/static/img/@img_guide2.jpg" width="80%">

- 동작 순서

1. Resource Owner가 Client에 접속.

2. Client가 Resource Owner의 Resource Server 자원을 사용해야 하는 상황 발생(로그인 요청).

3. 로그인 요청(버튼)은 다음의 URI를 링크.<br/>
   ex) https://resource.server/?client_id=1&scope=B,C&redirect_uri=https://client/callback<br/>

4. Resource Owner가 버튼을 누르면 해당 URI로 요청이 발생.

5. Resource Server는 Resource Owner의 '최초 가입 여부'와 '현재 로그인이 되어있는 상태'인지 확인.<br/>
   최초가입 o, 로그인여부 o => client_id와 redirect_uri의 유효성 판단 -> Resource Server DB에서 최초 유무 확인 -> Client에게 권한 부여 확인 메시지<br/>
   최초가입 o, 로그인여부 x => 로그인 화면 요청 -> client_id와 redirect_uri의 유효성 판단 -> Resource Server DB에서 최초 유무 확인 -> Client에게 권한 부여 확인 메시지<br/>
   최초가입 x, 로그인여부 o => client_id와 redirect_uri의 유효성 판단 -> Resource Server DB에서 최초 유무 확인<br/>
   최초가입 x, 로그인여부 x => 로그인 화면 요청 -> client_id와 redirect_uri의 유효성 판단 -> Resource Server DB에서 최초 유무 확인

6. Resource Server의 DB에 Resource Owner가 허용했다는 정보를 저장.

<br/>

## 5. Resource Server의 승인

7. Resource Server는 Resource Owner에게 임시 비밀번호로 사용되는 authorization code를 전송.<br/>
   Resource Owner가 사용중인 웹 브라우저가 ex) https://client/callback?code=3 로 리다이렉트 시킴.

8. Client는 authorization code를 부여받게 됨.

9. Client는 accessToken을 부여받기 위해 Resource Server에 authorization code를 중심으로 하는 요청 재전송.<br/>
   ex) https://resource.server/token?grant_type=authorization_code&code=3&redirect_uri=https://client/callback&client_id=1&client_secret=2

<br/>

## 6. Access token

10. Resource Server는 받은 정보가 모두 일치하는지 확인 후, 내부에서 authorization_code를 제거.

11. Resource Server가 accessToken을 생성 후 Client에게 응답.

12. Client는 내부적으로(DB같은 곳) accessToken을 저장.

<br/>

## 7. API 호출

Client는 Resource Server가 허용한 방식대로 데이터를 요청해야 함.<br/>
Client에 데이터를 제공하기 위한 접점의 조작 장치를 API라고 함.

- curl

클라이언트에서 커맨드 라인이나 소스코드로 손 쉽게 웹 브라우저 처럼 활동할 수 있도록 해주는 기술 (커맨드라인 Tool 혹은 라이브러리).

서버와 통신할 수 있는 커맨드 명령어 툴이다.<br/>
웹개발에 매우 많이 사용되고 있는 무료 오픈소스이다 curl의 특징으로는 다음과 같은 수 많은 프로토콜을 지원한다는 장점이 있다.<br/>
(DICT, FILE, FTP, FTPS, Gopher, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMB, SMBS, SMTP, SMTPS, Telnet, TFTP)

url을 가지고 할 수 있는 것들은 다할 수 있다.

```
curl -H "Authorization: Bearer {access_token}" {API_uri}
```

<br/>

## 8. Refresh token

access_token은 수명이 존재하며, 수명이 끝나면 API가 데이터를 주지 않는다.

새로 access_token을 발급받는 방법이 refresh_token.<br/>
보통 access_token을 발급할 때, refresh_token도 같이 발급한다.

Resource Server로 부터 Invalid Token Error가 발생(토큰 만료)하면,<br/>
Authorization Server로 refresh_token을 보냄으로써 access_token을 재발급 받는다.<br/>
이 때, refresh_token의 재발급 및 갱신 여부 처리는 Resource Server마다 다르다.

<br/>

## 9. 마치며

OAuth의 가장 매력적인 부분은,<br/>
Client가 제 3자인 Resource Server를 통해 Resource Owner의 신원을 인증할 수 있다는 것.

<br/>
