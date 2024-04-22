## 개념
- 사용자의 기본 정보를 저장하는 인터페이스
- [[Spring Security]]에서 사용하는 사용자 타입
- 저장된 사용자 정보는 추후 인증 절차를 위해 [[Authentication]] 객체에 포함
- 구현체로서 User 클래스 제공
## 구조
![](https://i.imgur.com/7EQXpYB.png)
- `getAuthorities()` : 사용자에게 부여된 권한을 반환
- `getUsername()` : 사용자 이름 반환
- `isEnabled()` : 사용자의 활성화 여부
- `isAccountNonLocked()` : 사용자가 잠겨 있는지 아닌지를 나타냄
- `getPassword()` : 사용자 비밀번호 반환
- `isAccountNonExpired()` : 사용자 계정의 유효 기간이 지났는지 나타냄
- `isCredentialsNonExpired()` : 사용자의 비밀번호가 유효 기간이 지났는지 나타냄
## 흐름
![](https://i.imgur.com/2gvPiM2.png)
1. AuthenticationProvider가 UserDetailsService에게 username으로 사용자 정보 검색 위임
2. [[UserDetailsService]]는 UserRepository를 이용해 DB로부터 사용자 정보를 가져옴
3. 시스템에서 사용하는 유저 엔티티(가정 : UserInfo)로부터 UserDetails로 맵핑
4. 맵핑한 UserDetails 객체를 [[AuthenticationProvider]]에게 전달
5. AuthenticationProvider가 UserDetails 정보를 이용해 Authentication 객체 생성