## 도입 배경
- JSP 만으로 비즈니스 로직과 뷰 렌더링을 모두 처리하기에는 너무 많은 역할을 가지며 유지보수가 어려워짐
- 비즈니스 로직과 UI의 변경 주기가 다름
- JSP 같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 있기 때문에 이 업무만 담당하는 것이 가장 효과적
## 개요
- JSP로 처리하던 것을 Controller와 View 영역으로 역할 분리
![](https://i.imgur.com/6haft68.png)
### 컨트롤러
- HTTP 요청을 받아서 파라미터 검증
- 비즈니스 로직 실행
- 뷰에 전달할 결과 데이터를 조회해 모델에 담음
### 모델
- 뷰에 출력할 데이터를 담음
- 뷰가 비즈니스 로직이나 데이터 접근을 모르게 하며, 화면을 렌더링하는데 집중할 수 있게 함
### 뷰
- 모델에 담겨 있는 데이터를 사용해서 화면을 그림
## 사용
### 컨트롤러 & 모델
```java
package hello.servlet.web.servletmvc;
  
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")  
public class MvcMemberSaveServlet extends HttpServlet {  
  
    private final MemberRepository memberRepository = MemberRepository.getInstance();  
  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
       String username = request.getParameter("username");  
       int age = Integer.parseInt(request.getParameter("age"));  
  
       Member member = new Member(username, age);  
       memberRepository.save(member);  
  
       // Model에 데이터 보관  
       request.setAttribute("member", member);  
  
       String viewPath = "/WEB-INF/views/save-result.jsp";  
       RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);  
       dispatcher.forward(request, response);  
    }  
}
```
- `dispatcher.forward()` : 다른 서블릿이나 JSP로 이동할 수 있는 기능이며 서버 내부에서 다시 호출 발생
	> **Redirect vs Forward**
	> - 리다이렉트 : 클라이언트가 요청하며, 인지 가능하고 URL 경로도 실제로 변경
	> - 포워드 : 서버 내부에서 일어나는 호출이며 클라이언트 인지 X
- `/WEB-INF` : 외부에서 직접 JSP를 호출할 수 없음 -> 오직 컨트롤러를 통한 호출
## 뷰
```jsp
<%--  
  Created by IntelliJ IDEA.  User: seongha  Date: 1/14/25  Time: 20:18  To change this template use File | Settings | File Templates.--%>  
<%@ page contentType="text/html;charset=UTF-8" language="java" %>  
<html>  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
</head>  
<body>  
  
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">  
    username:   <input type="text" name="username" />  
    age:        <input type="text" name="age"/>  
    <button type="submit">전송</button>  
</form>  
  
</body>  
</html>
```
- 절대경로로 시작하려면 (`/save`), 상대경로로 시작하려면 (`save`)
- 현재 URL이 속한 계층 경로 + save 호출
	- 현재 계층 경로 : `/servlet-mvc/members/`
	- 결과 : `/servlet-mvc/members/save`
## 한계
### 포워드 중복
```java
RequestDispatcher dispather = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```
### ViewPath 중복
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```
- 만약 jsp가 아닌 다른 뷰로 변경 시 전체 코드를 모두 변경해야 함
### 사용하지 않는 코드
```java
HttpServletRequest request, HttpServletResponse response
```
### 공통 처리 어려움
- 기능이 복잡할 수록 공통으로 처리해야 하는 부분이 많아짐
- 공통 기능을 메서드로 뽑게 되면, 결과적으로 해당 메서드를 항상 호출해야 하며 호출하지 않으면 문제가 발생함
## 해결
- **프론트 컨트롤러 패턴**을 도입 -> 입구를 하나로 만듦