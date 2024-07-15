### 2.2.1 추상화 계층 및 코드 품질의 핵심 요소
- 가독성
- 모듈화
- 재사용성 및 일반화성
- 테스트 용이성
### 2.3.2 함수

> 단일 업무 수행
> 잘 명명된 다른 함수를 호출해서 더 복잡한 동작 구성

#### 너무 많은 일을 하는 함수

```java
SentConfirmation? sendOwnerALetter(
	Vehicle vehicle, Letter letter
) {
	Address? ownersAddress = null;
	if (vehicle.hasBennScraped()) {
		ownersAddress = SCRAPYARD_ADDRESS;
	} else {
		Purchase? mostRecentPurchase = vehicle.getMostRecentPurchase();
		if (mostRecentPurchase == null) {
			ownersAddress = SHOWROOM_ADDRESS;
		} else {
			ownersAddress = mostRecentPurchase.getBuyersAddress();
		}
	}
	if (ownersAddress == null) {
		return null;
	}
	return sendLetter(ownersAddress, letter);
}
```
- 위 함수는 여러 로직이 합쳐져 있음
	- 소유자의 주소를 찾기 위한 자세한 로직
	```java
	Address? ownersAddress = null;
	if (vehicle.hasBennScraped()) {
		ownersAddress = SCRAPYARD_ADDRESS;
	} else {
		Purchase? mostRecentPurchase = vehicle.getMostRecentPurchase();
		if (mostRecentPurchase == null) {
			ownersAddress = SHOWROOM_ADDRESS;
		} else {
			ownersAddress = mostRecentPurchase.getBuyersAddress();
		}
	}
	```
	- 조건부로 편지를 보내는 로직
	```java
	if (ownersAddress == null) {
		return null;
	}
	return sendLetter(ownersAddress, letter);
	```
#### 더 작은 함수
```java
SentConfirmation? sendOwnerALetter(Vehicle vehicle, Letter letter) {
	Address? ownersAddress = getOwnersAddress(vehicle);
	if (ownersAddress == null) {
		return null;
	}
	return sendLetter(ownersAddress, letter);
}

private Address? getOwnersAddress(Vehicle vehicle) {
	if (vehicle.hasBeenScraped()) {
		return SCRAPYARD_ADDRESS;
	}
	Purcase? mostRecentPurchase = vehicle.getMostRecentPurchase();
	if (mostRecentPurchase == null) {
		return SHOWROOM_ADDRESS;
	}
	return mostRecentPurchase.getBuyersAddress();
}
```
### 2.3.3 클래스
**응집력**
- 한 클래스 내의 모든 요소들이 얼마나 잘 속해 있는지
- 순차적 응집력 : 한 요소의 출력이 다른 요소에 댇한 입력으로 필요할 때
- 기능적 응집력 : 몇 가지 요소들이 모여서 하나의 일을 성취하는데 기여할 때
**관심사의 분리**
- 시스템이 각각 별개의 문제를 다루는 개별 구성 요소로 분리
#### 너무 큰 클래스
```java
class TextSummarizer {
	String summarizeText(String text) {
		return splitIntoParagraphs(text)
			.filter(paragraph -> calculateImportacne(paragraph) >= IMPORTANCE_THRESHOLD)
			.join("\n\n");
	}

	private Double calculateImportance(String paragraph) {
		List<String> nouns = extractImportantNouns(paragraph);
		List<String> verbs = extractImportantVerbs(paragraph);
		List<String> adjectives = extractImportantAdjectives(paragraph);
		//...

		return importanceScore;
	}

	private List<String> extractImportantNouns(String text) {...}
	private List<String> extractImportantVerbs(String text) {...}
	private List<String> extractImportantAdjectives(String text) {...}

	private List<String> splitIntoParagraphs(String text) {
		List<String> paragraphs = [];
		Int? start = detectParagraphsStartOffset(text, 0);
		while (start != null) {
			Int? end = detectParagraphEndOffset(text, start);
			if (end == null) {
				break;
			}
			paragraphs.add(text.subString(start, end));
			start = detectParagraphStartOffset(text, end);
		}
		return paragraphs;
	}

	private Int? detectParagraphStartOffset(
		String text, Int fromOffset
	) {...}

	private Int? detectParagraphEndOffset(
		String text, Int fromOffset
	) {...}
}
```
- 같은 클래스 내에 '텍스트 요약', '중요도 계산', '명사, 형용사, 동사 추출', '테스트 단락 나누기', '단락 시작 찾기', '단락 끝 찾기' 같은 너무 많은 개념 포함
- 단위 테스트 불가
- 기능 일부분 교체 불가
- 하위 문제들에 대한 해결책 재사용 불가
#### 각 개념에 대한 별도 클래스
```java
class TextSummarizer {
	private final ParagraphFinder paragraphFinder;
	private final TextImportanceScorer importantScorer;

	TextSummarizer (
		ParagraphFinder paragraphFinder,
		TextImportanceScorer importanceScorer
	) {
		this.paragraphFinder = paragraphFinder;
		this.importantScorer = importantScorer;
	}

	static TextSummarizer createDefault() {
		return new TextSummarizer (
			new ParagraphFinder(),
			new TextImportanceScorer()
		);
	}

	String summarizeText(String text) {
		return paragraphFinder.find(text)
			.filter(paragraph -> importanceScorer.isImportant(paragraph))
			.join("\n\n");
	}
}

class PragraphFinder {
	List<String> find(String text) {
		List<String> paragraphs = [];
		Int? start = detectParagraphStartOffset(text, 0);
		while (start != null) {
			Int? end = detectParagraphEndOffset(text, start);
			if (end == null) {
				break;
			}
			paragraphs.add(text.subString(start, end));
			start = detectParagraphStartoffset(text, end);
		}
		return paragraphs;
	}

	private Int? detectParagraphStartOffset(
		String text, Int fromOffset
	) {...}

	private Int? detectParagraphEndOffset(
		String text, Int fromOffset
	) {...}
}

class TextImportanceScorer {
	Boolean isImportant(String text) {
		return calculateImportance(text) >= IMPORTANE_THRESHOLD;
	}

	private Double calculateImportance(String text) {
		List<String> nouns = extractImportantNouns(text);
		List<String> verbs = extractImportantNouns(text);
		List<String> adjectives = extractImportantAdjectives(text);

		//...
		return importance;
	}

	private List<String> extractImportantNouns(String text) {...}
	private List<String> extractImportantVerbs(String text) {...}
	private List<String> extractImportantAdjectives(String text) {...}
}
```
- 재사용 & 테스트 용이성 향상
### 2.3.4 인터페이스
- 계층 사이를 뚜렷이 구분하고 계층 사이에 구현 세부 사항이 유출되지 않도록 하기 위해 사용
- 상위 계층은 하위 클래스에 의존하지 않고 인터페이스에 정의된 대로 코드 구현
- 인터페이스만을 위한 인터페이스를 작성해서는 안
#### 장점
- 퍼블릭 API를 명확하게 보여줌
- 테스트 용이성
- 재사용성
#### 단점
- 더 많은 작업 필요
- 코드 복잡해질 가능성 존재
### 2.3.5 층이 너무 얇아질 때
- 분할을 위한 분할은 의미가 없음
- 다른 곳에서 사용될 가능성이 없는 코드는 계층 분리할 필요가 없음음