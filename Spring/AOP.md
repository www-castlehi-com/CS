## 요소
### Advisor
- AOP Advice와 Advice 적용 가능성을 결정하는 포인트컷을 가진 기본 인터페이스
### MethodInterceptor
- Advice
- 대상 객체를 호출하기 전, 후에 추가 작업을 수행하기 위한 인터페이스
- 수행 이후 실제 대상 객체의 조인포인트 호출(메서드 호출)을 위해 `Joinpoint.proceed()` 호출
### Pointcut
- Advice가 적용될 메소드나 클래스를 정의하는 것
- Advice가 실행되어야 하는 '적용 지점'이나 '조건' 지정
- ClassFilter나 MethodMatcher를 사용해서 어떤 클래스 및 어떤 메서드에 Advice를 적용할 것인지 결정
## 초기화
1. `AnnotationAwareAspectJAutoProxyCreator`가 현재 어플리케이션 컨텍스트 내의 모든 AspectJ 어노테이션과 스프링 Advisor를 빈으로 등록하여 처리
2. 포인트컷 조건에 해당하는 클래스와 메서드를 찾고(`execution(* io.security.MyService.*(..))`) 대상 클래스의 프록시를 `CGLibAopProxy`가 생성함