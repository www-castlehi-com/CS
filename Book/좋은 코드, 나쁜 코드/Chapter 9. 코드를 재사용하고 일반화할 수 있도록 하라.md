## 9.1 가정을 주의하라
- 코드 작성 시 가정을 하면 코드가 더 단순해지거나 효율적으로 될 수 있음
- 가정으로 인해 코드가 더 취약해지고 활용도가 낮아져 재사용하기에 안전하지 않을 수 있음
	- 어떤 가정이 이루어졌는지 정확히 추적하는 것이 어려움
### 9.1.1 가정은 코드 재사용 시 버그를 초래할 수 있다
```java
class Article {
	
	private List<Section> sections;

	List<Image> getAllImages() {
		for (Section section in sections) {
			if (section.containImages()) {
				return section.getImages();
			}
		}
		return [];
	}
}
```
- 이미지 섹션이 하나만 있을 것이라는 가정
### 9.1.2 해결책 : 불필요한 가정을 피하라
> **섣부른 최적화**
> - 코드 최적화는 일반적으로 비용이 든다
> - 큰 효과 없는 코드 최적화를 하기보다는 코드를 읽을 수 있고, 유지보수 가능하며, 견고하게 만드는 데 집중하는 것이 좋다
> - 코드의 어떤 부분이 여러 번 실행되고 그 부분을 최적화하는 것이 성능 향상에 큰 효과를 볼 수 있다는 점이 명백해질 때에 최적화 작업
```java
class Article {

	private List<Section> sections;

	List<Image> getAllImages() {
		List<Image> images = [];
		for (Section section in sections) {
			images.addAll(section.getImages());
		}
		return images;
	}
}
```
### 9.1.3 해결책 : 가정이 필요하면 강제적으로 하라
```java
class Article {

	private List<Section> sections;

	Section? getOnlyImageSection() {
		List<Section> imageSections = sections.filter(section -> section.containImages());

		assert(imageSections.size() <= 1, "기사가 여러 개의 이미지 섹션을 갖는다");

		return imageSections.first();
	}
}
```
- 어서션을 사용해 최대 하나의 이미지 섹션이 있다고 가정할 때 코드 호출
- `getOnlyImageSection()`으로 변경해 이 함수의 호출자에게 이미지 섹션이 하나만 있다고 가정
## 9.2 전역 상태를 주의하라
- 프로그램 내의 모든 콘텍스트에 영향
- 전역변수를 사용할 때는 누구도 해당 코드를 다른 목적으로 재사용하지 않을 것이라는 암묵적인 가정 전제
- 전역 상태는 코드를 매우 취약하게 만들고 재사용하기도 안전하지 않기 때문에 일반적으로 이점보다 비용이 더 크다
### 9.2.1 전역 상태를 갖는 코드는 재사용하기에 안전하지 않을 수 있다
```java
class ShoppingBasket {

	private static List<Item> items = [];

	static void addItem(Item item) {
		items.add(item);
	}

	static void List<Item> getItems() {
		return List.copyOf(items);
	}
}
```
- 여러 코드 내에서 재사용될 경우, 인스턴스마다 동일한 객체를 사용하게 될 수 있음
	- 고객들이 원하지 않는 물건이 주문
	- 장바구니에 넣은 물건을 다른 사용자가 보게 되어 사생활 침해 발생
### 9.2.2 해결책 : 공유 상태에 의존성 주입하라
```java
class ShoppingBasket {

	private List<Item> items = [];

	void addItem(Item item) {
		items.add(item);
	}

	void List<Item> getItems() {
		return List.copyOf(items);
	}
}
```
- 각 인스턴스는 독립적으로 존재
## 9.3 기본 반환값을 적절하게 사용하라
- 기본값 제공 조건
	- 어떤 기본값이 합리적인지
	- 더 상위 계층의 코드는 기본값을 받든지 명시적으로 설정된 값을 받든지 상관하지 않음
- 상위 수준의 코드는 특정 사용 사례에 더 밀접하게 결합하므로 코드의 모든 용도에 맞는 기본값을 선택하기 더 쉬움
- 낮은 수준의 코드는 보다 근본적인 하위 문제를 해결하여 여러 사용 사례에 더 광범위하게 재사용되는 경향이 있어 적합한 기본값을 선택하기 어려움
### 9.3.1 낮은 층위의 코드의 기본 반환값은 재사용성을 해칠 수 있다
```java
class UserDocumentSettings {

	private final Font? font;

	Font getPreferredFont() {
		if (font != null) {
			return font;
		}
		return Font.ARIAL;
	}
}
```
- 기본 글꼴로 Arial을 원하지 않는 경우에 재사용에 어려움이 있음
- 사용자가 특별히 Arial을 선택한 것인지 아니면 선호하는 폰트를 설정하지 않아 기본값이 반환된 것인지 구분하는 것이 불가능
- 기본값과 관련된 요구 사항이 변경되는 경우 어려움이 있음
### 9.3.2 해결책 : 상위 수준의 코드에서 기본값을 제공하라
```java
class UserDocumentSettings {

	private final Font? font;

	Font? getPreferredFont() {
		return font;
	}
}
```
- 호출하는 상위 수준 코드에서 원하는 방식으로 하위 문제를 해결하고, 코드의 재사용성을 높여줌
```java
class DefaultDocumentSettings {

	Font getDefaultFont() {
		return Font.ARIAL;
	}
}

class DocumentSettings {

	private final UserDocumentSettings userSettings;
	private final DefaultDocumentSettings defaultSettings;

	DocumentSettings(
		UserDocumentSettings userSettings,
		DefaultDocumentSettings defaultSettings) {
		this.userSettings = userSettings;
		this.defaultSettings = defaultSettings;	
	}

	Font getFont() {
		Font? userFont = userSettings.getPreferredFont();
		if (userFont != null) {
			return userFont;
		}
		return defaultSettings.getFont();
	}
}
```
- 기본값과 사용자가 제공한 값 중에서 선택
- 기본값과 사용자가 제공한 값에 대한 모든 구현 세부 정보를 숨기며, 의존성 주입을 사용하여 구현 세부 정보를 재설정할 수 있음
## 9.4 함수의 매개변수를 주목하라
- 클래스 내에 포함된 모든 정보가 있어야 하는 경우 해당 함수가 객체나 클래스의 인스턴스를 매개변수로 받는 것이 타당
- 함수가 한두 가지 정보만 필요로 할 때는 객체나 클래스의 인스턴스를 매개변수로 사용하는 것은 코드의 재사용성을 해칠 수 있음
### 9.4.1 필요 이상으로 매개변수를 받는 함수는 재사용하기 어려울 수 있다
```java
class TextBox {
	
	private final Element textContainer;

	void setTextStyle(TextOptions options) {
		setFont(...);
		setFontSize(...);
		setLineHeight(...);
		setTextColor(options);
	}

	void setTextColor(TextOptions options) {
		textContainer.setStyleProperty(
			"color", options.getTextColor().asHexRgb()
		);
	}
}
```
- 몇 가지 옵션만 사용하지만 인스턴스 자체를 매개변수로 사용
```java
void styleAsWarning(TextBox textBox) {
	TextOptions style = new TextOptions(
		Font.ARIAL,
		12.0,
		14.0,
		Color.RED
	);
	textBox.setTextColor(style);
}
```
- 사용하는 함수에서 인스턴스를 만들어야 해서 재사용이 어려움
### 9.4.2 해결책 : 함수는 필요한 것만 매개변수로 받도록 하라
```java
class TextBox {

	private final Element textElement;

	void setTextStyle(TextOptions options) {
		setFont(...);
		setFontSize(...);
		setLineHeight(...);
		setTextColor(options.getTextColor());
	}

	void setTextColor(Color color) {
		textElement.setStyleProperty("color", color.asHexRgb());
	}

	void styleAsWarning(TextBox textBox) {
		textBox.setTextColor(Color.RED);
	}
}
```
- 10가지 항목을 캡슐화하는 클래스가 있고, 8개를 필요로 하는 함수가 있다면 캡슐화 객체 전체를 함수에 전달하는 것이 합리적
## 9.5 제네릭의 사용을 고려하라
- 코드의 일반화 향상
### 9.5.1 특정 유형에 의존하면 일반화를 제한한다
```java
class RandomizedQueue {

	private final List<String> values = [];

	void add(String value) {
		values.add(value);
	}

	String? getNext() {
		if (values.isEmpty()) {
			return null;
		}
		Int randomIndex = Math.randomInt(0, values.size());
		values.swap(randomIndex, values.size() - 1);
		return values.removeLast();
	}
}
```
- 특정 자료형에 대한 의존도가 높기 때문에 다른 유형을 저장하는데는 사용할 수 없음
### 9.5.2 해결책 : 제네릭을 사용하라
```java
class RandomizedQueue<T> {

	private final List<T> values = [];

	void add(T value) {
		values.add(value);
	}

	T? getNext() {
		if (values.isEmpty()) {
			return null;
		}
		Int randomIndex = Math.randomInt(0, values.size());
		values.swap(randomIndex, values.size() - 1);
		return values.removeLast();
	}
}
```
- 호출 코드에서 어떤 유형을 사용할지 지시할 수 있는 자리 표시자를 지정
- 일반화되고 재사용이 가능