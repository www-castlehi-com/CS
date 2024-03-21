# ModelMapper란?
- 객체 간의 매핑을 쉽고 자동으로 해주는 라이브러리
	> ex) DTO <-> Entity
- **리플렉션**을 사용하여 Runtime에 객체의 클래스 정보를 읽어 메타데이터를 얻음
- `TyepMap`과 `Converter` 기반으로 매핑 전략을 수립하여 소스로부터 목적지 객체로 매핑
## Mapping
### Source 객체
```java
class Order {
	Customer customer; 
	Address billingAddress; 
}

class Customer {
	Name name;
 } 
 
 class Name {
	String firstName;
	String lastName; 
}

class Address {
	String street;
	String city;
}
```
### Destination 객체
```java
class OrderDTO {
	String customerFirstName;
	String customerLastName;
	String billingStreet;
	String billingCity;
}
```
### 사용
```java
ModelMapper modelMapper = new ModelMapper();
OrderDTO orderDTO = modelMapper.map(order, OrderDTO.class);
```
### 검증
```java
assertEquals(order.getCustomer().getName().getFirstName(), orderDTO.getCustomerFirstName()); assertEquals(order.getCustomer().getName().getLastName(), orderDTO.getCustomerLastName()); assertEquals(order.getBillingAddress().getStreet(), orderDTO.getBillingStreet()); assertEquals(order.getBillingAddress().getCity(), orderDTO.getBillingCity());
```
### 원리
`map()` 호출 시, matching strategy & configuration에 따라 암시적으로 일치하는 속성 결정
#### matching strategy
1️⃣ **Standard**
- 소스 속성을 대상 속성과 지능적으로 매칭
- 모든 대상 속성이 매칭되어야 함
- 모든 소스 속성 이름에 하나 이상의 토큰이 일치해야 함
- 토큰은 어떤 순서로든 일치되어야 함
2️⃣ **Loose**
- 마지막 대상 속성만 일치하도록 요구
- 토큰은 어떤 순서로든 일치되어야 함
- 마지막 대상 속성 이름에 모든 토큰이 일치
- 마지막 소스 속성 이름에 하나 이상의 토큰이 일치
- 계층 구조가 매우 다른 모델에서 사용하기 이상적
3️⃣ **Strict**
- 불일치, 모호함 없이 완벽
- 토큰은 엄격한 순서로 일치
- 모든 대상 속성 이름의 토큰이 일치
- `TypeMap`을 검사하지 않고도 모호함, 예상치 못한 매핑이 발생하지 않도록 하려는 경우 이상적
## map()
```java
public <D> D map(Object source, Class<D> destinationType) {  
  Assert.notNull(source, "source");  
  Assert.notNull(destinationType, "destinationType");  
  return mapInternal(source, null, destinationType, null);  
}
```
**mapInternal**
```java
private <D> D mapInternal(Object source, D destination, Type destinationType, String typeMapName) {  
  if (destination != null)  
    destinationType = Types.<D>deProxy(destination.getClass());  
  return engine.<Object, D>map(source, Types.<Object>deProxy(source.getClass()), destination,  
      TypeToken.<D>of(destinationType), typeMapName);  
}
```
**deProxy**
```java
public static <T> Class<T> deProxy(Class<?> type) {  
  // Ignore JDK proxies  
  if (type.isInterface())  
    return (Class<T>) type;  
  
  if (isProxied(type)) {  
    final Class<?> superclass = type.getSuperclass();  
    if (!superclass.equals(Object.class) && !superclass.equals(Proxy.class))  
      return (Class<T>) superclass;  
    else {  
      Class<?>[] interfaces = type.getInterfaces();  
      if (interfaces.length > 0)  
        return (Class<T>) interfaces[0];  
    }  
  }  
  
  return (Class<T>) type;  
}
```
1. type이 interface인지 여부 확인 -> 리플렉션 API
2. **isProxied** : 프록시 객체인지 확인
	```java
	public static boolean isProxied(Class<?> type) {  
	  if (type.getName().contains("$ByteBuddy$"))  
	    return true;  
	  if (type.getName().contains("$$EnhancerBy"))  
	    return true;  
	  if (type.getName().contains("$HibernateProxy$"))  
	    return true;  
	  if (type.getName().contains("$MockitoMock$"))  
	    return true;  
	  if (Proxy.isProxyClass(type))  
	    return true;  
	  return isProxiedByJavassist(type);  
	}
	```
	1) **ByteBuddy** : 직접 프록시 객체를 만들 때 사용
	2) **EnhancerBy(CGLIB)** : 싱글톤 빈을 생성하거나 트랜잭션 관리를 적용할 때
	3) **Hibernate** : lazy loading
	4) **MokitoMock** : 단위 테스트 중 생성되는 목 객체
	5) **JDK 동적** : 인터페이스 기반 프록시 객체
	6) **Javassist** : 바이트코드 조작 (클래스 파일 조작 후, 메소드 실행 전후에 로깅을 추가하는 프록시 객체)
3. 프록시 객체라면
	1. superClass를 가져옴
	2. Object.class나 Proxy.class가 아닐 경우, 해당 슈퍼클래스를 원본 타입으로 간주하고 반환
	3. Object.class나 proxy.class나, 슈퍼클래스를 찾을 수 없는 경우 첫 번째 인터페이스 반환 -> 인터페이스 기반 프록시 객체 처리
		```java
		public interface PaymentService {
		    void processPayment();
		}
		
		public interface AuditLog {
		    void logTransaction();
		}
		
		public class PaymentServiceImpl implements PaymentService, AuditLog {
		    public void processPayment() {
		        // 결제 처리 로직
		    }
		
		    public void logTransaction() {
		        // 트랜잭션 로깅 로직
		    }
		}
		
		// 프록시 객체 생성 (가정)
		PaymentService proxy = createProxy(new PaymentServiceImpl());
		```
4. 프록시 객체가 아니라면  해당 객체를 원본이라 간주하고 반환

> 💡 프록시 객체의 경우 추가적인 작업을 수행하기 위해 사용되기 때문에 `deProxy` 작업을 함

**engine.map(S source, Class\<S> sourceType, D destination,  TypeToken\<D> destinationTypeToken, String typeMapName)**
```java
public <S, D> D map(S source, Class<S> sourceType, D destination,  
    TypeToken<D> destinationTypeToken, String typeMapName) {  
  MappingContextImpl<S, D> context = new MappingContextImpl<S, D>(source, sourceType,  
      destination, destinationTypeToken.getRawType(), destinationTypeToken.getType(),  
      typeMapName, this);  
  D result = null;  
  
  try {  
    result = map(context);  
  } catch (ConfigurationException e) {  
    throw e;  
  } catch (ErrorsException e) {  
    throw context.errors.toMappingException();  
  } catch (Throwable t) {  
    context.errors.errorMapping(sourceType, destinationTypeToken.getType(), t);  
  }  
  
  context.errors.throwMappingExceptionIfErrorsExist();  
  return result;  
}
```
1. MappinContext 생성 -> 매핑 과정에서 필요한 소스, 목적지, 타입 정보 등을 저장
2. `map()`으로 실제 매핑 작업 수행
3. 매핑 오류 검사
4. 목적지 객체 반환
**engine.map(MappingContext<S, D> context)**
```java
public <S, D> D map(MappingContext<S, D> context) {  
  MappingContextImpl<S, D> contextImpl = (MappingContextImpl<S, D>) context;  
  Class<D> destinationType = context.getDestinationType();  
  
  // Resolve some circular dependencies  
  if (!Iterables.isIterable(destinationType)) {  
    D circularDest = contextImpl.destinationForSource();  
    if (circularDest != null && circularDest.getClass().isAssignableFrom(contextImpl.getDestinationType()))  
      return circularDest;  
  }  
  
  D destination = null;  
  TypeMap<S, D> typeMap = typeMapStore.get(context.getSourceType(), context.getDestinationType(),  
      context.getTypeMapName());  
  if (typeMap != null) {  
    destination = typeMap(contextImpl, typeMap);  
  } else {  
    Converter<S, D> converter = converterFor(context);  
    if (converter != null && (context.getDestination() == null || context.getParent() != null))  
      destination = convert(context, converter);  
    else if (!Primitives.isPrimitive(context.getSourceType()) && !Primitives.isPrimitive(context.getDestinationType())) {  
      // Call getOrCreate in case TypeMap was created concurrently  
      typeMap = typeMapStore.getOrCreate(context.getSource(), context.getSourceType(),  
          context.getDestinationType(), context.getTypeMapName(), this);  
      destination = typeMap(contextImpl, typeMap);  
    } else if (context.getDestinationType().isAssignableFrom(context.getSourceType()))  
      destination = (D) context.getSource();  
  }  
  
  contextImpl.setDestination(destination, true);  
  return destination;  
}
```
1. iterable, 순환 참조 확인
2. 소스 타입과 목적지 타입 간의 매핑 규칙을 정의한 `TypeMap` 조회 -> 있을 경우 이를 사용하여 매핑
3. `TypeMap`이 없을 경우 `Converter`를 찾아 사용
4. `Converter`를 등록하지 않았을 경우, 소스와 목적지 타입이 모두 Primitive가 아닐 때 `TypeMap`을 찾거나 새로 생성
5. 소스와 목적지 타입이 동일하거나 소스 타입이 목적지 타입의 하위 타입일 경우 소스 객체를 목적지 객체로 직접 할당

> 💡 Entity <-> DTO일 경우, name이 같기 때문에 ModelMapper가 제공하는 `TypeMap`을 쓰는 경우가 많음