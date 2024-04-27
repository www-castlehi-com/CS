## 개요
- 최신 방식의 CSRF 공격 방어 방법
- 서버가 쿠키를 설정할 때 SameSite 속성 지정 -> 크로스 사이트 간 쿠키 전송에 대한 제어
- SpringSecurity가 아닌 SpringSession이 지원
- 오래된 브라우저는 지원 X
- 유일한 방어 수단이 아닌 심층 강화 방어 일환으로 사용
## 속성
### 1️⃣ Strict
- 동일 사이트에서 오는 모든 요청에 쿠키 포함
- 크로스사이트간 HTTP 요청에 쿠키 포함 X
![](https://i.imgur.com/imDTjlV.png)
### 2️⃣ Lax
- 기본 설정
- 동일 사이트에서 오거나 Top Level Navigation에서 오는 요청 및 메소드가 읽기 전용인 경우 쿠키 전송
	- \<a>, window.location.replace, 302 리다이렉트 등의 이동은 쿠키 포함
	- \<iframe>, \<img>, AJAX 통신은 쿠키 전송 X
![](https://i.imgur.com/uX964xB.png)
### 3️⃣ None
- 동일 사이트, 크로스 사이트 요청의 경우에도 쿠키 전송
- HTTPS에 의한 Secure 쿠키로 설정
![](https://i.imgur.com/pgJfCys.png)
