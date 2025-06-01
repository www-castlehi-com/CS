## 11.1 기능뿐만 아니라 동작을 시험하라
- 함수는 여러 개의 동작을 수행할 수 있고, 한 동작이 여 함수에 걸쳐 있을 수 있음
- 함수별로 테스트 케이스를 하나만 작성하면 중요한 동작을 놓칠 수 있음
### 11.1.1 함수당 하나의 테스트 케이스만 있으면 적절하지 않을 때가 많다
#### 구현 코드
```java
class MortgageAssessor {
	private const Double MORTAGE_MULTIPLIER = 10.0;

	MortgageDecision assess(Customer customer) {
		if (!isEligibleForMortgage(customer)) {
			return MortgageDecision.rejected();
		}
		return MortgageDecision.approve(getMaxLoanAmount(customer));
	}

	private static Boolean isEligibleForMortgage(Customer customer) {
		return customer.hasGoodCreditRating() &&
			!customer.hasExistingMortgage() &&
			!customer.isBanned();
	}

	private static MonetaryAmount getMaxLoanAmount(Customer customer) {
		return customer.getIncome()
			.minus(customer.getOutgoings())
			.multiplyBy(MORTGAGE_MULTIPLIER);
	}
}
```
- `assess()` 함수의 동작
	- 신용 등급이 좋고, 기존 대출이 없으며, 대출이 금지되지 않은 고객에게 대출 승인
	- 최대 대출 금액은 고객의 수입에서 지출을 뺀 금액에 10을 곱한 값
#### 테스트 코드
```java
testAssess() {
	Customer customer = new Customer(
		income: new MonetaryAmount(50000, Currency.USD),
		outgoings: new MonetaryAmount(20000, Currency.USD),
		hasGoodCreditRating: true,
		hasExistingMortgage: false,
		isBanned: false
	);
	MortgageAssessor mortgageAssessor = new MortgageAssessor();

	MortgageDecision decision = mortgageAssessor.assess(customer);

	assertThat(decision.isApproved()).isTrue();
	assertThat(decision.getMaxLoanAmount()).isEqualTo(new MonetaryAmount(300000, Currency.USD));
}
```
- 하나의 테스트 케이스만 작성하며 동작이 아니라 기능 테스트에 집중됨
### 11.1.2 해결책: 각 동작을 테스트하는 데 집중하라
#### 모든 동작이 테스트되었는지 거듭 확인하라
- 삭제해도 여전히 컴파일되거나 테스트가 통과하는 코드 라인이 있는가?
- if  문의 참 거짓 논리를 반대로 해도 테스트가 통과하는가?
- 논리 연산자나 산술 연산자를 다른 것으로 대체해도 테스트가 통과하는가?
- 상숫값이나 하드 코딩된 값을 변경해도 테스트가 통과하는가?
#### 오류 시나리오를 잊지 말라
- 예외가 발생하는지 확인하고 발생한 예외에 예상하는 오류 메시지가 포함되어 있는지 확인
## 11.2 테스트만을 위해 퍼블릭으로 만들지 말라
### 11.2.1 프라이빗 함수를 테스트하는 것은 바람직하지 않을 때가 많다
- 원하는 최종 결과와 중간 구현 세부 사항을 섞지 말아야 함
- 프라이빗 함수인 구현 세부 사항이 리팩터링 될 시 실패하는 테스트가 없도록 해야함
### 11.2.2 해결책: 퍼블릭 API를 통해 테스트하라
- 클래스가 복잡하여 퍼블릭 API를 통해 모든 동작을 테스트하는 것이 까다로울 경우 코드의 추상화 계층이 너무 크다는 것을 의미하며 코드를 더 작은 단위로 분리해야함
### 11.2.3 해결책: 코드를 더 작은 단위로 분할하라
#### 복잡한 구현 코드
```java
class MortgageAssessor {
	private const Double MORTGAGE_MULTIPLIER = 10.0;
	private const Double GOOD_CREDIT_SCORE_THRESHOLD = 880.0;

	private final CreditScoreService creditScoreService;

	MortgageDecision assess(Customer customer) {
		//...
	}

	private Result<Boolean, Error> isEligibleForMortgage(Customer customer) {
		if (customer.hasExistingMortgage() || customer.isBanned()) {
			return Result.ofValue(false);
		}
		return isCreditRatingGood(customer.getId());
	}

	/** 테스트를 위해서만 공개 */
	Result<Boolean, Error> isCreditRatingGood(Int customerId) {
		CreditScoreResponse response = creditScoreService.query(customerId);

		if (response.errorOccurred()) {
			return Result.ofError(response.getError());
		}
		return Result.ofValue(response.getCreditScore() >= GOOD_CREDIT_SCORE_THRESHOLD);
	}
}
```
#### 분할된 코드
```java
class CreditRatingChecker {

	privat const Double GOOD_CREDIT_SCORE_THRESHOLD = 880.0;

	private final CreditScoreService creditScoreService;

	Result<Boolean, Error> isCreditRatingGood(Int customerId) {
		CreditScoreResponse response = creditScoreService.query(customerId);

		if (response.errorOccurred()) {
			return Result.ofError(response.getError());
		}
		return Result.ofValue(response.getCreditScore() >= GOOD_CREDIT_SCORE_THRESHOLD);
	}
}

class MortgageAssessor {
	private const Double MORTGAGE_MULTIPLIER = 10.0;

	private final CreditScoreService creditScoreService;

	MortgageDecision assess(Customer customer) {
		//...
	}

	private Result<Boolean, Error> isEligibleForMortgage(Customer customer) {
		if (customer.hasExistingMortgage() || customer.isBanned()) {
			return Result.ofValue(false);
		}
		return creditRatingChecker.isCreditRatingGood(customer.getId());
	}
}
```
## 11.3 한 번에 하나의 동작만 테스트하라
### 11.3.1 여러 동작을 한꺼번에 테스트하면 테스트가 제대로 안 될 수 있다
```java
void testGetValidCoupons_allBehaviors() {
	Customer customer1 = new Customer("test customer 1");
	Customer customer2 = new Customer("test customer 2");
	
	Coupon redeemed = new Coupon(
		alreadyRedeemed: true, hasExpired: false,
		issuedTo: customer1, value: 100	
	);

	Coupon expired = new Coupon(
		alreadyRedeemed: false, hasExpired: true,
		issuedTo: customer1, value: 100
	);

	Coupon issuedToSomeoneElse = new Coupon(
		alreadyRedeemed: false, hasExpired: false,
		issuedTo: customer2, value: 100
	);

	Coupon valid1 = new Coupon(
		alreadyRedeemed: false, hasExpired: false,
		issuedTo: customer1, value: 100
	);
	Coupon valid2 = new Coupon(
		alreadyRedeemed: false, hasExpired: false,
		issuedTo: customer1, value: 150
	);

	List<Coupon> validCoupons = getValidCoupons(
		[redeemed, expired, issuedToSomeoneElse, valid1, valid2],
		customer1
	);

	assertThat(validCoupons)
		.containsExactly(valid2, valid1)
		.inOrder();
}
```
- 테스트 케이스가 정확히 무엇을 하고 있는지 이해하기 어렵다
- 변경으로 인해 테스트 케이스가 실패할 경우 정확히 무엇이 변경 됐는지 알 수 없으며, 무언가 변경됐다는 사실만 알려준다
- 코드가 의도적으로 변경되었을 때 그 변경으로 인해 어떤 동작이 영향을 받았고 어떤 동작이 영향을 받지 않았는지 정확히 알기 어려움
### 11.3.2 해결책: 각 동작은 자체 테스트 케이스에서 테스트하라
```java
void testGetValidCoupons_validCoupon_included() {
	Customer customer1 = new Customer("test customer 1");

	Coupon valid = new Coupon(
		alreadyRedeemed: false, hasExpired: false,
		issuedTo: customer1, value: 100
	);

	List<Coupon> validCoupons = getValidCoupons([valid], customer);

	assertThat(validCoupons).containsExactly(valid);
}

void testGetValidCoupons_alreadyRedeemed_excluded() {
	Customer customer1 = new Customer("test customer 1");

	Coupon redeemed = new Coupon(
		alreadyRedeemed: true, hasExpired: false,
		issuedTo: customer1, value: 100	
	);

	List<Coupon> validCoupons = getValidCoupons([redeemed], customer);

	assertThat(validCoupons).isEmpty();
}

void testGetValidCoupons_expired_excluded() {...}
void testGetValidCoupons_issuedToDifferentCustomer_excluded() {...}
void testGetValidCoupons_returnedInDescendingValueOrder() {...}
```
- 테스트 케이스에 사용된 값과 설정이 일부 사소한 차이를 제외하고 거의 동일한 경우 매개변수화된 테스트를 사용
### 11.3.3 매개변수를 사용한 테스트
```java
[TestCase(true, false, TestName = "이미 사용함")]
[TestCase(false, true, TestName = "유효 기간 만료")]
void testGetValidCoupons_excludesInvalidCoupons(
	Boolean alreadyRedeemed, Boolean hasExpired
) {
	Customer customer = new Customer("test customer");
	Coupon coupon = new Coupon(
		alreadyRedeemed: alreadyRedeemed,
		hasExpired: hasExpired,
		issuedTo: customer, value: 100
	);

	List<Coupon> validCoupons = getValidCoupons([coupon], customer);

	assertThat(validCoupons).isEmpty();
}
```
- 자바의 경우, JUnit은 매개변수를 사용한 테스트를 지원함
## 11.4 공유 설정을 적절하게 사용하라
- `BeforeAll`, `OneTimeSetUp` : 테스트 케이스가 실행되기 전에 단 한 번 실행됨
- `BeforeEach`, `SetUp` : 각 테스트 케이스가 실행되기 전에 매번 실행
- `AfterAll`, `OneTimeTearDown` : 모든 테스트 케이스가 실행된 후 실행
- `AfterEach`, `TearDown` : 각 테스트 케이스가 실행된 후에 매번 실행
### 11.4.1 상태 공유는 문제가 될 수 있다
```java
Class OrderManager {

	private final Database database;

	void processOrder(Order order) {
		if (order.containsOutOfStockItem() || !order.isPaymentComplete()) {
			database.setOrderStatus(order.getId(), OrderStatus.DELAYED);
		}
	}
}
```
- 가변상태인 `Database`를 설정하는데 테스트 케이스 간에 가변적인 상태를 공유하면 다른 테스트 케이스에 의해 변경된 상태가 다른 테스트 케이스에 영향을 미칠 수 있음
### 11.4.2 해결책: 상태를 공유하지 않거나 초기화하라
- 테스트 더블 사용
```java
class OrderManagerTest {

	private Database database;

	@BeforeAll
	void oneTimeSetUp() {
		database = Database.createInstance();
		database.waitForReady();
	}

	@AfterEach
	void tearDown() {
		database.reset();
	}
}
```
- `AfterEach` 사용
### 11.4.3 설정 공유는 문제가 될 수 있다
```java
class OrderPostageManager {

	PostageLabel getPostageLabel(Order order) {
		return new PostageLabel(
			address: order.getCustomer().getAddress(),
			isLargePackage: order.getItems().size() > 2
		);
	}
}
```
```java
class OrderPostgageManagerTest {

	private Order testOrder;

	@BeforeEach
	void setUp() {
		testOrder = new Order(
			customer: new Customer(
				address: new Address("Test address")
			),
			items: [
				new Item(name: "Test item 1"),
				new Item(name: "Test item 2")
			]
		);
	}

	void testGetPostageLabel_twoItems_smallPackage() {
		PostageManager postageManager = new PostageManager();

		PostageLabel label = postageManager.getPostageLabel(testOrder);

		assertThat(label.isLargePackage()).isFalse();
	}
}
```
- 다른 개발자가 공유된 설정에 items를 추가할 경우 테스트는 실패하게 됨
### 11.4.4 해결책: 중요한 설정은 테스트 케이스 내에서 정의하라
```java
class OrderPostageManagerTest {

	void testGetPostageLabel_twoItems_smallPackage() {
		Order order = createOrderWithItems([
			new Item(name: "Test item 1"),
			new Item(name: "Test item 2")
		])

		PostageManager postageManager = new PostageManager();

		PostageLabel label = postageManager.getPostageLabel(testOrder);

		assertThat(label.isLargePackage()).isFalse();
	}

	void testGetPostageLabel_hazardousItem_isHazardous() {
		Order order = createOrderWithItems([
			new Item(name: "Hazardous item", isHazardous: true)
		])
		PostageManager postageManager = new PostageManager();

		PostageLabel label = postageManager.getPostageLabel(order);

		assertThat(label.isHazardous()).isTrue();
	}

	private static Order createOrderWithItems(List<Item> items) {
		return new Order(
			customer: new Customer(
				address: new Address("Test address")
			),
			items: items
		);
	}
}
```
### 11.4.5 설정 공유가 적절한 경우
- 테스트 케이스와 인스턴스의 일부 메타데이터가 전혀 관계가 없는 경우

> 함수 매개변수는 꼭 필요한 것만 갖는 것이 이상적이다
## 11.5 적절한 어서션 확인자를 사용하라
### 11.5.1 부적합한 확인자는 테스트 실패를 잘 설명하지 못할 수 있다
### 11.5.2 해결책: 적절한 확인자를 사용하라
- `containsAtLeast()` : 순서에 상관없이 특정 항목을 포함하고 있는지 검증
## 11.6 테스트 용이성을 위해 의존성 주입을 사용하라
- 테스트 코드에서 사용하는 의존성의 특정 인스턴스를 테스트 코드가 제공해야 하는 경우 테스트 더블을 사용할 수 없음
### 11.6.1 하드 코딩된 의존성은 테스트를 불가능하게 할 수 있다
```java
class InvoiceReminder {

	private final AddressBook addressBook;
	private final EmailSender emailSender;

	InvoiceReminder() {
		this.addressBook = DataStore.getAddressBook();
		this.emailSender = new EmailSenderImpl();
	}

	@CheckReturnValue
	Boolean sendReminder(Invoice invoice) {
		EmailAddress? address = addressBook.lookupEmailAddress(invoice.getCustomerId());
		if (address == null) {
			return false;
		}
		return eamilSender.send(address, InvoiceReminderTemplate.generate(invoice));
	}
} 
```
- 실제 의존성을 주입할 경우 데이터베이스에 연결하여 테스트로부터 외부 세계를 보호할 수 없음
### 11.6.2 해결책: 의존성 주입을 사용하라
```java
class InvoiceReminder {

	private final AddressBook addressBook;
	private final EmailSender emailSender;

	InvoiceReminder(
		AddressBook addressBook,
		EmailSender emailSender
	) {
		this.addressBook = addressBook;
		this.emailSender = emailSender;
	}
}
```
- 테스트 더블 사용 가능