# Spring Framework란?
- 자바 플랫폼을 위한 오픈소스 애플리케이션 프레임워크
- 엔터프라이즈급 애플리케이션을 개발하기 위한 모든 기능을 종합적으로 제공하는 경량화된 솔루션
> **엔터프라이즈급 애플리케이션**
> - 주로 대규모 데이터 처리와 트랜잭션이 동시에 여러 사용자로부터 행해지는 매우 큰 규모의 환경
>
> **경량 컨테이너**
> - 자바 객체를 담고 직접 관리
> - 객체의 생성 및 소멸, 라이프사이클을 관리하며 Spring 컨테이너로부터 필요한 객체를 가져와 사용

# 특징
## POJO 프로그래밍
- **Plain Old Java Object** : 순수 Java만을 통해서 생성한 객체
- 외부 기술이 사용되고 만약 이 기술이 deprecated되었을 때 해당 기술들을 사용하고 있는 모든 객체의 코드를 변경해야 하므로 POJO 프로그래밍을 지향해야한다
- 유연하게 변화와 확장에 대처할 수 있다

### 1. IOC
- **Inversion of Control : 제어의 역전**
- 일반적인 프로그래밍 : 객체 결정 및 생성 -> 의존성 객체 생성 -> 객체 내의 메소드 호출
- Spring framework : 제어의 흐름을 컨트롤 하지 않고 위임한 특별한 객체에 모든 것을 맡김

1. **DI**
	- Dependency Injection
	- 의존성 주입
	- 각 클래스 사이에 필요로 하는 의존관계를 빈 설정 정보를 바탕으로 컨테이너가 자동으로 연결해줌

2. **DL**
	- Dependency Lookup
	- 의존성 검색
	- 컨테이너에서 객체들을 관리하기 위해 별도의 저장소에 빈을 저장하고, 개발자들이 컨테이너에서 제공하는 API를 이용하여 사용하고자 하는 빈을 검색하는 방법
	- DI / Application Context

### 2. AOP
- **Aspect Oriented Programming : 관심 지향 프로그래밍**
- 핵심 관심 사항 : 애플리케이션의 핵심 기능과 관련된 관심 사항
- 공통 관심 사항 : 모든 핵심 관심 사항에 공통적으로 적용되는 관심 사항
- 공통 관심 사항을 수행하는 로직이 변경되면 모든 중복 코드를 변경해야하므로 별도의 객체를 분리해냄

### 3. PSA
- **Portable Service Abstraction : 일관된 서비스 추상화**
- 특정 기술과 관련된 서비스를 추상화하여 일관된 방식으로 사용될 수 있도록 한 것

>**JDBC** : 데이터베이스에 접근하는 방법을 규정한 인터페이스
>- 데이터베이스를 만든 회사가 자신의 데이터베이스에 접근하는 드라이버를 Java 코드의 형태로 배포하고, 이 드라이버에 해당하는 코드의 클래스가 JDBC를 구현
>- 이후에 데이터베이스를 바꿔도 기존에 작성한 데이터베이스 접근 로직을 그대로 사용할 수 있음

## 구조
1. **Core Container**: 이는 Spring Framework의 기반
    - **Beans**: Spring에서 객체(Bean)의 생성, 관리, 그리고 의존성 주입이 이루어지는 핵심 컴포넌트
    - **Core**: Spring의 기본적인 부분으로, Spring Framework의 다른 모든 부분에 사용되는 기능을 제공
    - **Context**: Spring의 Application Context, 즉 Spring 애플리케이션에서 사용되는 정보의 저장소 및 액세스 포인트를 제공
2. **Data Access/Integration**: 데이터 접근과 통합 기능
    - **JDBC (Java Database Connectivity)**: 데이터베이스 연결 및 SQL 실행을 단순화
    - **ORM (Object-Relational Mapping)**: JPA, Hibernate 등의 ORM 프레임워크와의 통합
3. **Web**: 웹 애플리케이션 개발에 관련된 기능
    - **Web MVC**: Spring의 Model-View-Controller
    - **Web Socket**: 웹소켓 기반 통신
