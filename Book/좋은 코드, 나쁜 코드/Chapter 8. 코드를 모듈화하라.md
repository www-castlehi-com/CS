## 8.1 의존성 주입의 사용을 고려하라
하위 문제를 재구성할 수 있는 방식으로 코드를 작성하는 것이 유용
### 8.1.1 하드 코드화된 의존성은 문제가 될 수 있다
```java
class RoutePlanner {
	private final RoadMap roadMap;

	RoutePlanner() {
		this.roadMap = new NorthAmericaRoadMap();
	}

	Route planRoute(LatLong startPoint, LatLong endPoint) {}
}

interface RoadMap {
	List<Road> getRoads();
	List<Junction> getJunctions();
}

class NorthAmericaRoadMap implements RoadMap {
	override List<Road> getRoads();
	override List<Junction> getJunctions();
}
```
`RoutePlanner` 클래스는 생성자에서 `NorthAmericaRoadMap`을 생성하는데, 이는 특정 구현에 대한 의존성이 하드코딩 되어 있는 것을 의미한다
### 8.1.2 해결책: 의존성 주입을 사용하라
#### 1️⃣ 종속성 주입
```java
class RoutePlanner {
	private final RoadMap roadMap;

	RoutePlanner(RoadMap roadMap) {
		this.roadMap = roadMap;
	}

	Route planRoute(LatLong startPoint, LatLong endPoint) {}
}
```
생성자가 복잡해짐
#### 2️⃣ 팩토리 함수
```java
class RoutePalannerFactory {
	static RoutePlanner createEuropeRoutePlanner() {
		return new RoutePlanner(new EuropeRoadMap());
	}

	static RoutePlanner createDefaultNorthAmericaRoutePlanner() {
		return new RoutePlanner(new NorthAmericaRoadMap(true, false));
	}
}
```
많은 클래스에 대해 만들어야 한다면 코드가 반복됨
#### 의존성 주입 프레임워크
## 8.2 인터페이스에 의존하라
**의존성 역전 원리**
구체적인 구현보다는 추상화에 의존하는 것이 낫다
## 8.3 클래스 상속을 주의하라
두 가지 사물이 진정한 **is-a** 관계를 갖는다면 (ex: a car **is a** vehicle) 상속이 적절함
상속을 사용할 수 있는 상황에서 **구성** _(클래스를 확장하기보다 인스턴스를 가지고 있음)_ 을 대신 사용할 수도 있음
### 8.3.1 클래스 상속은 문제가 될 수 있다
#### 1️⃣ 상속은 추상화 계층에 방해가 될 수 있다
- 상속 시 클래스는 슈퍼 클래스의 모든 기능을 상속
- 원하는 것보다 더 많은 기능을 노출할 수도 있어 추상화 계층이 복잡해지고 구현 세부 정보가 드러날 수 있다
#### 2️⃣ 상속은 적응성 높은 코드의 작성을 어렵게 만들 수 있다
### 8.3.2 해결책: 구성을 사용하라
클래스를 확장하기보다 해당 클래스의 인스턴스를 가짐으로써 하나의 클래스를 다른 클래스로부터 **구성**함
```java
class IntFileReader {
	private final FileValueReader fileValueReader;

	IntFileReader(FileValueReader valueReader) {
		this.valueReader = valueReader;
	}

	Int? getNextInt() {
		String? nextValue = valueReader.getNextValue();
		if (nextValue == null) {
			return null;
		}
		return Int.parse(nextValue, Radix.BASE_10);
	}

	void close() {
		valueReader.close();
	}
}
```
### 8.3.3 진정한 is-a 관계는 어떤가?
#### 상속의 문제점
**1️⃣ 취약한 베이스 클래스 문제**
- 서브클래스가 슈퍼클래스에서 상속되고 해당 슈퍼클래스가 추후 수정되면 서브클래스가 작동하지 않을 수 있음
- 코드를 변경할 때 그 변경이 문제없을지 판단하기가 어려울 수 있음
**2️⃣ 다이아몬드 문제**
- 두 개 이상의 슈퍼클래스를 확장할 수 있는 **다중 상속**
- 슈퍼클래스가 동일한 함수의 각각 다른 버전을 제공하는 경우 문제 발생
- 어떤 슈퍼클래스로부터 해당 함수를 상속해야 하는지 모호
**3️⃣ 문제가 있는 계층 구조**
- **단일 상속**시 문제 발생 가능
- Car클래스와 AirCraft 클래스가 있을 때, '하늘을 나는 자동차'가 발명되었을 경우 계층 구조에 포함할 방법이 없음
#### 해결 방법
1. 인터페이스 사용
2. 구성을 사용
## 8.4 클래스는 자신의 기능에만 집중해야 한다
### 8.4.1 다른 클래스와 지나치게 연관되어 있으면 문제가 될 수 있다
다른 클래스에 관련된 사항만을 다룬다면 문제가 발생함
### 8.4.2 해결책: 자신의 기능에만 충실한 클래스를 만들라
**디미터의 법칙**
- 한 객체가 다른 객체의 내용이나 구조에 대해 가능한 한 최대한으로 가정하지 않아야 한다
- 특히 한 객체는 직접 관련된 객체와만 상호작용 해야 한다
## 8.5 관련 있는 데이터는 함께 캡슐화하라
너무 많은 것들이 한 클래스 안에 있을 때 문제가 발생할 수 있지만,
한 클래스 안에 함께 두는 것이 합리적일 때에는 그렇게 하는 것이 좋다
### 8.5.2 해결책: 관련된 데이터는 객체 또는 클래스로 그룹화하라
```java
class TextOptions {
	private final Font font;
	private final Double fontSize;
	private final Double lineHeight;
	private final Color textColor;

	TextOptions(Font font, Double fontSize, Double lineHeight, Color textColor) {
		this.font = font;
		this.fontSize = fontSize;
		this.lineHeight = lineHeight;
		this.textColor = textColor;
	}
}

class UiSettings {
	TextOptions getTextStyle() {}
}

class UserInterface {
	private final TexstBox messageBox;
	private final UiSettings uiSettings;

	void displayMessage(String message) {
		messageBox.renderText(message, uiSettings.getTextStyle());
	}
}

class TextBox {
	void renderText(String text, TextOptions textStyle) {}
}
```
여러 데이터가 따로 떨어져서는 별 의미가 없을 정도로 **서로 밀접하게 연관**되어 있거나,
캡슐화된 데이터 중에서 **일부만 원하는 경우가 아니라면** 캡슐화하는 것이 합리적
## 8.6 반환 유형에 구현 세부 정보가 유출되지 않도록 주의하라
간결한 추상화 계층을 가지기 위해서는 각 계층의 구현 세부 정보가 유출되지 않아야 한다
### 8.6.1 반환 형식에 구현 세부 사항이 유출될 경우 문제가 될 수 있다
```java
class ProfilePictureService {
	private final HttpFetcher httpFetcher;

	ProfilePictureREsult getProfilePicture(Int64 userId) {}
}

class ProfilePictureResult {
	HttpResponse.Status getStatus() {}
	HttpResponse.Payload? getImageData() {}
}
```
`HttpResponse`와 관련된 유형을 사용하고 있기 때문에 변경된 요구 사항을 지원하려면 확인 & 변경해야 할 코드가 너무 많아짐
### 8.6.2 해결책: 추상화 계층에 적합한 유형을 반환하라
```java
class ProfilePictureService {
	private final HttpFetcher httpFetcher;

	ProfilePictureResult getProfilePicture(Int64 userId) {}
}

class ProfilePictureResult {
	enum Status {
		SUCCESS,
		USER_DOES_NOT_EXIST,
		OTHER_ERROR,
	}

	Status getStatus() {}

	List<Byte>? getImageData() {}
}
```
## 8.7 예외 처리 시 구현 세부 사항이 유출되지 않도록 주의하라
호출하는 쪽에서 복구하고자 하는 오류에 대해 비검사 예외를 사용하는 경우 예외 처리 시 구현 세부 정보를 유출하는 것은 문제가 될 수 있음
### 8.7.2 해결책: 추상화 계층에 적절한 예외를 만들라
```java
class TextSummarizerException extends Exception {
	TextSummarizerException(Throwable cause) {}
}

class TextSummarizer {
	private final TextImportanceScorer importanceScorer;

	String summarizeText(String text) throws TextSummarizerException {
		try {
			return paragraphFinder.find(text)
				.filter(paragraph => importanceScorer.isImportant(paragraph))
				.join("\n\n");
		} catch (TextImportanceScorerException e) {
			throw new TextSummarizerException(e);
		}
	}
}

Class TextImportanceScorerException extends Exception {
	TextImportanceScorerException(Throwable cause) {}
}

interface TextImportanceScorer {
	Boolean isImportant(String text) throws TextImportanceScorerException;
}

class ModelBasedScorer implements TextImportanceScorer {
	Boolean isImportant(String text) throws TextImportanceScorerException {
		try {
			return model.predict(text) >= MODEL_THRESHOLD;
		} catch (PredictionModelException e) {
			throw new TextImportanceScorerException(e);
		}
	}
}
```
오류 처리를 위해 추가로 작성해야 되는 반복 코드가 늘어나지만, 클래스의 동작이 예측 가능하고 모듈화가 개선되기 때문에 장점의 이점이 크다
