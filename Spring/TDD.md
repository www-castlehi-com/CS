# 정의
- **Test Driven Development**
- 테스트 주도 개발
- 짧은 개발 주기의 반복에 의존

# 개발주기
![](https://i.imgur.com/TjUuLxE.png)
1. **RED** : 실패하는 테스트 코드 작성
2. **Green** : 테스트 코드를 성공시키기 위한 실제 코드
3. **Blue** : 리팩토링

# 비교
## 폭포수 모델 (일반 개발 방식)
![](https://i.imgur.com/zuknxJM.png)

요구 사항 분석 -> 설계 -> 개발 -> 테스트 -> 배포

개발을 느리게 함
- 소비자의 요구사항이 처음부터 명확하지 않을 수 있음
- 소스코드의 품질 저하 : 재설계로 인한 불필요한 코드가 남거나 중복처리가 될 가능성이 큼
- 자체 테스트 비용 증가 : 작은 부분의 기능 수정에도 전체 테스트가 진행되어 전체적인 버그를 검출하기 어려움

## TDD
![](https://i.imgur.com/J6qxlxm.png)

테스트 코드를 작성한 뒤에 실제 코드 작성

- 프로그래밍 목적을 반드시 미리 정의
- 무엇을 테스트해야 할지 미리 정의

# TDD 툴
## JUnit

**JUnit5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

1. JUnit Platform

- 테스트 프레임워크를 JVM에서 실행하기 위한 기반을 제공
- **TestEngine**_(테스트를 발견, 실행하고 결과를 보고하는 역할)_ API를 지원하여 JUnit5 생태계 내에서 다른 테스팅 라이브러리와 프레임워크를 사용할 수 있게 함.
- 플랫폼을 명령 줄에서 시작하기 위한 **Console Launcher** 제공
- 하나 이상의 TestEngine을 사용하여 사용자 정의 테스트를 실행하기 위한 **JUnit Platform Suite Engine** 제공
- 여러 IDE (IntelliJ IDEA, Eclipse, Visual Studio Code 등) 및 빌드 도구 (Maven, Gradle 등) 지원

2. JUnit Jupiter

- 테스트 작성 및 실행을 담당
- **Programming model** 지원 (@BeforeEach, @AfterEach, @Test 등) : 테스트를 작성하는 데 사용하는 규칙과 주석의 집합
- **ExtensionModel** 지원 (@ExtendWith 등) : 테스트 수명 주기 동안 실행할 수 있는 사용자 지정 작업을 생성하기 위해 구현할 수 있는 일련의 API 및 인터페이스를 제공
- 자체 **TestEngine**을 제공하여 JUnit Jupiter 프로그래밍 모델로 작성된 테스트를 실행

3. JUnit Vintage

- JUnit3 또는 JUnit4로 작성된 테스트를 실행
- JUnit4.12 이상일 시 사용 가능