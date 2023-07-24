# 싱글톤 컨테이너

`#인강/2.스프링 핵심원리/기본편#`

## 웹 애플리케이션과 싱글톤

* 스프링은 태생이 기업용 온라인 서비스 기술을 지원하기 위해 탄생했다.
* 대부분의 스프링 애플리케이션은 웹 애플리케이션이다. 물론 웹이 아닌 애플리케이션 개발도 얼마든지 개발할 수있다.
* 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.
<img src="https://ifh.cc/g/5zBV13.jpg">

**스프링 없는 순수한 DI 컨테이너 테스트**
```java
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

public class SingletonTest {
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();
        //1. 조회 : 호출할 때 마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();

        //2. 조회 : 호출할 때 마다 객체를 생성
        MemberService memberService2 = appConfig.memberService();

        // 참조값이 다른 것을 확인
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        // memberService1 != memberService2
        Assertions.assertThat(memberService1).isNotSameAs(memberService2);
    }
}

```

* 우리가 만들었던 스프링 없는 순수한 DI 컨테이너인 AppConfig는 요청을 할 때 마다 객체를 새로 생성한다.
* 고객 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸된다! → 메모리 낭비가 심하다.
* 해결방안은 해당 객체가 딱 1개만 생성되고, 공유하도록 설계하면 된다. → 싱글톤 패턴

## 싱글톤 패턴
* 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴
* 그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.
  * private 생성자를 사용해서 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.

**싱글톤 패턴을 적용한 예제 코드를 보자 `main`이 아닌 `test` 위치에 생성하자.**
```java
package hello.core.singleton;

import org.junit.jupiter.api.Test;

public class SingletonService {
    
    private static final SingletonService instance = new SingletonService();
    
    public static SingletonService getInstance() {
        return instance;
    }

    private SingletonService() {
    }
    
    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```
* 1. static 영역에 객체 instance를 미리 하나 생성해서 올려둔다.
* 2. 이 객체 인스턴스가 필요하면 오직 `getIntance()`메서드를 통해서만 조회할 수 있다. 이 메서드를 호출하면 항상 같은 인스턴스를 반환한다.
* 3. 딱 1개의 객체 인스턴스만 존재해야 하므로, 생성자를 private으로 막아서 혹시라도 외부에서 new키워드로 객체 인스턴스가 생성되는 것을 막는다.

**싱글톤 패턴을 사용하는 테스트 코드를 보자.**
```java
package hello.core.singleton;

import org.junit.jupiter.api.Test;

public class SingletonService {

    private static final SingletonService instance = new SingletonService();

    public static SingletonService getInstance() {
        return instance;
    }

    private SingletonService() {
    }

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```
* private으로 new키워드를 막아두었다.
* 호출할 때 마다 같은 객체 인스턴스를 반환하는 것을 확인할 수 있다.

> 참고: 싱글톤 패턴을 구현하는 방법은 여러가지가 있다. 여기서는 객체를 미리 생성해두는 가장 단순하고 안전한 방법을 선택했다.

싱글톤 패턴을 적용하면 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 사용할 수 있다. 하지만 싱글톤 패턴은  다음과 같은 수 많은 문제점들을 가지고 있다.

**싱글톤 패턴 문제점**
* 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
* 의존관계상 클라이언트가 구체 클래스에 의존한다. → DIP를 위반한다.
* 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
* 테스트하기 어렵다.
* 내부 속성을 변경하거나 초기화 하기 어렵다.
* private 생성자로 자식 클래스를 만들기 어렵다.
* 결론적으로 유연성이 떨어진다.
* 안티패턴이라고 불리기도 한다.


## 싱글톤 컨테이너

스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다.
지금까지 학습한 스프링 빈이 바로 싱글톤으로 관리되는 빈이다.

**싱글톤 컨테이너**
* 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
  * 이전에 설명한 컨테이너 생성 과정을 자세히 보자. 컨테이너는 객체를 하나만 생성해서 관리한다.
* 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
* 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
  * 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  * DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.

**스프링 컨테이너를 사용하는 테스트 코드**
```java
@Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContatiner() {
//        AppConfig appConfig = new AppConfig();
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        assertThat(memberService1).isSameAs(memberService2);

    }
}
```

**싱글톤 컨테이너 적용 후**
<img src="https://ifh.cc/g/cBY4cv.png">
* 스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용 사용할 수 있다.

> 참고: 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아니다. 요청할 때 마다 새로운 객체를 생성해서 변환하는 기능도 제공한다. 자세한 내용은 뒤에 빈 스코프에서 설명하겠다.


## 싱글톤 방식의 주의점
* 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 객체 인스턴스를 공유하기 때문에 상태를 유지(stateful)하게 설계하면 안된다.
* 무상태(sateless)로 설계해야한다!
  * 특정 클라이언트에 의존적인 필드가 있으면 안된다.
  * 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!
  * 가급적 읽기만 가능해야 한다.
  * 필드 대신에 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal등을 사용해야 한다.
* 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다!!
**상태를 유지할 경우 발생하는 문제점 예시**
```java
package hello.core.singleton;

public class StatefulService {
  private int price; // 상태를 유지하는 필드

  public void order(String name, int price) {
    System.out.println("name = " + name + " price = " + price);
    this.price = price; // 여기가 문제!


  }

  public int getPrice() {
    return price;
  }
}

```
**상태를 유지할 경우 발생하는 문제점 예시**
```java
package hello.core.singleton;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;


class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConifg.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        // ThreadA: A사용자 10000원 주문
        statefulService1.order("userA", 10000);
        // ThreadB: B사용자 20000원 주문
        statefulService2.order("userB", 20000);

        //ThreadA: 사용자A 주문 금액 조회
        int price = statefulService1.getPrice();
        System.out.println("price = " + price);

        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }

    static class TestConifg {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```
* 최대한 단순히 설명하기 위해, 실제 쓰레드는 상요하지 않았다.
* ThreadA가 사용자A 코드를 호출하고, ThreadB가 사용자B코드를 호출한다 가정하자.
* `StatefulService`의 `price`필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경한다.
* 사용자A의 주문금액은 10000원이 되어야 하는데, 20000원이라는 결과가 나왔다.
* 실무에서 이런 경우를 종종 보는데, 이로인해 정말 해결하기 어려운 큰 문제들이 터진다. (몇년에 한번씩 꼭 만난다.)
* 진짜 공유필드는 조심해야 한다! 스프링 빈은 항상 무상태(stateless)로 설계하자!

## @Configuration과 싱글톤

그런데 이상한점이 있따. 다음 AppConfig 코드를 보자.
```java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());     // 생성자 주입
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(
                memberRepository(),
                discountPolicy());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }



}

/*
* @Configuration → 설정정보를 담당함
* 각각의 메서드에 @Bean을 달아주게 되면 각각의 메서드들이 컨테이너에 등록이 됌
*/
```
* memberService 빈을 만드는 코드를 보면 `memberRepository()`를 호출한다.
  * 이 메서드를 호출하면 `new MemoryMemberRepository()`를 호출한다.
* orderService 빈을 만드는 코드로 동일하게 `memberRepository()`를 호출한다.
  * 이 메서드를 호출하면 `new MemoryMemberRepository()`를 호출한다.

결과적으로 각각 다른 2개의 `MemoryMemberRepository`가 생성되면서 싱글톤이 깨지는 것 처럼 보인다. 스프링 컨테이너는 이 문제를 어떻게 해결할까?