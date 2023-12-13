# Lombok이란?
반복적인 코드 작성을 줄이고 보다 깔끔하고 가독성 높은 코드를 작성할 수 있도록 돕는다

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