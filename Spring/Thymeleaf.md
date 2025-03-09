# 사용 선언
`<html xmlns:th="http://www.thymeleaf.org>`
# 핵심
- `th:xxx`가 붙을 경우 **서버 사이드 렌더링** 되며, 기존 것을 대체. 없을 경우, 기존 html의 속성이 적용
- HTML을 직접 열 경우, 웹 브라우저는 `th:` 속성을 알지 못하므로 무시됨
# 특징
## 1️⃣ 서버 사이드 렌더링 (SSR)
- 백엔드 서버에서 HTML을 동적으로 렌더링하는 용도로 사용
## 2️⃣ 네츄럴 템플릿
- 순수 HTML을 최대한 유지하면서 뷰 템플릿으로 사용
## 3️⃣ 스프링 통합 지원
- 스프링과 통합되며, 스프링의 다양한 기능을 편리하게 사용할 수 있도록 지원
# 기능
## 속성 변경
### th:href
`th:href="@{/css/bootstrap.min.css}"`
- `href="xxx"`을 `th:href="xxx"`의 값으로 변경
#### URL 링크 표현식
- URL 링크 사용
- 서블릿 컨텍스트를 자동으로 포함
- 경로 변수 뿐만 아니라 쿼리 파라미터 생성 가능
	`th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}`
	= `th:href="@{|/basic/items/${item.id}|}"`
	=> `http://localhost:8080/basic/items/1?query=test`
### th:onclick
`th:onclick="|location.href='@{/basic/items/add}'|"`
#### 리터럴 대체
- `|...|`
- 타임리프에서 문자와 표현식은 분리되어 있어 더해서 사용해야 함
- 리터럴 대체 문법을 사용할 경우, 더하기 없이 편리하게 사용 가능
	`<span th:text="|Welcome to our application, ${user.name}!|">`
### th:action
- `action`에 값이 없을 경우 현재 URL에 데이터를 전송
- HTTP 메서드로 기능 구분
## 반복
### th:each
`<tr th:each="item : ${items}">`
- 모델에 포함된 컬렉션 데이터가 변수에 하나씩 포함됨
- 컬렉션의 수 만큼 `<tr>..</tr>` 이 하위 태그를 포함해서 자동 생성
- `List`, 배열, `java.util.Iterable`, `java.util.Enumeration`을 구현한 모든 객체를 사용 가능
- `Map`을 사용할 경우 `Map.Entry`가 변수에 담김
### 반복 상태 유지
`<tr th:each="user, userStat : ${users}">`
- 두번째 파라미터를 설정해서 반복 상태 확인
- 생략 가능하며, 생략 시 지정한 변수명 (`user`) + `Stat`이 됨
#### 기능
- `index` : 0부터 시작하는 값
- `count` : 1부터 시작하는 값
- `size` : 전체 사이즈
- `even`, `odd` : 홀수, 짝수 여부 (`boolean`)
- `first`, `last` : 처음, 마지막 여부 (`boolean`)
- `current` : 현재 객체
#### 변수 표현식
- `${...}`
- 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회
- 프로퍼티 접근법 사용 (`item.getPrice()`)
## 텍스트
### th:text
`<td th:text="${item.price}">10000</td>`
- 내용의 값을 `th:text`의 값으로 변경
- HTML 콘텐츠 영역 안에서 직접 데이터를 출력할 경우 `[[$data]]`로 사용
- Unescape시 `th:utext`, `[(...)]`로 변경

> **Escape**
> HTML에서 사용하는 특수 문자를 HTML 엔티티로 변경하는 것
> - HTML 문서는 `<`, `>` 같은 특수 문자를 기반으로 정의
> - `<b></b>`로 강조를 원할 경우에도 `&lt;b&gt`로 변경됨 
> - escape를 사용하지 않을 경우 HTML이 정상 렌더링 되지 않는 경우가 있으므로 필요할 때만 Unescape를 사용
## 변수 - SpringEL
### `.username`
- `user.username` : user의 username 프로퍼티 접근 (`user.getUsername()`)
- `users[0].username` : List에서 첫 번째 회원을 찾고 username 프로퍼티 접근 (`list.get(0).getUsername()`)
- `userMap['userA'].username` : Map에서 userA를 찾고, username 프로퍼티 접근 (`map.get("userA").getUsername()`)
### `['username']`
- `user['username']` : user의 username 프로퍼티 접근 (`user.getUsername()`)
- `users[0]['username']` : List에서 첫 번째 회원을 찾고 username 프로퍼티 접근 (`list.get(0).getUsername()`)
- `users[0].getUsername()` : List에서 첫 번째 회원을 찾고 메서드 직접 호출
### `.getUsername()`
- `user.getUsername()` : user의 `getUsername()` 직접 호출
-  `users[0].getUsername()` : List에서 첫 번째 회원을 찾고 메서드 직접 호출
- `userMap['userA'].getUsername()` : Map에서 userA를 찾고 메서드 직접 호출
### 지역 변수
- `th:with`
- 선언한 태그 안에서만 사용
## 기본 객체
`#xx`는 `HttpServletRequest` 객체가 그대로 제공되기 때문에 데이터를 조회하려면 `reuqest.getParameter("data")` 처럼 불편하게 접근해야 함
- `${#request}`
- `${#response}`
- `${#session}`
- `${#servletContext}`
- `${#locale}`
### 편의 객체
- `param` : HTTP 요청 파라미터 접근
	- _`${param.paramData`_
- `session` : HTTP 세션 접근
	- _`${session.sessionData}`_
- `@` : 스프링 빈 접근
	- _`${@helloBean.hello('Spring!')}`_
## 유틸리티 객체
https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-utility-objects
### 자바 8 날짜
`LocalDate`, `LocalDateTime`, `Instant` 지원
#### 라이브러리
`thymeleaf-extras-java8time`

> 스프링 부트 3.2이상을 사용할 경우, 기본 포함
#### 유틸리티 객체
`#temporals`
## URL 링크
### 단순한 URL
`@{/hello}` -> `/hello`
### 쿼리 파라미터
`@{/hello(param1=${param1}, param2=${param2})}` -> `/hello?param1=data1&param2=data2`
- `()`에 있는 부분은 쿼리 파라미터로 처리
### 경로 변수
`@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}` -> `/hello/data1/data2`
- URL 경로상에 변수가 있으면 `()` 부분은 경로 변수로 처리

> 경로 변수와 파라미터 함께 사용 가능
## 리터럴
소스 코드 상에 고정된 값
- 문자 : `'hello'`
- 숫자 : `10`
- 불린 : `true`, `false`
- null : `null`

**문자 리터럴**은 항상 `'` (작은 따옴표)로 감싸야 하지만, 공백 없이 쭉 이어진다면 생략 가능
### 리터럴 대체
`<span th:text="|hello ${data}|">`
## 속성 값 설정
### 타임리프 태그 속성 (Attribute)
#### 속성 설정
`th:*`로 속성 적용 시 기존 속성을 대체하고, 없으면 새로 만듦
#### 속성 추가
- `th:attrappend` : 속성 값의 **뒤**에 값을 추가
- `th:attrprepend` : 속성 값의 앞에 값을 추가
- `th:classappend` : class 속성에 추가
#### checked 처리
HTML에서는 checked 속성이 있을 경우 무조건 checked 처리가 됨
타임리프의 경우, `th:checked = false`일 경우, `checked` 속성 자체를 제거
## 조건부 평가
### if, unless
해당 조건이 맞지 않을 경우 태그 자체를 렌더링하지 않음
### switch
`*`은 만족하는 조건이 없을 때 사용하는 default
## 주석
### 표준 HTML 주석
- `<!-- -->`
- 타임리프가 렌더링 하지 않고, 남겨둠
### 타임리프 파서 주석
- `<!--/* */-->`
- 렌더링에서 주석 부분 제거
### 타임리프 프로토타입 주석
- `<!--/*/ /*/-->`
- 웹 브라우저에서 열 경우, HTML 주석이기 때문에 웹 브라우저가 렌더링하지 않음
- 타임리프 렌더링을 할 경우, 정상 렌더링 됨
## 블록
```java
<th:block th:each="user : ${users}">
	<div>
		사용자 이름1 <span th:text="${user.username}"></span>
		사용자 나이1 <span th:text="${user.age}"></span> 
	</div>
	<div>  
		요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
	</div>
 </th:block>
```
- `<th:block>`
- HTML 태그가 아닌 타임리프의 유일한 자체 태그
## 자바스크립트 인라인
- 자바스크립트에서 타임리프를 편리하게 사용할 수 있는 기능
- `<script th:inline="javascript">`
### 텍스트 렌더링
`var username = [[${user.username}]];`
- 인라인 사용 전 : `var username = userA;`
- 인라인 사용 후 : `var username = "userA";`
### 네추럴 템플릿
`var username2 = /*[[${user.username}]]*/;`
- 인라인 사용 전 : `var username2 = /*userA*/;`
- 인라인 사용 후 : `var username2 = "userA";`
### 객체
`var user = [[${user}]];`
- 인라인 사용 전 : `var user = BasicController.User(username=userA, age=10);`
- 인라인 사용 후 : `var user = {"username":"userA", "age":10};`

인라인 사용 전, 객체의 `toString()` 호출
인라인 사용 후, 객체를 JSON으로 변환
## 템플릿 조각
일부 코드 조각을 가지고와서 사용
### insert
- `<div th:insert="~{template/fragment/footer :: copy}"></div>`
- 현재 태그(`div`) 내부에 추가
### replae
- `<div th:replace="~{template/fragment/footer :: copy}"></div>
- 현재 태그(`div`) 대체
### 파라미터 사용
- `<div th:relace="~{template/fragment/footer :: copyParam ('데이터1', '데이터2')}"></div>`
- 파라미터 전달 후 동적으로 조각 렌더링
## 템플릿 레이아웃
- 코드 조각을 레이아웃에 넘겨서 사용
- `<head>`에 공통 정보들을 한 곳에 모아두고 공통으로 사용하되, 각 페이지마다 필요한 정보를 더 추가해서 사용
### 확장 
- `<html>` 전체에 적용
- 레이아웃 페이지의 `<html>`에 `th:fragment` 속성을 정의하여 필요한 내용을 부분부분 변경
- 메인 페이지의 `<html>` 자체를 `th:replace`를 사용해서 변경
## 체크 박스
### HTML
#### 코드
`<input type="checkbox" id="open" name="open" class="form-check-input">`
#### 상황
- 체크 박스 선택 : HTML Form에서 `open=on`을 넘김 -> 스프링에서 `on`을 `true` 타입으로 변경
- 체크 박스 미선택 : HTML Form에서 `open`이라는 필드 자체를 서버로 전송하지 않음
#### 문제
수정의 경우, 사용자가 의도적으로 체크된 값을 해제해도 아무 값도 넘어가지 않아 서버에서는 값이 오지 않은 것으로 판단하여 변경하지 않을 수 있음
#### 해결
**히든 필드**
`<input type="hidden" name="_open" value="on"/>`
- 체크 박스 선택 : `open=on&_open=on`, 스프링 MVC가 `open`에 값이 있는 것을 확인 후 사용하며 `_open`은 무시
- 체크 박스 미선택 : `_open=on`, 스프링 MVC가 `_open`만 있는 것을 확인하고, `open`의 값이 체크되지 않았다고 인식하여 `false`로 전달
### 타임리프
#### 코드
`<input type="checkbox" th:field="*{open}" class="form-check-input">`
#### 결과
```html
<div class="form-check">
	<input type="checkbox" id="open" class="form-check-input" name="open" value="true"> 
	<input type="hidden" name="_open" value="on"/>  
	<label for="open" class="form-check-label">판매 오픈</label>
</div>
```
히든 필드와 관련된 부분도 함께 추가
## 오류 검증
### `#fields`
- `BindingResult`가 제공하는 검증 오류 접근
### th:errors
- 해당 필드에 오류가 있는 경우 태그 출력
### th:errorclass
- `th:errors`에서 지정한 필드에 오류가 있을 경우 `class `정보 추가