- 빈 설정 메타 정보 인터페이스
	- `@Bean`, `<bean>`당 각각 하나씩 메타 정보가 생성
- 스프링이 다양한 설정 형식을 지원하도록 도와줌
- **역할과 구현을 개념적으로 나눔**
	- 스프링 컨테이너는 자바 코드인지, XML인지 몰라도 되며 오직 BeanDefinition만 알면 됨
![](https://i.imgur.com/ZtRBzZm.png)
```java
public class AnnotationConfigApplicationContext 
	extends GenericApplicationContext implements AnnotationConfigRegistry {  
    
    private final AnnotatedBeanDefinitionReader reader;

	//...
}
```
```java
public class GenericXmlApplicationContext extends GenericApplicationContext {  
 
	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);

	//...
}
```
- `AnnotationConfigApplcationContext`를 이용할 경우 팩토리 메소드 빈을 이용함
	- `beanDefinitionName = orderService beanDefinition = Root bean: class [null]; factoryBeanName=appConfig; factoryMethodName=orderService; defined in hello.spring_core_principlesbasic.AppConfig`
- `GenericXmlApplicationContext`를 이용할 경우 등록된 빈에 대한 정보가 바로 보임
	- `beanDefinitionName = discountPolicy beanDefinition = Generic bean: class [hello.spring_core_principlesbasic.discount.RateDiscountPolicy]; defined in class path resource [appConfig.xml]`