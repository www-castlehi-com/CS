## 개요
- 권한 부여 이벤트 처리를 할 때 발생하는 이벤트 수신
- 이벤 트 수신 시 `ApplicationEventPublisher`나 `AuthorizationEventPublisher`를 사용
- `AuthorizationEventPublisher`의 구현체로 `SpringAuthorizationEventPublisher` 제공
	```java
	package org.springframework.security.authorization;  
	  
	public final class SpringAuthorizationEventPublisher implements AuthorizationEventPublisher {  
	
		//...
		
		@Override  
		public <T> void publishAuthorizationEvent(Supplier<Authentication> authentication, T object,  
			 AuthorizationDecision decision) {  
		  if (decision == null || decision.isGranted()) {  
			 return;  
		  }  
		  AuthorizationDeniedEvent<T> failure = new AuthorizationDeniedEvent<>(authentication, object, decision);  
		  this.eventPublisher.publishEvent(failure);  
		}  
	
	}
	```
## 발행 방법
- `ApplicationEventPublisher.publishEvent(ApplicationEvent)`
- `AuthorizationEventPublsiher.publishAuthorizationEvent(Supplier<Authentication>, T, AuthorizationDecision)`
## 이벤트 수신 방법
```java
@Component
public class AuthorizationEvents {
	@EventListener
	public void onAuthorization(AuthorizationDeniedEvent failure) {...}
	@EventListener
	public void onAuthorization(AuthorizationGrantedEvent success) {...}
}
```
