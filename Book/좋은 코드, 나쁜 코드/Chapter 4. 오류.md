### 4.1.1 복구 가능한 오류
- 네트워크 오류
- 중요하지 않은 작업 오류
### 4.1.2 복구할 수 없는 오류
- 코드와 함께 추가되어야 하는 리소스가 없음
- 잘못된 입력 인수로 호출
- 일부 필요한 상태를 사전에 초기화하지 않음
### 4.2.1 신속하게 실패하라
- 문제의 근원에 최대한 가까이서 실패한 것으로 보여줌
### 4.2.2 요란하게 실패하라
- 오류가 발생했는데도 불구하고 아무도 모르는 상황을 막아야함
- 예외 발생, 로그 기록 등
### 4.3.2 명시적 방법 : 검사 예외
- 호출하는 쪽에서 예외를 인지하도록 함
- 예외를 포착하지 않거나, 예외가 발생할 수 있음을 선언하지 않을 경우 컴파일이 되지 않음
#### 오류 전달
```java
class NegativeNumberException extends Exception {
	private final Double erroneousNumber;

	NegativeNumberException(Double erroneousNumber) {
		this.erroneousNumber = erroneousNumber;
	}

	Double getErroneousNumber() {
		return erroneousNumber;
	}
}

Double getSquareRoot(Double value) 
		throws NegatvieNumberException {
	if (value < 0.0) {
		throw new NegativeNumberException(value);
	}
	return Math.sqrt(value);
}
```
#### 예외 처리
```java
void displaySquareRoot() {
	Double value = ui.getInputNumber();
	try {
		ui.setOutput("Square root is : " + getSqaureRoot(value));
	} catch (NegativeNumberException e) {
		ui.setError("Can't get square root of negative number : " + e.getErroneousNumber());
	}
}
```
- 상위 함수가 예외를 처리하지 않는 경우 예외가 발생할 수 있음을 선언하고 더 상위 객체가 처리해야함
### 4.3.3 암시적 방법 : 비검사 예외
#### 오류 전달
```java
/**
* @throws NegativeNumberException 값이 음수일 경우
*/
Double getSquareRoot(Double value) {
	if (value < 0.0) {
		throw new NegativeNumberException(value);
	}
	return Math.sqrt(value);
}
```
- 비검사 예외의 경우 문서화가 권장
#### 예외 처리
```java
void displaySquareRoot() {
	Double value = ui.getInputNumber();
	try {
		ui.setOutput("Square root is : " + getSqaureRoot(value));
	} catch (NegativeNumberException e) {
		ui.setError("Can't get square root of negative number : " + e.getErroneousNumber());
	}
}
```
- 상위 함수가 예외를 확인하고 처리하지 않아도 됨
### 4.3.4 명시적 방법 : 널값이 가능한 반환 유형
### 4.3.5 명시적 방법 : 리절트 반환 유형
- 널값이나 옵셔널 타입을 반환할 때 오류 정보를 전달할 수 없음
- 호출자에게 값을 얻을 수 없음 & 값을 얻을 수 없는 이유까지 알림
- `getValue` 호출 전, `hasError`를 호출 해야 함
```java
class Result<V, E> {
	private final Optional<V> value;
	private final Optional<E> error;

	private Result(Optional<V> value, Optional<E> error) {
		this.value = value;
		this.error = error;
	}

	static Result<V, E> ofValue(V value) {
		return new Result(Optional.of(value), Optional.empty());
	}

	static Result<V, E> ofError(E error) {
		return new Result(Optinal.empty(), Optional.of(error));
	}

	Boolean hasError() {
		return error.isPresent();
	}

	V getValue() {
		return value.get();
	}

	E getError() {
		return error.get();
	}
}
```
### 4.3.6 명시적 방법 : 아웃컴 반환 유형
- 함수가 수행한 동작의 결과를 나타내는 값을 반환함
#### 오류 전달
```java
Boolean sendMessage(Channel channel, String message) {
	if (channel.isOpen()) {
		channel.send(message);
		return true;
	}
	return false;
}
```
#### 무시되지 않도록 보장
```java
@CheckReturnValue
Boolean sendMessage(Channel channel, String message) {
	//...
}
```
### 4.3.7 암시적 방법 : 프로미스 또는 퓨처
### 4.3.8 암시적 방법 : 매직값 반환
## 4.4 복구할 수 없는 오류 전달
- 암시적인 기술을 사용함 -> 오류 전달 외에 할 수 있는 것이 없기 때문
## 4.5 호출하는 쪽에서 복구하기를 원할 수도 있는 오류의 전달
- 명시적, 암시적인 기술을 사용할 때 구현 세부 사항은 노출할 필요 없음
### 4.5.1 비검사 예외를 사용해야 한다는 주장
- 대부분의 오류 처리는 상위 계층에서 이루어지기 때문에 사이의 중간 계층에서는 오류에 대해서 알지 않아도 됨
- 클래스를 교환할 경우 예외가 발생할 가능성이 있는지 알 수 없음
- 명시적 기법을 사용할 경우, 모든 예외를 찾다가 다음과 같이 변질될 수 있음
	```java
	Boolean isDataFileValid(byte[] fileContents) {
		try {
			DataFile.parse(fileContents);
			return true;
		} catch (Exception e) {
			return false;
		}
	}
	```
### 4.5.2 명시적 기법을 사용해야 한다는 주장
- 모든 오류를 매끄럽게 처리할  수 있는 단일 계층을 가질 수 있음
- 실수로 오류를 무시할 수 없음

> **표준 예외 유형 고수**
> - `ArgumentException`이나 `StateException`과 같은 표준 예외 유형 혹은 그것들의 서브 클래스 사용
> - 서로 다른 오류 시나리오를 구분하는 기능을 제한할 수도 있음