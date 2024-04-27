## 개요
- 인증이 성공하면 `AuthenticationSuccessEvent`, 실패하면 `AuthenticationFailureEvent` 발생
- 이벤트를 수신할 때 `ApplicationEventPublisher`나 `AuthenticationEventPublisher` 사용
- `AuthenticatinoEventPublisher`의 구현체로 `DefaultAuthenticationEventPublisher` 제공
## 이벤트 발행 방법
- `ApplicationEventPublisher.publishEvent(ApplicationEvent)`
- `AuthenticationEventPublisher.publishAuthenticationSuccess(Authentication)`
- `AuthenticationEventPublisher.publishAuthenticationFailure(AuthenticationException, Authentication)`
## 이벤트 수신 방법
```java
@Component
public class AuthenticationEvents {
	@EventListener
	public void onSuccess(AuthenticationSuccessEvent success) {...}
	@EventListener
	public void onFaliure(AbstractAuthenticationFailureEvent failures) {...}
}
```
## 이벤트 종류
### 상위 이벤트
- `AbstractAuthenticationEvent`
- `AbstractAuthenticationFailureEvent`
### 성공 이벤트
- `AuthenticationSuccessEvent`
- `InteractiveAuthenticationSuccessEvent`
### 실패 이벤트
- `AuthenticationFailureBadCredentialsEvent`
- `AuthenticationFailureCredentialsExpiredEvent`
- `AuthenticationFailureDisabledEvent`
- `AuthenticationFailureExpiredEvent`
- `AuthenticationFailureLockedEvent`
- `AuthenticationFailureProviderNotFoundEvent`
- `AuthenticationFailureProxyUntrustedEvent`
- `AuthenticationFailureServiceExceptionEvent`