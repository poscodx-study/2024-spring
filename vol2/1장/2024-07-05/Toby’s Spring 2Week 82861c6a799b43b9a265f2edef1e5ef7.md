# Toby’s Spring 2Week

# vol2

# 1.2 IoC/DI를 위한 빈 설정 메타정보 작성

- 컨테이너는 빈 설정 메타정보를 통해 빈의 클래스와 이름을 제공 받음
- 빈을 만들기 위한 설정 메타정보는 파일이나 애노테이션 같은 리소스로부터 전용 리더를 통해 읽혀서 BeanDefinition 타입의 오브젝트로 변환됨

![컨테이너가 활용하는 빈 설정 메타정보](Toby%E2%80%99s%20Spring%202Week%2082861c6a799b43b9a265f2edef1e5ef7/Untitled.png)

컨테이너가 활용하는 빈 설정 메타정보

- 적절한 리더나 BeanDefinition 생성기를 사용할 수만 있다면 빈 설정 메타정보를 담은 소스는 어떤 식으로 만들어도 상관없음

## 1.2.1 빈 설정 메타정보

- 빈 설정 메타정보 항목
    
    ![Untitled](Toby%E2%80%99s%20Spring%202Week%2082861c6a799b43b9a265f2edef1e5ef7/Untitled%201.png)
    
    ![Untitled](Toby%E2%80%99s%20Spring%202Week%2082861c6a799b43b9a265f2edef1e5ef7/Untitled%202.png)
    
    ```java
    @Component
    @Scope(value = "prototype")
    public class ProtoTypeBean {
    
    }
    ```
    

## 1.2.2 빈 등록 방법

보통 외부 리소스(XML문서, 프로퍼티 파일, 소스코드 애노테이션)로 빈 메타정보를 작성하고 리더나 변환기를 통해 애플리케이션 컨텍스트가 사용할 수 있는 정보로 변환한다.

1. XML: **<bean> 태그**
    - 세밀한 제어 가능
    - id와 class 속성 필요 (id는 생략가능)
    - 내부 빈(inner bean)
        - 다른 빈의 설정 안에 정의되는 빈
        - 특정 빈에서만 참조하는 경우에 사용(강한 결합)
        - 다른 빈에서는 참조 불가능(아이디가 없음)
        
        ```xml
        <bean id = "hello" class = "springbook.learningtest.spring.ioc.bean.Hello">
        	<property name = "printer">
        		<bean class = "springbook.learningtest.spring.ioc.bean.StringPrinter"/>
        	</property>
        </bean>
        ```
        
        - 내부 빈을 사용하지 않았을 때
            
            ```xml
            <bean id= "hello" class = "springbook2.learningtest.ioc.Hello">
            		<property name = "printer" ref = "printer"/>
            	</bean>
            
            	<bean id = "printer" class = "springbook2.learningtest.ioc.StringPrinter"/>
            </beans>
            ```
            
        
    
2. **XML: 네임스페이스와 전용 태그**

스프링은 기술적인 설정과 기반 서비스를 빈으로 등록할 때를 위해 의미가 잘 드러나는 네임스페이스와 태그를 가진 설정 방법을 제공

```xml
<aop:pointcut id="mypointcut" expression="execution(...)" />

--------------------------------------------------------------------------------

<bean id="mypointcut"
	class="org.springframework.aop.aspectj.AspectJExpressionPointcit">
		<property name="expression" value="execution(...)" />
</bean>
```

장점: 내용이 분명하게 드러나고 선언 자체도 깔끔해짐.
         애플리케이션 로직을 담은 다른 빈 설정과 혼동되지도 않는다.  
         aop 스키마가 있기 때문에 XML 문서 편집 중에도 즉시 애트리뷰트의 타입과 필스 사용 여부 등을 검증 가능(컴파일 에러 시점)

만약, 일정한 명명 규칙이 있다면 프레젠테이션 계층, 서비스 계층, 데이터 엑세스 계층의 빈을 세 개로 나누어 등록하지 않고 다음과 같은 태그로 한 번에 등록할 수도 있다.

```xml
<aop:module id-prefix="user" class-prfix="User" package="com.mycompany.user" />

----------------------------------------------------------------------------------

<bean id="userController" class="com.mycompnay.user.UserController">
	<property name="service" ref="userService" />
</bean>
<bean id="userService" class="com.mycompnay.user.UserService">
	<property name="dao" ref="userDao" />
</bean>
<bean id="userDao" class="com.mycompnay.user.UserDao">
</bean>
```

![mysite3/src/main/java/resources/applicationContext.xml](Toby%E2%80%99s%20Spring%202Week%2082861c6a799b43b9a265f2edef1e5ef7/Untitled%203.png)

mysite3/src/main/java/resources/applicationContext.xml

1. 자동인식을 이용한 빈 등록: 스테레오타입 애노테이션과 빈 스캐너
빈 스캐닝을 통한 자동인식 빈 등록 기능: 특정 애노테이션이 붙은 클래스를 자동으로 찾아서 빈으로 등록해주는 방식
    
    
    빈 스캐너: 이러한 스캐닝 작업을 담당하는 오브젝트
    - 지정된 클래스패스 아래에 있는 모든 패키지의 클래스를 대상으로 필터를 적용해서 빈 등록을 위한 클래스들을 선별
    - 빈 스캐너에 내장된 디폴트 필터는 `@Component` 애노테이션(or `@Component` 를 메타 애노테이션으로 가진 애노테이션)이 부여된 클래스를 선택하도록 되어있다.
    
    * 스테레오타입 애노테이션: 스프링에서, 디폴트 필터에 적용되는 애노테이션을 말함
    
    ```java
    @Component
    public class AnotatedHello {
    	...
    }
    ```
    
    클래스 이름: AnnotatedHello
    
    빈의 아이디: annotatedHello(빈 스캐너는 기본적으로 클래스 이름을 빈의 아이디로 사용-첫 글자는 소문자)
    
    `AnnotationConfigApplicationContext`: 빈 스캐너를 내장하고 있는 ApplicationContext 구현 클래스
    
    ```java
    ApplicationContext ctx = new AnnotationConfigApplicationContext("springbook.learningtest.spring.ioc.bean");
    AnnotatedHello hello = ctx.getBean("annotatedHello", AnnotatedHello.class);
    ```
    
    - 빈 이름 지정 방법
        
        ```java
        @Component("myAnnotatedHello")
        public class AnotatedHello {
        	...
        }
        ```
        
    
    **스테레오타입 에노테이션의 종류**
    
    `@Component`, `@Repository`, `@Service`, `@Controller`  ⇒ 빈 클래스 자동인식을 위한 애노테이션
     * 특정 계층으로 분류하기 힘든 경우에는 `@Component` 를 사용하는 것이 바람직함
    
    ❔   @Component 애노테이션만 사용하면 안 되나?
    
    1. 계층별로 빈의 특성이나 종류를 나타내려는 목적
    2. AOP의 적용 대상 그룹을 만들기 위해(특정 계층의 빈에 부가기능 부여 가능
    - 예시코드
        
        ```java
        @Component
        @Aspect
        public class MeasureExcecutionTimeAspect {
        	@Around("execution(* *..*.repository.*.*(..)) || execution(* *..*.service.*.*(..)) || execution(* *..*.controller.*.*(..))")
        	public Object AdviceAround(ProceedingJoinPoint pjp) throws Throwable {
        		// before
        		StopWatch sw = new StopWatch();
        		sw.start();
        		
        		Object result = pjp.proceed();
        		
        		//after
        		sw.stop();
        		
        		long totalTime = sw.getTotalTimeMillis();
        		String className = pjp.getTarget().getClass().getName(); //실행되는 빈의 클래스 네임
        		String methodName = pjp.getSignature().getName();
        		String taskName = className + "." + methodName;
        		
        		System.out.println("[Execution TIme][" + taskName + "]" + totalTime + "millis");
        		
        		return result;
        	}
        }
        ```
        
    
    **커스텀 스테레오타입 애노테이션 정의하기**
    
    ```java
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Component
    public @interface BusinessRule {
    	String value() default "";
    }
    ```
    
    1. 자바 코드에 의한 빈 등록: @Configuration 클래스의 @Bean 메소드
    
    빈 설정 메타정보를 담고 있는 자바 코드는 `@Configuration` 애노테이션이 달린 클래스를 이용해 작성.
    
    해당 클래스에 `@Bean` 이 붙은 메소드를 정의하여 빈을 정의할 수 있다.
    
    XML 문서의 루트인 <beans> → @Configuration, <bean> → @Bean
    
    ```java
    @Configuration
    public class AnnotatedHelloConfig {
        @Bean
        public AnnotatedHello annotatedHello() {
            return new AnnotatedHello();
        }
    }
    ```
    
    ```java
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AnnotatedHelloConfig.class);
    AnnotatedHello hello = ctx.getBean("annotatedHello", AnnotatedHello.class);
    
    AnnotatedHelloConfig config = ctx.getBean("annotatedHelloConfig", AnnotatedHelloConfig.class);
    
    asswertThat(config.annotatedHello(), is(not(sameInstance(hello)))); //실패
    ```
    
    ```java
    @Configuration
    public class HelloConfig {
    	@Bean
    	public Hello hello(){
    		Hello hello = new Hello();
    		hello.setPrinter(printer()); //DI
    		return hello;
    	}
    
    	@Bean
    	public Hello hello2(){
    		Hello hello = new Hello();
    		hello.setPrinter(printer()); //DI
    		return hello;
    	}
    
    	@Bean
    	public Printer printer(){
    		return new StringPrinter();
    	}
    }
    ```
    
    **자바 코드에 의한 설정이 XML 같은 외부 설정파일을 이용하는 것 보다 유용한 점**
    
    - 컴파일러나 IDE를 통한 타입 검증이 가능하다.
    ⇒ XML은 텍스트 문서이므로 오류를 손쉽게 검증할 수 없다.
    - 자동완성과 같은 IDE 지원 기능을 최대한 이용할 수 있다.
    - 복잡한 빈 설정이나 초기화 작업을 손쉽게 적용할 수 있다.
    ⇒ @Bean 메소드를 이용해 빈을 정의하면 하나의 클래스 안에 여러 개의 빈을 만들 수 있다.
    
    1. 자바 코드에 의한 빈 등록: 일반 빈 클래스의 @Bean 메소드
    
    @Configuration이 붙은 클래스가 아닌 일반 POJO 클래스에도 @Bean을 사용할 수 있다.
    
    ```java
    public class HelloService {
    	private Printer printer;
    	
    	public void setPrinter(Printer printer) {
    		this.printer = printer;
    	}
    	
    	@Bean
    	public Hello hello() {
    		Hello hello = new Hello();
    		hello.setPrinter(this.printer);
    		return hello;
    	}
    
    	@Bean
    	public Hello hello2(){
    		Hello hello = new Hello();
    		hello.setPrinter(this.printer); 
    		return hello;
    	}
    
    	@Bean
    	public Printer printer(){
    		return new StringPrinter();
    	}
    }
    ```
    
    <빈 등록 메타정보 구성 전략>
    
    - XML 단독 사용
    장점: 컨텍스트에서 생성되는 모든 빈을 XML에서 확인할 수 있음
    단점: 빈의 개수가 많아지면 XML 파일을 관리하기 번거로울 수 있다.
    - XML과 빈 스캐닝 혼용
    대부분의 기술 서비스나 컨테이너 설정용 빈은 초기에 XML에 등록하고, 개발이 진행되면서 만들어지는 애플리케이션 빈들은 스테레오 타입 애노테이션을 부여
        - * 주의: 스캔 대상이 되는 클래스를 위치시킬 패키지를 미리 결정해야함
            
            ![Untitled](Toby%E2%80%99s%20Spring%202Week%2082861c6a799b43b9a265f2edef1e5ef7/Untitled%204.png)
            
        
        XML 을 사용하는 애플리케이션 컨텍스트를 기본으로 하고, 다음과 같이 빈 스캐너를 context 스키마의 <context:component-scan> 태그를 이용해 등록해주면 된다.
        
        ```xml
        <context:component-scan base-package="com.mycompany.app" />
        ```
        
        ![mysite3/src/main/java/resources/applicationContext.xml](Toby%E2%80%99s%20Spring%202Week%2082861c6a799b43b9a265f2edef1e5ef7/Untitled%205.png)
        
        mysite3/src/main/java/resources/applicationContext.xml
        
    
    - XML 없이 빈 스캐닝 단독 사용
        
        장점: 빈의 설정정보를 타입에 안전한 방식으로 작성 가능
        
        단점: 스프링이 제공하는 스키마에 정의된 전용 태그 사용 불가능(aop, tx 등)