## 개요 
- 요청 이후 사용자가 인증되었는지 감시, 인증된 경우 [[세션 고정 보호]],  [[동시 세션 제어]] 등을 수행하기 위해 설정된 `SessionAuthenticationStrategy`를 호출하는 필터 클래스
- 스프링 시큐리티 6 이상에서는 SessionManagementFilter가 기본적으로 설정되지 않고 API 설정을 통해 생성
## 구성
![](https://i.imgur.com/ROhZNTb.png)
- `SessionManagementFilter`는 내부적으로 `SessionAuthenticationStrategy`를 가짐
- `SessionAuthenticationStrategy`는 네 가지 구현체
