## 개념
- 사용자의 아이디, 비밀번호가 유효한지 검증
- 다양한 인증 매커니즘 지원 (예 : 사용자 이름-비밀번호 기반, 토큰 기반, 지문 인식 등)
- 성공적인 인증 후 User와 권한을 포함한 [[Authentication]] 객체 반환
- 인증 과정 중 문제가 발생할 경우, AuthenticationException 발생
## 구조
![](https://i.imgur.com/xSzbNRN.png)
- `authenticate()` : [[AuthenticationManager]]로부터 Authentication 객체를 전달 받아 인증 수행
- `supports()` : 인증을 수행할 수 있는 조건인지 확인
## 흐름
![](https://i.imgur.com/iUNBett.png)
1. `AuthenticationManager`가 username + password로 `Authentication` 객체를 만들어 `AuthenticationProvider`에게 제공
2. `AuthenticationProvider`은 사용자 유무, 비밀번호 검증, 보안 강화 처리 등을 지원
	- 인증이 성공할 시 , UserDetails + 권한으로 `Authentication` 객체를 만들어 `AuthenticationManager`에게 제공
	- 인증 실패 시, `AuthenticationException` 발생
## 사용 방법
### 1️⃣ 일반 객체로 생성
- Bean X
- POJO
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	AuthenticationManagerBuilder managerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
	managerBuilder.authenticationProvider(new CustomAuthenticationProvider());
	http.authenticationProvider(new CustomAuthenticationProvider2());

	http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
	http.formLogin(Customizer.withDefaults());

	return http.build();
}
```
### 2️⃣ 빈으로 생성
#### 1. 빈을 한 개만 정의
1. 기존 빈 대체
```java
@Bean
public AuthenticationProvider customAuthenticationProvider() {
	return new CustomAuthenticationProvider();
}
```
- `DaoAuthenticationProvider` 자동 대체
![](https://i.imgur.com/MObUSb7.png)
2. 빈 커스텀
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http, AuthenticationManagerBuilder builder, AuthenticationConfiguration configuration) throws Exception {
	AuthenticationManagerBuilder managerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
	managerBuilder.authenticationProvider(customAuthenticationProvider());
	ProviderManager providerManager = (ProviderManager)configuration.getAuthenticationManager();
	providerManager.getProviders().remove(0);
	builder.authenticationProvider(new DaoAuthenticationProvider());

	http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
	return http.build();
}
```
- `configuration.getAuthentcationManager` == `builder.authenticationProvider` : parent의 providers 제공
![](https://i.imgur.com/DQYpy0l.png)
#### 2. 빈을 두 개 이상 정의
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	AuthenticationManagerBuilder managerBuilder = http.getSharedObject(AuthenticaitonManagerBuilder.class);
	managerBuilder.authenticationProvider(customAuthenticationProvider());
	managerBuilder.authenticationProvider(customAuthenticationProvider2());

	http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
	http.formLogin(Customizer.withDefaults());

	return http.build();
}

@Bean
public AuthenticationProvider customAuthenticationProvider() {
	return new CustomAuthenticationProvider();
}

@Bean
public AuthenticationProvider customAuthenticationProvider2() {
	return new CustomAuthenticationProvider2();
}
```
- 일반 객체와 동일한 구조이지만 빈을 쓸 수 있음
![](https://i.imgur.com/gjPkL2U.png)
## 코드
**Provider 생성**
```java
package org.springframework.security.config.annotation.authentication.configuration;  
  
class InitializeAuthenticationProviderBeanManagerConfigurer extends GlobalAuthenticationConfigurerAdapter { 

	//...
	
	class InitializeAuthenticationProviderManagerConfigurer extends GlobalAuthenticationConfigurerAdapter {  
	  @Override  
	  public void configure(AuthenticationManagerBuilder auth) {  
		 if (auth.isConfigured()) {  
			return;  
		 }  
		 AuthenticationProvider authenticationProvider = getBeanOrNull(AuthenticationProvider.class);  
		 if (authenticationProvider == null) {  
			return;  
		 }  
		 auth.authenticationProvider(authenticationProvider);  
	  }  
		
	//...
	
   }  
  
}
```
- `AuthenticationProvider` 구현체 중 하나인 `DaoAuthenticationProvider`는 `InitializeAuthenticationProviderBeanManagerConfigurer`에 의해 초기화
- Configurer이므로 `init()`, `configure()` 메소드 실행

- `auth.isConfigured()`
	```java
	package org.springframework.security.config.annotation.authentication.builders;  
	  
	public class AuthenticationManagerBuilder  
	      extends AbstractConfiguredSecurityBuilder<AuthenticationManager, AuthenticationManagerBuilder>  
	      implements ProviderManagerBuilder<AuthenticationManagerBuilder> {  
	
		//...
		
		public boolean isConfigured() {  
		  return !this.authenticationProviders.isEmpty() || this.parentAuthenticationManager != null;  
		}  
		
		//...
	}
	```
	- `AuthenticatioinProvider` 객체를 가지고 있는지 확인
- `getBeanOrNull()` : `AuthenticationProvider`이름으로 빈이 생성되어 있는지 확인
	```java
	private <T> T getBeanOrNull(Class<T> type) {  
		String[] beanNames = InitializeUserDetailsBeanManagerConfigurer.this.context.getBeanNamesForType(type);  
		if (beanNames.length != 1) {  
		  return null;  
		}  
		return InitializeUserDetailsBeanManagerConfigurer.this.context.getBean(beanNames[0], type);  
	}
	```
**UserDetails 생성**
```java
package org.springframework.security.config.annotation.authentication.configuration;  
  
class InitializeUserDetailsBeanManagerConfigurer extends GlobalAuthenticationConfigurerAdapter {  

	//...
	
	class InitializeUserDetailsManagerConfigurer extends GlobalAuthenticationConfigurerAdapter {  
	
	  @Override  
	  public void configure(AuthenticationManagerBuilder auth) throws Exception {  
		 if (auth.isConfigured()) {  
			return;  
		 }  
		 UserDetailsService userDetailsService = getBeanOrNull(UserDetailsService.class);  
		 if (userDetailsService == null) {  
			return;  
		 }  
		 PasswordEncoder passwordEncoder = getBeanOrNull(PasswordEncoder.class);  
		 UserDetailsPasswordService passwordManager = getBeanOrNull(UserDetailsPasswordService.class);  
		 DaoAuthenticationProvider provider = new DaoAuthenticationProvider();  
		 provider.setUserDetailsService(userDetailsService);  
		 if (passwordEncoder != null) {  
			provider.setPasswordEncoder(passwordEncoder);  
		 }  
		 if (passwordManager != null) {  
			provider.setUserDetailsPasswordService(passwordManager);  
		 }  
		 provider.afterPropertiesSet();  
		 auth.authenticationProvider(provider);  
	  }  
	
	//...
	
	}  
  
}
```
- `new DaoAuthenticationProvider()` : `DaoAuthenticationProvider` 객체 생성
## 결과
```java
package org.springframework.security.authentication;  
  
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {  

	//...
	
	@Override  
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {  
		Class<? extends Authentication> toTest = authentication.getClass();  
		AuthenticationException lastException = null;  
		AuthenticationException parentException = null;  
		Authentication result = null;  
		Authentication parentResult = null;  
		int currentPosition = 0;  
		int size = this.providers.size();  
		for (AuthenticationProvider provider : getProviders()) {  
		 if (!provider.supports(toTest)) {  
			continue;  
		 }  
		 if (logger.isTraceEnabled()) {  
			logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",  
				  provider.getClass().getSimpleName(), ++currentPosition, size));  
		 }  
		 try {  
			result = provider.authenticate(authentication);  
			if (result != null) {  
			   copyDetails(authentication, result);  
			   break;            }  
		 }  
		 catch (AccountStatusException | InternalAuthenticationServiceException ex) {  
			prepareException(ex, authentication);  
			// SEC-546: Avoid polling additional providers if auth failure is due to  
			// invalid account status            throw ex;  
		 }  
		 catch (AuthenticationException ex) {  
			lastException = ex;  
		 }  
		}  
		if (result == null && this.parent != null) {  
		 // Allow the parent to try.  
		 try {  
			parentResult = this.parent.authenticate(authentication);  
			result = parentResult;  
		 }  
		 catch (ProviderNotFoundException ex) {  
			// ignore as we will throw below if no other exception occurred prior to  
			// calling parent and the parent            // may throw ProviderNotFound even though a provider in the child already            // handled the request         }  
		 catch (AuthenticationException ex) {  
			parentException = ex;  
			lastException = ex;  
		 }  
		}  
		if (result != null) {  
		 if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {  
			// Authentication is complete. Remove credentials and other secret data  
			// from authentication            ((CredentialsContainer) result).eraseCredentials();  
		 }  
		 // If the parent AuthenticationManager was attempted and successful then it  
		 // will publish an AuthenticationSuccessEvent         // This check prevents a duplicate AuthenticationSuccessEvent if the parent         // AuthenticationManager already published it         if (parentResult == null) {  
			this.eventPublisher.publishAuthenticationSuccess(result);  
		 }  
		
		 return result;  
		}  
		
		// Parent was null, or didn't authenticate (or throw an exception).  
		if (lastException == null) {  
		 lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",  
			   new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));  
		}  
		// If the parent AuthenticationManager was attempted and failed then it will  
		// publish an AbstractAuthenticationFailureEvent      // This check prevents a duplicate AbstractAuthenticationFailureEvent if the      // parent AuthenticationManager already published it      if (parentException == null) {  
		 prepareException(lastException, authentication);  
		}  
		throw lastException;  
	}  
	
	//...
  
}
```
- provider list
	![](https://i.imgur.com/oLGGep5.png)
