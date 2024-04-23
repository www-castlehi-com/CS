## 개요
- [[동시 세션 제어]]
- 각 요청에 대해 SessionRegistry에서 SessionInformation을 검색하고 세션이 만료된 경우 로그아웃 -> 세션 무효화
- 각 요청에 대해 `SessionRegistry.refreshLastRequest(String)`을 호출하여 세션들이 항상 **마지막 업데이트** 날짜/시간을 가짐
## 흐름
![](https://i.imgur.com/UqHf3WC.png)
1. Client의 인증 요청 & 성공 -> 세션 추가
2. [[SessionManagementFilter]]가 개수가 초과된 사용자의 세션을 만료 설정(`session.expireNow()`)함
3. Client의 재 접속
4. ConcurrentSessionFilter가 사용자의 요청을 처리
	> 💡 ConcurrentSessionFilter는 모든 요청에 대해 처리함
	
	- 세션이 만료되었다면, [[Logout]] 처리를 하고 `SessionInformationExpiredStrategy`를 통해 만료 세션 처리
	- 만료되지 않았다면, 세션의 날짜/시간을 최신으로 업데이트
## 시퀀스 다이어그램
![](https://i.imgur.com/eEJ2RzD.png)
## 코드
```java
package org.springframework.security.web.authentication.session;  
  
public class ConcurrentSessionControlAuthenticationStrategy  
      implements MessageSourceAware, SessionAuthenticationStrategy {  

	//...
	
	@Override
	public void onAuthentication(Authentication authentication, HttpServletRequest request,  
		 HttpServletResponse response) {  
	  int allowedSessions = getMaximumSessionsForThisUser(authentication);  
	  if (allowedSessions == -1) {  
		 // We permit unlimited logins  
		 return;  
	  }  
	  List<SessionInformation> sessions = this.sessionRegistry.getAllSessions(authentication.getPrincipal(), false);  
	  int sessionCount = sessions.size();  
	  if (sessionCount < allowedSessions) {  
		 // They haven't got too many login sessions running at present  
		 return;  
	  }  
	  if (sessionCount == allowedSessions) {  
		 HttpSession session = request.getSession(false);  
		 if (session != null) {  
			// Only permit it though if this request is associated with one of the  
			// already registered sessions            for (SessionInformation si : sessions) {  
			   if (si.getSessionId().equals(session.getId())) {  
				  return;  
			   }  
			}  
		 }  
		 // If the session is null, a new one will be created by the parent class,  
		 // exceeding the allowed number      }  
	  allowableSessionsExceeded(sessions, allowedSessions, this.sessionRegistry);  
	}  
	
	//...
  
}
```
- `getMaximumSessionsForThisUser(...)` : 세션 최대 허용 개수 로드
- `sessionCount < allowedSessions` : 현재 세션 개수와 최대 허용 개수 비교
- `sessionCount == allowedSessions & getSessionid().equals(...)` : 세션 개수가 최대이고 등록된 세션 아이디와 동일하다면 문제 X
	- `allowableSessionsExceeded(...)` : 다른 세션 아이디일 경우
		```java
		protected void allowableSessionsExceeded(List<SessionInformation> sessions, int allowableSessions,  
      SessionRegistry registry) throws SessionAuthenticationException {  
		   if (this.exceptionIfMaximumExceeded || (sessions == null)) {  
		      throw new SessionAuthenticationException(  
		            this.messages.getMessage("ConcurrentSessionControlAuthenticationStrategy.exceededAllowed",  
		                  new Object[] { allowableSessions }, "Maximum sessions of {0} for this principal exceeded"));  
		   }  
		   // Determine least recently used sessions, and mark them for invalidation  
		   sessions.sort(Comparator.comparing(SessionInformation::getLastRequest));  
		   int maximumSessionsExceededBy = sessions.size() - allowableSessions + 1;  
		   List<SessionInformation> sessionsToBeExpired = sessions.subList(0, maximumSessionsExceededBy);  
		   for (SessionInformation session : sessionsToBeExpired) {  
		      session.expireNow();  
		   }  
		}
		```
		- `exceptionIfMaximumExceeded` : 사용자 인증 차단을 할 것인지 사용자 세션 강제 만료를 할 것인지 여부
			- 사용자 인증을 차단할 경우 `SessionAuthenticaitonException` 발생
			- 사용자 세션 강제 만료를 할 경우 `session.expireNow()`로 이전 세션을 만료시킴
```java
package org.springframework.security.web.session;  
  
public class ConcurrentSessionFilter extends GenericFilterBean {  
  
	//...
  
	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)  
		 throws IOException, ServletException {  
	  HttpSession session = request.getSession(false);  
	  if (session != null) {  
		 SessionInformation info = this.sessionRegistry.getSessionInformation(session.getId());  
		 if (info != null) {  
			if (info.isExpired()) {  
			   // Expired - abort processing  
			   this.logger.debug(LogMessage  
				  .of(() -> "Requested session ID " + request.getRequestedSessionId() + " has expired."));  
			   doLogout(request, response);  
			   this.sessionInformationExpiredStrategy  
				  .onExpiredSessionDetected(new SessionInformationExpiredEvent(info, request, response));  
			   return;            }  
			// Non-expired - update last request date/time  
			this.sessionRegistry.refreshLastRequest(info.getSessionId());  
		 }  
	  }  
	  chain.doFilter(request, response);  
	}  

	//...

}
```
- `isExpired()` : 세션이 만료될 경우
	- `doLogout(...)` : 로그아웃 처리
- `onExpiredSessionDetected(...)`
	```java
	private static final class ResponseBodySessionInformationExpiredStrategy  
      implements SessionInformationExpiredStrategy {  
  
	   @Override  
	   public void onExpiredSessionDetected(SessionInformationExpiredEvent event) throws IOException {  
	      HttpServletResponse response = event.getResponse();  
	      response.getWriter()  
	         .print("This session has been expired (possibly due to multiple concurrent "  
	               + "logins being attempted as the same user).");  
	      response.flushBuffer();  
	   }  
	  
	}
	```