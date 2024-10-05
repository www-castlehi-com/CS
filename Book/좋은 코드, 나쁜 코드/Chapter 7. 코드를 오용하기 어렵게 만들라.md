## 7.1 불변 객체로 만드는 것을 고려하라
### 7.1.1 가변 클래스는 오용하기 쉽다
```java
class TextOptions {
	private Font font;
	private Double fontSize;

	TextOptions(Font font, Double fontSize) {
		this.font = font;
		this.fontSize = fontSize;
	}

	void setFont(Font font) {
		this.font = font;
	}

	void setFontSize(Double fontSize) {
		this.fontSize = fontSize;
	}

	Font getFont() {
		return font;
	}

	Double getFontSize() {
		return fontSize;
	}
}
```
다른 클래스에서 `TextOptions`의 폰트, 폰트 사이즈를 변경할 수 있다
### 7.1.2 해결책: 객체를 생성할 때만 값을 할당하라
```java
class TextOptions {
	private final Font font;
	private final Double fontSize;

	TextOptions(Font font, Double fontSize) {
		this.font = font;
		this.fontSize = fontSize;
	}

	Font getFont() {
		return font;
	}

	Double getFontSize() {
		return fontSize;
	}
}
```
멤버 변수는 final로 표시되며 인스턴스 생성 시에만 설정할 수 있다
멤버 변수가 필수값이 아닐 경우에도 모든 변수 값이 필요하다
### 7.1.3 해결책: 불변성에 대한 디자인 패턴을 사용하라
#### 빌더 패턴
- 값을 하나씩 설정할 수 있는 빌더 클래스
- 빌더에 의해 작성된 불변적인 읽기 전용 클래스
```java
class TextOptions {
	private final Font font;
	private final Double? fontSize;

	TextOptions(Font font, Double? fontSize) {
		this.font = font;
		this.fontSize = fontSize;
	}

	Font getFont() {
		return font;
	}

	Double? getFontSize() {
		return fontSize;
	}
}

class TextOptionsBuilder {
	private final Font font;
	private Double? fontSize;

	TextOptionsBuilder(Font font) {
		this.font = font;
	}

	TextOptionsBuilder setFontSize(Double fontSize) {
		this.fontSize = fontSize;
		return this;
	}

	TextOptions build() {
		return new TextOptions(font, fontSize);
	}
}
```
- `font`가 필수값이고, `fontSize`가 옵션값일 경우 빌더 패턴
#### 쓰기 시 복사 패턴
```java
class TextOptions {
	private final Font font;
	private final Double? fontSize;

	TextOptions(Font font) {
		this(font, null);
	}

	private TextOptions(Font font, Double? fontSize) {
		this.font = font;
		this.fontSize = fontSize;
	}

	Font getFont() {
		return font;
	}

	Double? getFontSize() {
		return fontSize;
	}

	TextOptions withFont(Font newFont) {
		return new TextOptions(newFont, fontSize);
	}

	TextOptions withFontSize(Double newFontSize) {
		return new TextOptions(font, newFontSize);
	}
}
```
필드값이 변경된 새 객체를 반환
## 7.2 객체를 깊은 수준까지 불변적으로 만드는 것을 고려하라
**깊은 가변성** : 멤버 변수 자체가 가변적인 유형이고 다른 코드가 멤버 변수에 액세스할 수 있는 경우
### 7.2.1 깊은 가변성은 오용을 초래할 수 있다
```java
...
List<Font> fontFamily = [Font.ARIAL, Font.VERDANA];

TextOptions textOptions = new TextOptions(fontFamily, 12.0);

fontFamily.clear();
fontFamily.add(Font.COMIC_SANS);
...
```
```java
...
TextOptions textOptions = new TextOptions([Font.ARIAL, Font.VERDANA], 12.0);

List<Font> fontFamily = textOptions.getFontFamily();
fontFamily.clear();
fontFamily.add(Font.COMIC_SANS);
...
```
- **참조**를 가지고 있는 필드값이 다른 외부 환경에 의해 변경된다.
### 7.2.2 해결책: 방어적으로 복사하라
```java
class TextOptions {
	private final List<Font> fontFamily;
	private final Double fontSize;

	TextOptions(List<Font> fontFamily, Double fontSize) {
		this.fontFamily = fontFamily;
		this.fontSize = fontSize;
	}

	List<Font> getFontFamily() {
		return List.copyOf(fontFamily);
	}

	Double getFontSize() {
		return fontSize;
	}
}
```
#### 단점
- 복사 비용 증가
- 클래스 내부에서 `fontFamily.add()`를 하는 것처럼 변경을 막아주지 못하는 경우가 있다
### 7.2.3 해결책 : 불변적 자료구조를 사용하라
Guava 라이브러리의 `ImmutableList` 클래스
```java
Class TextOptions {
	private final ImmutableList<Font> fontFamily;
	private final Double fontSize;

	TextOptions(ImmutableList<Font> fontFamily, Double fontSize) {
		this.fontFamily = fontFamily;
		this.fontSize = fontSize;
	}

	ImmutuableList<Font> fontFamily() {
		return fontFamily;
	}

	Double getFontSize() {
		return fontSize;
	}
}
```
## 7.3 지나치게 일반적인 데이터 유형을 피하라
정수나 리스트와 같은 유형으로 표현이 '가능'하다고 해서 그것이 반드시 '좋은' 방법은 아니다
설명이 부족하고 허용하는 범위가 넓을수록 코드 오용이 쉬워진다
### 7.3.1 지나치게 일반적인 유형은 오용될 수 있다
### 7.3.2 페어 유형은 오용하기 쉽다
### 7.3.3 해결책: 전용 유형 사용
class를 만들어 사용한다
## 7.4 시간 처리
### 7.4.1 정수로 시간을 나타내는 것은 문제가 될 수 있다
- 한 순간의 시간인지 시간의 양인지 알아내기 어렵다
- 단위가 일치하지 않을 수 있다
- 서로 다른 시간대에 살고 있는 경우 시간대를 처리하는데 오류가 발생할 수 있다
### 7.4.2 해결책: 적절한 자료구조를 사용하라
자바의 `java.time` 패키지의 클래스
- `Duration`, `Instant`유형으로 구분
## 7.5 데이터에 대해 진실의 원천을 하나만 가져야 한다
- **기본 데이터** : 코드에 제공해야 할 데이터이며 코드에 이 데이터를 알려주지 않고는 코드가 처리할 방법이 없음
- **파생 데이터** : 주어진 기본 데이터에 기반해서 코드가 계산할 수 있는 데이터
### 7.5.1 또 다른 진실의 원천은 유효하지 않은 상태를 초래할 수 있다
```java
class UserAccount {
	private final Double credit;
	private final Double debit;
	private final Double balance;

	UserAccount(Double credit, Double debit, Double balance) {
		this.credit = credit;
		this.debit = debit;
		this.balance = balance;
	}

	Double getCredit() {
		return credit;
	}

	Double getDebit() {
		return debit;
	}

	Double getBalance() {
		return balance;
	}
}
```
balance를 생성할 때 잘못을 범할 경우 버그로 이어진다
### 7.5.2 해결책: 기본 데이터를 유일한 진실의 원천으로 사용하라
```java
class UserAccount {
	private final Double credit;
	private final Double debit;

	UserAccount(Double credit, Double debit) {
		this.credit = credit;
		this.debit = debit;
	}

	Double getCredit() {
		return credit;
	}

	Double getDebit() {
		return debit;
	}

	Double getBalance() {
		return credit - debit;
	}
}
```
## 7.6 논리에 대한 진실의 원천을 하나만 가져야 한다
### 7.6.1 논리에 대한 진실의 원천이 여러 개 있으면 버그를 유발할 수 있다
```java
class DataLogger {
	private final List<Int> loggedValues;
	...

	saveValues(FileHandler file) {
		String serializedValue = loggedValues
			.map(value -> value.toString(Radix.BASE_10))
			.join(",");
		file.write(serializedValues);
	}
}
```
```java
class DataLoader {
	...

	List<Int> loadValues(FileHandler file) {
		return file.readAsString()
			.split(",")
			.map(str -> Int.parse(str, Radix.BASE_10));
	}
}
```
한 클래스가 수정되지 않고 다른 클래스가 수정되지 않으면 문제가 발생한다
### 7.6.2 해결책: 진실의 원천은 단 하나만 있어야 한다
```java
class IntListFormat {
	private const String DELIMITER = ",";
	private const Radix RADIX = Radix.BASE_10;

	String serialize(List<Int> values) {
		return values
			.map(value -> value.toString(RADIX))
			.join(DELIMITER);
	}

	List<Int> deserialize(String serialized) {
		return serialized
			.split(DELIMITER)
			.map(str -> Int.parse(str, RADIX));
	}
}
```
두 개의 다른 코드가 수행하는 논리가 일치하도록 한다