- 필터 기반 프레임워크
- 서버 구동 시, [[Spring Security 초기화]]가 일어남
# 시큐리티 인증/인가 흐름
![](https://i.imgur.com/ssjggnr.png)
1. Client의 요청
2. `DelegatingFilterProxy`가 `FilterChainProxy`로부터 `AuthenticatinoFilter` 호출
3. `AuthenticationFilter`가 사용자 정보에 대한 [[Authentication]] 객체 생성
4. `Authentication` 객체를 [[AuthenticationManager]]에게 전달
5. `AuthenticationManager`가 이를 `AuthenticationProvider`에게 위임 (사용자의 아이디, 패스워드 검증)
6. [[UserDetailsService]]를 이용해 사용자 정보를 가져오고 [[UserDetails]]를 만들어 반환
7. [[AuthenticationProvider]]가 `UserDetails`를 받고 사용자 패스워드와 맞는지 `PasswordEncoder`를 통해 검증
8. 검증 완료 후, user 정보와 authorities를 이용해 `Authentication` 객체 생성
9. `AuthenticationFilter`가 `Authentication` 객체를 받고 `SecurityContextHolder`를 통해서 [[SecurityContext]]에 저장
	- `SecurityContext` 내에 `Authentication`과  `UserDetails` 정보 저장됨