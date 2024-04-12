# 초기화
- 인증, 인가와 관련된 작업
- 종합적으로 [[Security Builder & Security Configurer]]가 처리
## 자동 설정
- 서버 기동 시, 스프링 시큐리티 초기화 & 보안 설정이 자동으로 이루어짐
  
1. 모든 요청에 대해 인증 여부를 검증하고 인증 승인 시, 자원 접근
2. 인증 방식으로 **[[form Login]] 방식**과 **httpBasic 로그인 방식** 제공
3. 인증 시도할 수 있는 로그인 페이지 자동 생성 & 렌더링
4. 한 개의 계정 기본적으로 제공
	 - SecurityProperties 설정 클래스에서 생성
	 - username : user
	 - password : 랜덤 문자열
### 원리

```java
@Configuration(proxyBeanMethods = false)  
@ConditionalOnWebApplication(type = Type.SERVLET)  
class SpringBootWebSecurityConfiguration {  
	
	@Configuration(proxyBeanMethods = false)  
	@ConditionalOnDefaultWebSecurity  
	static class SecurityFilterChainConfiguration {  
	
	  @Bean  
	  @Order(SecurityProperties.BASIC_AUTH_ORDER)  
	  SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {  
		 http.authorizeHttpRequests((requests) -> requests.anyRequest().authenticated());  
		 http.formLogin(withDefaults());  
		 http.httpBasic(withDefaults());  
		 return http.build();  
	  }  
	}

	//...
}
```
- 초기화 과정을 수행
- `@ConditionalOnDefaultWebApplication`에 의해 조건에 의해서만 `SecurityFilterChain` 생성
	```java
	@Target({ ElementType.TYPE, ElementType.METHOD })  
	@Retention(RetentionPolicy.RUNTIME)  
	@Documented  
	@Conditional(DefaultWebSecurityCondition.class)  
	public @interface ConditionalOnDefaultWebSecurity {  
	  
	}
	```

	```java
	class DefaultWebSecurityCondition extends AllNestedConditions {  
  
	   DefaultWebSecurityCondition() {  
	      super(ConfigurationPhase.REGISTER_BEAN);  
	   }  
	  
	   @ConditionalOnClass({ SecurityFilterChain.class, HttpSecurity.class })  
	   static class Classes {  
	  
	   }  
	   @ConditionalOnMissingBean({ SecurityFilterChain.class })  
	   static class Beans {  
	  
	   }  
	}
	```
	- 두 조건을 모두 만족해야만 기본 보안 작동이 이루어짐
	
	1. `SecurityFilterChain`, `HttpSecurity` 클래스가 클래스패스에 존재하는가?
		- 참 => SpringSecurity 의존성을 추가했기 때문
	2. `SecurityFilterChain`이 빈으로 등록되어 있지 않을 경우 참
		- 참 => 생성한 적이 없기 때문
- 모든 요청 (`anyRequest()`)은 인증을 받아야만 (`authenticated()`)  서버에 접근이 허락된다 (``authorizeRequests()`)
- 인증을 받지 못했을 경우 인증 방식으로 폼 로그인 방식(`formLogin()`), httpBasic 방식(`httpBasic()`)을 제공한다
## 계정
```java
package org.springframework.boot.autoconfigure.security;  
  
import java.util.ArrayList;  
import java.util.EnumSet;  
import java.util.List;  
import java.util.Set;  
import java.util.UUID;  
  
import org.springframework.boot.context.properties.ConfigurationProperties;  
import org.springframework.boot.web.servlet.DispatcherType;  
import org.springframework.boot.web.servlet.filter.OrderedFilter;  
import org.springframework.core.Ordered;  
import org.springframework.util.StringUtils;  
  
/**  
 * Configuration properties for Spring Security. * * @author Dave Syer  
 * @author Andy Wilkinson  
 * @author Madhura Bhave  
 * @since 1.0.0  
 */@ConfigurationProperties(prefix = "spring.security")  
public class SecurityProperties {  
  
   //...
  
   public static class User {  
  
      /**  
       * Default user name.       */      
       private String name = "user";  
  
      /**  
       * Password for the default user name.       */      
       private String password = UUID.randomUUID().toString();  
  
      /**  
       * Granted roles for the default user name.       */      
       private List<String> roles = new ArrayList<>();  
  
      private boolean passwordGenerated = true;  
  
      public String getName() {  
         return this.name;  
      }  
  
      public void setName(String name) {  
         this.name = name;  
      }  
  
      public String getPassword() {  
         return this.password;  
      }  
  
      public void setPassword(String password) {  
         if (!StringUtils.hasLength(password)) {  
            return;  
         }  
         this.passwordGenerated = false;  
         this.password = password;  
      }  
  
      public List<String> getRoles() {  
         return this.roles;  
      }  
  
      public void setRoles(List<String> roles) {  
         this.roles = new ArrayList<>(roles);  
      }  
  
      public boolean isPasswordGenerated() {  
         return this.passwordGenerated;  
      }  
   }  
}
```
- `private String name = "user";` : 유저의 이름 'user'
- `private String password = UUID.randomUUID().toString();` : 패스워드 'UUID' (랜덤값)
### 계정 저장
- 일반적으로 웹에서 username, password를 db에 저장한 후 인증
- Spring Security은 db에 연결은 되어있지 않지만 메모리에 저장하는 과정 수반

```java
@Bean  
public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,  
      ObjectProvider<PasswordEncoder> passwordEncoder) {  
   SecurityProperties.User user = properties.getUser();  
   List<String> roles = user.getRoles();  
   return new InMemoryUserDetailsManager(User.withUsername(user.getName())  
      .password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))  
      .roles(StringUtils.toStringArray(roles))  
      .build());  
}
```
- 유저 계정을 관리하는 매니저 클래스 빈으로 생성
- `SeuciryProperties.User.getXX()`를 통해 정보를 가져와서 등록
### 문제점
- 계정 추가 문제
- 계정의 권한을 다르게 줄 수 없는 문제
- 세부적인 보안기능 필요
### 예시
**의존성 추가**
```java
dependencies {  
    implementation 'org.springframework.boot:spring-boot-starter-security'  
    implementation 'org.springframework.boot:spring-boot-starter-web'    
}
```
**컨트롤러**
```java
@RestController  
public class IndexController {  
    @GetMapping("/")  
    public String index() {  
        return "index";  
    }  
}
```
**콘솔**
![](https://i.imgur.com/phIlstz.png)
**웹페이지**
![](https://i.imgur.com/QjnLSpM.png)
- Username : user
- Password : 콘솔의 generated security password
**로그인 성공 시**
![](https://i.imgur.com/kXLpUes.png)
