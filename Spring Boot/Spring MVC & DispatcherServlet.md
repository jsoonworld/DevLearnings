# 목차
- Spring MVC란 무엇인가?
- Spring MVC의 등장 배경
- DispatcherServlet 의 역할
- DispatcherServlet 의 동작 과정

# Spring MVC

면접관: Spring을 사용하는 이유는 무엇인가요?

???: 유연함과 확장성 때문입니다.

면접에서는 위와 같이 답할 것 같지만 실제로 유연함과 확장성에서 편리함을 느껴서
Spring을 사용하고 있는지는 모르겠다. 그렇다는 건 반대로 아직 spring의 유연함과 확장성에 대해서 잘 모르기 때문에 그런 게 아닐까? 이번 기회를 통해서 Spring의 유연함과 확장성에 대해서 파악하고 발전시켜 Spring을 제대로 사용해보자!

```java
@RestController  
@RequestMapping("/member")  
@RequiredArgsConstructor  
public class MemberController {  
  
    private final MemberService memberService;  
  
    @PostMapping  
    public ResponseEntity<SignupResponse> save(@RequestBody final SignupRequest request) {  
        return ResponseEntity.ok(memberService.save(request));  
    }  
}
```

위와 같이 작성하면 내부 동작을 모르고도 간편하게 구현이 가능했기에 숨어있는 유연함과 확장성에 대해서 느끼지 못했던 것일까? 당연해서..? a.k.a Spring Framework의 편리한 기분 구조 덕분..

Spring Framework 는 OCP 원칙 즉, Open for extension, Closed for modification Principal 을 지향하며 만들어졌다. 이런 확장성을 활용해서 Spring JDBC, Spring MVC, Spring AOP, Spring ORM 와 같은 여러가지 Framework를 만들었다. 이러한 Framework들은 기존에 불편함들을 개선하고자 만들어졌고 그 중 WEB에 최적화된 framework 가 Spring MVC이다.

## What is Spring Web MVC?

### Spring Web MVC

Spring Web MVC is the original web framework built on the Servlet API and has been included in the Spring Framework from the very beginning. The formal name, "Spring Web MVC," comes from the name of its source module ([`spring-webmvc`](https://github.com/spring-projects/spring-framework/tree/main/spring-webmvc)), but it is more commonly known as "Spring MVC".

## Keyword

- Spring: spring 을 활용한다.
- Web: web에 최적화된 프레임워크다.
- MVC: MVC 기반으로 만들어져있다.

## Spring MVC의 등장 배경

기본적으로 framework는 기존의 불편함을 개선하고자 만들어졌다.

- 중복코드
- 디버깅 어려움
- 확장 어려움
- 테스트 어려움
- 유지보수 어려움
- 등등,,

그렇다면 어떻게 이러한 불편함을 해소하였을까?

Spring MVC를 카페 운영에 비유하여 설명해보겠습니다.

### 카페 운영 비유

**1. 사장님 (Controller):** 사장님은 카페의 운영을 총괄합니다. 고객이 무엇을 원하고 있는지 확인하고 그 요청을 적절한 직원(알바)에게 전달합니다. 사장님은 고객과 직원들 사이에서 중요한 역할을 합니다.

**2. 직원 (Service):** 직원들은 사장님의 지시에 따라 실제 업무를 수행합니다. 예를 들어, 바리스타는 커피를 만들고, 캐셔는 계산을 하고, 청소 직원은 카페를 청소합니다. 각각의 직원은 자신의 역할에 맞는 업무를 처리합니다.

**3. 메뉴판 (View):** 메뉴판은 고객에게 제공되는 정보입니다. 메뉴판에는 다양한 커피와 디저트가 나열되어 있고, 고객은 이 메뉴판을 보고 주문을 결정합니다. 메뉴판은 고객과의 직접적인 인터페이스입니다.

**4. 재료 창고 (Model):** 재료 창고에는 커피 원두, 우유, 설탕 등 다양한 재료가 보관되어 있습니다. 직원들이 커피를 만들 때 필요한 재료는 모두 이 창고에서 가져갑니다. 재료 창고는 데이터를 저장하고 제공하는 역할을 합니다.

### Spring MVC 비유

**1. Controller:** Spring MVC에서 Controller는 카페 사장님과 같습니다. 사용자가 웹 애플리케이션에 요청을 보내면, Controller는 이 요청을 받아서 어떤 작업을 해야 할지 결정하고, 적절한 Service를 호출합니다.

**2. Service:** Service는 Spring MVC에서 실제 비즈니스 로직을 처리하는 부분입니다. 직원들이 실제로 커피를 만들고, 계산을 처리하는 것처럼, Service는 데이터를 처리하고 필요한 작업을 수행합니다.

**3. View:** View는 메뉴판과 같습니다. 사용자가 볼 수 있는 웹 페이지입니다. Spring MVC에서는 주로 JSP, Thymeleaf 등을 사용하여 View를 구성합니다. View는 Controller에서 전달된 데이터를 사용자에게 표시합니다.

**4. Model:** Model은 재료 창고와 비슷합니다. 애플리케이션의 데이터를 저장하고 관리하는 역할을 합니다. Controller는 Model에서 데이터를 가져와서 View에 전달합니다.

### 실생활 예시

1. **고객이 카페에 들어와서 커피를 주문합니다. (HTTP Request)**
2. **사장님이 주문을 받아 어떤 직원에게 전달할지 결정합니다. (Controller가 요청을 받음)**
3. **바리스타에게 커피를 만들라고 지시합니다. (Controller가 Service를 호출)**
4. **바리스타는 재료 창고에서 커피 원두, 우유 등을 가져와서 커피를 만듭니다. (Service가 Model에서 데이터를 가져와 처리)**
5. **완성된 커피를 고객에게 전달합니다. (Service의 결과를 Controller가 받아서 View에 전달)**
6. **고객은 메뉴판에 있는 커피를 받습니다. (View에서 데이터를 사용자에게 표시)**

참 쉽죠!?

코드로 살펴보기 전에 서블릿(Servlet)의 개념을 알고 가면 좋을 것 같습니다.

Servlet은 Java 기반의 웹 애플리케이션을 만들기 위한 기술로, 서버 측에서 실행되는 프로그램입니다. Servlet은 클라이언트(주로 웹 브라우저)로부터 HTTP 요청을 받아서 그에 대한 응답을 생성하는 역할을 합니다. 웹 서버(예: Apache Tomcat)에 의해 실행되며, 웹 애플리케이션의 비즈니스 로직을 처리하는 데 사용됩니다.

### Servlet의 주요 개념

1. **Servlet 클래스:**
    
    - Javax.servlet.http.HttpServlet 클래스를 상속받아야 합니다.
    - HTTP 요청을 처리하기 위해 doGet(), doPost() 등의 메서드를 오버라이드합니다.
2. **Servlet 라이프 사이클:**
    
    - **init():** Servlet이 처음 로드될 때 한 번 호출됩니다. 초기화 작업을 수행합니다.
    - **service():** 각 HTTP 요청마다 호출됩니다. 요청의 메서드 타입(GET, POST 등)에 따라 doGet(), doPost() 등을 호출합니다.
    - **destroy():** Servlet이 언로드될 때 한 번 호출됩니다. 리소스를 해제하는 등의 정리 작업을 수행합니다.
3. **HTTP 요청 및 응답:**
    
    - **HttpServletRequest:** 클라이언트의 요청 정보를 담고 있는 객체입니다. 요청 파라미터, 헤더 정보 등을 제공합니다.
    - **HttpServletResponse:** 서버의 응답 정보를 담고 있는 객체입니다. 응답 데이터, 상태 코드 등을 설정할 수 있습니다.

### Servlet의 동작 방식

1. **클라이언트 요청:**
    
    - 클라이언트(웹 브라우저)가 특정 URL로 HTTP 요청을 보냅니다.
2. **웹 서버 요청 처리:**
    
    - 웹 서버가 요청을 받아 해당 URL과 매핑된 Servlet을 찾습니다.
3. **Servlet 호출:**
    
    - 웹 서버는 해당 Servlet의 인스턴스를 생성(이미 생성된 경우 재사용)하고, service() 메서드를 호출합니다.
    - service() 메서드는 요청의 HTTP 메서드 타입(GET, POST 등)에 따라 doGet(), doPost() 등을 호출합니다.
4. **응답 생성:**
    
    - doGet(), doPost() 메서드는 비즈니스 로직을 처리하고, HttpServletResponse 객체를 사용해 응답을 생성합니다.
    - 생성된 응답은 웹 서버를 통해 클라이언트로 전송됩니다.

### 예제 코드

간단한 Servlet 예제를 통해 위 개념들을 설명하겠습니다.
```java
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloWorldServlet extends HttpServlet {
    
    @Override
    public void init() throws ServletException {
        // Servlet 초기화 코드
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // 클라이언트가 GET 요청을 보낼 때 처리하는 코드
        response.setContentType("text/html");
        response.setCharacterEncoding("UTF-8");
        response.getWriter().println("<html><body>");
        response.getWriter().println("<h1>Hello, World!</h1>");
        response.getWriter().println("</body></html>");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        // 클라이언트가 POST 요청을 보낼 때 처리하는 코드
        // 예를 들어, 폼 데이터를 처리하는 로직
    }

    @Override
    public void destroy() {
        // Servlet 종료 시 정리 작업
    }
}
```

### Servlet의 장점

- **확장성:** 여러 클라이언트 요청을 동시에 처리할 수 있습니다.
- **안정성:** Java의 장점인 안정성과 강력한 예외 처리 메커니즘을 제공합니다.
- **유연성:** 다양한 프로토콜과의 통합이 가능하며, 다른 Java EE 컴포넌트와 쉽게 통합됩니다.

### Servlet의 단점

- **복잡성:** 초기 설정 및 구성 과정이 복잡할 수 있습니다.
- **유지 보수:** 대규모 애플리케이션에서는 Servlet 코드가 복잡해져 유지 보수가 어려울 수 있습니다.


이제 Servlet에 대해서 알아봤으니 Servlet의 불편함을 해결하기 위한 Spring MVC가 탄생하게 된 과정을 이해해봅시다!

Spring Web MVC는 Servlet의 중복 코드 문제와 복잡한 요청 처리 로직을 해결하기 위해 만들어졌습니다. Servlet을 사용한 전통적인 Java 웹 애플리케이션 개발 방식은 요청 처리, 비즈니스 로직, 뷰 렌더링 등의 코드가 혼재되어 복잡성과 유지보수성 문제가 있었습니다. Spring Web MVC는 이러한 문제를 해결하기 위해 다양한 기능과 패턴을 도입했습니다.

### Servlet의 문제점

1. **중복 코드:**
    - 각 Servlet에서 공통적으로 사용되는 초기화, 요청 처리, 응답 생성 코드가 반복됩니다.
2. **복잡성:**
    - 요청 처리 로직과 비즈니스 로직, 뷰 렌더링 코드가 한 곳에 섞여 있어서 코드가 복잡하고 유지보수가 어렵습니다.
3. **확장성 부족:**
    - 새로운 기능을 추가하거나 변경하기 어렵습니다. 코드의 응집도가 높아 변경이 다른 부분에 영향을 미칠 수 있습니다.

### Spring Web MVC의 등장 배경

Spring Web MVC는 이러한 문제를 해결하고자 등장했습니다. Spring Web MVC는 MVC 패턴을 사용하여 애플리케이션을 명확하게 분리하고, 중복 코드를 줄이며, 유지보수성을 높였습니다.

### Spring Web MVC의 핵심 개념

1. **DispatcherServlet:**
    - 모든 요청을 중앙에서 처리하는 프론트 컨트롤러 역할을 합니다. 요청을 적절한 컨트롤러로 위임합니다.
2. **Controller:**
    - 사용자 요청을 처리하고 모델 데이터를 준비하여 적절한 뷰로 반환합니다. 비즈니스 로직은 서비스 레이어로 위임합니다.
3. **Service:**
    - 비즈니스 로직을 처리하고, 데이터 액세스 로직을 DAO 레이어로 위임합니다.
4. **Model:**
    - 애플리케이션에서 사용되는 데이터 구조를 정의합니다. 컨트롤러는 모델 객체를 생성하고 뷰에 전달합니다.
5. **View:**
    - 모델 데이터를 사용자에게 표시합니다. JSP, Thymeleaf 등 다양한 뷰 템플릿을 사용할 수 있습니다.

### Spring Web MVC의 동작 과정

1. **요청 수신 (DispatcherServlet):**
    - 클라이언트 요청이 DispatcherServlet으로 전달됩니다. DispatcherServlet은 모든 요청을 중앙에서 처리하고 적절한 컨트롤러로 전달합니다.
2. **핸들러 매핑 (HandlerMapping):**
    - DispatcherServlet은 요청 URL을 분석하여 적절한 컨트롤러를 찾습니다. 이는 HandlerMapping 인터페이스를 통해 이루어집니다.
3. **컨트롤러 호출:**
    - HandlerMapping에 의해 선택된 컨트롤러가 호출됩니다. 컨트롤러는 요청을 처리하고, 비즈니스 로직을 수행하며, 모델 데이터를 준비합니다.
4. **모델과 뷰 반환:**
    - 컨트롤러는 처리 결과를 모델에 담고, 뷰 이름을 DispatcherServlet에 반환합니다. DispatcherServlet은 ViewResolver를 사용하여 뷰를 찾습니다.
5. **뷰 렌더링 (ViewResolver):**
    - ViewResolver는 뷰 이름에 해당하는 실제 뷰를 찾고, 모델 데이터를 사용하여 뷰를 렌더링합니다.
6. **응답 전송:**
    - 최종적으로 렌더링된 뷰는 클라이언트에게 응답으로 전송됩니다.

# Spring MVC Architecture

![](https://velog.velcdn.com/images/harperkwon/post/c4a5eac1-8309-4419-8f50-47c55805075e/image.png)


여기서 보이는 Front Controller의 역할이 바로 Dispatcher Servlet 이라고 볼 수 있습니다.
# Dispatcher Servlet 이란?

The DispatcherServlet is **the front controller in Spring web applications**. It's used to create web applications and REST services in Spring MVC. In a traditional Spring web application, this servlet is defined in the web. xml file.

- Dispatch : 보내다 / 파견하다
- Servlet : 웹 요청과 응답을 처리하는 객체

# Dispatcher Servlet 의 역할

Spring MVC 구조에서 웹 어플리케이션이 동작할 수 있도록 만드는 엔진

## 동작 방법 

![](https://velog.velcdn.com/images/harperkwon/post/07949fab-bf34-4500-8158-870189d5c64b/image.png)


## 역할

### 1. 공통 작업 처리 후 컨트롤러에게 요청을 위임

- **요청을 할 때 `HttpServletRequest`로 반환을 해서 Controller에 전달을 함:**
  - 사용자가 특정 URL로 요청을 보내면, 이 요청은 먼저 `DispatcherServlet`으로 전달됩니다.
  - `DispatcherServlet`은 이 요청을 `HttpServletRequest` 객체로 포장하여 처리합니다.

- **전달을 해야할 Controller를 찾아주는 `HandlerMapping`을 통해 Controller를 조회 해줌:**
  - `DispatcherServlet`은 요청을 처리하기 위해 적절한 컨트롤러를 찾아야 합니다.
  - 이를 위해 `HandlerMapping`을 사용하여 요청 URL에 매핑된 컨트롤러를 조회합니다.

- **보낼 때는 `HandlerAdapter`가 알맞은 형태(파라미터)를 맞춰 반환 해둔다 (like MemberSaveRequest에 맞게 변환할 수 있도록):**
  - `HandlerMapping`을 통해 컨트롤러를 찾은 후, `DispatcherServlet`은 `HandlerAdapter`를 사용하여 요청을 처리할 수 있는 형태로 변환합니다.
  - `HandlerAdapter`는 컨트롤러 메서드가 필요로 하는 파라미터를 준비하고, 요청 데이터를 적절한 객체로 변환하여 컨트롤러에 전달합니다. 예를 들어, JSON 데이터를 `MemberSaveRequest` 객체로 변환할 수 있습니다.

- **이로써 Controller는 제약 없이 확장성 있게 대응이 가능하다:**
  - `HandlerAdapter` 덕분에 컨트롤러는 요청 데이터를 직접 처리할 필요가 없으며, 필요한 형태로 제공된 데이터를 사용하여 비즈니스 로직을 처리할 수 있습니다.
  - 이는 컨트롤러가 다양한 요청 형식에 유연하게 대응할 수 있게 해줍니다.

### 2. 뷰에게 모델을 전달하여 최종 결과물 생성:

1. **Controller에서 받은 ViewName을 토대로 ViewResolver가 View를 생성합니다:**
    - Controller는 비즈니스 로직을 수행한 후 뷰 이름(ViewName)과 모델(Model)을 반환합니다.
    - DispatcherServlet은 ViewName과 Model을 받습니다.
    - ViewResolver는 ViewName을 사용하여 실제 View 객체를 찾거나 생성합니다.

2. **View는 Model에 전달받은 값을 사용하여 최종 결과물을 생성합니다:**
    - View 객체는 Model 데이터를 사용하여 HTML, JSON 등 최종 결과물을 렌더링합니다.
    - View 객체는 Model에서 데이터를 꺼내어 클라이언트에게 보낼 내용을 생성합니다.
    - 최종 결과물은 DispatcherServlet에 의해 클라이언트에게 전송됩니다.

# Dispatcher Servlet의 역할 정리

1. 클라이언트의 요청을 받아 공통적인 작업을 수행합니다.
2. 컨트롤러로 세부 작업을 위임합니다.
3. 클라이언트에게 보낼 뷰를 선택해서 최종 결과를 생성합니다.

이렇게,,,Spring의 유연함과 확장성을 시작으로 기존의 불편함을 해소해주는 Spring MVC를 알아보았습니다..이제는 면접에서 spring mvc 왜 사용하나요 -> 확장성, 유연성!!



