- 특정 구현체 X
- Bean Validation 2.0 (JSR-380) 기술 표준
- 검증 애노테이션과 여러 인터페이스 모음
  ![](https://i.imgur.com/2XEsfZA.png)
> 일반적으로 사용하는 구현체는 **하이버네이트 Validator**
> ![](https://i.imgur.com/LchdNfa.png)
> (ORM 관련 X)
# Bean Validator
- 스프링 부트가 `spring-boot-starter-validation` 라이브러리를 이용하여 자동으로 Bean Validator를 인지하고 스프링에 통합
- `LocalValidatorFactoryBean`을 글로벌 Validator로 등록
	- `@NotNull` 같은 애노테이션을 보고 검증 수행
	- 글로벌 Validator가 적용되어 있어 `@Valid`, `@Validated` 만 적용해도 검증 가능
	- 검증 오류가 발생할 시, `FieldError`, `ObjectError`를 생성해서 `BindingResult`에 주입
# 검증 순서
1. `@ModelAttribute` 각각의 필드에 타입 변환 시도
	- 실패할 경우 `typeMismatch`로 `FieldError` 추가
2. Validator 적용
# 한계
등록과 수정에서 검증 요건이 다를 때 조건의 충돌이 발생할 수 있음
## 방법
### 1️⃣ BeanValidation의 groups 기능 사용
#### 1. 수정, 저장용 groups 생성
```java
public interface SaveCheck { }

public interface UpdateCheck { }
```
#### 2. Validated value 옵션에 적용
```java
public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item) {}

public String editV2(@Validated(value = UpdateCheck.class) @ModelAttribute Item item) { }
```

> `@Valid`에는 groups를 적용하는 기능 X
### 2️⃣ Item을 직접 사용하지 않고, 폼 전송을 위한 별도의 모델 객체 생성
#### 순서
**HTML Form -> DTO -> Controller -> Item 생성 -> Repository**
- 검증 중복 X
- 보통 데이터 등록과 수정시 다루는 데이터가 다르기 때문에 분리 가능
#### `@ModelAttribute`vs `@RequestBody`
**`@ModelAttribute`**
- 필드 단위로 세밀하게 적용
- 특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리되며, Validator를 사용한 검증도 적용 가능
**`@RequestBody`
- `HttpMessageConverter`는 전체 객체 단위로 적용
- 메시지 컨버터의 작동이 성공해서 객체를 만들어야 `@Valid`, `@Validated`적용
- 바인딩에 실패할 경우 컨트롤러 호출 X