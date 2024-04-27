# 동기 통합
## 개요
- Spring MVC 인자에 대한 `Authentication.getPrincipal()`을 자동으로 해결할 수 있는 `AuthenticationPrincipalArgumentResolver` 제공
- `@AuthenticationPrincipal`을 메서드 인수에 선언하게 되면 Spring Security와 독립적으로 사용 가능
## 비교
**Spring Security**
```java
@RequestMapping("/user")
public void findUser() {
	Authentication authentication = SecurityContextHolder.getContextHolderStrategy().getContext().getAuthentication();
	CustomerUser custom = (CustomUser) authentication == null ? null : authentication.getPrincipal();
}
```
**MVC 통합**
```java
@RequestMapping("/user")
public Customer findUser(@AuthhenticationPrincipal CustomUser customUser) {...}
```
**표현식**
```java
@RequestMapping("/user")
public Customer findUser(@AuthenticationPrincipal(expression = "customer") Customer customer) {...}
```
-  사용자 세부 정보가 Principal 내부에 중첩된 객체일 경우 사용
**메타 주석**
```java
@Target({ElementType.PARAMETER, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@AuthenticationPrincipal
public @interface CurrentUser {}

@RequestMapping("/user")
public void user(@CurrentUser CustomUser customUser) {...}
```
# 비동기 통합
## 개요
- Spring MVC Controller에서 Callable을 실행하는 비동기 스레드에 SecurityContext를 자동으로 설정
- `WebAsyncManager`와 통합하여 SecurityContextHolder에서 사용 가능한 SecurityContext를  Callable에서 접근 가능하도록 해줌

> ⚠️ `@Async`나 다른 비동기 기술은 [[Spring Security]]와 통합되어 있지 않아 비동기 통합이 되지 않음
> `SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);`를 통해 Async로 통합
## 구조
### WebAsyncManagerIntegrationFilter
- SecurityContext와 `WebAsyncManager` 사이의 통합 제공
- `WebAsyncManager` 생성, `SecurityContextCallablepProcessingInterceptor`를 이에 등록
### WebAsyncManager
- 스레드 풀의 비동기 스레드 생성
- Callable를 받아 실행시킴
- 등록된 `SecurityContextCallableProcessingInterceptor`를 통해 현재 스레드가 보유하고 있는 SecurityContext 객체를 비동기 스레드의 ThreadLocal에 저장
## 예시
![](https://i.imgur.com/dXZ9mvf.png)
## 흐름
![](https://i.imgur.com/9VE9ToT.png)
1. Client의 요청
2. 부모 스레드인 Main Thread가 요청 받음
3. `WebAsyncManagerIntegrationFilter`가 `WebAsyncManager`, `SecurityContextCallableProcessingInterceptor` 생성
	```java
	package org.springframework.security.web.context.request.async;  
	  
	public final class WebAsyncManagerIntegrationFilter extends OncePerRequestFilter {  
	
		//...
	  
		@Override  
		protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)  
			 throws ServletException, IOException {  
		  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);  
		  SecurityContextCallableProcessingInterceptor securityProcessingInterceptor = (SecurityContextCallableProcessingInterceptor) asyncManager  
			 .getCallableInterceptor(CALLABLE_INTERCEPTOR_KEY);  
		  if (securityProcessingInterceptor == null) {  
			 SecurityContextCallableProcessingInterceptor interceptor = new SecurityContextCallableProcessingInterceptor();  
			 interceptor.setSecurityContextHolderStrategy(this.securityContextHolderStrategy);  
			 asyncManager.registerCallableInterceptor(CALLABLE_INTERCEPTOR_KEY, interceptor);  
		  }  
		  filterChain.doFilter(request, response);  
		}
	     
		//...
		
}
	```
4.  `WebAsyncManager`가 부모 스레드의 security context를 `SecurityContextCallableProcessingInterceptor`에 저장
	```java
	package org.springframework.security.web.context.request.async;  
	  
	public final class SecurityContextCallableProcessingInterceptor implements CallableProcessingInterceptor {  
	
		//...
		
		@Override  
		public <T> void beforeConcurrentHandling(NativeWebRequest request, Callable<T> task) {  
		  if (this.securityContext == null) {  
			 setSecurityContext(this.securityContextHolderStrategy.getContext());  
		  }  
		}  
		
		//...
	  
	}
	```
5. `WebAsyncManager`가 `ThreadPoolExecutor`를 이용하여 비동기 스레드 생성
6. 비동기 스레드의 ThreadLocal에 `SecurityContextCallableProcessingInterceptor`를 이용하여 부모 스레드의 security context를 가져와 저장
	```java
	package org.springframework.web.context.request.async;  
	  
	public final class WebAsyncManager {  
	
		//...
	
		public void startCallableProcessing(final WebAsyncTask<?> webAsyncTask, Object... processingContext)  
	      throws Exception {  
	  
		   Assert.notNull(webAsyncTask, "WebAsyncTask must not be null");  
		   Assert.state(this.asyncWebRequest != null, "AsyncWebRequest must not be null");  
		  
		   if (!this.state.compareAndSet(State.NOT_STARTED, State.ASYNC_PROCESSING)) {  
		      throw new IllegalStateException(  
		            "Unexpected call to startCallableProcessing: [" + this.state.get() + "]");  
		   }  
		  
		   Long timeout = webAsyncTask.getTimeout();  
		   if (timeout != null) {  
		      this.asyncWebRequest.setTimeout(timeout);  
		   }  
		  
		   AsyncTaskExecutor executor = webAsyncTask.getExecutor();  
		   if (executor != null) {  
		      this.taskExecutor = executor;  
		   }  
		  
		   List<CallableProcessingInterceptor> interceptors = new ArrayList<>();  
		   interceptors.add(webAsyncTask.getInterceptor());  
		   interceptors.addAll(this.callableInterceptors.values());  
		   interceptors.add(timeoutCallableInterceptor);  
		  
		   final Callable<?> callable = webAsyncTask.getCallable();  
		   final CallableInterceptorChain interceptorChain = new CallableInterceptorChain(interceptors);  
		  
		   this.asyncWebRequest.addTimeoutHandler(() -> {  
		      if (logger.isDebugEnabled()) {  
		         logger.debug("Servlet container timeout notification for " + formatUri(this.asyncWebRequest));  
		      }  
		      Object result = interceptorChain.triggerAfterTimeout(this.asyncWebRequest, callable);  
		      if (result != CallableProcessingInterceptor.RESULT_NONE) {  
		         setConcurrentResultAndDispatch(result);  
		      }  
		   });  
		  
		   this.asyncWebRequest.addErrorHandler(ex -> {  
		      if (logger.isDebugEnabled()) {  
		         logger.debug("Servlet container error notification for " + formatUri(this.asyncWebRequest) + ": " + ex);  
		      }  
		      Object result = interceptorChain.triggerAfterError(this.asyncWebRequest, callable, ex);  
		      result = (result != CallableProcessingInterceptor.RESULT_NONE ? result : ex);  
		      setConcurrentResultAndDispatch(result);  
		   });  
		  
		   this.asyncWebRequest.addCompletionHandler(() ->  
		         interceptorChain.triggerAfterCompletion(this.asyncWebRequest, callable));  
		  
		   interceptorChain.applyBeforeConcurrentHandling(this.asyncWebRequest, callable);  
		   startAsyncProcessing(processingContext);  
		   try {  
		      Future<?> future = this.taskExecutor.submit(() -> {  
		         Object result = null;  
		         try {  
		            interceptorChain.applyPreProcess(this.asyncWebRequest, callable);  
		            result = callable.call();  
		         }  
		         catch (Throwable ex) {  
		            result = ex;  
		         }  
		         finally {  
		            result = interceptorChain.applyPostProcess(this.asyncWebRequest, callable, result);  
		         }  
		         setConcurrentResultAndDispatch(result);  
		      });  
		      interceptorChain.setTaskFuture(future);  
		   }  
		   catch (Throwable ex) {  
		      Object result = interceptorChain.applyPostProcess(this.asyncWebRequest, callable, ex);  
		      setConcurrentResultAndDispatch(result);  
		   }  
		}
	
		//...
	}
	```
	- `callable.call()`
		![](https://i.imgur.com/AoNDC4M.png)
		```java
		@RestController  
		@RequiredArgsConstructor  
		public class IndexController {  
		
			//...
		  
		    @GetMapping("/callable")  
		    public Callable<Authentication> call(){  
		        //...
		  
		        return new Callable<Authentication>() {  
		            @Override  
		            public Authentication call() throws Exception {  
		                SecurityContext securityContext = SecurityContextHolder.getContextHolderStrategy().getContext();  
		                System.out.println("securityContext = " + securityContext);  
		                System.out.println("Child Thread : " + Thread.currentThread().getName());  
		                return securityContext.getAuthentication();  
		            }  
		        };  
		    }  
		
			//...
			
		}
		```
7. `Callable`을 수행하는 비동기 스레드는 자신의 ThreadLocal에 저장되어 있는 security context 참조 가능