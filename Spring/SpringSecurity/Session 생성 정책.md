## 전략
`SessionCreationPolicy`로 설정
### 1️⃣ SessionCreationPolicy.ALWAYS
- 인증 여부에 상관없이 항상 세션 생성
- `ForceEagerSessionCreationFilter`클래스 추가 구성, 세션 강제 생성
### 2️⃣ SessionCreationPolicy.NEVER
- Spring Security가 세션 생성 X
- 애플리케이션이 이미 생성한 세션은 사용할 수 있음
### 3️⃣ SessionCreationPolicy.IF_REQUIRED
- 기본값
- 필요한 경우에만 세션 생성
- 인증이 필요한 자원에 접근할 때 세션 생성
### 4️⃣ SessionCreationPolicy.STATELESS
- 세션을 전혀 생성하거나 사용하지 않음
- [[SecurityContext]]를 세션에 저장하지 않음 -> JWT 등 세션을 사용하지 않는 방식에 유용
- [[SecurityContextHolderFilter]]는 Request 단위로 항상 새로운 context를 생성하므로 context 영속성이 유지되지 않음

>💡 **인증** 차원에서만 stateless
> - CSRF 기능은 사용자의 세션을 생성해 CSRF 토큰을 저장함
> - 인증 프로세스의 context 영속성에 영향 X
## SessionManagement API
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	http.sessionManagement((session) -> session
		.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
	);

	return http.build();
}
```

