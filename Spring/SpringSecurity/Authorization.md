## 개념
- 특정 자원에 접근할 수 있는 사람을 결정하는 것
- `GrantedAuthority` 클래스를 통해 권한 목록 관리하며 사용자의 [[Authentication]] 객체와 연결
![](https://i.imgur.com/oXaoL9h.png)
1. Manager 권한을 가진 Client가 요청
2. Authentication은 사용자가 시스템에 존재하는지 확인
3. 사용자가 시스템에 존재하고, Authentication 객체에서 권한 확인
4. Authorization에서 권한과 매핑된 자원 확인
## GrantedAuthority
### 개념
- 권한 목록 저장소
- 인증 주체에게 부여된 권한을 사용하도록 함
- [[AuthenticationManager]]에 의해 Authentication 객체에 삽입
- 스프링 시큐리티가 인가 결정을 내릴 때 [[AuthorizationManager]]를 사용하여 Authentication 객체로부터 GrantedAuthority를 읽어 처리
### 구조
![](https://i.imgur.com/CFMN7Fz.png)
- `getAuthority()` : AuthorizationManager가 GrantedAuthority의 정확한 문자열 표현을 얻기 위해 사용