# Toby’s Spring 2week

# 제어의 역전(Inversion of Control)

## 오브젝트 팩토리

UserDaoTest는 초난감 DAO 리팩토링 과정 중 대충 넘어감

```java
public class UserDaoTest {
	public statis void main(String[] args) throws ClassNotFoundException, SQLException{
    	ConnectionMaker connectionMaker = new DConnectionMaker();
        
        UserDao dao = new UserDao(connectionMaker);
        
        ...
    }
}
```

살펴 보면 UserDaoTest는 UserDao의 기능을 테스트하기 위함.

but, ConnectionMaker 구현 클래스를 사용할지를 결정하는 역할을 떠맡음

→ 분리 시켜야 함.

### 팩토리

위의 UserDaoTest에서 오브젝트를 만드는 것과 그 오브젝트 간의 관계를 맺어주는 기능을 분리해보자.

객체의 생성 방법을 결정하고 그 오브젝트를 리턴해주는 클래스

→ `팩토리(factory)` 라고 한다.

<aside>
💡 유의!!
팩토리(factory) ≠ 추상 팩토리 패턴, 팩토리 메소드 패턴

</aside>

이 팩토리 역할을 맡을 클래스

```java
// DaoFactory
public class DaoFactory {
	public UserDao userDao() {
    	ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
    }
}
```

UserDao, ConnectionMaker 관련 생성 작업을 옮기고 UserDaoTest는 DaoFactory에 요청해 만든 오브젝트를 가져와 사용하게 만든다.

```java
// 수정한 UserDaoTest
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
    	UserDao dao = new DaoFactory().userDao();
    }
}
```

이렇게 분리하면 UserDaoTest는 UserDao가 어떻게 만들어지는지 어떻게 초기화되어 있는지 신경 X

### 설계도로서의 팩토리

![다운로드.png](Toby%E2%80%99s%20Spring%202week%20b6dbe6b38a3445458ad36e7236794892/%25EB%258B%25A4%25EC%259A%25B4%25EB%25A1%259C%25EB%2593%259C.png)

1. UserDao와 ConnectionMaker는 핵심 데이터 로직과 기술로직 담당
2. DaoFactory는 이런 어플리케이션의 오브젝트들을 구성하고 관계 정의

a → 실질적 로직을 담당하는 컴포넌트

b→ 컴포넌트의 구조와 관계를 정의하는 설계도의 역할

위의 구조대로면 새로운 ConnectionMaker 구현 클래스로 변경 필요 시 DaoFactory만 수정하면 됨. 

→ UserDao 변경 필요 X, DB연결 방식 확장 자유로움.

`컴포넌트 역할을 하는 오브젝트와 구조를 결정하는 오브젝트를 분리시킴`

## 오브젝트 팩토리 활용

DaoFactory 에 다른 DAO 생성기능을 넣는다면?

→ ConnectionMaker 구현 클래스의 오브젝트를 생성하는 코드가 메소드마다 중복되는 문제점

생성 메소드 추가로 인해 중복이 발생하는 코드

```java
public class DaoFactory {
	public UserDao userDao() {
    	return new UserDao(new DConnectionMaker());
    }
    
    public AccountDao accountDao() {
    	return new AccountDao(new DConnectionMaker());
    }
    
    public MessageDao messageDao() {
    	return new MessageDao(new DConnectionMaker());
    }
}
```

DAO가 많아지면 ConnectionMaker의 구현 클래스를 바꿀 때마다 모든 메소드를 일일이 수정해야 된다는 문제점.

→ ConnectionMaker 생성용 메소드를 따로 뽑아내자

수정된 코드

```java
public class DaoFactory {
	public UserDao userDao() {
    	return new UserDao(connectionMaker());
    }
    
    public AccountDao accountDao() {
    	return new AccountDao(connectionMaker());
    }
    
    public MessageDao messageDao() {
    	return new MessageDao(connectionMaker());
    }
    
    public ConnectionMaker connectionMaker() {
    	return new DConnectionMaker();
    }
}
```

→ 수정 필요 시 connectionMaker() 함수만 수정하면 됨.

## 제어권의 이전을 통한 제어 관계 역전

> 제어의 역전, 쉽게 얘기하면 프로그램의 제어 흐름 구조가 바뀌는 것.

일반적으로 프로그램에선 오브젝트는 프로그램의 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여함.

제어의 역전 적용 시
`오브젝트는 자신이 사용할 오브젝트를 스스로 선택 X, 생성 X
자신이 어떻게 생성되고 사용되는지도 모른다.
→ 모든 제어 권한을 자신이 아닌 다른 대상에게 위임`
main()과 같은 엔트리 포인트를 제외하고 모든 오브젝트는 특별한 오브젝트에게 제어 권한을 넘겨 결정되고 만들어짐.
> 

제어의 역전이 적용된 예시

1. 서블릿
- 서블릿은 배포는 가능하지만 실행을 제어하는 건 불가. 대신, 서블릿에 대한 제어 권한을 가진 컨테이너가 서블릿 클래스의 오브젝트를 만들고 메소드 호출.
2. 템플릿 메소드 패턴
- 제어권을 상위 템플릿 메소드에 넘기고 자신은 필요할 때 호출되어 사용되도록 함.
3. 프레임워크(framework)
- 프레임워크 ≠ 라이브러리 
라이브러리를 사용하는 어플리케이션 코드는 코드가 흐름 직접 제어
프레임워크는 애플리케이션 코드가 프레임워크에 의해 사용됨
4. UserDao & DaoFactory
- ConnectionMaker에 대한 제어권 DaoFactory에게 위임.

---

# 스프링의 IoC

## 오브젝트 팩토리를 이용한 스프링 IoC

### 애플리케이션 컨텍스트와 설정 정보

- 스프링 빈(Bean)
    
    - 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트
    - 자바 빈에서 말하는 빈과 비슷한 오브젝트 단위의 애플리케이션 컴포넌트를 말한다.
    - 스프링 컨테이너가 생성과 관계 설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트(객체)
    
- 컴포넌트(Component)란?
    
    컴포넌트(Component)란 **프로그래밍에 있어 재사용이 가능한 각각의 독립된 모듈
    Spring에서 컴포넌트는 주로 서비스(Service), 컨트롤러(Controller), 레포지토리(Repository) 등의 역할을 하는 클래스**
    

<aside>
💡 빈 팩토리(Bean Factory) 
`스프링`에서는 빈의 생성과 관계설정 같은 제어를 담당하는 `IoC 오브젝트`를 `빈 팩토리`(`bean factory`)라고 부른다.

보통 이를 좀 더 확장한 `애플리케이션 컨텍스트`(`application context`)를 주로 사용한다. (빈 팩토리 = 애플리케이션 컨텍스트)

빈 팩토리(Bean Factory) → `빈을 생성하고 관계를 설정하는 IoC의 기본 기능에 초점`

- 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트

애플리케이션 컨텍스트(Application Context) → `애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하는 IoC 엔진이라는 의미에 초점`

</aside>

애플리케이션 컨텍스트(Application Context)

→ 애플리케이션 컨텍스트는 별도의 정보를 참고해서 빈(오브젝트)의 생성, 관계설정 등의 제어 작업을 총괄

애플리케이션의 로직을 담고 있는 컴포넌트와 설계도 역할을 하는 팩토리를 구분했었다. 바로 이 `설계도`라는 게 바로 이런 `애플리케이션 컨텍스트`와 그 설정정보를 말한다고 보면 된다.

- 애플리케이션 컨텍스트 자세히
    
    > **[ 애플리케이션 컨텍스트(Application Context)란? ]**
    > 
    > 
    > Spring에서는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC(Inversion of Control) 컨테이너인 빈 팩토리(Bean Factory)가 존재
    > 
    > 실제로는 빈의 생성과 관계설정 외에 추가적인 기능이 필요
    > 
    > 이러한 이유로 Spring에서는 빈 팩토리를 상속받아 확장한 애플리케이션 컨텍스트(Application Context)를 주로 사용
    > 
    > 애플리케이션 컨텍스트는 별도의 설정 정보를 참고하고 IoC를 적용하여 빈의 생성, 관계설정 등의 제어 작업을 총괄
    > 
    > 애플리케이션 컨텍스트에는 직접 오브젝트를 생성하고 관계를 맺어주는 코드가 없고, 그런 생성 정보와 연관관계 정보에 대한 설정을 읽어 처리
    > 
    > 예를 들어 @Configuration과 같은 어노테이션이 대표적인 IoC의 설정정보
    > 
    
    > [ 빈(Bean) 요청 시 처리 과정 ]
    > 
    
- 빈(Bean) 요청 시 처리 과정
    
    클라이언트에서 해당 빈을 요청하면 애플리케이션 컨텍스트는 다음과 같은 과정을 거쳐 빈을 반환
    
    1. ApplicationContext는 @Configuration이 붙은 클래스들을 설정 정보로 등록해두고, @Bean이 붙은 메소드의 이름으로 빈 목록을 생성
    
    2. 클라이언트가 해당 빈을 요청
    
    3. ApplicationContext는 자신의 빈 목록에서 요청한 이름이 있는지 찾음
    
    4. ApplicationContext는 설정 클래스로부터 빈 생성을 요청하고, 생성된 빈 반환
    
    ![다운로드 (1).png](Toby%E2%80%99s%20Spring%202week%20b6dbe6b38a3445458ad36e7236794892/%25EB%258B%25A4%25EC%259A%25B4%25EB%25A1%259C%25EB%2593%259C_(1).png)
    
    https://mangkyu.tistory.com/151
    

### 애플리케이션 컨텍스트 사용법

> 스프링을 적용하긴 했지만 사실 앞에서 만든 DaoFactory를 직접 사용한 것과 기능적으로 다를 바 없다.
> 
1. 먼저 스프링이 빈 팩토리를 위한 오브젝트 설정을 담당하는 클래스라고 인식할 수 있도록 @Configuration이라는 애노테이션을 추가
2. 그리고 오브젝트를 만들어주는 메소드에는 @Bean이라는 애노테이션을 붙여준다.

```java
@Configuration // 애플리케이션 컨텍스트 또는 빈 팩토리가 사용할 설정정보라는 표시
public class DaoFactory {
	@Bean // 오브젝트 생성을 담당하는 IoC용 메소드라는 표시
    public UserDao userDao() {
    	return new UserDao(connectionMaker());
    }
    
    @Bean
    public ConnectionMaker connectionMaker() {
    	return new DConnectionMaker();
    }
}
```

애플리케이션 컨텍스트 적용한 UserDaoTest

```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
    	ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);
        ...
    }
}
```

DaoFactory는 자바 코드로 작성된 스프링 전용 설정 정보로 봐야함.

이 설정 정보를 사용하는 애플리케이션 컨텍스트

→ 애플리케이션 컨텍스트는 ApplicationContext 타입의 오브젝트
@Configuration이 붙은 자바 코드를 설정 정보로 사용하려면 AnnotationConfigApplicationContext 사용하면 됨.
코

설명

**getBean() 메소드**

ApplicationContext가 관리하는 오브젝트를 요청하는 메소드

**1. 매개변수**

ex) getBean("userDao", UserDao.class); // 위에서 사용했던 코드

1) 첫번째 파라미터 > "userDao"

a) ApplicationContext에 등록된 빈의 이름

b) 메소드 이름이 빈의 이름으로 등록됨

(DaoFactory에서 @Bean이라는 어노테이션을 userDao라는 이름의 메소드에 붙임)

c) userDao라는 이름의 빈을 가져온다는 것의 의미 > DaoFactory의 userDao() 메소드를 호출한 결과를 가져옴

d) UserDao를 생성하는 방식이나 구성을 다르게 가져갈 수 있기 때문에 메소드명으로 빈 생성

(specialUserDao()라는 메소드 만들고 getBean("specialUserDao", UserDao.class)로 가져옴)

2) 두번째 파라미터 > UserDao.class

- 위 클래스를 가져온다는 의미 

`getBean()`은 기본적으로 Object 타입으로 리턴하게 되어 있어서 매번 리턴되는 오브젝트에 다시 캐스팅을 해줘야 하는 부담이 있다. 

자바 5 이상의 제네릭`generic` 메소드 방식을 사용해 getBean()의 두 번째 파라미터에 리턴 타입을 주면, 지저분한 캐스팅 코드를 사용하지 않아도 된다!

- **스프링 기능 사용 시 필요한 라이브러리**

1. 스프링 배포판 다운로드

2. 예제의 lib 폴더에서 필요한 파일을 클래스패스에 포함시키기

### 애플리케이션 컨텍스트의 동작방식

![다운로드 (2).png](Toby%E2%80%99s%20Spring%202week%20b6dbe6b38a3445458ad36e7236794892/%25EB%258B%25A4%25EC%259A%25B4%25EB%25A1%259C%25EB%2593%259C_(2).png)

<aside>
💡 오브젝트 팩토리 vs 애플리케이션 컨텍스트

1. 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없음

1) DaoFactory

- 애플리케이션 발전 시 DaoFactory처럼 IoC를 적용한 오브젝트로 계속 추가될 것

- 클라이언트가 필요한 오브젝트를 가져오려면 어떤 팩토리 클래스를 사용해야 할지 알아야 함

- 필요할 때마다 팩토리 오브젝트를 생성해야 함

2) 애플리케이션 컨텍스트

- 애플리케이션 컨텍스트를 사용하면 오브젝트 팩토리가 아무리 많아져도 이를 알아야하거나 직접 사용할 필요X

- 애플리케이션 컨텍스트 사용 시 일관된 방식으로 원하는 오브젝트 불러올 수 있음

- DaoFactory처럼 자바 코드를 작성하는 대신 XML처럼 단순한 방법을 사용해 애플리케이션 컨텍스트가 사용할 IoC 설정정보를 만들 수도 있음

2. 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해줌

- 애플리케이션 컨텍스트의 역할

1) 오브젝트 생성과 다른 오브젝트와의 관계설정

2) 오브젝트가 만들어지는 방식, 시점과 전략을 다르게 가져갈 수도 있음

3) 자동생성, 오브젝트에 대한 후처리, 정보의 조합, 설정방식의 다변화, 인터셉팅 등 오브젝트를 효과적으로 활용할 수 있는 다양한 기능을 제공

4) 빈이 사용할 수 있는 기반기술 서비스나 외부 시스템과의 연동 등을 컨테이너 차원에서 제공

3. 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공함

1) getBean() > 빈의 이름을 이용해 빈을 찾아줌

2) 이외에도 타입만으로 빈 검색, 특별한 애노테이션 설정이 되어 있는 빈 검색 가능

</aside>

DaoFactory와 같은 오브젝트 팩토리에서 사용했던 IoC 원리를 그대로 적용하는 데 애플리케이션 컨텍스트를 사용하는 이유는 범용적으로 유연한 방법으로 IoC 기능을 확장하기 위함

### 스프링 IoC 용어정리

**빈(Bean)**

1. 용어 : 빈 또는 빈 오브젝트라고 부름

2. 의미

1) 스프링이 IoC 방식으로 관리하는 오브젝트라는 뜻

2) 관리되는 오브젝트(managed object)라고 부르기도 함

3. 주의할 점

1) 스프링을 사용하는 애플리케이션에서 만들어지는 모든 오브젝트가 다 빈은 아님

2) 스프링이 직접 그 생성과 제어를 담당하는 오브젝트만을 빈이라고 부름

**빈 팩토리(Bean Factory)**

1. 용어 : 빈 팩토리

2. 의미

1) 스프링의 IoC를 담당하는 핵심 컨테이너

2) 빈 등록, 생성, 조회, 리턴, 이 외 부가적인 빈 관리 가능 담당

3. 주의할 점

1) 보통은 빈 팩토리 바로 사용X > 이를 확장한 애플리케이션 컨텍스트 이용

2) BeanFactory라고 붙여쓰면 빈 팩토리가 구현하고 있는 가장 기본적인 인터페이스의 이름

(이 인터페이스에 getBean()과 같은 메소드가 정의되어 있음)

**애플리케이션 컨텍스트(Application context)**

1. 용어 : 빈팩토리와 비교

1) 빈팩토리 : 빈의 생성과 제어의 관점

2) 애플리케이션 컨텍스트 : 스프링이 제공하는 애플리케이션 지원 기능을 모두 포함

2. 의미

1) 빈 팩토리를 확장한 IoC 컨테이너

2) 빈 등록 및 관리하는 기본적인 기능은 빈 팩토리와 동일

3) 여기에 스프링이 제공하는 각종 부가 서비스를 추가로 제공

3. 주의할 점

1) 빈팩토리보다 애플리케이션 컨텍스트를 더 많이 사용

2) AppicationContext라고 적으면 애플리케이션 컨텍스트가 구현해야 하는 기본 인터페이스를 가리킴

(ApplicationContext는 BeanFactory를 상속함)

**설정정보/설정 메타정보(Configuration Metadata)**

1. 용어 : 스프링의 설정정보 혹은 설정 메타정보, 애플리케이션의 형상 정보, 애플리케이션의 전체 그림이 그려진 청사진 등

2. 의미 : 애플리케이션 컨텍스트 또는 빈 팩토리가 IoC를 적용하기 위해 사용하는 메타정보

3. 주의할 점

1) 스프링의 설정정보는 컨테이너에 어떤 기능을 세팅하거나 조정하는 경우에도 사용

2) 하지만 위 경우보다는 IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성할 때 사용됨

**컨테이너(Container) 또는 IoC 컨테이너**

1) 컨테이너 또는 IoC컨테이너 : IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트나 빈 팩토리를 부르는 말 > 주로 빈 팩토리의 관점

2) 컨테이너 또는 스프링 컨테이너

- 애플리케이션 컨텍스트를 가리킴

- 컨테이너라는 말 자체가 IoC의 개념을 담고 있기 때문에 이름이 긴 애플리케이션 컨텍스트 대신에 스프링 컨테이너라고 부르는 걸 선호하는 사람도 있음 > 애플리케이션 컨텍스트는 그 자체로 ApplicationContext 인터페이스를 구현한 오브젝트를 가리키기도 하는데, 애플리케이션 컨텍스트 오브젝트는 하나의 애플리케이션에서 보통 여러 개가 만들어져 사용됨 > 이를 통틀어서 스프링 컨테이너라고 부를 수 있음

3) 스프링

- 컨테이너라는 말을 떼고 스프링이라고 부를 때도 스프링 컨테이너를 가리키는 말일 수 있음

- '스프링에 빈을 등록하고'  = '스프링 컨테이너 또는 애플리케이션 컨텍스트에 빈을 등록하고'

**스프링 프레임워크**

- IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 때 주로 사용
- 줄여서 스프링이라고 말하기도 함

---

# 싱클톤 레지스트리와 오브젝트 스코프

DaoFactory의 userDao() 메소드를 두번 호출해서 받는 UserDao 오브젝트는 동일한가?

<aside>
💡 동일성과 동등성
동일한 오브젝트 ≠ 동일한 정보를 담고 있는 오브젝트

동일한 오브젝트인가? → 동일성(identity) → ==
두 오브젝트가 동일 → 사실 상 하나의 오브젝트
이 오브젝트가 두 개의 레퍼런스 변수를 가지고 있을 뿐.

두 오브젝트의 값이 같은가? → 동등성(equality) → equals()

if) equals() 메소드 따로 구현하지 않은 경우

- 최상위 클래스인 Object 클래스에 구현되어 있는 equals() 메소드 사용

: 두 오브젝트의 동일성을 비교해 그 결과를 돌려줌 -> 동일한 오브젝트여야 동등한 오브젝트라고 여김
의미 : 두 개의 각기 다른 오브젝트가 메모리상에 존재, 동등성 기준에 따라 두 오브젝트의 정보가 동등하다고 판단

`동일한 오브젝트는 동등하다`

`동등한 오브젝트라고 동일하진 않다.`

</aside>

userDao 메소드를 호출 시 마다 new 연산자에 의해 새로운 오브젝트가 매번 만들어진다.

→ 동일한 오브젝트 X

그렇다면 애플리케이션 컨텍스트에 DaoFactory를 설정정보로 등록하고 getBean() 메소드로 userDao 오브젝트를 가져온다면?

→ 동일한 오브젝트 O

**스프링은 여러 번에 걸쳐 빈을 요청해도 동일한 오브젝트를 돌려줌**

## 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

애플리케이션 컨텍스트 → IoC 컨테이너
→ 싱글톤을 저장하고 관리하는 싱글톤 레지스트리(singleton registry)

*스프링은 default로 빈 오브젝트를 싱글톤으로 만든다*

### 서버 애플리케이션과 싱글톤

왜 싱글톤으로 만드는가?

→ 스프링은 엔터프라이즈 시스템을 위해 고안된 기술이기 때문.

→ 엔터프라이즈 서버 환경은 로직 복잡 & 부하 높음

→ 객체 매번 생성 시 서버 감당이 힘듬

→ 서비스 오브젝트 개념 도입

서블릿

→ JAVAEE 에서 가장 기본이 되는 서비스 오브젝트

→ 강제는 아니지만 대부분의 멀티스레드 환경에서 싱글톤으로 동작

→ 서블릿 클래스 당 하나의 오브젝트만 만들어두고 사용자의 요청을 담당하는 여러 스레드에서 공유해 동시에 사용

<aside>
💡 싱글톤 패턴
→ 애플리케이션 안에 제한된 수, 대개 한 개의 오브젝트만 만들어서 사용하는 원리.
→ 서버환경에서 권장됨
→ 하나만 존재하도록 강제.
→ 이 오브젝트는 애플리케이션 내에서 전역적 접근 간으
→ 단일 오브젝트만 존재해야 하며 여러 곳에서 공유하는 경우 사용

주의!
싱글톤 패턴은 사용하기 까다롭고 여러 문제점 존재
피해야 할 패턴이라하며 (Anti Pattern)이라 하기도 함.

</aside>

싱글톤 패턴의 한계

**자바에서 싱글톤 구현하는 방법**

1. 클래스 밖에서는 오브젝트를 생성하지 못하도록 생성자를 private으로 만듦

2. 생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 static 필드 정의

3. 생성

3-1) 호출 시점에 생성

- static 팩토리 메소드인 getInstance()를 만들고 이 메소드가 최초로 호출되는 시점에서 한 번만 오브젝트가 생성되도록 함

- 생성된 오브젝트는 static 필드에 저장

3-2) static 필드의 초기값으로 오브젝트를 미리 만듦

4. 한 번 싱글톤 오브젝트가 만들어지고 난 후에는 getInstance() 메소드를 통해 이미 만들어져 static 필드에 저장해둔 오브젝트 넘겨줌

이 싱글톤 패턴을 UserDao에 적용해보자

```java
public class UserDao{
	private static UserDao INSTANCE;
	...
	
		private UserDao(ConnectionMaker connectionMaker){
			this.connectionMaker = connectionMaker;
		}
		
		public static synchronized UserDao getInstance(){
			if(INSTANCE==null) INSTANCE = new UserDao(???);
			return INSTANCE;
		}
	...
}
```

→ 코드가 지저분

→ 생성자가 private으로 바뀌어서 ConnectionMaker 객체 넣어주는 게 불가능해짐.

→ 좋지 못한 코드

<aside>
💡 **싱글톤 패턴 구현 방식의 문제점**

1. private 생성자를 갖고 있기 때문에 상속 불가능

- 기술적인 서비스만 제공하는 경우라면 상관없겠지만, 애플리케이션의 로직을 담고 있는 일반 오브젝트의 경우 싱글톤으로 만들었을 때 객체지향적인 설계의 장점을 적용하기가 어려움

- 상속과 다형성 같은 객체지향의 특징이 적용되지 않는 static 필드와 메소드를 사용하는 것도 동일한 문제 발생시킴

2. 싱글톤은 테스트가 어렵거나 경우에 따라 테스트 자체가 불가능함

- 만들어지는 방식이 제한적이기 때문에 테스트에서 사용될 때 mock 오브젝트 등으로 대체하기 힘듦

- 싱글톤은 초기화 과정에서 생성자 등을 통해 사용할 오브젝트를 다이나믹하게 주입하기도 힘들기 때문에 필요한 오브젝트는 직접 오브젝트를 만들어 사용해야 함 > 테스트용 오브젝트로 대체 힘듦

3. 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못함

- 싱글톤이 여러개 생기는 예시
    1. 애플리케이션 서버는 여러 웹 애플리케이션을 호스팅합니다.
    각 웹 애플리케이션은 자신만의 클래스 로더를 가지고 있습니다.
    SingletonClass가 각 웹 애플리케이션 내에서 로드됩니다.
    결과적으로, 각 웹 애플리케이션은 SingletonClass의 고유한 인스턴스를 가지게 됩니다.
    2. 부모 클래스 로더가 SingletonClass를 로드합니다.
    자식 클래스 로더도 동일한 SingletonClass를 로드합니다.
    부모와 자식 클래스 로더는 동일한 클래스의 서로 다른 인스턴스를 가지게 됩니다.
    3. 웹 애플리케이션이 업데이트되어 서버에서 재배포됩니다.
    새로운 클래스 로더가 이전의 싱글톤 클래스와 다른 인스턴스를 로드합니다.
    결과적으로, 서버에는 이전 인스턴스와 새로운 인스턴스가 모두 존재하게 됩니다.

- 서버에서 클래스 로더를 어떻게 구성하고 있느냐에 따라 싱글톤 클래스임에도 하나 이상의 오브젝트가 만들어질 수 있음

- 자바 언어를 이용한 싱글톤 패턴 기법은 서버환경에서는 싱글톤이 꼭 보장된다고 볼 수 없음

- 여러 개의 JVM에 분산돼서 설치가 되는 경우에도 각각 독립적으로 오브젝트가 생김

4. 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못함

- 싱글톤은 사용하는 클라이언트가 정해져 있지 않음

- 싱글톤의 static 메소드를 이용해 언제든지 싱글톤에 쉽게 접근할 수 있기 때문에 애플리케이션 어디서든지 사용가능 > 자연스럽게 전역상태(global state)로 사용되기 쉬움

- 아무 객체나 자유롭게 접근하고 수정하고 공유할 수 있는 전역 상태를 갖는 것은 객체지향 프로그래밍에서는 권장되지 않는 프로그래밍 모델

</aside>

### 싱글톤 레지스트리

스프링 → 서버에서 서비스 오브젝트로 싱글톤 적극 권장

→ 그 어려움과 단점을 해결해주고자 스프링이 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능 제공

→ `싱글톤 레지스트리(singleton registry)`

→ 평범한 자바 클래스도 싱글톤으로 활용하게 해줌.

→ 왜냐? 스프링에선 평범한 자바 클래스도 제어권을 컨테이너에 넘기기 때문.
→ 즉, 생성에 관한 모든 권한이 애플리케이션 컨텍스트에 존재하기 때문.

`스프링은 IoC(Inversion Of Control) 컨테이너일 뿐만 아니라, 고전적인 싱글톤 패턴을 대신해 싱글톤을 만들고 관리해주는 싱글톤 레지스트리`

싱글톤 레지스트리의 장점

- 스태틱 메소드와 private 생성자를 사용해야 하는 비정상적인 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 활용하게 해준다.
- 싱글톤 방식으로 사용될 어플리케이션 컨텍스트도 public으로 만들 수 있다.
- 테스트용 목 오브젝트 대체가 간단하다.
- 무엇보다도, `객체지향적 설계원칙`, `디자인패턴`을 적용할 수 있다!

### 싱글톤과 오브젝트 상태

 싱글톤은 멀티스레드 환경에서 여러 스레드가 동시에 접근해서 사용 가능하기 때문에 상태 관리에 주의 

**무상태성**

> 기본적으로 싱글톤이 멀티스레드 환경에서 서비스 형태의 오브젝트로 사용되는 경우에는 상태정보를 내부에 갖고 있지 않은 무상태(`stateless`) 방식으로 만들어져야 한다. → 읽기 전용의 값이면 예외
> 

Q: 상태가 없는 방식으로 클래스를 만드는 경우에 각 요청에 대한 정보나, DB나 서버의 리소스로부터 생성한 정보는 어떻게 다뤄야 할까? 

A: 파라미터와 로컬 변수, 리턴 값 등을 이용하면 된다!

- 상태성(stateful)
    
    객체가 내부적으로 데이터를 저장하고 그 데이터를 바탕으로 동작하는 것.
    
- 무상태성(stateless)
    
    객체가 내부적으로 어떠한 데이터를 저장하지 않는 것을 의미.
    

무상태 방식으로 클래스 만드는 방법

- 메소드 파라미터나 메소드 안에서 생성되는 로컬 변수를 사용
    - 매번 새로운 값을 저장할 독립적인 공간이 만들어지기 때문에 싱글톤이라 하여도 여러 스레드가 변수의 값을 덮어 쓸 일 X

UserDao에 인스턴스 변수를 적용한다면?

```java
public class UserDao{

	// 초기에 설정하면 사용 중에는 바뀌지 않는 읽기전용 인스턴스 변수
	private ConnectionMaker connectionMaker;
    
    // 매번 새로운 값으로 바뀌는 정보를 담은 인스턴스 변수 > 심각한 문제 발생
    private Connection c;
    private User user;
    
    public User get(String id) throws ClassNotFoundException, SQLException {
    	this.c = connectionMaker.makeConnection();
        
        ...
        
        this.user = new User();
        this.user.setId(rs.getString("id"));
        this.user.setName(rs.getString("name"));
        this.user.setPassword(rs.getString("password"));
        
        ...
        
        return this.user;
    }
}
```

→ Connection과 User를 클래스의 인스턴스 필드로 선언

→ 싱글톤으로 만들어져 멀티스레드 환경에서 사용 시 심각한 문제.

→ 스프링의 싱글톤 빈으로 사용되는 클래스 만들 땐, 기존의 UserDao처럼 개별적으로 바뀌는 정보는 로컬변수로 정의 or 파라미터로 주고받으면서 사용하게 해야함.

→ ConnectionMaker는 읽기전용 → 인스턴스 변수로 정의 상관 X

→ DaoFactory에 @Bean → 스프링이 관리하는 빈 → 별다른 설정 없으면 싱글톤

`단순 읽기 전용이면 static final이나 final로 선언하는게 good` 

### 스프링 빈의 스코프

빈의 스코프(scope)

1. 정의 : 스프링이 관리하는 오브젝트, 즉 빈이 생성되고, 존재하고, 적용되는 범위

2. 종류(여기서는 간단하게 4가지만 소개 - 10장에서 자세히 알아볼 예정)

1) 싱글톤(singleton) 스코드

- 스프링 빈의 기본 스코프

- 스프링에서 만들어지는 대부분의 빈은 싱글톤 스코프를 가짐

- 컨테이너 내에 한 개의 오브젝트만 만들어져서 강제로 제거하지 않는 한 스프링 컨테이너가 존재하는 동안 계속 유지

2) 프로토타입(prototype) 스코프

- 컨테이너에 빈을 요청할 때마다 매번 새로운 오브젝트 만들어줌

3) 요청(request) 스코프

- 웹을 통해 새로운 HTTP 요청이 생길 때마다 생성

4) 세션(session) 스코프

- 웹의 세션과 스코프가 유사