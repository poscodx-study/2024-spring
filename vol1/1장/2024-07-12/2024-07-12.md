# Toby’s Spring 3week

# 의존 관계 주입(DI)

## 제어의 역전(IoC)와 의존관계 주입

스프링의 IoC 방식의 핵심을 제대로 정의하기 위해 DI(Dependency Injection) 이라는 용어 정의

<aside>
💡 DI(Dependency Injection)
DI는 오브젝트 레퍼런스를 외부로부터 제공(주입) 받고 이를 통해 여타 오브젝트와 다이내믹하게 의존관계가 만들어지는 것이 핵심

</aside>

## 런타임 의존관계 설정

### 의존관계

![images_jakeseo_me_post_5dfd7fb5-b195-4543-b90a-3fef2ef3a439_image.png](Toby%E2%80%99s%20Spring%203week%20e9f65b594bbe4a1896f4022a1df7cd06/images_jakeseo_me_post_5dfd7fb5-b195-4543-b90a-3fef2ef3a439_image.png)

UML 모델에서 의존관계는 다음과 같이 점선으로 된 화살표로 표현

위 그림의 의미

1. A가 B에 의존
2. A와 B가 의존 관계를 가짐

의존한다는건?

→ 의존 대상이 변한다면 그것이 의존하는 대상에게 영향을 미친다는 뜻

→ 여기서는 B가 변하면 A에 영향을 미침

ex) A에서 B에 정의된 메소드 호출하는 경우 같은 사용에 대한 의존관계

방향성이 존재하는데 여기서 A는 B에 의존하지만 B는 A에 의존하지 않는다.

### UserDao의 의존관계

![images_jakeseo_me_post_2ced4953-9bb8-4bec-bd8e-9c35dd32f51c_image.png](Toby%E2%80%99s%20Spring%203week%20e9f65b594bbe4a1896f4022a1df7cd06/images_jakeseo_me_post_2ced4953-9bb8-4bec-bd8e-9c35dd32f51c_image.png)

위 그림은 UserDao가 ConnectionMaker 인터페이스를 사용하는 것을 나타냄.

→ UserDao는 ConnectionMaker 인터페이스에만 의존.

→ ConnectionMaker가 바뀌면 영향을 받지만 DConnectionMaker는 바뀌어도 영향 X

→ 결합도가 낮은 상태

이처럼 인터페이스에 대해서만 의존관계를 만들어두면 결합도가 낮아져 변경에서 자유로워짐.

UserDao는 ConnectionMaker에게만 직접 의존, DConnectionMaker는 존재조차 모름

UML에서 말하는 의존관계는 이렇듯 설계 모델의 관점에서 얘기하는 것

이렇게 가시적으로 보이는 의존관계 말고, `런타임 시에 오브젝트 사이에서 만들어지는 의존관계도 존재한다`

→ 런타임 의존관계 or 오브젝트 의존관계 → 설계 시점의 의존관계가 실체화 된 것

이렇게 인터페이스를 통해 느슨한 의존관계를 갖으면 UserDao 의 오브젝트가 런타임 시에 사용할 오브젝트가 어떤 클래스로 만든 것인지 미리 알지 못함.

이렇듯 런타임 시에 의존관계를 맺는 대상, 즉 실제 사용 대상인 오브젝트를 `의존 오브젝트(dependent object)` 라고 한다.

**의존관계 주입** → 구체적인 의존 오브젝트와 그것을 사용할 주체, 보통 클라이언트라 부르는 오브젝트를 런타임 시에 연결해주는 작업

<aside>
💡 의존관계 주입이란 다음 세가지 조건을 충족하는 작업을 말한다.

- 클래스 모델이나 코드에서는 런타임 시점의 의존관계가 드러나지 않아야 한다. 즉, 인터페이스에만 의존해야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리(이전의 `DaoFactory`)와 같은 제3의 존재가 결정한다.
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.
</aside>

이런 의존관계 주입을 해주는 `제 3의 존재`는 관계 설정 책임을 가진 코드를 분리해서 만들어진 오브젝트다.

→ ex) DaoFactory, 스프링의 애플리케이션 컨텍스트, 빈 팩토리, IoC 컨테이너 등..

### UserDao의 의존관계 주입

관계 설정 책임 분리 전의 UserDao 클래스

```java
public UserDao() {
		connectionMaker = new DConnectionMaker();
}
```

→ 이렇게 사용할 클래스를 구체적으로 알아야 한다는 문제점

→ 런타임시에의 의존관계가 이미 코드 안에서 완성되어있음

→ 그래서 DaoFactory 만듬

DaoFactory를 만든 시점에서 의존관계 주입을 이용했었음

위의 3가지 조건을 만족하기 때문에 DI가 이루어졌다고 봄

여기서 제 3의 존재는 DaoFactory가 맡음

DaoFactory는 의존관계 주입 작업을 주도, 동시에 IoC 방식으로 오브젝트의 생성과 초기화 및 제공 등의 작업 수행하는 컨테이너

→ DI 컨테이너(IoC/DI 컨테이너)로 볼 수 있다

DI 컨테이너는 UserDao를  만드는 시점에서 생성자의 파라미터로 이미 만들어진 DConnectionMaker의 오브젝트를 전달.

→ 자바에서 오브젝트에 무엇인가 넣는 것 → 메소드를 실행하면서 파라미터로 오브젝트의 레퍼런스를 넘겨는 방법 뿐

→ 생성자로 전달하는 것이 가장 쉬움

DI가 이루어진 이후

```java
public class UserDao {
    ConnectionMaker connectionMaker;

    // DI (Dependency Injection) 를 이용한 방법
    public UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }
    
    ...
```

→ DI 컨테이너에 의해 런타임 시에 의존 오브젝트를 사용할 수 있도록 그 레퍼런스를 전달받는 과정이 마치 메소드(생성자)를 통해 DI 컨테이너가 UserDao에게 주입해주는 것 같다 하여 이를 의존관계 주입이라고 부른다.

![76219248-e2cdbe80-6258-11ea-95c0-1684d251d46b.png](Toby%E2%80%99s%20Spring%203week%20e9f65b594bbe4a1896f4022a1df7cd06/76219248-e2cdbe80-6258-11ea-95c0-1684d251d46b.png)

런타임 의존관계 주입과 그로 인해 발생하는 런타임 사용 의존관계의 모습

## 의존관계 검색과 주입

IoC에는 `의존관계 검색` 이라는 방법도 존재.

→ 런타임 시에 의존관계 결정하는 건 동일

→ 관계를 맺는 방법이 주입이 아닌 검색

<aside>
💡 의존관계 검색(Dependency Lookup)
자신이 필요로 하는 의존 오브젝트를 능동적으로 찾음
의존 오브젝트를 결정하고 생성하는 것은 외부 컨테이너가
이를 가져올 때 요청하는 방식

</aside>

```java
public class UserDao {
    ConnectionMaker connectionMaker;

    // DL (Dependency Lookup) 를 이용한 방법
    public UserDao() {
        ApplicationContext applicationContext
                = new AnnotationConfigApplicationContext(DaoFactory.class);

        this.connectionMaker = applicationContext.getBean(ConnectionMaker.class);
    }
```

→ getBean()을 이용해 의존관계 검색

```java
@Configuration
// `@Configuration`은 `애플리케이션 컨텍스트` 혹은 `빈 팩토리`가 사용할 설정 정보라는 표시이다.
// `@Component`와는 다르게 의존 정보를 설정할 수 있는 곳이다.
// `@Component`에서 아래와 같이 내부에서 직접 생성하는 메소드를 사용하면,
// `Method annotated with @Bean called directly. use dependency injection` 이라는 에러 문구가 뜬다.
// `@Configuration`에서는 내부에서 직접 생성하는 메소드를 사용해도 빈 의존관계로 취급된다.
public class DaoFactory {
    @Bean // 오브젝트 생성을 담당하는 IoC용 메소드라는 표시이다.
    public UserDao userDao() {
        return new UserDao(simpleConnectionMaker());
    }

    @Bean // 오브젝트 생성을 담당하는 IoC용 메소드라는 표시이다.
    public ConnectionMaker simpleConnectionMaker() {
        return new NConnectionMaker();
    }
}
public class UserDaoTest {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        UserDao userDao = new UserDao();

        User user = new User();
        user.setId("15");
        user.setName("제이크15");
        user.setPassword("jakejake");

        userDao.add(user);

        System.out.println(user.getId() + " register succeeded");

        User user2 = userDao.get(user.getId());
        System.out.println(user2.getName());
        System.out.println(user2.getPassword());

        System.out.println(user2.getId() + " query succeeded");
    }
}
```

→ 클라이언트인 UserDaoTest에서 의존관계 주입을 해주지 않고, UserDao에서 직접 애플리케이션 컨텍스트를 검색해 의존할 오브젝트를 찾음

### 의존관계 검색 vs 의존관계 주입

의존관계 검색은 의존관계 주입의 장점을 거의 다 가지고 있음

IoC 원칙에도 잘 들어맞음

but, 대부분의 경우 의존관계 주입을 사용하는 편이 좋다.

1. 코드 부분에서 의존관계 주입이 깔끔
2. 검색 방법은 스프링 API나 오브젝트 팩토리 클래스가 코드 내에 나타남
    1. 이는 애플리케이션 컴포넌트가 컨테이너와 같이 성격이 다른 오브젝트에 의존하는 것. → 바람직하지 않음

의존관계 검색이 필요한 경우

→ `UserDaoTest`와 같은 클라이언트에서는 스프링 IoC와 DI를 컨테이너를 적용했다고 하더라도, 애플리케이션의 기동 시점에서 적어도 한 번은 `의존관계 검색` 방식을 사용해 오브젝트를 가져와야 한다. 스태틱 메소드인 `main()`에서는 DI를 이용해 오브젝트를 주입받을 방법이 없기 때문이다.

서블릿도 그러하지만 스프링이 미리 만들어서 제공하기 때문에 신경 X

<aside>
💡 의존관계 검색과 주입의 가장 큰 차이
**주입에서는 주입받는 오브젝트 자신도 스프링 빈이어야 함
검색에서는 그럴 필요 X**

</aside>

- DI 받는다
    
    DI 동작 방식은 외부로부터의 주입
    외부젱서 파라미터로 오브젝트를 넘겨준다고 전부 DI는 아님
    주입받는 메소드 파라미터가 특정 클래스로 고정되어 있으면 DI 아니다.
    

## 의존관계 주입의 응용

DI는 객체 지향 설계와 프로그래밍의 원칙을 그대로 따랐을때 얻을 수 있는 장점을 그대로 가짐.

### 기능 구현의 교환

개발 시에 DB는 함부로 건들면 안됨. 부하가 항상 높기 때문

개발 시 로컬 DB로 개발하고 배포 시 운영DB를 적용한다 할때

DI를 적용하지 않으면 모든 DAO가 LocalDBConnectionMaker에 의존

→ 이 경우 운영DB로 전환 시 DAO를 모두 수정해야 하며 실수하면 에러

DI 적용 시 모든 DAO는 생성 시점에 ConnectionMaker 타입의 오브젝트를 컨테이너로부터 제공받음.

→ 구체적 사용 클래스 이름은 컨테이너가 사용할 설정 정보에 있음

→ ex) @Configuration이 붙은 DaoFactory

→ 수정 시 ConnectionMaker만 바꿔 주면 됨

개발 시 사용

```java
@Bean
	public ConnectionMaker connectionMaker() {
			return new LocalDBConnectionMaker();
	} 

```

배포 시에는

```java
@Bean
	public ConnectionMaker connectionMaker() {
			return new ProductionDBConnectionMaker();
	}
```

### 부가기능 추가

만일 Dao가 DB를 얼마나 많이 연결해서 사용하는지 알고 싶다면?

DI 컨테이너를 이용해 간단하게 추가

```java
public class CountingConnectionMaker implements ConnectionMaker{

    int counter = 0;
    private ConnectionMaker realConnectionMaker;

    CountingConnectionMaker(ConnectionMaker realConnectionMaker){
        this.realConnectionMaker = realConnectionMaker;
    }
    
    @Override
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        this.counter++;
        return realConnectionMaker.makeConnection();
    }

    public int getCounter(){
        return this.counter;
    }
}
```

DAO가 의존할 대상이 될 것이기에 ConnectionMaker 인터페이스를 구현해서 만듬.

→ 내부에서 DB 커넥션 만들지 않는다. 대신 DAO가 DB 커넥션을 가져올 때마다 호출하는 makeConnection()에서 DB연결 횟수 카운팅

생성자에 CountingConnectionMaker도 DI를 받음.

위 클래스가 추가 되기 전

UserDao 는 ConnectionMaker 타입의 DConnectionMaker에 의존.

→ UserDao 오브젝트가 DI 받는 대상의 설정을 바꿔 CountingConnectionMaker에 의존하게 하면  UserDao가 DB커넥션을 가져올 때 마다 카운터 증가.

이렇게 하면 의존관계가

 `ConnectionMaker` ← `CountingConnectionMaker` ← `UserDao` 

가 된다.

```java
@Configuration
public class CountingDaoFactory {
	@Bean
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}
	
	@Bean
	public ConnectionMaker connectionMaker() {
		return new CountingConnectionMaker(realConnectionMaker());
	}
	
	@Bean
	public ConnectionMaker realConnectionMaker() {
		return new DConnectionMaker();
	}
	
}
```

### 생성자를 이용한 의존관계 주입

지금까지는 `UserDao`의 의존관계 주입을 위해서 생성자를 사용

일반 메서드를 통해서 의존 오브젝트와의 관계를 주입해줄 수 있다.

```java
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private Engine engine;

    // 생성자를 통한 의존성 주입
    public Car(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
        System.out.println("Car is driving");
    }
}

public class Main {
    public static void main(String[] args) {
        Engine engine = new Engine();
        Car car = new Car(engine); // 의존성 주입
        car.drive();
    }
}

```

---

### 수정자(Setter) 메서드를 이용한 주입

수정자는 파라미터로 전달된 값을 내부의 인스턴스 변수에 저장

이러한 수정자는 DI 방식에서 활용하기 good

```java
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private Engine engine;

    // 세터 메소드를 통한 의존성 주입
    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        if (engine == null) {
            System.out.println("Engine is not set");
            return;
        }
        engine.start();
        System.out.println("Car is driving");
    }
}

public class Main {
    public static void main(String[] args) {
        Engine engine = new Engine();
        Car car = new Car();
        car.setEngine(engine); // 의존성 주입
        car.drive();
    }
}

```

---

### 일반 메서드를 이용한 주입

수정자는 정해진 형태가 있고 파라미터는 한개만 받을 수 있음. 이게 싫으면 일반 메소드 사용 가능

하지만 **파라미터의 개수가 많아져 비슷한 타입이 여러 개라면 실수 가능성 높아짐**

하지만, 여러개의 파라미터를 받아서 여러 개의 초기화 메서드도 만들 수 있기 때문에 이런것에 대해서 장점이 존재

```java
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

class Car {
    public void drive(Engine engine) { // 메소드를 통한 의존성 주입
        engine.start();
        System.out.println("Car is driving");
    }
}

public class Main {
    public static void main(String[] args) {
        Engine engine = new Engine();
        Car car = new Car();
        car.drive(engine); // 의존성 주입
    }
}

```

스프링은 보통 메서드를 이용한 DI 방식중에서는 **수정자 메서드를 가장 많이 사용**

기존 생성자를 제거하고 `setConnectionMaker()` 라는 메서드를 하나 추가한 뒤 파라미터로 `ConnectionMaker` 타입의 오브젝트를 받도록 선언

```java
public class UserDao{
    private ConnectionMaker connectionMaker;

    public void setConnectionMaker(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }
}
```

이런식으로 만들어 주었다면 `DaoFactory`의 코드도 함께 수정

```tsx
@Bean
public UserDao userDao() {
    UserDao userDao = new UserDao();
    userDao.setConnectionMaker(connetionMaker());
    return userDao;
}
```

이러면 의존관계를 주입하는 시점과 방법만 달라지고 결과는 동일

# XML을 이용한 설정

오브젝트 사이의 의존정보를 일일이 자바 코드로 만들어주려면 번거로움

→ 틀에 박힌 구조가 반복됨

→ 구성이 바뀔 때 마다 수정하고 다시 컴파일도 귀찮.

이를 해결하고자 xml로 설정

**XML**을 이용하는 방법

<aside>
💡 **XML의 장점**

- 단순한 텍스트 파일이기 떄문에 다루기 쉬움
- 쉽게 이해할 수 있고 컴파일과 같은 별도의 빌드작업이 필요 x
- 환경이 달라져서 오브젝트의 관계가 바뀌는 경우에도 빠르게 변경사항을 반영
- 스키마나 DTD를 이용해서 정해진 포맷을 따라 작성되었는지 손쉽게 확인
</aside>

---

## XML 설정

스프링 Application Context는 XML에 담긴 DI 정보를 활용

우리는 `<beans>` 를 루트 엘리먼트로 사용해서 사용.

 이러한 `<beans>` 안에는 여러개의 `<bean>`  가능

본래대로면 `@Bean` 메서드에서는 알 수 있는 빈의 DI 정보는 3가지

- 빈의 이름 : 메서드의 이름 → getBean()에서 사용됨
- 빈의 클래스 : 빈 오브젝트를 어떤 클래스에 이용해서 만들지 결정
- 빈의 의존 오브젝트 : 빈의 생성자나 수정자 메서드 등을 통해서 의존 오브젝트를 넣어줌. 다수 가능

XML에서도 마찬가지로 이러한 3가지 정보를 정의 가능

---

### connectionMaker() 전환

| 목록 | 자바 코드 설정정보 | XML 설정정보 |
| --- | --- | --- |
| 빈 설정파일 | @Configuration | <beans> |
| 빈의 이름 | @Bean 메서드이름() | <bean id=”메서드이름” |
| 빈의 클래스 | return new BeanClass(); | class=”a.b.c...BeanClass”> |

@Bean 메소드에 담긴 정보를 1:1로 XML의 태그와 애트리뷰트(속성)로 전환해주면 됨.

여기서 `<bean>` 에 들어가는 class 속성에 지정하는 것은 자바 메서드에서 오브젝트를 만들 때 사용하는 클래스 이름

때문에 메서드의 리턴 타입을 class의 속성에 넣으면 X

```java
@Bean -> <bean
public ConnectionMaker connectionMaker(){    -> id="connectionMaker"
    return new AConnectionMaker();           -> class="springbook...AConnectionMaker"/>
}
```

이를 변경하면

```xml
<bean id="connectionMaker" class="springbook... AConnectionMaker"/>
```

DI 컨테이너는 이러한 XML의 `<bean>` 태그의 정보를 읽어서 우리가 작성했던 `connectionMaker()` 메서드 와 같은 작업을 진행

---

### userDao()전환

이번에는 `userDao` 를 XML로 변환. 

자바 빈의 관례에 따라 수정자 메소드는 프로페티가 된다.

만약 수정자 메서드를 사용하면 `setConnectionMaker()` 가 있다면 `set` 을 제외한 `connectionMaker` 라는 프로퍼티를 가짐.

XML에서는 `<property>` 태그를 사용해서 의존 오브젝트와의 관계를 정의

- `name` : 프로퍼티의 이름 → 이를 통해서 수정자 메서드를 알 수 있음
- `ref` : 수정자 메서드를 통해서 주입해줄 오브젝트의 빈 이름 DI할 오브젝트도 빈. 그 빈의 이름을 지정

그래서 만약에 다음과 같은 방식으로 `UserDaoTest` 에서 사용을 한다면

```less
userDao.setConnectionMaker(connectionMaker());
```

여기서 `connectionMaker()`는 `userDao` 빈의 `connectionMaker()` 라는 프로퍼티를 이용해서 의존관계 정보를 주입

그래서 `ref`

이를 XML로 고치면 다음과 같습니다.

```xml
<bean id="userDao" class="springbook.dao.UserDao">
    <property name="connectionMaker" ref="connectionMaker" />
</bean>
```

---

### XML의 의존관계 주입정보

아래처럼 <beans>로 전환한 두 개의 <bean> 태그를 감사주면 DaoFactory로부터 XML 전환 작업 완료

```xml
<beans>
    <bean id="connectionMaker" class="springbook.user.dao.AConnectionMaker" />
    <bean id-="userDao" class="springbook.user.dao.UserDao">
        <property name="connectionMaker" ref="connectionMaker"/>
    </bean>
</beans>
```

현재는 `property` 태그의 `name` 과 `ref` 가 같지만 이름이 같더라도 어떤 차이가 있는지 구별가능해야함.

- `name` 속성은 DI에 사용할 **수정자 메서드의 프로퍼티 이름**
- `ref` 속성은 **주입할 오브젝트를 정의한 빈의 ID**

보통 둘 이름이 같은 경우 다수

이러한 프로퍼티의 이름은 보통 주입할 빈 오브젝트의 인터페이스를 따름. 빈의 이름도 인터페이스 이름 사용하는 경우가 많음.

만약 빈의 이름이 중복되거나 의미를 좀 더 잘 드러낼 수 있는 이름이 있다면 변경을 할때 `property` 태그의 `ref` 속성의 값도 함께 변경

만약 `connectionMaker` 에서 `myConnectionMaker` 로 변경했다고 하면 

```xml
<beans>
    <bean id="**myConnectionMaker**" class="springbook.user.dao.AConnectionMaker" />
    <bean id-="userDao" class="springbook.user.dao.UserDao">
        <property name="connectionMaker" ref="**myConnectionMaker**"/>
    </bean>
</beans>
```

만약, 인터페이스를 구현한 의존 오브젝트를 여러개 만들어 놓고 골라서 사용하는 경우 존재

 그럴때는 다음과 같이 작성

```xml
<bean>
    <bean id="**localDBConnectionMaker**" class="...LocalDBConnectionMaker" />
    <bean id="testDBConnectionMaker" class="...testDBConnectionMaker"/>
    <bean id="productionDBConnectionMaker" class="...productionDBConnectionMaker"/>

<bean id-="userDao" class="springbook.user.dao.UserDao">
            <property name="connectionMaker" ref="**localDBConnectionMaker**"/>
    </bean>
</bean>
```

이런식으로 사용하고 `ref` 만 내가 사용할 것을 놓으면 된다. 바뀌면 위 파일도 바꾸면 됨

---

## XML을 이용하는 애플리케이션 컨텍스트

이제 Application Context가 `DaoFactory` 대신 XML 설정정보를 활용하도록 

XML을 사용하면 IoC/DI 작업에서는 `GenericXmlApplicationContext` 를 사용

 이에대한 생성자 파라미터로 XML 파일의 클래스패스 정의

클래스패스를 정해야하기 때문에 보통은 클래스패스 최상단에 둠.

이러한 설정파일을 만들 떄는 보통 `applicationContext.xml` 

![mysite3의 applicationContext.xml](Toby%E2%80%99s%20Spring%203week%20e9f65b594bbe4a1896f4022a1df7cd06/Untitled.png)

mysite3의 applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframwork.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframwork.org/schema/beans
            http://www.springframwork.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="connectionMaker" class="springbook.user.dao.AConnectionMaker"/>

    <bean id="userDao" class="springbook.user.dao.UserDao">
        <property name="connectionMaker" ref="connectionMaker"/>
    </bean>
</beans>
```

이제 `UserDaoTest`를 수정

 이제 `AnnotationConfigApplicationContext` 대신에 `GenericXmlApplicationContext`를 사용

```ebnf
ApplicationContext context = new GenericXmlApplicationContext(
        "applicationContext.xml");
```

이런식으로 바꿔서 사용

또한, `GenericXmlApplicationContext` 가 아닌 `ClassPathXmlApplicationContext` 도 존재, 이는 XML 파일을 클래스패스에서 가져올 때 사용할 수 있는 편리한 기능이 추가된 것

보통 **XML** 파일의 경로를 가져와야하는데 귀찮으면 자동으로 클래스 오브젝트를 사용해서 다음과 같이 변환

```coffeescript
new GenericXmlApplicationContexct("springbook/user/dao/daoContext.xml");

new ClassPathXmlApplicationContext("daoContext.xml", UserDao.class);
```

---

## DataSource 인터페이스로 변환

### DataSource 인터페이스 적용

우리는 `ConnectionMaker` 를 만들어 DB 커넥션을 생성해주는 기능 하나만을 정의한 매우 단순한 인터페이스 사용

`Datasource` 라는 인터페이스가 이미 존재

`Datasource` 에서 `getConnection()` 메서드는 `makeConnection()` 과 동일한 기능을 하는 메서드.

```java
import javax.sql.DataSource;

public class UserDao{
    private DataSource dataSource;

    public void SetDataSource(DataSource dataSource){
        this.dataSource = dataSource;
    }

    public void add(User user) throws SQLException{
        Connection c = dataSource.getConnection();
  }
}
```

UserDao에 주입될 의존 오브젝트의 타입을 DataSource로 변경 makeConnection()을 getConnection() 으로 변경

구현을 하였는데 `DataSource` 구현 클래스가 필요. 

우리는 스프링에서 제공하는 테스트환경에서 간단히 사용할 수 있는 `SimpleDriverDataSource` 를 사용해서 이 클래스를 사용하도록 DI를 재구성.

---

### 자바 코드 설정 방식

먼저 `DaoFactory`의 설정방식을 이용. 

기존의 `connectionMaker()` 메서드를 `dataSource()`로 변경하고 `SimpleDriverDataSource`의 오브젝트를 리턴

```tsx
// DaoFactory
@Bean
public DataSource dataSource() {
    SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

    dataSource.setDriverClass("com.mysql.jdbc.Driver.class");
    dataSource.setUrl("jdbc:mysql://localhost/springbook");
    dataSource.setUsername("spring");
    dataSource.setPassword("book");

    return dataSource;
}
```

`DaoFactory`의 `userDao()` 메서드도 수정

```tsx
@Bean
public UserDao userDao() {
    UserDao userDao = new UserDao();
    // UserDao는 이제 DataSource 타입의 dataSource()를 DI 받음
    userDao.setDataSource(dataSource()); 
    return userDao;
}
```

이런식으로 `connectionMaker()` 를 `dataSource()` 로 변경

---

### XML 설정 방식

먼저 id가 `connectionMaker` 인 `<bean>` 을 없앤 뒤 `dataSource` 라는 이름의 `<bean>`을 등록

그리고 `SimpleDriverDataSource` 로 변경

```java
<bean id="dataSource"
    class="org.springframwork.jdbc.datasource.SimpleDriverDataSource"/>
```

이런식으로 `SimpleDriverDataSource`의 오브젝트를 만드는 것까지 가능하지만 `datasource()` 메서드에서 `SimpleDriverDataSource` 의 오브젝트의 수정자로 넣어준 DB접속정보는 X

`UserDao` 처럼 다른 빈에 의존하면 그냥 `property` 태그랑 `ref` 속성을 사용해서 의존할 빈의 이름을 넣어주면 되었는데 `datasource()` 인경우는 어떤식으로 해결?

---

## 프로퍼티 값의 주입

### 값 주입

`DaoFactory` 의 `datasource()` 메서드에서 본것 처럼 수정자 메서드에는 다른 빈이나 오브젝트 뿐만 아니라 스트링 같은 단순 값을 넣어줄 수 있다 ( 값을 주입한다.)

```java
@Bean
public DataSource dataSource() {
    SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

    dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
    dataSource.setUrl("jdbc:mysql://localhost/springbook");
    dataSource.setUsername("spring");
    dataSource.setPassword("book");

    return dataSource;
}
```

이중 `setDriverClass()` 메서드는 Class 타입의 오브젝트를 넣었지만 다른 빈 오브젝트를 DI 방식으로 가져와서 넣는 것은 아님.

이렇게 사용하는 수정자 또한, `<property>`를 사용해서 값을 주입가능. 

성격은 다르지만 일종의 DI.

```less
dataSource.setDriverClass("com.mysql.jdbc.Driver.class");
dataSource.setUrl("jdbc:mysql://localhost/springbook");
dataSource.setUsername("spring");
dataSource.setPassword("book");
```

이렇게 된 코드들을 XML을 사용해서 DB 연결정보를 설정 가능.

```xml
<property name="driverClass" value="com.mysql.jdbc.Driver"/>
<property name="url" value="jdbc:mysql://localhost/springbook"/>
<property name="username" value="spring"/>
<property name="password" value="book"/>
```

다음과 같은 `ref` 대신 `value` 를 쓴다.

---

### value 값의 자동 변환

 `url`, `username`, `password` 들은 모두 스트링 타입이기 때문에 텍스트로 정의되는 `value` 속성 값을 사용하는 것은 문제 X

하지만 현재 `driverClass` 를 보면 `java.lang.Class` 타입

텍스트로 정의되어 있다고 했는데 별다른 타입정보가 없이 그냥 클래스이름이 텍스트형태로 현재 `value`에 주입

다음과 같은 상황.

```ebnf
Class driverClass = "com.mysql.jdbc.Driver";
```

이러면 본래 스트링 값을 Class에 넣는것 이기 때문에 컴파일조차 X

하지만 우리는 컴파일을 하면 아무런 문제 없이 성공

→ 스프링이 프로퍼티의 값을 수정자 메서드의 파라미터 타입을 참고로 해서 적절한 형태로 변환

`"com.mysql.jdbc.Driver"` 라는 텍스트 값을 오브젝트로 자동 변환

밑에처럼

```vbnet
Class driverClass = Class.forName("com.mysql.jdbc.Driver");
dataSource.setDriverClass(driverClass);
```

이런식으로 스프링은 `value`에 지정한 텍스트 값을 적절한 자바 타입으로 변환.

결론

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframwork.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframwork.org/schema/beans
            http://www.springframwork.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="dataSource"
            class="org.springframwork.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost/springbook"/>
        <property name="username" value="spring"/>
        <property name="password" value="book"/>
    </bean>

    <bean id="userDao" class="springbook.user.dao.UserDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>
```

# 1장 정리

- UserDao 책임 분리
    
    → 관심사 분리, 리팩토링
    
- 변경이 생길 수 있는 클래스는 인터페이스 구현 및 이 인터페이스를 통해서만 접근 가능하게 만듬
    
    →  전략 패턴
    
- 개방 폐쇄 원칙
    
    → 불필요한 변화 막고 기능은 확장과 변경에 자유로음
    
- 낮은 응집도, 높은 결합도
    
    → 한쪽의 기능변화가 다른 쪽 변경요구 X, 자신의 책임과 관심사에만 순수하게 집중
    
- 제어의 역전/IoC
    
    → 생성 및 관계의 제어권을 별도의 오브젝트 팩토리에 넘김 or IoC 컨테이너로 넘김
    
- 싱글톤 레지스트리
    
    → 스프링 빈의 디폴트는 싱글톤, 싱글톤의 단점을 극복하고자 설계된 컨테이너(싱글톤 레지스트리)
    
- DI 컨테이너
    
    → 설계 시점에는 클래스와 인터페이스 사이의 느슨한 의존관계
    
    → 런타임 시에 구체적 의존 오브젝트를 DI컨테이너(제 3자)의 도움으로 주입 → 다이내믹한 관계(의존관계 주입/DI)
    
- 생성자 주입과 수정자 주입(setter)
- XML 설정 방법