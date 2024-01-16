# Lombok이란?

반복적인 코드 작성을 줄이고 보다 깔끔하고 가독성 높은 코드를 작성할 수 있도록 돕는다

**컴파일 시점**에 바이트코드를 변환하여 원하는 부분을 주입함

# 과정

1. 자바 컴파일러가 소스파일을 파싱하여 AST 트리를 만듦
2. Lombok AnnotationProcessor에 따라 AST 트리를 동적으로 수정하고 새 노드를 추가하고 바이트 코드를 분석 및 생성
3. 컴파일러는 Lombok Annotation Processor에 의해 수정된 AST를 기반으로 Byte code를 생성

## Annotation Processor

- 어노테이션에 대한 코드베이스를 검사, 수정, 생성하는 훅

1. 자바 컴파일러가 알고 있는 상태에서 컴파일을 수행
2. Annotation Processor 들이 각자의 역할에 맞게 구현되어 있는 상태에서 실행되지 않은 Annotaton Processor 실행
3. Annotation Processor 내부에서 Annotation에 대한 처리
4. 자바 컴파일러가 모든 Annotation Processor가 실행되었는지 검사하고, 모든 Annotation Processor가 실행되지 않았다면 반복

# 장점

- 코드 중복 감소
- 가독성 향상
- 유지보수 용이

# 예시

1. `@Getter` / `@Setter`
2. `@AllArgsContructor`, `@NoArgsConstructor`, `@RequiredArgsConstructor`
3. `@ToString`
4. `@Builder`

# 주의 사항

1. 디버깅 및 코드 추적의 복잡성
    - 소스 코드에 직접 나타나지 않는 코드를 생성하므로 디버깅 시에 실제 실행되는 코드를 추적하기 어려울 수 있음
2. API의 변경 가능성
    - Lombok은 라이브러리이므로, API가 변경될 수 있고 라이브러리 업데이트 시 발생할 수 있는 변경 사항에 민감해질 수 있다