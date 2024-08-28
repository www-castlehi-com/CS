## 6.1 매직값을 반환하지 말아야 한다
### 6.1.2 해결책 : 널, 옵셔널 또는 오류를 반환하라
### 6.1.3 때때로 매직값이 우연히 발생할 수 있다
```java
Int minValue(List<Int> values) {
	Int minValue = Int.MAX_VALUE;
	for (Int value in values) {
		minValue = Math.min(value, minValue);
	}
	return minValue;
}
```
values 리스트가 비어있을 경우 우연히 `Int.MAX_VALUE`가 반환된다
```java
Int? minValue(List<Int> values) {
	if (values.isEmpty()) {
		return null;
	}
	Int minValue = Int.MAX_VALUE;
	for (Int value in values) {
		minValue = Math.min(value, minValue);
	}
	return minValue;
}
```
values 리스트가 비어 있으면 널 값, 옵셔널을 반환하거나 오류를 전달한다
## 6.3 예상치 못한 부수 효과를 피하라
#### 부수 효과
- 어떤 함수의 호출이 함수 외부에 초래한 상태 변화
- 함수가 반환하는 값 외에 다른 효과가 있을 경우

> **유형**
> - 사용자에게 출력 표시
> - 파일이나 데이터베이스에 무언가를 저장
> - 다른 시스템을 호출하여 네트워크 트래픽 발생
> - 캐시 업데이트 혹은 무효화
### 6.3.1 분명하고 의도적인 부수 효과는 괜찮다
```java
class UserDisplay {
	private final Canvas canvas;

	void displayErrorMessage(String message) {
		canvas.drawText(message, Color.RED);
	}
}
```
 오류 메시지로 캔버스를 업데이트하는 것은 호출하는 쪽에서 예상하는 바
### 6.3.2 예기치 않은 부수 효과는 문제가 될 수 있다
```java
class UserDisplay {
	private final Canvas canvas;

	Color getPixel(int x, int y) {
		canvas.redraw();
		PixelData data = canvas.getPixel(x, y);
		return new Color (
			data.getRed(),
			data.getGreen(),
			data.getBlue()
		);
	}
}
```
#### 부수 효과는 비용이 많이 들 수 있다
- `canvas.redraw()` 호출 작업은 잠재적으로 비용이 많이 들 수 있다
#### 호출한 쪽의 가정을 깨뜨리기
- 사용자의 화면이 깜빡거리는 등의 문제가 발생할 수 있지만 `getPixel()` 함수의 이름이 이를 나타내지 않는다
#### 다중 스레드 코드의 버그
- 스레드 1이 스레드 2가 캔버스를 그리고 있는 도중에 캔버스로부터 픽셀 데이터를 읽을 수도 있다
### 6.3.3 해결책: 부수 효과를 피하거나 그 사실을 분명하게 하라
1. `canvas.redraw()`를 호출하는 것이 정말로 필요한지 확인
2. 필요하다면, `getPixel()`의 이름을 `redrawAndGetPixel()`로 변경하여 부수 효과가 발생할 것임을 분명히 하기
## 6.4 입력 매개변수를 수정하는 것에 주의하라
### 6.4.1 입력 매개변수를 수정하면 버그를 초래할 수 있다
```java
List<Invoice> getBillableInvoices(
	Map<User, Invoice> userInvoices,
	Set<User> userWithFreeTrial
) {
	userInvoices.removeAll(usersWithFreeTrial);
	return userInvoices.values();
}

void processOrders(OrderBatch orderBatch) {
	Map<User, Invoice> userInvoices = orderBatch.getUserInvoices();
	Set<User> usersWithFreeTrial = orderBatch.getFreeTrialUsers();

	sendInvoices(getBillableInvoices(userInvoices, usersWithFreeTrial));
	enableOrderedServices(userInvoices);
}
```
`getBillableInvoices()` 함수에서 무료 평가판을 사용할 수 있는 유저를 삭제해 `userInvoices`를 변경했다
`processOrders()` 함수에서 `getBillableInvoices()`, `enableOrderServices()`를 호출하기 때문에 무료 평가판을 사용하는 유저는 `enableOrderServices()`를 사용할 수 없게 된다
### 6.4.2 해결책: 변경하기 전에 복사하라
```java
List<Invoice> getBillableInvoices(
	Map<User, Invoice> userInvoices,
	Set<User> usersWithFreeTrial
) {
	return userInvoices
		.entries()
		.filter(entry -> 
			!usersWithFreeTrial.contains(entry.getKey()))
		.map(entry -> entry.getValue());
}
```
 `filter()`는 조건에 맞는 값을 새로운 리스트에 복사한다
## 6.5 오해를 일으키는 함수는 작성하지 말라
### 6.5.1 중요한 입력이 누락되었을 때 아무것도 하지 않으면 놀랄 수 있다
```java
class UserDisplay {
	private final LocalizedMessages messages;

	void displayLegalDisclaimer(String? legalText) {
		if (legalText == null) {
			return ;
		}
		displayOverlay(
			title: messages.getLegalDsiclaimerTitle(),
			message: legalText,
			textColor: Color.RED
		);
	}
}

class LocalizedMessages {

	String getLegalDisclaimerTitle();
}
```
호출하는 쪽에서 널을 확인해야 하는 것을 피하기 위해 널을 받아들이지만, 아무것도 하지 않는 것은 오해의 소지가 있고 예상을 벗어나는 코드를 초래할 수 있다
### 6.5.2 해결책: 중요한 입력은 필수 항목으로 만들라
```java
calss SignupFlow {
	private final UserDisplay userDisplay;
	private final LocalizedMessages messages;

	@CheckReturnValue
	Boolean ensureLegalCompliance() {
		String? signupDisclaimer = messages.getSignupDisclaimer();
		if (signupDisclaimer == null) {
			return false;
		}
		userDisplay.displayLegalDisclaimer(signupDisclaimer);
		return true;
	}
}
```
필수값이 null일 경우 `false`를 반환하고 `@CheckReturnValue`를 이용해 이 반환값이 무시되지 않도록 한다
## 6.6 미래를 대비한 열거형 처리
### 6.6.1 미래에 추가될 수 있는 열것값을 암묵적으로 처리하는 것은 문제가 될 수 있다
```java
enum PredictedOutcome {
	COMPANY_WILL_GO_BUST,
	COMPANY_WILL_MAKE_A_PROFIT
}

Boolean isOutcomeSafe(PredictedOutcome prediction) {
	if (prediction == PredictedOutcome.COMPANY_WILL_GO_BUST) {
		return false;
	}
	return true;
}
```
`if-else` 문으로 `COMPANY_WILL_GO_BUST` 이외의 값들은 암시적으로 안전한 것으로 처리된다
`WORLD_WILL_END` 항목이 추가되었을 경우, true를 반환해버린다
### 6.6.2 해결책: 모든 경우를 처리하는 스위치 문을 사용하라
```java
enum PredictedOutcome {
	COMPANY_WILL_GO_BUST,
	COMPANY_WILL_MAKE_A_PROFIT,
}

Boolean isOutcomeSafe(PredictedOutcome prediction) {
	switch (prediction) {
		case COMPANY_WILL_GO_BUST:
			return false;
		case COMPANY_WILL_MAKE_A_PROFIT:
			return true;
	}
	throw new UncheckedException(
		"Unhandled prediction: " + prediction;
	)
}
```