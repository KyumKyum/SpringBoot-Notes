# MVC Pattern

항상 Express 서버 기반의 router, 혹은 graphql로 백엔드 서버 개발을 구축, 기획 해왔던 나에겐 MVC Pattern은 개념만 아는 정도의 정보였다. 멘토링을 해주신 교수님 말씀으로도 언어는 상관없지만 Spring 처럼 생각하는 사고는 중요하다고 하셨고, 이 패턴 또한 그 중 하나인 것 같다. 여기에 MVC 패턴은 어떻게 구현이 되어있는지 간단하게 서술한다.

## MVC

- MVC는 Model, View, Controller의 줄임말이므로, 각각 다음과 같은 것을 뜻한다.
    - Model: 데이터, DB와 같은 정보를 뜻한다.
    - View: 사용자에게 보여지는 화면, UI를 뜻한다. Model로 정보를 얻은 후 이를 표시한다.
    - Controller: Model과 View 사이에서 데이터와 Business Logic 간 상호 동작을 관리한다. Model과 View가 직접적인 소통을 하지 않도록 한다.
- Spring MVC는 다음과 같이 작동한다.
    1. 유저가 브라우저에서 특정 URL로 요청을 한다. 해당 요청은 DispatcherServlet에 의해 처리된다.
        - Servlet: Web Application Server에서 실행되는 Java Program으로서, 클라이언트로부터 요청을 받고 응답을 return함.
    2. 헤당 요청에 해당하는 (즉 요청받은 uri에 해당하는) controller을 반환한다. 이를 위해 HanlderMapping에게서 해당하는 HandlerMethod를 return한다.
    3. Return된 HandlerMethod를 통해 해당하는 Controller에서 요청을 처리한다.
        - Controller에서 Repository등등과 관련된 데이터 처리, 요청 처리 등등을 진행한다. Service Logic, Business Logic, Model 처리 등이 Controller하에서 처리된다고 생각하자.
    4. Controller의 처리 결과는 View로 생성되게 된다. 이때 DispatcherServlet은 ViewResolver를 통해 해당하는 View를 mapping시킨다. (+.html)
    5. 해당 View를 DispatcherServer은 실행하고, 응답을 브라우저로 보낸다.
    6. 

## DTO/DAO

  Controller가 Database에 직접적으로 접근하기보다는, 이를 처리하는 방법을 하나하나 나눈 방법이다. 나중에 서비스 확장성을 위해 익혀두는 것이 좋다.

### DTO - Data Transfer Object

- 데이터 transfer. Database에 접근하는 로직이 아닌, 오직 getter/setter를 가지고 있는 데이터 객체이다.
- *VO: Value Object로서, getter만 가지고 있는 DTO의 Read-only version이다. DTO는 데이터를 전달하는 일종의 상자의 개념이라면, VO는 Value를 표현하는 constant value의 개념이다.

### DAO - Data Access Object

- 데이터 Access. Database에 직접 접근하여 CRUD를 실행한다.

> 기본 원리
> 
- Creation: 유저가 입력한 데이터는 곧 바로 Create되는 것이 아니라, DTO object에 넣어져서 보내진다. 이후 서버는 DAO를 사용하여 DTO의 데이터를 Database에 저장한다.
- Query: 유저가 검색한 데이터에 대해서는 DAO object를 통해 query합니다. 해당 데이터를 DTO object에 넣어 유저가 요청한 데이터만 보여줍니다.

## Directory Structure

  Spring Boot의 MVC를 기반으로 한 dir 구조이다. 위의 MVC의 개념을 충실히 따른 구조이니 최대한 따르도록 연습하자.

### /Controller

- Annotation: **@Controller, @RestController**
- URI에 해당하는 Controller class들이 위치해있으며, Request에 대해 해당하는 controller는 해당 요청을 GET/POST 등에 따라 service로 전달.

### /Service (/api)

- Annotation: @Service
- Controller의 요청을 처리하기 위한 Business/Service Logic을 처리한다

### /Domain

- Annotation: @Entity
- DTO 관리.

### /Repository

- Annotation: @Repository
- DAO 관리. 실제 Database와 접근 후 처리