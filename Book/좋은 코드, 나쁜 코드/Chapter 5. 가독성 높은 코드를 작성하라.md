### 5.1.3 해결책 : 서술적인 이름 짓기
- 변수, 함수 및 클래스를 별도로 설명할 필요 없는 이름 짓기
## 5.2 주석문의 적절한 사용
1. 코드가 **무엇**을 하는지 설명
2. 코드가 **왜** 그 일을 하는지 설명
3. 사용 지침 등 기타 정보 제공
### 5.2.1 중복된 주석문은 유해할 수 있다
```java
String generateId(String firstName, String lastName) {
	// "{이름}.{성}"의 형태로 ID를 생성한다.
	return firstName + "." + lastName;
}
```
- 코드 자체로 설명이 되기 때문에 주석문은 쓸모가 없음
- 코드를 변경할 경우 주석문을 수정해야 함
### 5.2.2 주석문으로 가독성 높은 코드를 대체할 수 없다
#### 주석문이 있는 이해하기 어려운 코드
```java
String generateId(String[] data) {
	//data[0]는 유저의 이름이고 data[1]은 성이다.
	//"{이름}.{성}"의 형태로 ID를 생성한다.
	return data[0] + "." + data[1];
}
```
#### 가독성이 더 좋아진 코드
```java
String generateId(String[] data) {
	return firstName(data) + "." + lastName(data);
}

String firstName(String[] data) {
	return data[0];
}

String lastName(String[] data) {
	return data[1];
}
```
### 5.2.3 주석문은 코드의 이유를 설명하는 데 유용하다
- 제품 또는 비즈니스 의사 결정
- 이상하고 명확하지 않은 버그에 대한 해결책
- 의존하는 코드의 예상을 벗어나는 동작에 대처

```java
Class User {
	private final Int username;
	private final String firstName;
	private final String lastName;
	private final Version signupVersion;

	String getUserId() {
		if (signupVersion.isOlderThan("2.0")) {
			// (v2.0 이전에 등록한) 레거시 유저는 이름으로 ID가 부여된다.
			// 자세한 내용은 #4218 이슈를 보라.
			return fisrtName.toLowerCase() + "." + lastName.toLowerCase();
		}
		// (v2.0 이후에 등록한) 새 유저는 username으로 ID가 부여된다.
		return username;
	}
}
```
- 코드가 지저분해질 수 있지만 얻는 이점이 더 큼
### 5.2.4 주석문은 유용한 상위 수준의 요약 정보를 제공할 수 있다
- 클래스가 수행하는 작업 및 다른 개발자가 알고 있어야 할 중요한 세부 사항을 개괄적으로 설명하는 문서
- 함수에 대한 입력 매개변수 또는 기능을 설명하는 문서
- 함수의 반환값이 무엇을 나타내는지 설명하는 문서
```java
/**
* 스트리밍 서비스의 유저에 대한 자세한 사항을 갖는다.
*
* 이 클래스는 데이터베이스에 직접 연결하지 않는다. 대신 메모리에 저장된 값으로 생성된다.
* 따라서 이 클래스가 생성된 이후에 데이터베이스에서 이뤄진 변경 사항으 반영하지 않을 수 있다.
*/
Class User {
}
```
### 5.3.2 더 많은 줄이 필요하더라도 가독성 높은 코드를 작성하라
#### 간결하지만 이해하기 어려운 코드
```java
Boolean isIdValid(UInt16 id) {
	return countSetBits(id & 0x7FFF) % 2 == ((id & 0x8000) >> 15);
}
```
#### 코드의 양은 더 많지만 가독성은 높은 코드
```java
Boolean isIdValid(UInt16 id) {
	return extractEncodedParity(id) == calculateParity(getIdValue(id));
}

private const UInt16 PARITY_BIT_INDEX = 15;
private const UInt16 PARITY_BIT_MASK = (1 << PARITY_BIT_INDEX);
private const UInt16 VALUE_BIT_MASK = ~PARITY_BIT_MASK;

private UInt16 getIdValue(UInt16 id) {
	return id & VALUE_BIT_MASK;
}

private UInt16 extractEncodedParity(UInt16 id) {
	return (id & PARITY_BIT_MASK) >> PARITY_BIT_INDEX;
}

// 패리티 비트는 1인 비트의 수가 짝수이면 0이고
// 홀수이면 1이다.
private UInt16 calculateParity(UInt16 value) {
	return countSetBits(value) % 2;
}
```
### 5.4.2 스타일 가이드를 채택하고 따르라
**린터**
-  스타일 가이드를 따르지 않은 코드를 찾아 알려주는 도구
### 5.5.2 중첩을 최소화하기 위한 구조 변경
#### 깊이 중첩된 if문
```java
Address? getOwnersAddress(Vehicle vehicle) {
	if (vehicle.hasBeenScraped()) {
		return SCRAPYARD_ADDRESS;
	} else {
		Purchase? mostRecentPurchase = vehicle.getMostRecentPurchase();
		if (mostRecentPurchase == null) {
			return SHOWROOM_ADDRESS;
		} else {
			Buyer? buyer = mostRecentPurchase.getBuyer();
			if (buyer != null) {
				return buyer.getAddress();
			}
		}
	}
	return null;
}
```
#### 중첩이 최소화된 코드
```java
Address? getOwnersAddress(Vehicle vehicle) {
	if (vehicle.hasBeenScraped()) {
		return SCRAPYARD_ADDRESS;
	}
	Purchase? mostRecentPurchase = vehicle.getMostRecentPurchase();
	if (mostRecentPurchase == null) {
		return SHOWROOM_ADDRESS;
	}
	Buyer? buyer = mostRecentPurchase.getBuyer();
	if (buyer != null) {
		return buyer.getAddress();
	}
	return null;
}
```
- 더 작은 함수로 분리하며 중첩 최소화
### 5.6.3 서술적 유형 사용
#### **함수 호출에서의 서술적 유형**
```java
class MessagePriority {
	MessagePriority(Int priority) {}
}

enum RetryPolicy {
	ALLOW_RETRY,
	DISALLOW_RETRY
}

void sendMessage(
	String message,
	MessagePriority priority,
	RetryPolicy retryPolicy
) {}

sendMessage("hello", new MessagePriority(1), RetryPolicy.ALLOW_RETRY);
```
> 라인의 가독성을 높여주는 위와 같은 방법이 없을 경우, 주석문을 사용하는 것보다 그대로 두는 것이 좋음
> 주석문을 사용할 경우, 해당 값이 바뀔 때 주석도 같이 바뀌어야 하기 때문
### 5.7.2 잘 명명된 상수를 사용하라
#### 명명 상수를 사용하지 않은 경우
```java
class Vehicle {
	Double getMassUsTon() {}
	Double getSpeedMph() {}
	Double getKineticEnergyJ() {
		return 0.5 *
			getMassUsTon() * 907.1847 *
			Math.pow(getSpeedMph() * 0.44704, 2);
	}
}
```
#### 잘 명명된 상수
```java
class Vehicle {
	private const Double KILOGRAMS_PER_US_TON = 907.1847;
	private const Double METERS_PER_SECOND_PER_MPH = 0.44704;

	Double getKineticEnergyJ() {
		return 0.5 *
			getMassUsTon() * KILOGRAMS_PER_US_TON *
			Math.pow(getSpeedMph() * METERS_PER_SECOND_PER_MPH, 2);
	}
}
```
### 5.7.3 잘 명명된 함수를 사용하라
#### 공급자 함수
```java
class Vehicle {
	//...

	private static Double kilogramsPerUsTon() {
		return 907.1847;
	}

	private static Double metersPerSecondPerMph() {
		return 0.44704;
	}
}
```
- 상수를 사용하는 것과 동일
#### 헬퍼 함수
```java
class Vehicle {
	//...

	private static Double usTonsToKilograms(Double usTons) {
		return usTons * 907.1847;
	}

	private static Double mphToMetersPerSecond(Double mph) {
		return mph * 0.44704;
	}
}
```
- 수량 변환 시 사용되는 계수가 있다는 사실은 함수 호출자는 모르는 구현 세부사항이므로 하위 문제로 만들어 헬퍼 함수가  처리
- 다른 개발자들이 재사용할 가능성이 있다면 별도의 유틸리티 클래스에 두는 것이 좋음
### 5.8.1 익명 함수는 간단한 로직에 좋다
### 5.8.2 익명 함수는 가독성이 떨어질 수 있다
```java
List<UInt16> getValidIds(List<UInt16> ids) {
	return ids
		.filter(id -> id != 0)
		.filter(id -> countSetBits(id & 0x7FFF) % 2 == ((id & 0x8000) >> 15));
}
```
명명 함수를 사용하지 않을 경우, 함수 이름으로 유추할 수 없기 때문에 복잡한 로직에서는 가독성이 떨어질 수 있음
### 5.8.3 대신 명명함수를 사용하라
- 구현 세부사항을 별도의 명명함수로 빼내는 것이 좋음
```java
List<UInt16> getValidIds(List<UInt16> idx) {
	return idx
		.filter(id -> id != 0)
		.filter(isParityBitCorrect);
}

private boolean isParityBitCorrect(UInt16 id) {}
```
익명 함수가 길 경우, 작은 단위의 명명 함수로 작성
### 5.9.3 작업에 가장 적합한 도구를 사용하라
#### 스트림을 사용하여 맵에서 값을 얻기
```java
Optional<String> value = map
	.entrySet()
	.stream()
	.filter(entry -> entry.getKey().equals(key))
	.map(Entry::getValue)
	.findFirst();
```
#### 더 적합한 도구
```java
String value = map.get(key);
```
`Stream`과 같은 새 기능은 코드를 개선할 수 있지만, 작업에 적합한 도구를 사용하는 것이 효과적임
