# Toby’s Spring 1Week

# vol2

## 1.1 IoC 컨테이너

스프링 어플리케이션에서는 오브젝트의 생성과 관계 설정, 사용, 제거 등의 작업을 애플리케이션 코드 대신 독립된 컨테이너가 담당한다. 이를 컨테이너가 코드 대신 오브젝트에 대한 제어권을 갖고 있다고 해서 IoC라고 부른다.

⇒ 객체 관리의 많은 부분이 개발자 코드가 아닌 Spring 컨테이너에 의해 자동으로 처리된다는 의미

- Spring을 사용하지 않는 경우와 사용하는 경우의 코드를 비교
    
    <spring을 사용하지 않는 경우 - 개발자 코드로 객체 관리>
    
    ```java
    // 서비스 클래스
    public class UserService {
        private UserRepository userRepository;
    
        public UserService() {
            this.userRepository = new UserRepository();
        }
    
        public void registerUser(User user) {
            userRepository.save(user);
        }
    }
    
    // 메인 클래스
    public class Application {
        public static void main(String[] args) {
            UserService userService = new UserService();
            User user = new User("John");
            userService.registerUser(user);
        }
    }
    ```
    
    <spring을 사용하는 경우 - Spring 컨테이너에 의해 자동으로 처리>
    
    ```xml
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <bean id="userRepository" class="com.example.UserRepository"/>
    
        <bean id="userService" class="com.example.UserService">
            <property name="userRepository" ref="userRepository"/>
        </bean>
    </beans>
    
    ```
    
    ```java
    //서비스 클래스
    public class UserService {
        private UserRepository userRepository;
    
        // Setter for dependency injection
        public void setUserRepository(UserRepository userRepository) {
            this.userRepository = userRepository;
        }
    
        public void registerUser(User user) {
            userRepository.save(user);
        }
    }
    
    //메인 클래스
    public class Application {
        public static void main(String[] args) {
            ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
            UserService userService = context.getBean(UserService.class);
    
            User user = new User("John");
            userService.registerUser(user);
        }
    }
    ```
    
- 스프링 컨테이너
    - 스프링 컨테이너란?
        
        자바 객체의 생명 주기를 관리하며, 생성된 자바 객체(Bean)들에게 추가적인 기능을 제공한다.
        즉, 내부에 존재하는 빈의 생명주기를 관리하며, 생성된 빈에게 추가적인 기능을 제공하는 스프링 프레임워크의 핵심 컴포넌트이다.
        
        ![프로젝트 mysite3/src/main/resources/applicationContext.xml](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled.png)
        
        프로젝트 mysite3/src/main/resources/applicationContext.xml
        

- **스프링 컨테이너의 종류**

![Untitled](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%201.png)

스프링에선 IoC를 담당하는 컨테이너를 빈 팩토리 또는 어플리케이션 컨텍스트라고 부른다.

빈 팩토리: DI 작업

애플리케이션 컨텍스트: DI 작업 + DI를 위한 빈 팩토리에 엔터프라이즈 애플리케이션을 개발하는 데 필요한 여러가지 컨테이너 기능을 추가한 것

⇒ 스프링 컨테이너 or IoC 컨테이너라고 말하는 것은 ApplicationContext 인터페이스를 구현한 클래스의 오브젝트

### 1.1.1 IoC 컨테이너를 이용해 애플리케이션 만들기

- IoC 컨테이너 생성하기(빈 컨테이너)

```java
StaticApplicationContext ac = new StaticApplicationContext();
```

- POJO의 규칙
    1. 특정 규약에 종속되지 않는다.
    -자바와 꼭 필요한 API외에는 종속되지 않아야 한다.
    2. 특정 환경에 종속적이지 않는다.
        
        -웹 환경에 종속되는 HttpServletRequest나 HttpSession과 관련된 API를 직업 이용해서는 안된다.
        
    3. 객체 지향적 원리에 충실해야 한다.
- 빈(Bean): 스프링 컨테이너가 관리하는 오브젝트
- 설정 메타정보: 빈을 어떻게 만들고 어떻게 동작하게 할 것인가에 관한 정보
애플리케이션 컨텍스트는 BeanDefinition으로 만들어진 메타정보를 담은 오브젝트를 사용해 IoC와 DI 작업을 수행한다.
-빈 아이디, 이름, 별칭: 빈 오브젝트를 구분할 수 있는 식별자
-클래스 또는 클래스 이름: 빈으로 만들 POJO 클래스 또는 서비스 클래스 정보
-스코프: 싱글톤, 프로토타입과 같은 빈의 생성 반식과 존재 범위
-프로퍼티 값 or 참조: DI에 사용할 프로퍼티 이름과 값 or 참조할 빈의 이름
-생성자 파라미터 값 or 참조: DI에 사용할 생성자 파라미터 이름과 값 또는 참조할 빈의 이름
-지연된 로딩 여부, 우선 빈 여부, 자동와이어링 여부, 부모 빈 정보, 빈팩토리 이름 등

```java
StaticApplicationContext ac = new StaticApplicationContext();
ac.registerSingleton("hello1", Hello.class);
```

- 직접 BeanDefinition 타입의 설정 메타정보를 만들어서 IoC 컨테이너에 빈 등록하기

```java
BeanDefinition helloDef = new RootBeanDefinition(Hello.class);
helloDef.getPropertyValues().addPropertyValue("name", "Spring");
ac.registerBeanDefinition("hello2", helloDef);
```

```xml
<bean id="hello2" class="...Hello"> 
		<property name="name" value="Spring" /> 
</bean>
```

### 1.1.2 IoC 컨테이너의 종류와 사용 방법

1. StaticApplicationContext
코드를 통해 빈 메타정보를 등록하기 위해 사용
실전에서는 사용하면 안됨!!! (상세한 정보는 API 문서 참조 바람)
2. GenericApplicationContext
가장 일반적인 애플리케이션 컨텍스트의 구현 클래스
xml 파일과 같은 외부의 리소스에 있는 빈 설정 메타정보를 reader를 통해 읽어들여서 메타정보로 전환해서 사용

```java
public void genericApplicationContext() {
	GenericApplicationContext ac = new GenericApplicationContext();
	XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(ac);
	reader.loadBeanDefinitions(".../genericApplicationContext.xml");
	
	ac.refresh(); //모든 메타정보가 등록이 완료됐으니 애플리케이션 컨테이너를 초기화
								//reader를 사용해서 설정을 읽은 경우에는 반드시 refresh()를 통해 초기화해야 함
	
	Hello hello = ac.getBean("hello", Hello.class);
	hello.print();
```

1. GenericXmlApplicationContext
GenericApplicationContext + XmlBeanDefinitionReader

```java
GenericXmlApplicationContext ac = new GenericXmlApplicationContext(".../genericApplicationContext.xml");
Hello hello = ac.getBean("hello", Hello.class);
hello.print();
```

1. WebApplicationContext
스프링 애플리케이션에서 가장 많이 사용되는 애플리케이션 컨텍스트
ApplicationContext를 확장하여 웹 환경에서 사용할 때 필요한 기능을 추가함

```java
WebApplicationContext ac = new WebApplicationContext(".../webApplicationContext.xml");
Hello hello = ac.getBean("hello", Hello.class);
hello.print();
```

**웹 애플리케이션에서 스프링 애플리케이션을 기동시키는 방법**

1) main() 메소드 역할을 하는 서블릿을 생성

2) 애플리케이션 컨텍스트 생성

3) 요청이 서블릿으로 들어올 때마다 getBean()으로 필요한 빈을 가져와 정해진 메소드 실행

![main() 메소드 안에서 애플리케이션 컨텍스트를 만들고 설정 메타정보를 읽어들여서 초기화한 후에 애플리케이션의 기동 책임을 맡은 POJO 빈을 요청해서 메소드를 실행](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%202.png)

main() 메소드 안에서 애플리케이션 컨텍스트를 만들고 설정 메타정보를 읽어들여서 초기화한 후에 애플리케이션의 기동 책임을 맡은 POJO 빈을 요청해서 메소드를 실행

⇒ 스프링은 DispatcherServlet이라는 이름의 서블릿을 제공하여, 서블릿은 web.xml에 등록하는 것만으로 웹 환경에서 스프링 컨테이너가 만들어지고 애플리케이션을 실행하는데 필요한 준비를 자동으로 마쳐줌

### 1.1.3 IoC 컨테이너 계층구조

계층구조 안의 모든 컨텍스트는 각자 독립적으로 자신이 관리하는 빈을 갖고 있긴 하지만 DI를 위해 빈을 찾을 때는 부모 애플리케이션 컨텍스트 빈까지 모두 검색. (원하는 빈을 찾지 못하면 가장 위에 존재하는 루트 컨텍스트까지 요청이 전달됨)

-부모 컨텍스트는 자식 컨텍스트의 빈 사용 불가능

-형제 컨텍스트의 빈 사용 불가능

-부모 컨텍스트의 빈을 오버라이드 했을 경우(자식 컨텍스트의 빈이 부모 컨텍스트 의 빈과 중복될 때) 자식 컨텍스트의 것이 우선한다.

**<컨텍스트 계층구조 테스트>**

```xml
<!-- 부모설정파일.  -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<bean id= "hello" class = "springbook2.learningtest.ioc.Hello">
		<property name = "name" value = "Parent"/>
		<property name = "printer" ref = "printer"/>
	</bean>

	<bean id = "printer" class = "springbook2.learningtest.ioc.StringPrinter"/>
</beans>
```

```xml
<!-- 자식설정파일.  -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<bean id= "hello" class = "springbook2.learningtest.ioc.Hello">
		<property name = "name" value = "Child"/>
		<property name = "printer" ref = "printer"/>
	</bean>
</beans>
```

childContext.xml의 설정은 parentContext.xml에 의존적(hello 빈의 printer 프로퍼티가 참조하고 있는  printer 라는 이름의 빈이 존재하지 않음)

### 1.1.4 웹 애플리케이션의 IoC 컨테이너 구성

프론트 컨트롤러 패턴: 중앙집중식으로 모든 요청을 다 받아서 처리하는 방식

많은 웹 요청을 한 번에 받을 수 있는 대표 서블릿을 등록해두고, 공통적인 선행 작업을 수행하게 한 후에, 각 요청의 기능을 담당하는 핸들러라고 불리는 클래스를 호출하는 방식으로 개발하는 경우가 일반적임

![프로젝트 mysite2/src/main/java/com/poscodx/mysite/controller/GuestBookServlet.java](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%203.png)

프로젝트 mysite2/src/main/java/com/poscodx/mysite/controller/GuestBookServlet.java

GuestBookServlet이 대표 서블릿, 4개의 Action들이 핸들러라고 볼 수 있음

 

웹 애플리케이션 안에서 동작하는 IoC 컨테이너는 두 가지 방법으로 만들어진다.

1. 스프링 애플리케이션의 요청을 처리하는 서블릿 안에서 (DispatcherServlet)
2. 웹 애플리케이션 레벨에서 (ContextLoaderListener)

⇒ 일반적으로는 이 두가지 방식을 모두 사용해 컨테이너를 만든다.

**<웹 애플리케이션의 컨텍스트 계층구조>**

웹 애플리케이션 레벨에 등록되는 컨테이너는 보통 루트 웹 애플리케이션 컨텍스트라고 불린다.

웹 애플리케이션에는 하나 이상의 스프링 애플리케이션의 프론트 컨트롤러 역할을 하는 서블릿이 등록될 수 있다. 이런 경우 각 서블릿이 공유하게 되는 공통적인 빈들이 있을 것이고, 이런 빈들을 웹 애플리케이션 레벨의 컨텍스트에 등록하면 된다.

![Untitled](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%204.png)

**<웹 애플리케이션의 컨텍스트 구성 방법>**

1. 서블릿 컨텍스트와 루트 애플리케이션 컨텍스트 계층구조(스프링 웹 기술 사용하는 경우)
서블릿 컨텍스트 ⇒ 웹 관련 빈
루트 애플리케이션 컨텍스트 ⇒ 나머지
* 루트 컨텍스트는 모든 서블릿 레벨 컨텍스트의 부모 컨텍스트가 된다.
2. 루트 애플리케이션 컨텍스트 단일구조
when? 스프링 웹 기술을 사용하지 않고 서드파티 웹 프레임워크나 서비스 엔진만을 사용
3. 서블릿 컨텍스트 단일구조(스프링 웹 기술 사용하는 경우)
when? 스프링 웹 기술을 사용하면서 스프링 이외의 프레임워크나 서비스 엔진에서 스프링의 빈을 이용하지 않음

**<루트 애플리케이션 컨텍스트 등록>**

서블릿의 이벤트 리스너를 이용하면 간단하게 웹 애플리케이션 레벨에 만들어지는 루트 웹 애플리케이션 컨텍스트를 등록할 수 있다.

스프링은 웹 애플리케이션 시작과 종료 시 발생하는 이벤트를 처리하는 리스너인 ServletContextListener를 이용한다.

ServletContextListener 인터페이스를 구현한 리스너를 이용해서 시작될 때 루트 애플리케이션 컨텍스트를 만들어 초기화하고, 종료될 때 컨텍스트를 함께 종료한다.

⇒ 스프링에서는 ContextLoaderListener를 제공하여 
애플리케이션 전체에서 사용할 수 있는 빈들을 관리하는 WebApplicationContext를 생성

![ContextLoaderListener 등록 (프로젝트 mysite4/src/main/webapp/WEB-INF/web.xml)](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%205.png)

ContextLoaderListener 등록 (프로젝트 mysite4/src/main/webapp/WEB-INF/web.xml)

디폴트 설정

-애플리케이션 컨텍스트 클래스: XmlWebApplicationContext(WebApplicationContext 구현체)

-XML 설정 파일 위치: /WEB-INF/applicationContext.xml

**설정 파일 위치 지정하기**

![Untitled](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%206.png)

![설정파일 위치 지정(프로젝트 mysite3/src/main/webapp/WEB-INF/web.xml)](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%207.png)

설정파일 위치 지정(프로젝트 mysite3/src/main/webapp/WEB-INF/web.xml)

- ANT 스타일의 경로표시 방법
    
    /WEB-INF/*Context.xml: WEB-INF 밑의 Context.xml로 끝나는 모든 파일을 지정한다.
    
    /WEB-INF/**/*Context.xml: WEB-INF 밑의 **모든 서브폴더**에서 Context.xml로 끝나는 파일을 찾는다.
    

**컨텍스트 클래스 지정하기**

주의: 사용될 컨텍스트는 반드시 WebApplicationContext 인터페이스를 구현해야함

현재 스프링에서는 컨텍스트 클래스로 XmlWebApplicationContext와, 이를 대체 가능한 AnnotationConfigWebApplicationContext 만을 제공

**AnnotationConfigWebApplicationContext**

- XML 설정 대신 소스코드 내의 애노테이션 선언과 특별하게 만들어진 자바 코드를 설정 메타정보를 활용함.
- contextConfigLocation 파라미터 선언 필수
- contextConfigLocation의 value값으로 XML 파일의 위치가 아니라 설정 메타정보를 담고 있는 클래스 또는 빈 스캐닝 패키지를 지정할 수 있다.

![컨텍스트 클래스 변경(프로젝트 mysite4/src/main/webapp/WEB-INF/web.xml)](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%208.png)

컨텍스트 클래스 변경(프로젝트 mysite4/src/main/webapp/WEB-INF/web.xml)

**<서블릿 애플리케이션 컨텍스트 등록>**

**DispatcherServlet**

- 스프링의 웹 기능을 지원하는 프론트 컨트롤러 서블릿
- web.xml에 등록해서 사용가능
- 하나의 웹 애플리케이션에 여러 개의 DispatcherServlet 등록 가능(이름은 달라야함)
- 각 DispatcherServlet은 서블릿이 초기화될 때 자신만의 컨텍스트를 생성하고 초기화한다.

![서블릿 컨텍스트를 위한 서블릿 등록](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%209.png)

서블릿 컨텍스트를 위한 서블릿 등록

**<servlet-name>**

네임스페이스는 <servlet-name>으로 지정한 서블릿 이름에 -servlet을 붙여서 만든다.

ex) <servlet-name>spring</servlet-name> 의 네임스페이스는 spring-servlet

DispatcherServlet이 사용할 디폴트 XML 설정 파일의 위치는 /WEB-INF/spring-servlet.xml이 된다.

⇒ 여러개의 DispatcherServlet 이 등록되더라도 구분 가능 & 자신만의 디폴트 설정파일 가질 수 있음

![Untitled](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%2010.png)

![디폴트 설정 변경(프로젝트 mysite3/src/main/webapp/WEB-INF/web.xml)](Toby%E2%80%99s%20Spring%201Week%2096f56b451e5d4007bf10a5aca8eb0a8b/Untitled%2011.png)

디폴트 설정 변경(프로젝트 mysite3/src/main/webapp/WEB-INF/web.xml)

<load-on-startup>

서블릿 컨테이너가 등록된 서블릿을 언제 만들고 초기화할지, 또 그 순서는 어떻게 되는지를 지정하는 정수 값

-생략 or 음의 정수: 서블릿 컨테이너가 임의로 정한 시점에서 만들어지고 초기화 됨

-0이상의 값: 웹 애플리케이션이 시작되는 시점에서 서블릿을 로딩하고 초기화(다수의 서블릿이 등록되어 있을 시, 작은 수를 가진 서블릿이 우선적으로 만들어진다)