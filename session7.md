# 의존관계 자동 주입
`#인강/2.스프링 핵심원리/기본편#`

## 목차
[7.의존관계 자동 주입 - 다양한 의존관계 주입 방법](##다양한 의존관계 주입 방법)
<br>
[7.의존관계 자동 주입 - 옵션 처리](##옵션 처리)
<br>
[7.의존관계 자동 주입 - 생성자 주입을 선택 해라!](##생성자 주입을 선택 해라!)
<br>
[7.의존관계 자동 주입 - 롬복과 최신 트렌드](##롬복과 최신 트렌드)
<br>
[7.의존관계 자동 주입 - 조회 빈이 2개 이상 - 문제](##조회 빈이 2개 이상 문제)
<br>
[7.의존관계 자동 주입 - @Autowired 필드 명, @Quilifier, @Primary]
<br>
[7.의존관계 자동 주입 - 에노테이션 직접 만들기](##에노테이션 직접 만들기)
<br>
[7.의존관계 자동 주입 - 조회한 빈이 모두 필요할 때, List, Map]
<br>
[7.의존관계 자동 주입 - 자동, 수동의 올바른 실무 운영 기준]
<br>

## 다양한 의존관계 주입 방법
의존관계 주입은 크게 4가지 방법이 있다.
* 생성자 주입
* 수정자 주입(setter 주입)
* 필드 주입
* 일반 메서드 주입

### 생성자 주입
* 이름 그대로 생성자를 통해서 의존 관계를 주입 받는 방법이다.
* 지금까지 우리가 진행했던 방법이 바로 생성자 주입이다.
* 특징
  * 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
  * **불변, 필수** 의존관계에 사용

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepostory;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepostory = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```
**중요! 생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입 된다.** 물론, 스프링 빈에만 해당한다.
```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

### 수정자 주입(setter 주입)
* setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법이다.
* 특징
  * **선택, 변경** 가능성이 있는 의존관계에 사용
  * 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.