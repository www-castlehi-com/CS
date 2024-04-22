## 개념
- 사용자와 관련된 상세 데이터 로드
- 사용자 신원, 권한, 자격 증명 등과 같은 정보 포함
- [[AuthenticationProvider]]가 인터페이스 사용 : 사용자의 존재 유무, 검색 & 인증 과정 수행
## 구조
![](https://i.imgur.com/rB7qCsS.png)
**username**을 통해 사용자 데이터 검색하고, 해당 데이터를 UserDetails 객체로 반환
## 흐름
![](https://i.imgur.com/k0O53rZ.png)
1. AuthenticationProvider가 UserDetailsService에게 username으로 사용자 정보 검색 위임
	- 존재하지 않는다면, `UserNotFoundException` 발생
2. UserDetailsService는 UserRepository를 이용해 DB로부터 사용자 정보를 가져옴
3. 시스템에서 사용하는 유저 엔티티(가정 : UserInfo)로부터 UserDetails로 맵핑
4. 맵핑한 UserDetails 객체를 AuthenticaitonProvider에게 전달

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
- `ProviderManager`에 provider를 설정하지 않을 시 AnonymousProvider, DaoProvider가 존재함
- `provider.authenticate()` : Provider의 인증 처리
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
- `retrieveUser()` : 사용자 정보를 가져오는 로직
```java
package org.springframework.security.authentication.dao;  
  
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {  

	//...
  
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

	//...
	
}
```
- `DaoAuthenticationProvider` : 커스텀한 UserDetailsService 객체를 사용할 provider
	![](https://i.imgur.com/kcc0pmN.png)
- `getUserDetailsService().loadUserByUsername(username)` : UserDetailsService의 `loadUserByUsername()` 메소드를 이용해 사용자 이름으롤부터 [[UserDetails]] 객체 추출