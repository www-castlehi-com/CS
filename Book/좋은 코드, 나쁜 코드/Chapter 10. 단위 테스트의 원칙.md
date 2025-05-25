##  10.1 단위 테스트 기초
### 테스트 중인 코드
- 실제 코드
- 테스트의 대상이 되는 코드
### 테스트 코드
- 단위 테스트를 구성하는 코드
- 테스트 중인 코드와 일대일 매핑
### 테스트 케이스
- 특정 동작이나 시나리오 테스트
#### 1️⃣ 준비 (given)
- 테스트할 특정 동작 호출 전 설정 수행
- 일부 테스트 값 정의, 의존성 설정, 테스트 대상이 되는 클래스의 인스턴스를 올바르게 설정하고 생성하는 것 등등
### 2️⃣ 실행 (when)
- 테스트 중인 동작을 실제로 호출하는 코드
### 3️⃣ 단언 (then)
- 실제로 올바른 일이 발생했는지 확인
## 10.2 좋은 단위 테스트는 어떻게 작성할 수 있는가?
### 10.2.1 훼손의 정확한 감지
#### 코드에 대한 초기 신뢰
- 새로운 코드나 코드 변경 사항과 함께 작성 시 코드가 코드 베이스로 병합되기 전에 실수를 발견하고 수정 가능
#### 미래의 훼손 방지
- 모든 올바른 동작을 테스트를 통해 확인
- 코드 변경으로 인해 잘 돌아가던 기능이 작동하지 않는지 **회귀 테스트** 진행
### 10.2.2 세부 구현 사항에 독립적
- **기능 변화**, **리팩터링**으로 코드 변화
- 기능 변화의 경우 테스트를 수정할 것이라고 기대하고 예상하지만 리팩터링의 경우 구현 세부 사항만 변경되며 결과는 변경되지 않음
- 테스트는 구현 세부 사항에 의존하지 않아야 두 가지 코드 변화에 신뢰적
### 10.2.3 잘 설명되는 실패
- 각 테스트 케이스에 대해 서술적인 이름 사용
### 10.2.4 이해 가능한 테스트 코드
- 한 번에 너무 많은 것을 테스트하지 않아야 함
- 너무 많은 공유 테스트를 설정하지 않아야 함
- 테스트 코드를 사용 설명서로 사용할 수 있어야 함
### 10.2.5 쉽고 빠른 실행
## 10.3 퍼블릭 API에 집중하되 중요한 동작은 무시하지 말라
- 퍼블릭  API에 초점을 맞추면 세부 사항이 아닌 코드 사용자가 궁극적으로 신경 쓸 동작에 집중할 수밖에 없게 됨
- 세부 사항은 목적을 이루기 위한 수단일 뿐
### 10.3.1 중요한 동작이 퍼블릭 API 외부에 있을 수 있다
- 테스트 대상 코드는 수많은 다른 코드에 의존하는 경우가 많음
- 의존하는 코드로부터 외부 입력이 제공되거나 테스트 대상 코드가 의존하는 코드에 부수 효과를 일으킬 수 있음
	- **서버와 상호작용하는 코드**
		- 서버로부터 필요한 값을 받을 수 있도록 서버를 설정하고 시뮬레이션하는 코드
		- 서버를 얼마나 자주 호출하는지, 요청이 유효한 형식인지 등
	- **데이터베이스에 값을 저장하거나 읽는 코드**
		- 데이터베이스에 저장된 여러 다른 값으로 코드를 테스트
		- 데이터베이스에 어떤 값을 저장하는지 확인 등
## 10.4 테스트 더블
- 의존성을 실제로 사용하는 것에 대한 대안
- 목, 스텁, 페이크
### 10.4.1 테스트 더블을 사용하는 이유
#### 1️⃣ 테스트 단순화
- 많은 매개변수를 지정해야 하거나, 하위 의존성을 많이 설정해야 하는 경우 존재
- 테스트 코드에 설정을 위한 코드가 많아지고, 수많은 구현 세부 정보와 밀접하게 연결될 수 있음
- 테스트 더블과만 상호작용 시 설정과 부수 효과 검증을 할 수 있으며 테스트 속도가 빨라짐
- 테스트 더블을 설정 시 의존성을 실제로 사용하는 것보다 더 복잡할 때가 있을 수 있음
#### 2️⃣ 테스트로부터 외부 세계 보호
- 테스트할 때 실제 의존성을 사용할 시 부수 효과가 실제로 일어날 수 있음
- 사용자는 이상하고 혼란스러운 값을 볼 수 있으며, 모니터링 및 로깅에 영향을 미칠 수 있음
#### 3️⃣ 외부로부터 테스트 보호
- 실제 세계에서 달라지는 값을 사용할 시 테스트의 결과가 매번 달라질 수 있어 **플래키 테스트**가 될 수 있으며 신뢰성이 떨어짐
### 10.4.2 목
- 클래스나 인터페이스를 시뮬레이션하는 데 멤버 함수에 대한 호출을 기록하는 것 이외에 어떠한 일도 수행하지 않음
- 함수 호출 시 인수에 제공되는 값 기록
```java
class PaymentManager {
	//...
	
	PaymentResult settleInvoice(BankAccount customerBankAccount, Invoice invoice) {
		customerBankAccount.debit(invoice.getBalance());
		return PaymentResult.paid(invoice.getId());
	}
}
```
- `PaymentManager` 테스트는 `BankAccount` 종속성과 상호작용하여 원하는 부수 효과가 발생하는지 확인 필요
- `BankAccount` 는 외부 환경과 상호작용하기 때문에 테스트로부터 외부 세계를 보호할 수 없음
```java
void testSettleInvoice_accountDebited() {
	BankAccount mockAccount = createMock(BankAccount);
	MonetaryAmount invoiceBalance = new MonetaryAmount(5.0, Currency.USD);
	Invoice invoice = new Invoice(invoiceBalance, "test-id");
	PaymentManager paymentManager = new PaymentManager();

	paymentManager.settleInvoice(mockAccount, invoice);

	verifyThat(mockAccount.debit)
		.wasCalledOnce()
		.withArguments(invoiceBalance);
}
```
- `BanckAccount` 종속성에 대한 목 객체를 만들고 부수 효과를 일으키는 함수가 올바른 인수로 호출되는지 확인
- 테스트로부터 외부 세계를 보호하는데 성공하지만, 테스트가 비현실적이고 중요한 버그를 잡지 못할 위험이 있음
### 10.4.3 스텁
- 함수가 호출되면 미리 정해 놓은 값을 반환함으로써 함수를 시뮬레이션
- 테스트 대상 코드가 의존하는 코드로부터 어떤 값을 받아야 하는 경우 의존성을 시뮬레이션하는 데 유용
```java
class PaymentManager {

	//...

	PaymentResult settleInvoice(BankAccount customerBankAccount, Invoice invoice) {
		if (customerBankAccount.getBalance().isLessThan(invoice.getBalance())) {
			return paymentReulst.insufficientFunds(invoice.getId());
		}
		customerBankAccount.debit(invoice.getBalance());
		return PaymentResult.paid(invoice.getId());
	}
}
```
- 인출 전 계좌 잔액이 충분한지 확인하도록 기능 추가
- 실제 의존성 객체와 상호작용할 경우 잔액이 변경되기 때문에 외부로부터 테스트를 보호할 수 없음
```java
void testSettleInvoice_insufficientFundsCorrectResultReturned() {
	MonetaryAmount invoiceBalance = new MonetaryAmount(10.0, Currency.USD);
	Invoice invoice = new Invoice(invoiceBalance, "test-id");
	BankAccount mockAccount = createMock(BankAccount);
	when(mockAccount.getBalance())
		.thenReturn(new MonetaryAmount(9.99, Currency.USD));
	PaymentManager paymentManager = new PaymentManager();

	PaymentResult result = paymentManager.settleInvoice(mockAccount, invoice);
	assertThat(result.getStatus()).isEqualTo(INSUFFICIENT_FUNDS);
}
```
- 스텁이 미리 정한 값을 반환하도록 설정
### 10.4.4 목과 스텁은 문제가 될 수 있다
#### 목과 스텁은 실제적이지 않은 테스트를 만들 수 있다
```java
interface BankAccount {
	/**
	* @throws ArgumentException 0보다 적은 금액으로 호출되는 경우
	*/
	void debit(MonetaryAmount amount);

	/**
	* @throws ArgumentException 0보다 적은 금액으로 호출되는 경우
	*/
	void credit(MonetaryAmount amount);
}
```
- 테스트에서는 목을 사용하기 때문에 0보다 적은 금액으로 호출 시 버그가 드러나지 않음
```java
interface BankAccount {

	/**
	* @return 가장 가까운 10의 배수로 반내림한 계좌의 잔액
	* 예를 들어 실제 잔액이 19달러라면 이 함수는 10달러를 반환한다.
	* 이것은 보안을 위한 것인데 정확한 잔액은 보안 확인을 위한 질문으로 은행이 사용하기 때문이다.
	*/
	MonetaryAmount getBalance();
}
```
- 스텁을 구성할 때 의존성 코드가 실제로 반환하는 값인지에 대해서는 아무런 검증을 하지 않음
#### 목과 스텁을 사용하면 테스트가 구현 세부 정보에 유착될 수 있다
```java
PaymentResult settleInvoice(...) {

	//...
	MonetaryAmount balance = invoice.getBalance();
	if (balance.isPositive()) {
		customerBankAccount.debit(balance);
	} else {
		customerBankAccount.credit(balance.absoluteAmount());
	}
}
```
```java
void testSettleInvoice_positiveInvoiceBalance() {

	//...

	verifyThat(mockAccount.debit)
		.wasCalledOnce()
		.withArguments(invoiceBalance);
}

void testSettleInvoice_negativeInvoiceBalance() {

	//...

	verifyThat(mockAccount.credit)
		.wasCalledOnce()
		.withArguments(invoiceBalance.absoluteAmount());
}
```
- 어떤 함수가 호출되는지는 목적을 위한 수단이기 때문에 구현 세부 사항에 속함
- 실제 관심을 갖는 동작을 직접 테스트하지 않음
```java
interface BankAccount {

	//...

	/**
	* 지정된 금액을 계좌로 송금한다.
	* 금액이 0보다 적으면 계좌로부터 인출하는 효과를 갖는다.
	*/
	void transfer(MonetaryAmount amount);
}
```
- 리팩터링 시 구현 세부 사항만 변경되며 동작은 변경되지 않음
- 테스트가 구현 세부 사항을 확인하면 리팩터링 시 테스트 케이스가 실패됨
### 10.4.5 페이크
- 클래스의 대체 구현체
- 실제 의존성에 대한 코드 계약이 변경되면 페이크의 코드 계약도 동일하게 변경되어야 함
```java
class FakeBankAccount implements BankAccount {

	private MonetaryAmount balance;

	FakeBankAccount(MonetaryAmount startingBalance) {
		this.balance = startingBalance;
	}

	override void debit(MonetaryAmount amount) {
		if (amount.isNegative()) {
			throw new ArgumentException("액수는 0보다 적을 수 없음");
		}
		balance = balance.subtract(amount);
	}

	override void credit(MonetaryAmount amount) {
		if (amount.isNegative()) {
			throw new ArgumentException("액수는 0보다 적을 수 없음");
		}
		balance = balance.add(amount);
	}

	oveerride void transfer(MonetaryAmount amount) {
		balance.add(amount);
	}

	override MonetaryAmount getBalance() {
		return roundDownToNearest10(balance);
	}

	MonetaryAmount getActualBalance() {
		return balance;
	}
}
```
- 페이크로 인해 보다 실질적인 테스트가 이루어질 수 있음
- 페이크를 사용하면 구현 세부정보로부터 테스트를 분리할 수 있음
## 10.5 테스트 철학으로부터 신중하게 선택하라