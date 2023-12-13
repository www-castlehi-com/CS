# 빌드 도구란
## 정의

소스 코드를 컴파일하고(컴파일러 호출), 테스트를 실행하고, 소프트웨어의 구성 요소를 배포 가능한 애플리케이션으로 패키징하는 소프트웨어 도구로서 프로세스를 자동화하고 관리

소프트웨어 개발 수명 주기의 필수적인 부분

## 역할

**1. 코드 컴파일**

- 프로그램이 언어로 작성된 소스 코드를 대상 환경에서 실행할 수 있는 실행 가능한 바이너리 또는 중간 코드로 컴파일

**2. 테스트 실행**

- 단위 테스트, 통합 테스트 등 자동화된 테스트를 실행

**3. 종속성 관리**

- 프로젝트가 의존하는 외부 라이브러리, 프레임워크, 모듈 등을 다운로드하고 버전을 관리

**4. 패키징 및 배포**

- 컴파일된 코드와 필요한 리소스를 JAR, WAR 또는 실행 파일과 같은 배포 가능한 아티팩트로 패키징

**5. 정적 분석**

- 정적 코드 분석 및 코드 품질 검사를 제공하여 잠재적인 문제, 코딩 표준 위반 및 기타 코드 관련 문제를 식별

> Ex) Gradle의 SpotBugs, Checkstyle 등등

**6. 환경 구성**

- 환경별 구성 및 속성을 관리할 수 있어 다양한 배포 환경에 맞게 애플리케이션 빌드 가능

**7. 자동화**

- 반복적인 작업을 자동화하여 수동 개입을 줄여줌- 일관성과 안전성을 보장

# JAVA 빌드 도구

자바 빌드 도구에는 Maven, Gradle, Ant, Bazel, Buildr, SBuild, Buildship 등등이 있지만 가장 많이 사용하는 두 가지는 Apache Maven과 Gradle이다.

## Apache Maven

'구성보다 규칙을 우선시하는 원칙'을 따르며 프로젝트 설정을 간소화하기 위해 특정 기본 규칙을 적용함

### Project Object Model (POM)

- 프로젝트 정보, 빌드 구성, 종속성 및 플러그인을 정의하는 XML 파일

- 프로젝트의 중앙 구성 파일 역할

- POM에 정의된 종속성은 원격 리포지터리에서 자동으로 다운로드하여 관리됨

```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>my-java-project</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <dependencies>
        <!-- Define your project's dependencies here -->
    </dependencies>
</project>
```

### LifeCycle

Phase의 시퀀스이며, 각 Phase는 순차적으로 실행되어야 함  

1. **Clean Lifecycle**
    
    - **clean :** 빌드 출력 디렉토리(target)를 삭제하여 깨끗한 빌드 환경을 보장
    
2. **Default Lifecycle**
    
    - **validate** : 프로젝트와 그 구성의 유효성 검사
    - **compile** : 소스 코드를 컴파일
    - **test** : 단위 테스트 실행
    - **package** : 컴파일된 코드의 배포 가능한 패키지(JAR, WAR)를 생성
    - **verify** : 통합 테스트를 실행하고 결과를 확인
    - **install** : 다른 프로젝트에서 사용할 수 있도록 패키지를 로컬 리포지토리에 설치
    - **deploy** : 다른 개발자와 공유하거나 서버에 배포하기 위해 원격 저장소에 패키지를 배포
    
3. **Site Lifecycle**
    
    - **site** : 프로젝트 문서 및 보고서 생성
    - **site-deploy** : 생성된 사이트 문서를 공유할 수 있도록 원격 위치에 배포
    

Goal을 사용하여 각 Phase내에서 특정 작업을 수행

Goal은 Phase에 바인딩되며, Phase 실행 시 해당 Phase와 관련된 Goal을 실행

> ex)  
> mvn package -> 'compile', 'test', 'package' Phase

## Gradle

### Groovy DSL (도메인 특화 언어)

- JVM을 위한 동적 스크립팅 언어인 Groovy로 작성

- 메타프로그래밍을 지원하는 동적 프로그래밍 언어이기 때문에 정적 언어와 다르게 런타임에 코드를 생성하고 조작할 수 있어서 빌드 스크립트를 유연하게 작성할 수 있음

- 자바에 비해 간결한 문법이기 때문에 읽기 쉽고 작성하기 편함

- 클로저를 지원하기 때문에 스크립트를 추상화할 수 있고 사용자 정의 기능을 쉽게 추가할 수 있음

- Convention Over Configuration을 지원하여 일반적인 작업에 대한 기본 구성을 제공하면서 필요한 경우 사용자 정의 설정을 추가할 수 있는 유연성을 제공함

```groovy
plugins {  
   id 'java'  
   id 'org.springframework.boot' version '3.0.5'  
   id 'io.spring.dependency-management' version '1.1.0'  
}  
  
jar {  
   enabled(false)  
} 
  
repositories {  
   mavenCentral()  
}  
  
dependencies {  
}  
  
tasks.named('test') {  
   useJUnitPlatform()  
}
```

### 증분 빌드

- 마지막 빌드 이후 변경된 프로젝트의 부분만 다시 빌드하고 테스트하므로 대규모 프로젝트일 경우, 타 빌드 도구보다 프로세스 속도가 빨라짐

### 의존성 캐싱

- 다운로드한 종속성을 캐싱하여 재다운로드하는 시간을 줄이고 빌드 시간을 단축

### 작업 지향 접근 방식

- 작업을 Task로 구성
- 사용자 지정 Task를 정의할 수 있으며, Task는 다른 Task에 종속성을 가질 수 있음
- 각 작업이 특정한 역할을 수행하도록 구조화하므로 명확하고 체계적인 빌드 프로세스를 만듦