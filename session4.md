# 4. 스프링 컨테이너와 스프링 빈

`#인강/2. 스프링 핵심원리/기본편#`

## 스프링 컨테이너 생성
**스프링 컨테이너가 생성되는 과정을 알아보자**
```java
//스프링 컨테이너 생성
ApplicationContext applicationContext = 
    new AnnotationConfigApplicationContext(AppConfig.class);
```
* `ApplicationContext`를 스프링 컨테이너라 한다.
* `ApplicationContext`는 인터페이스이다.
* 스프링 컨테이너는 XML을 기반으로 만들 수 있고, 에노테이션 기반의 자바 설정 클래스로 만들 수 있다.
* 직전에 `AppConfig`를 사용했던 방식이 에노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것이다.
* 자바 설정 클래스를 기반으로 스프링 컨테이너 `ApplicationContext`를 만들어보자.
  * `new AnnotationConfigApplicationContext(AppConfig.class)`
  * 이 클래스는 `ApplicationContext` 인터페이스의 구현체 이다.

>참고 : 더 정확히는 스프링 컨테이너를 부를 때 `BeanFactory`, `ApplicationContext`로 구분해서 이야기 한다. 이 부분은 뒤에서 나오겠지만, `BeanFactory`를 직접 사용하는 경우는 거의 없으므로 일반적으로 `ApplicationContext`를 스프링 컨테이너라고 한다.


### 스프링 컨테이너의 생성 과정
**1. 스프링 컨테이너 생성**
<img src="https://ifh.cc/g/NYJsPK.jpg">
* `new AnnotationConfigApplicationContext(AppConfig.class)`
* 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 한다.
* 여기서는 `AppConfig.class`를 구성 정보로 지정했다.

**2. 스프링 빈 등록**
<img src="https://ifh.cc/g/xor197.jpg">
* 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.

**빈 이름**
* 빈 이름은 메서드 이름을 사용한다.
* 빈 이름을 직접 부여할 수도 있다.
  * `@Bean(name="memberService2")`

> 주의 : **빈 이름은 항상 다른 이름을 부여**해야 한다. 같은 이름을 부여하면, 다른 빈이 무시되거나, 기존 빈을 덮어버리거나 설정에 따라 오류가 발생한다.


**3.스프링 빈 의존관계 설정 - 준비**
<img src="https://ifh.cc/g/7GMgOp.jpg">

**4. 스프링 빈 의존관계 설정 - 완료**
<img src="https://ifh.cc/g/8RGLOF.jpg">
* 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입(DI)한다.
* 단순히 자바 코드를 호출하는 것 같지만, 차이가 있다. 이 차이는 뒤에 싱글톤 컨테이너에서 언급한다.

**참고**
<br>

스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져 있다. 그런데 이렇게 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리된다. 여기서는 이해를 돕기 위해 개념적으로 나누어 설명했다. 자세한 내용은 의존관계 자동 주입에서 다시 언급하겠음.

**정리**
<br>

스프링 컨테이너를 생성하고, 설정(구성) 정보를 참고해서 스프링 빈도 등록하고, 의존관계도 설정했다.
이제 스프링 컨테이너에서 데이터를 조회해보자

## 컨테이너에 등록된 모든 빈 조회
**스프링 컨테이너에 실제 스프링 빈들이 잘 등록 되었는지 확인해보자.**
```java
package hello.core.beanfind;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ApplicationContextInfoTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력하기")
    void findAllBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + "object = " + bean);

        }
    }
    @Test
    @DisplayName("애플리케이션 빈 출력하기")
    void findApplicationBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            // Role ROLE_APPLICATION : 직접 등록한 애플리케이션 빈
            // Role ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈
            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + "object = " + bean);
            }
        }
    }
}

```
* 모든 빈 출력하기
  * 실행하면 스프링에 등록된 모든 빈 정보를 출력할 수 있다.
    * `ac.getBeanDefinitionNames()` : 스프링에 등록된 모든 빈 이름을 조회한다.
    * `ac.getBean()` : 빈 이름으로 빈 객체(인스턴스)를 조회한다.
  * 애플리케이션 빈 출력하기
    * 스프링이 내부에서 사용하는 빈은 제외하고, 내가 등록한 빈만 출력해보자
    * 스프링이 내부에서 사용하는 빈은 `getRole()`로 구분할 수 있다.
      * `ROLE_APPLICATION` : 일반적으로 사용자가 정의한 빈
      * `ROLE_INFRASTRUCTURE` : 스프링이 내부에서 사용하는 빈

## 스프링 빈 조회 - 기본

**스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법**
* `ac.getBean(빈이름, 타입)`
* `ac.getBean(타입)`
* 조회 대상 스프링 빈이 없으면 예외 발생
  * `NoSuchBeanDefinitionException : No bean named 'xxxx' available`

**예제 코드**
```java
package hello.core.beanfind;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestFactory;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

class ApplicationContextBasicFindTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName() {
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        System.out.println("memberService = " + memberService);
        System.out.println("memberService.getClass() = " + memberService.getClass());
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }
    @Test
    @DisplayName("이름 없이 타입으로만 조회")
    void findBeanByType() {
        MemberService memberService = ac.getBean( MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByName2() {
        MemberService memberService = ac.getBean("memberService", MemberServiceImpl.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회 x")
    void FindBeanByNameX() {
        assertThrows(NoSuchBeanDefinitionException.class,
                () -> ac.getBean("xxxx", MemberService.class));
    }
}

```

## 스프링 빈 조회 - 동일한 타입이 둘 이상
* 타입으로 조회 시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정하자.
* `ac.getBeansOfType()`을 사용하면 해당 타입의 모든 빈을 조회할 수 있다.

**예제 코드**
```java
package hello.core.beanfind;

import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

import static org.assertj.core.api.AssertionsForClassTypes.assertThat;


class ApplicationContextSameBeanFindTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회 시에 같은 타입이 둘 이상이 있으면, 중복 오류가 발생한다")
    void findBeanByTypeDuplicate() {
        Assertions.assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(MemberRepository.class));
    }

    @Test
    @DisplayName("타입으로 조회 시에 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
    void findBeanByName() {
        MemberRepository memberRepository = ac.getBean("memberRepository1",MemberRepository.class);
        assertThat(memberRepository).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입을 모두 조회하기")
    void findAllBeanByType() {
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + "value = " + beansOfType.get(key));
        }
        System.out.println("beansOfType = " + beansOfType);
        assertThat(beansOfType.size()).isEqualTo(2);
    }

    @Configuration
    static class SameBeanConfig{

        @Bean
        public MemberRepository memberRepository1() {
            return new MemoryMemberRepository();
        }
        @Bean
        public MemberRepository memberRepository2() {
            return new MemoryMemberRepository();
        }
    }
}

```

## 스프링 빈 조회 - 상속관계 ⭐️
* 부모 타입으로 조회하면, 자식 타입도 함께 조회한다.
* 그래서 모든 자바 객체의 최고 부모인 `Object` 타입으로 조회하면, 모든 스프링 빈을 조회한다.

<img src="https://ifh.cc/g/0VC4bh.jpg">


**예제 코드**
```java
package hello.core.beanfind;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;


class ApplicationContextExtendsFindTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

    @Test
    @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 중복 오류가 발생한다.")
    void findBeanByParentTypeDuplicate() {
        assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(DiscountPolicy.class));
    }
    @Test
    @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
    void findBeanByParentTypeBeanName() {
        DiscountPolicy reteDiscountPolicy = ac.getBean("reteDiscountPolicy", DiscountPolicy.class);
        assertThat(reteDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("특정 하위 타입으로 조회")
    void findBeanBySubType() {
        RateDiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
        assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회하기")
    void findAllBeanByParentType() {
        Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
        assertThat(beansOfType.size()).isEqualTo(2);
        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + "value = " + beansOfType.get(key));

        }
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회하기 - Object")
    void findAllBeanByObjectType() {
        Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + "value = " + beansOfType.get(key));
        }
    }

    @Configuration
    static class TestConfig {
        @Bean
        public DiscountPolicy reteDiscountPolicy() {
            return new RateDiscountPolicy();
        }

        @Bean
        public DiscountPolicy fixDiscountPolicy() {
            return new FixDiscountPolicy();
        }
    }
}

```


## BeanFactory와 ApplicationContext

beanFactory와 ApplicationContext에 대해 알아보자

**BeanFactory**
* 스프링 컨테이너의 최상위 인터페이스이다.
* 스프링 빈을 관리하고 조회하는 역할을 담당한다.
* `getBean()`을 제공한다.
* 지금까지 우리가 사용했던 대부분의 기능은 BeanFactory가 제공하는 기능이다.

**ApplicationContext**
* BeanFactory 기능을 모두 상속받아서 제공한다.
* 빈을 관리하고 검색하는 기능을 BeanFactory가 제공해주는데, 그러면 둘의 차이가 무엇일까?
* 애플리케이션을 개발할 때는 빈을 관리하고 조회하는 기능은 물론이고, 수 많은 부가기능이 필요하다.

**ApplicationContext**가 제공하는 부가기능
<img src="https://ifh.cc/g/jVykwL.png">

* **메시지소스를 활용한 국제화 기능**
  * 예를 들어서 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
  * 예) message_KR, message_UK, message_CA
* **환경변수**
  * 로컬, 개발, 운영등을 구분해서 처리
* **애플리케이션 이벤트**
  * 이벤트를 발행하고 구독하는 모델을 편리하게 지원
* **편리한 리소스 조회**
  * 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

```java
package org.springframework.context;

import org.springframework.beans.factory.HierarchicalBeanFactory;
import org.springframework.beans.factory.ListableBeanFactory;
import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
import org.springframework.core.env.EnvironmentCapable;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.lang.Nullable;

/**
 * Central interface to provide configuration for an application.
 * This is read-only while the application is running, but may be
 * reloaded if the implementation supports this.
 *
 * <p>An ApplicationContext provides:
 * <ul>
 * <li>Bean factory methods for accessing application components.
 * Inherited from {@link org.springframework.beans.factory.ListableBeanFactory}.
 * <li>The ability to load file resources in a generic fashion.
 * Inherited from the {@link org.springframework.core.io.ResourceLoader} interface.
 * <li>The ability to publish events to registered listeners.
 * Inherited from the {@link ApplicationEventPublisher} interface.
 * <li>The ability to resolve messages, supporting internationalization.
 * Inherited from the {@link MessageSource} interface.
 * <li>Inheritance from a parent context. Definitions in a descendant context
 * will always take priority. This means, for example, that a single parent
 * context can be used by an entire web application, while each servlet has
 * its own child context that is independent of that of any other servlet.
 * </ul>
 *
 * <p>In addition to standard {@link org.springframework.beans.factory.BeanFactory}
 * lifecycle capabilities, ApplicationContext implementations detect and invoke
 * {@link ApplicationContextAware} beans as well as {@link ResourceLoaderAware},
 * {@link ApplicationEventPublisherAware} and {@link MessageSourceAware} beans.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @see ConfigurableApplicationContext
 * @see org.springframework.beans.factory.BeanFactory
 * @see org.springframework.core.io.ResourceLoader
 */
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
```

**정리**
* ApplicationContext는 BeanFactory의 기능을 상속받는다
* ApplicationContext는 빈 관리기능 + 편리한 부가 기능을 제공한다.
* BeanFactory를 직접 사용할 일은 거의 없다. 부가기능이 포함된 ApplicationContext를 사용한다.
* BeanFactory나 ApplicationContext를 <U>스프링 컨테이너</U>라고 한다.


## 스프링 빈 설정 메타 정보 - BeanDefinition

* 스프링은 어떻게 이런 다양한 설정 형식을 지원하는 것일까? 그 중심에는 **BeanDefinition**이라는 추상화가 있다.
* 쉽게 이야기해서 역할과 **구현을 개념적으로 나눈 것**이다!
  * XML을 읽어서 BeanDefinition을 만들면 된다.
  * 자바 코드를 읽어서 BeanDefinition을 만들면 된다.
  * 스프링 컨테이너는 자바 코드인지, XML인지 몰라도 된다. 오직 BeanDefinition만 알면 된다.
* **BeanDefinition**을 빈 설정 메타정보라 한다.
  * `@Bean`, `<bean>` 당 각각 하나씩 메타 정보가 생성된다.
* 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.
<img src="https://ifh.cc/g/7RnNZG.png">

**코드 레벨로 조금 더 깊이 있게 들어가 보자.**
<img src="https://ifh.cc/g/4osPjS.jpg">
* `AnnotationConfigApplicationContext` 는 `AnnotatedBeanDefinitionReader`를 사용해서 `AppConfig.class`를 읽고 `BeanDefinition`을 생성한다.
* `GenericXmlApplicationContext`는 `XmlBeanDefinitionReader`를 사용해서 `appConfig.xml` 설정 정보를 읽고 `BeanDefinition`을 생성한다.
* 새로운 형식의 설정 정보가 추가되면, XxxBeanDefinition를 만들어서 `BeanDefinition`을 생성하면 된다.

### BeanDefinition 살펴보기
**BeanDefinition 정보**
* BeanClassName : 생성할 빈의 클래스 명 (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)
* factoryBeanName : 팩토리 역할의 빈을 사용할 경우 이름, 예)appConfig
* factoryMethodName : 빈을 생성할 팩토리 메서드 지정, 예) memberService
* Scope : 싱글톤 (기본값)
* lazyInit : 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연처리 하는지 여부
* InitMethodName : 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
* DestroyMethodName : 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
* Constructor arguments, Properties : 의존 관계 주입에서 사용한다. ( 자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음 )
```java
package hello.core.beandefinition;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class BeanDefinitionTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 설정 메타 정보확인")
    void findApplicationBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                System.out.println("beanDefinition + = " + beanDefinition +
                        "beanDefinition = " + beanDefinition);
            }
        }
    }
}

```

**정리**
* BeanDefinition을 직접 생성해서 스프링 컨테이너에 등록할 수도 있다. 하지만 실무에서 BeanDefinition을 직접 정의하거나 사용할 일은 거의 없다 → 어려우면 그냥 넘어가자
* BeanDefinition에 대해서는 너무 깊이있게 이해하기 보다는, 스프링이 다양한 형태의 설정 정보를 BeanDefinition으로 추상화해서 사용하는 것 정도만 이해하면 된다.
* 가끔 스프링 코드나 스프링 관련 오픈 소스의 코드를 볼 때, BeanDefinition 이라는 것이 보일 때가 있다. 이때 이러한 메커니즘을 떠올리면 된다.