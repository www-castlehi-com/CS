# DTO
## 특징
- 순수하게 데이터를 담아 계층 간으로 전달하는 객체
- 로직을 갖고 있지 않으며 getter / setter 만을 갖음
## 사용
- Controller - Service 사이에서 사용

1) 비즈니스 로직과 분리 : 서비스 레이어의 내부 구현을 숨김
2) 데이터를 유연하게 조정하기 위함


# VO
## 특징
- 값 그 자체
- 불변성 보장을 위해 생성자 사용
- 로직 포함 가능

## 사용
- View - Controller 사이에서 사용

1) 도메인 모델의 직접적 반영 : 불변이기 때문에 도메인의 특정 상태를 안정적으로 표현하는데 유리