## 개념
- 신원을 확인하는 방법
- 사용자 이름, 비밀번호를 입력받아 신원 확인(**인증**) 후, 권한 부여(**인가**)
- 토큰 개념이며 인증 후, [[SecurityContext]]에 저장되어 전역적으로 참조
## 구조
![](https://i.imgur.com/SqoVWoy.png)
- `getPrincipal()` 
	- 인증 주체
	- 인증 요청 시, 사용자 이름을 의미
	- 인증 후, [[UserDetails]] 타입의 객체
- `getCredentials()`
	- 비밀번호
- `getAuthorities()`
	- 인증 주체(`principal`)에게 부여된 권한
- `getDetails()`
	- 인증 요청에 대한 추가적인 세부 사항
	- IP 주소, 인증서 일련 번호 등
- `isAuthenticated()`
	- 인증 상태를 반환
- `setAuthenticated(boolean)`
	- 인증 상태를 설정
## 흐름
![](https://i.imgur.com/WERo9rf.png)
**인증 처리 전**
1. Client의 로그인 시도 -> username, password 입력
2. `AuthenticationFilter`가 `Authentication`토큰 객체를 생성 후, [[AuthenticationManager]]에 전달하여 인증
**인증 처리 후**
3.  `UserDetails` + authorities를 이용해 `Authentication` 객체를 생성
	- principal :  타입이 Object이기 때문에 `UserDetails`가 아닐 수 있음
	- credentials : 인증 후이므로 정보 삭제
	- authorities : `GrantAuthority` 타입
4. `AuthenticationManager`가 `Authentication` 객체를 `AuthenticationFilter`까지 전달
5. `Authentication` 객체를 `SecurityContextHolder`를 이용해 `SecurityContext`에 저장
## 코드
```java
package org.springframework.security.web.authentication;  
  
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean  
      implements ApplicationEventPublisherAware, MessageSourceAware {  

	//...
  
	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)  
		 throws IOException, ServletException {  
	  if (!requiresAuthentication(request, response)) {  
		 chain.doFilter(request, response);  
		 return;      }  
	  try {  
		 Authentication authenticationResult = attemptAuthentication(request, response);  
		 if (authenticationResult == null) {  
			// return immediately as subclass has indicated that it hasn't completed  
			return;  
		 }  
		 this.sessionStrategy.onAuthentication(authenticationResult, request, response);  
		 // Authentication success  
		 if (this.continueChainBeforeSuccessfulAuthentication) {  
			chain.doFilter(request, response);  
		 }  
		 successfulAuthentication(request, response, chain, authenticationResult);  
	  }  
	  catch (InternalAuthenticationServiceException failed) {  
		 this.logger.error("An internal error occurred while trying to authenticate the user.", failed);  
		 unsuccessfulAuthentication(request, response, failed);  
	  }  
	  catch (AuthenticationException ex) {  
		 // Authentication failed  
		 unsuccessfulAuthentication(request, response, ex);  
	  }  
	}  

	//...

	protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,  
	  Authentication authResult) throws IOException, ServletException {  
	SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();  
	context.setAuthentication(authResult);  
	this.securityContextHolderStrategy.setContext(context);  
	this.securityContextRepository.saveContext(context, request, response);  
	if (this.logger.isDebugEnabled()) {  
	  this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));  
	}  
	this.rememberMeServices.loginSuccess(request, response, authResult);  
	if (this.eventPublisher != null) {  
	  this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));  
	}  
	this.successHandler.onAuthenticationSuccess(request, response, authResult);  
	}

	//...
	
}
```
- `attemptAuthentication` : `Authenticaiton` 객체 생성
	```java
	package org.springframework.security.web.authentication;  
	  
	public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {  
	  
		//...
	
		@Override  
		public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)  
			 throws AuthenticationException {  
		  if (this.postOnly && !request.getMethod().equals("POST")) {  
			 throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());  
		  }  
		  String username = obtainUsername(request);  
		  username = (username != null) ? username.trim() : "";  
		  String password = obtainPassword(request);  
		  password = (password != null) ? password : "";  
		  UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,  
				password);  
		  // Allow subclasses to set the "details" property  
		  setDetails(request, authRequest);  
		  return this.getAuthenticationManager().authenticate(authRequest);  
		}  
	
		//...
		
	}
	```
	- username, password를 이용해 `Authentication` 토큰 객체 생성
	- `getAuthenticationManager().authenticate()` : `Authentication`객체를 `authenticaitonManager`에게 전달
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
		- `provider.authenticate()` : [[AuthenticationProvider]]에게 인증처리 위임
			```java
			package org.springframework.security.authentication.dao;  
			  
			public abstract class AbstractUserDetailsAuthenticationProvider  
			      implements AuthenticationProvider, InitializingBean, MessageSourceAware {  
			
				//...
				
				@Override  
				public Authentication authenticate(Authentication authentication) throws AuthenticationException {  
				  Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,  
						() -> this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.onlySupports",  
							  "Only UsernamePasswordAuthenticationToken is supported"));  
				  String username = determineUsername(authentication);  
				  boolean cacheWasUsed = true;  
				  UserDetails user = this.userCache.getUserFromCache(username);  
				  if (user == null) {  
					 cacheWasUsed = false;  
					 try {  
						user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);  
					 }  
					 catch (UsernameNotFoundException ex) {  
						this.logger.debug("Failed to find user '" + username + "'");  
						if (!this.hideUserNotFoundExceptions) {  
						   throw ex;  
						}  
						throw new BadCredentialsException(this.messages  
						   .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));  
					 }  
					 Assert.notNull(user, "retrieveUser returned null - a violation of the interface contract");  
				  }  
				  try {  
					 this.preAuthenticationChecks.check(user);  
					 additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);  
				  }  
				  catch (AuthenticationException ex) {  
					 if (!cacheWasUsed) {  
						throw ex;  
					 }  
					 // There was a problem, so try again after checking  
					 // we're using latest data (i.e. not from the cache)         cacheWasUsed = false;  
					 user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);  
					 this.preAuthenticationChecks.check(user);  
					 additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);  
				  }  
				  this.postAuthenticationChecks.check(user);  
				  if (!cacheWasUsed) {  
					 this.userCache.putUserInCache(user);  
				  }  
				  Object principalToReturn = user;  
				  if (this.forcePrincipalAsString) {  
					 principalToReturn = user.getUsername();  
				  }  
				  return createSuccessAuthentication(principalToReturn, authentication, user);  
				}  
				
				//...
			  
			}
			```
			- `retrieveUser()` : 사용자 정보를 [[UserDetailsService]]로부터 추출
				```java
				@Override  
				protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)  
				      throws AuthenticationException {  
				   prepareTimingAttackProtection();  
				   try {  
				      UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);  
				      if (loadedUser == null) {  
				         throw new InternalAuthenticationServiceException(  
				               "UserDetailsService returned null, which is an interface contract violation");  
				      }  
				      return loadedUser;  
				   }  
				   catch (UsernameNotFoundException ex) {  
				      mitigateAgainstTimingAttack(authentication);  
				      throw ex;  
				   }  
				   catch (InternalAuthenticationServiceException ex) {  
				      throw ex;  
				   }  
				   catch (Exception ex) {  
				      throw new InternalAuthenticationServiceException(ex.getMessage(), ex);  
				   }  
				}
				```
				- `getUserDetailsService().loadUserByUsername()` : username을 이용해 `UserDetails` 객체 추출
			- `additionalAuthenticationChecks()` : 패스워드 검증
				 ```java
				@Override  
				@SuppressWarnings("deprecation")  
				protected void additionalAuthenticationChecks(UserDetails userDetails,  
				      UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {  
				   if (authentication.getCredentials() == null) {  
				      this.logger.debug("Failed to authenticate since no credentials provided");  
				      throw new BadCredentialsException(this.messages  
				         .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));  
				   }  
				   String presentedPassword = authentication.getCredentials().toString();  
				   if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {  
				      this.logger.debug("Failed to authenticate since password does not match stored value");  
				      throw new BadCredentialsException(this.messages  
				         .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));  
				   }  
				} 
				```
				- `passwordEncoder.matches()` : 시스템이 가진 패스워드와 사용자 요청 패스워드가 같은지 확인
			- `createSuccessAuthentication()` : 최종 결과물로 `Authentication` 객체 생성
				```java
				@Override  
				protected Authentication createSuccessAuthentication(Object principal, Authentication authentication,  
				      UserDetails user) {  
				   boolean upgradeEncoding = this.userDetailsPasswordService != null  
				         && this.passwordEncoder.upgradeEncoding(user.getPassword());  
				   if (upgradeEncoding) {  
				      String presentedPassword = authentication.getCredentials().toString();  
				      String newPassword = this.passwordEncoder.encode(presentedPassword);  
				      user = this.userDetailsPasswordService.updatePassword(user, newPassword);  
				   }  
				   return super.createSuccessAuthentication(principal, authentication, user);  
				}
				```
- `successfulAuthentication` : security context내에 `Authentication` 객체 저장
	- `securityContextHolderStrategy.setContext()` : security context에 객체 저장
	- `securityContextHolderStrategy.saveContext()` : 세션에 security context 저장
